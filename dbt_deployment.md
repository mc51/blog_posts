# Running dbt-core in production on AWS: deploying and scheduling using ECS Fargate and Airflow


## Background

[dbt](https://getdbt.com) has established itself as a great tool for empowering analysts to take ownership of the analytical layer in the data warehouse. The era of data transformations based upon unversioned stored procedures and unreadable Python code is finally over.
*Not so fast!*
Usually, people don't like to change the way they work. And it seems that some Analysts **really** love stored procedures and spaghetti code. Hence, for increasing adaptation of dbt, it's not enough to point out the benefits. It's also crucial to make the life of analysts in your team as easy as possible when introducing dbt. And a smooth deployment to production is central for this. Your best bet for that is probably using dbt cloud. However, if you can't do that, there are some good [alternatives](https://medium.com/hashmapinc/deploying-and-running-dbt-on-aws-fargate-872db84065e4) out there.
In this post, I present a tested solution step by step, that has worked great in my experience. You might want to give it a try if you are on AWS and are looking for a reliable and maintainable way to let users deploy their dbt models into production.


## Solution overview

The simplified architecture diagram looks like this:

[![Architecture diagram](/img/dbt/architecture.png){:width=300px style="margin:auto; display: block;"}](/img/dbt/architecture.png "Architecture diagram of solution"){:class="mybox"}

Features:

* Uses CI/CD for build and deployment
* Uses a Docker Image as dbt run environment
* Runs Docker Image on AWS ECS using serverless Fargate instances
* Exports dbt run results to S3 bucket for auto generating documentation
* Triggers runs via schedule on Airflow
* Allows observability of run logs via CloudWatch

From the viewpoint of an Analyst deploying a model into production (obviously first into dev) is as simple as doing `git push`. Hard to beat this!


## Step by Step Implementation

While this solution strives for maximum simplicity of usage, the implementation itself is a bit more tricky. But worry not fellow Data Engineer, we'll go through the central parts together.
Everything is implemented via IaC using Terraform. So, instead of showing you where to click in the AWS console, I'll share the Terraform configuration.


### Docker configuration

First, let's configure the Docker image that will later be deployed to AWS ECS and which runs dbt for us. Here's the `Dockerfile`:


    :::bash
    FROM public.ecr.aws/bitnami/git:latest
    LABEL Maintainer="Data Engineer 51"
    # Supply at build with docker build --build-arg
    ARG PROJECT_NAME
    # Use supplied at build values, but can be overridden on docker run with -e
    ENV PROJECT_NAME=$PROJECT_NAME

    ENV ENV=dev
    ENV INSTALL_PATH /usr/src/app
    RUN mkdir -p $INSTALL_PATH
    WORKDIR $INSTALL_PATH

    # make this cachable by first installing packages
    RUN install_packages python3.9-minimal python3-pip awscli
    COPY requirements.txt .
    RUN pip3 install -r requirements.txt
    # and then copying the rest
    COPY . .
    RUN chmod -R 755 scripts/

    # we run everything through sh, so we can execute all we'd like
    ENTRYPOINT [ "/bin/sh", "-c" ]
    CMD ["scripts/run_dbt.sh"]


To be able to run dbt, we basically just need Python and git. After we installed Python, we install the required dbt packages via pip. The `requirements.txt` looks like this:

    :::bash
    boto3==1.24.20
    dbt-core~=1.3
    dbt-redshift~=1.3

Now, let's look at `scripts/run_dbt.sh` which is the script that will be invoked when running the Docker image:

    :::bash
    #!/bin/bash
    # Invoked in docker container. Will be run in ECS Fargate. Called by Orchestration Tool (i.e. Airflow)
    set -e

    # We use this to securely import our Redshift credentials on runtime for the dbt profile
    echo "Running .py script to get glue connection redshift credentials"
    python3 ./scripts/export_glue_redshift_connection.py # writes .redshift_credentials file

    echo "Exporting Redshift credentials as environment variables to be used by dbt"
    . ./.redshift_credentials

    echo "Running dbt:"
    echo ""
    dbt deps --profiles-dir ./dbt --project-dir ./dbt
    echo ""
    dbt debug --profiles-dir ./dbt --project-dir ./dbt
    echo ""
    dbt run --profiles-dir ./dbt --project-dir ./dbt
    echo ""
    dbt test --profiles-dir ./dbt --project-dir ./dbt
    echo ""
    dbt source freshness --profiles-dir ./dbt --project-dir ./dbt
    dbt docs generate --profiles-dir ./dbt --project-dir ./dbt
    echo ""

    echo "Copying dbt outputs to s3://dbt-documentation-$ENV/$PROJECT_NAME/ for hosting"
    aws s3 cp --recursive --exclude="*" --include="*.json" --include="*.html" dbt/target/\
        s3://dbt-documentation-$ENV/$PROJECT_NAME/

The script takes care of three tasks:

1. Import the credentials to be used by dbt for connecting to our DB on runtime
2. Run all dbt commands that are relevant for this project
3. Copy the resulting `.json` output files after dbt was run

I guess, there are numerous ways to achieve the first one. But most of those will be unsecure. For example, storing your credentials in the `Dockerfile` itself is a bad idea. I'm sure you can find many more anti patterns. Instead, what we do here is get the credentials in a secure way at runtime. To do so, we query a AWS Glue Connection (you'll need to create this before) via Python using the service role of the task that the container is run in (more on that later on). You could also query the Secret Manager, Parameter Store etc., if you prefer to save your credentials there. Just make sure, that the used service role has the appropriate permissions to access those services. Then, the Python script writes the credentials to a file (`.redshift_credentials`) that looks like this:

    :::bash
    export DBT_REDSHIFT_HOST=myhost.eu-central-1.redshift.amazonaws.com
    export DBT_REDSHIFT_USER=your_user
    export DBT_REDSHIFT_PASSWORD='your_pw'
    export DBT_REDSHIFT_SCHEMA=my_base_schema
    export DBT_REDSHIFT_DB=dev

Finally, we source this file using `. ./.redshift_credentials` so that the credentials are exported as environment variables. They will later be picked up by all dbt commands when reading the `dbt/profiles.yml` file.
In the last step of `scripts/run_dbt.sh`, we copy the artifacts that dbt creates after `dbt run` (manifest, lineage), `dbt test` (test results) and `dbt docs generate` (static `index.html` page containing the docs) to a S3 bucket. With those files, we can either directly host the documentation using the exported `index.html`. Or, we can export the `.json` artifacts to a data governance tool like [DataHub](https://datahubproject.io/) which can create documentation from several sources and projects.


### Dbt Configuration

The dbt configuration is pretty trivial. We have a `dbt/` directory in our repo's root directory containing the standard dbt project skeleton. We only need to adapt the [`dbt/profiles.yml`](https://docs.getdbt.com/docs/get-started/connection-profiles) file, like so:

    :::bash
    my-dbt-project:
    outputs:
        dev:
        host: "{{ env_var('DBT_REDSHIFT_HOST') }}"
        user: "{{ env_var('DBT_REDSHIFT_USER') }}"
        password: "{{ env_var('DBT_REDSHIFT_PASSWORD') }}"
        schema: "{{ env_var('DBT_REDSHIFT_SCHEMA') }}"
        dbname: "{{ env_var('DBT_REDSHIFT_DB') }}"
        port: 5439
        threads: 4
        type: redshift
    target: dev

This sets up our db connection using [environment variables](https://docs.getdbt.com/docs/get-started/connection-profiles#advanced-using-environment-variables). Notice, that we don't have different entries for `dev` and `live` here. Instead, the environment variables will be populated with different values in the separate deployment environments. This is because our `dev` environment is in a different AWS account then `live`. As such, the AWS Glue connection that we extract the credentials from will also refer to a different db.
For local runs, you can have a `set_credentials.sh` script in which users add their personal credentials to be exported as environment variables. They'll need to execute this before running any `dbt` command. To keep schemas separated between deployment and local development, the official documentation suggests [using user specific schemas](https://docs.getdbt.com/docs/get-started/connection-profiles#understanding-target-schemas). You might not want to do this because you want to limit schema clutter or simply don't allow your users to create schemas. In this case, an alternative could be to always use a single sandbox schema for local runs. To do so, create a  `dbt/macros/generate_schema_name.sql` file:

    :::bash
    -- Override default Macro:
    -- In deployment, run models in default schema
    -- Otherwise (When run locally) always run models in _sandbox schema
    {% macro generate_schema_name(custom_schema_name, node) -%}
        {% set myenv = env_var("ENV", "local") %}
        {%- set default_schema = target.schema -%}
        {%- set env_deployment = ["dev", "live"] -%}
        {%- if myenv in env_deployment -%}
            {%- if custom_schema_name is none -%}
                {{ default_schema }}
            {%- else -%}
                {{ default_schema }}_{{ custom_schema_name | trim }}
            {%- endif -%}
        {%- else -%}
                {{ default_schema }}_sandbox
        {%- endif -%}
    {%- endmacro %}

This overrides the default behavior of dbt. It adds a `_sandbox` suffix to the base schema name whenever the `ENV` environment variable is not set to `dev` or `live`. This will be the case on local runs. For deployment, we'll need to make sure to set it appropriately. One downside of this setup is that users can override each others tables in the sandbox by accident. To mitigate this, you can e.g. create an additional macro that adds a suffix to all tables names using the username.


### Infrastructure setup

We have mostly completed the programmatic part of the solution. Next, we'll learn how to setup the infrastructure on AWS that runs what we have built so far. I won't provide a complete solution here, as your setup will most probably be very different anyway. But I'll provide you with enough examples to get you going in the right direction.
First, we setup the ECR repository that will contain our Docker image (`ecr.tf`):

    :::terraform
    resource "aws_ecr_repository" "repo_dbt" {
    name                 = "${var.project_name}-${var.env}"
    image_tag_mutability = "MUTABLE"
    force_delete         = true

    image_scanning_configuration {
        scan_on_push = true
    }
    }

Nothing special here. Next, we configure ECS to run our Docker image as a task (`ecs.tf`):

    :::terraform
    resource "aws_ecs_task_definition" "task_dbt" {
    family                   = "${var.project_name}-dbt"
    requires_compatibilities = ["FARGATE"]
    network_mode             = "awsvpc"
    cpu                      = 512
    memory                   = 1024

    # This is the role of the agent / runner, need to pull images etc.
    execution_role_arn = "arn:aws:iam::${local.account_id}:role/${var.project_name}-ecs-task-execution"

    # Role used INSIDE the container, i.e. used when making API calls from within task
    task_role_arn = "arn:aws:iam::${local.account_id}:role/DBTServiceRole-${var.project_name}"

    # TODO: Make sure to first push image
    container_definitions = <<CONTAINER_DEFINITION
    [
    {
        "name": "${var.project_name}-dbt",
        "image": "${aws_ecr_repository.repo_dbt.repository_url}",
        "environment": [{"name": "ENV", "value": "${var.env}"}],
        "cpu": 512,
        "memory": 1024,
        "essential": true,
        "logConfiguration": {
            "logDriver": "awslogs",
            "secretOptions": null,
            "options": {
            "awslogs-group": "${aws_cloudwatch_log_group.logs_all.name}",
            "awslogs-region": "${local.region}",
            "awslogs-stream-prefix": "ecs"
            }
        }
    }
    ]
    CONTAINER_DEFINITION

    runtime_platform {
        operating_system_family = "LINUX"
        cpu_architecture        = "X86_64"
    }
    }


Things to watch out for here are the `execution_role_arn` and `task_role_arn` roles and the `network_mode`. Will come back to the `network_mode` later on, when we look at the Airflow DAG triggering the task. For now, let's define the two used service roles (`roles.tf`):

    :::terraform
    resource "aws_iam_role" "dbt" {
    name                 = "DBTServiceRole-${var.project_name}"
    description          = "Role for running dbt"
    max_session_duration = "7200"

    managed_policy_arns = [
        aws_iam_policy.dbt.arn
    ]

    assume_role_policy = jsonencode({
        Version = "2012-10-17"
        Statement = [
        {
            Action = "sts:AssumeRole"
            Effect = "Allow"
            Sid    = "AllowECSToAssumeRole"
            Principal = {
            # Allow those services to assume this role "run as this role on execution"
                Service = ["ecs-tasks.amazonaws.com"]
            }
        },
        {
            Action = "sts:AssumeRole"
            Effect = "Allow"
            Sid    = "AllowAirflowToAssumeRole"
            Principal = {
            AWS = [
                "arn:aws:iam::${local.account_id}:role/airflow_task_role",
                ]
            }
            }
        ]
    })
    }

    resource "aws_iam_role" "ecs_task_execution" {
    name                 = "${var.project_name}-ecs-task-execution"
    description          = "For running ECS task agent"
    max_session_duration = "7200"

    managed_policy_arns = [
        "arn:aws:iam::aws:policy/service-role/AmazonECSTaskExecutionRolePolicy",
    ]

    assume_role_policy = jsonencode({
        Version = "2012-10-17"
        Statement = [
        {
            Action = "sts:AssumeRole"
            Effect = "Allow"
            Sid    = "AllowECSToAssumeRole"
            Principal = {
            # Allow those services to assume this role "run as this role on execution"
                Service = ["ecs-tasks.amazonaws.com"]
            }
        },
        {
            Action = "sts:AssumeRole"
            Effect = "Allow"
            Principal = {
                AWS = [aws_iam_role.dbt.arn]
            }
        }
        ]
    })
    }


For both roles it is required to allow ECS to assume those roles. For the first one, also the role running Airflow will need to get this permission. This is because we want to allow Airflow to trigger our task later on. Now, lets look at the policies that we need to attach to the first role (`policies.tf`):

    :::terraform
    resource "aws_iam_policy" "dbt" {
      name        = var.project_name
      path        = "/"
      description = "Add to custom dbt service Role"
      policy      = data.aws_iam_policy_document.dbt.json
    }

    data "aws_iam_policy_document" "dbt" {

      statement {
        # allow retrieving the redshift credentials from the glue connection
        actions = [
          "glue:GetConnection"
        ]
        resources = ["*"]
        effect    = "Allow"
      }

      statement {
        actions = [
          "ecs:DescribeTasks"
        ]
        resources = [
          "arn:aws:ecs:${local.region}:${local.account_id}:task/*",
        ]
        effect = "Allow"
      }

      statement {
        actions = [
          "s3:GetBucketLocation",
          "s3:ListBucket",
          "s3:GetBucketAcl",
          "s3:GetObject",
          "s3:PutObject",
          "s3:DeleteObject"
        ]
        resources = [
          "arn:aws:s3:::dbt-documentation-${var.env}/*", # Documentation bucket
        ]
        effect = "Allow"
      }

      statement {
        actions = [
          "ecs:RunTask",
          "ecs:StopTask"
        ]
        # Get everything until last : of ARN. We dont want a single revision, but ALL
        # If we only have access for revision, it will not work.
        # see: https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/ecs_task_definition
        # "arn - Full ARN of the Task Definition (including both family and revision)."
        resources = [regex(".*[a-z]", aws_ecs_task_definition.task_dbt.arn)]
        effect    = "Allow"
      }

      statement {
        actions = ["iam:PassRole"]
        resources = [
          "arn:aws:iam::${local.account_id}:role/${var.project_name}-ecs-task-execution"
        ]
        effect = "Allow"
      }

    }


The somewhat tricky part is the regex used to get the ARN of the task without the revision suffix. Seems like there should be a better solution for this. But this works, so...
This should get you started in terms of the infrastructure. Let's automate stuff now!


### Airflow configuration

We use Airflow to trigger the run of dbt on a schedule. Here's a possible template for a DAG achieving that (Airflow v1.x):

    :::python
    import logging
    import os
    from datetime import datetime, timedelta

    import boto3
    from airflow import DAG
    from airflow.operators.python_operator import PythonOperator

    log = logging.getLogger(__name__)
    sh = logging.StreamHandler()
    sh.setFormatter(
        logging.Formatter("%(asctime)s - %(name)s - %(levelname)s - %(message)s")
    )
    log.addHandler(sh)
    log.setLevel(logging.DEBUG)

    PROJECT_NAME = "my-dbt-project"
    DAG_NAME = PROJECT_NAME

    AWS_PROFILE = os.getenv("AWS_PROFILE", "default")
    REGION = os.getenv("AWS_REGION", os.getenv("AWS_DEFAULT_REGION", "eu-west-1"))
    ACCOUNT_ID = boto3.client("sts").get_caller_identity()["Account"]

    ECS_CLUSTER_NAME = "todo"
    ECS_TASK_NAME = "todo"
    ECS_TASK_SUBNET = "todo"
    ECS_TASK_SECURITYGROUP = "todo"

    ROLE_TO_ASSUME = (
        f"DBTServiceRole-{PROJECT_NAME}"  # Assume when creating boto3 client(s)
    )

    # This is applied on a per task basis
    default_args = {
        "owner": "Data Engineer 51",
        "depends_on_past": False,
        "retries": 1,
        "retry_delay": timedelta(minutes=5)
    }

    def get_assumed_role_client(
        account_id: int, role_to_assume: str, service_name: str
    ) -> dict:
        """Assume role and return service client using the role"""
        log.info("Account ID: %s\nAssuming role: %s", account_id, role_to_assume)
        session = boto3.Session()
        client = session.client("sts", region_name=REGION)
        assumed_role_object = client.assume_role(
            RoleArn=f"arn:aws:iam::{account_id}:role/{role_to_assume}",
            RoleSessionName="mAssumeRoleFromAirflowOperator",
        )
        creds = assumed_role_object["Credentials"]
        session = boto3.Session(
            region_name=REGION,
            aws_access_key_id=creds["AccessKeyId"],
            aws_secret_access_key=creds["SecretAccessKey"],
            aws_session_token=creds["SessionToken"],
        )
        assumed_client = session.client(service_name=service_name)
        return assumed_client


    def run_ecs_task(jobname: str):
        """Run ECS Task in Fargate and wait for result"""
        client = get_assumed_role_client(ACCOUNT_ID, ROLE_TO_ASSUME, "ecs")
        log.info("Starting ECS-Task: %s", jobname)
        response = client.run_task(
            cluster=ECS_CLUSTER_NAME,
            launchType="FARGATE",
            taskDefinition=ECS_TASK_NAME,
            count=1,
            platformVersion="LATEST",
            networkConfiguration={
                "awsvpcConfiguration": {
                    "subnets": [ECS_TASK_SUBNET],
                    "securityGroups": [ECS_TASK_SECURITYGROUP],
                    "assignPublicIp": "DISABLED",
                }
            },
        )
        task_cluster_name = response["tasks"][0]["clusterArn"]
        task_arn = response["tasks"][0]["taskArn"]

        log.info("ECS-Task running. Waiting for it to finish.")
        waiter = client.get_waiter("tasks_stopped")
        waiter.wait(
            cluster=task_cluster_name,
            tasks=[task_arn],
            WaiterConfig={"Delay": 10, "MaxAttempts": 360},  # 1h
        )
        response = client.describe_tasks(
            cluster=task_cluster_name,
            tasks=[task_arn],
        )
        log.debug(response)
        exit_code = response["tasks"][0]["containers"][0]["exitCode"]
        stop_code = response["tasks"][0]["stopCode"]
        stopped_reason = response["tasks"][0]["stoppedReason"]
        log.info("Exit Code: %s - Stop code: %s\n", exit_code, stop_code)
        if exit_code != 0:
            log.error("Error:  %s", stopped_reason)
            raise Exception(f"Error finishing ECS-Task: {stopped_reason}")
        return True


    dag = DAG(
        DAG_NAME,
        default_args=default_args,
        description="Trigger dbt",
        schedule_interval="00 12 * * *",  # every day at 12:00 UTC
        start_date=datetime(2022, 12, 1),
        catchup=False,
    )

    run_dbt = PythonOperator(
        task_id="run_dbt",
        python_callable=run_ecs_task,
        op_kwargs={"jobname": ECS_TASK_NAME},
        dag=dag,
    )

    # pylint: disable=pointless-statement
    run_dbt


We use the `PythonOperator` to call a function. The function will first assume the role we defined earlier in Terraform (`DBTServiceRole-${var.project_name}`). That's why we had to allow the service role that Airflow runs as to assume this role before. Then, the function will trigger a task run on ECS using this assumed role. Recall that we defined the `network_mode` of the ECS task definition as `awsvpc` (which is the only viable option for FARGATE). That's why we need to provide a network configuration here. To do so, we must specify appropriate values for `ECS_TASK_SUBNET` and `ECS_TASK_SECURITYGROUP`. Make sure that the task can access your db using this configuration. Otherwise, you dbt run will time out.  
For bonus points, you can add callback functions that will share the run status. For example, posting those to a slack channel might be a good idea.


### CI/CD Setup

Your CI/CD Setup will depend on so many idiosyncrasies, that I'll just stick to a basic explanation. You go and figure out the rest (or ask your friendly DevOps person). What you want to achieve is this:

1. Trigger a build, when a (versioned) commit is pushed to the repo. The build should:
    1. Create the Docker image
    2. Create the terraform plan
    3. Export those as build artifacts
2. On successful build, trigger a deployment to `dev`:
    1. Get the build artifact
    2. Use terraform to create your ECR repo before anything else
    3. Push the docker image to the freshly created ECR repo
    4. Now, apply the rest of the terraform plan for creating the remaining infra
    5. Deploy the Airflow DAG (e.g. by copying the `.py` file to a bucket hooked to your Airflow instance)

Other things you can do include running some checks, i.e. linters and formatters, in the build stage. I'm a fan of using [pre-commit](https://pre-commit.com/) for that. You can not only check your Python code but also your [terraform config](https://github.com/antonbabenko/pre-commit-terraform) and even [dbt models (with SQLFluff](https://github.com/sqlfluff/sqlfluff).


## Conclusion

After this post, you'll be able to build a production ready process for deploying dbt models on AWS that follows best practices. Automating the process by using CI/CD for deploying and Airflow for scheduling dbt will reduce the workload of your Analysts to a simple `git push`. This will definitely encourage the adaptation of dbt in your org. Following, allowing you to focus on extracting and loading data instead of dealing with transformations. But watch out: Your Analysts might fall in love with you!
