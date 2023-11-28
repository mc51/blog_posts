+++
title = "A critical review of Marketing Mix Modeling — From hype to reality"
date = "2023-11-26"
tags = ["marketing-mix-modeling", "MMM", "data-science", "marketing-measurement"]
slug = "critical-review-of-marketing-mix-modeling"
type = "blog"
categories = ["Blog"]
author = "mc51"
summary = "Marketing Mix Modeling is promoted as the answer to the most important and challenging question in marketing measurement: determining the effect of marketing and marketing channels on sales. However, this claim is misleading and should not be taken at face value. Let's find out why!"
+++


# Context

Most companies spend large chunks of their budget on marketing. Often, without knowing the return of that investment. Marketing Mix Modeling has been promoted as the one method to shed light on the effect of marketing. Not quite coincidentally, this is mainly supported by people that have a self-serving interest to advocate MMM. Opposing standpoints are few and far between.  
In this post, I do a critical review of Marketing Mix Modeling and debunk the overinflated claims surrounding it. To do so, I lay out strong arguments based on statistics and take actual research into account. Moreover, I draw on my experience as a Data Scientist working in marketing analytics. This is where I gained extensive first hand knowledge of implementing Marketing Mix Models. This allows me to also delve into the practical issues of these models. Furthermore, I propose reasons for the ungrounded popularity of the method. Finally, I offer some strategies for dealing with the unreasonable expectations accompanying MMM projects. 


# What is Marketing Mix Modeling?

The objective of a Marketing Mix Model is to quantify the contribution of various marketing activities on sales / revenue. Hence, allowing companies to assign a return on ad spend (ROAS) to marketing channels.[^2a] As such, it is believed to be an indispensable tool for budget allocation between marketing channels. And for a good reason: the models are trusted to be an objective measure in a process in which decisions are regularly dictated by politics instead of provable arguments. Or, can you imagine the head of Social Media saying: "Our impact on the company's success was marginal last year. People weren't buying more toilet paper, even though we had this great influencer campaign with the Kardashians. Why don't we just cut my department's budget in half?"[^18]  
Underlying a MMM is a statistical model based on correlations. It uses historic time series data to uncover the relationship between several input factors and a target variable. Usually, the target variable is sales or revenue. In general, numerous factors like seasonality, weather, macro-economic conditions, competition etc. are used to model "base-sales". The focus, however, is on variables that can explain "incremental" sales. Those can be factors like product, price and promotions.[^2b] But most importantly, variables that depict marketing activities usually in the form of ad spending or reach per marketing channel.
The technical implementation of MMM can vary. Nonetheless, more often than not, the foundation is some kind of linear regression model. To account for lags, carry-over or saturation of marketing effects, those inputs are transformed. Normally, the data has a weekly granularity (classic media campaigns are long running) and a history of at least two years is needed to have "enough" data points.[^1a]


[^2a]: For a definition of ROAS see p.8 in [Jin, Y., Wang, Y., Sun, Y., Chan, D., & Koehler, J. (2017). Bayesian methods for media mix modeling with carryover and shape effects.](https://research.google/pubs/pub46001/)

[^2b]: See pp.1 in [Jin, Y., Wang, Y., Sun, Y., Chan, D., & Koehler, J. (2017). Bayesian methods for media mix modeling with carryover and shape effects.](https://research.google/pubs/pub46001/)

[^1a]: For a more detailed introduction to Marketing Mix Modeling, see [this article](https://towardsdatascience.com/market-mix-modeling-mmm-101-3d094df976f9) 

[^18]: I was trying to come up with the most ridiculous marketing campaign I could think of for this example. Through chance, I learned that this actually [happened](https://www.prnewswire.com/news-releases/kim-kardashian-opens-new-charmin-restrooms-location-in-the-heart-of-new-york-city-110103764.html).


# The issues with MMMs

There are a lot of theoretical problems with Marketing Mix Modeling and at least as many practical ones.[^1b] So many that I yet have to see or hear of a company that has implemented a MMM successfully. By successfully I mean that the model comes to robust and coherent results that are not based on arbitrary assumptions. Thereby, making it credible. So credible, in fact, that you would trust it to guide your multi-million dollar budget decisions.
Sure, a lot of companies _claim_ that they are getting value from their models. And don't get me started on consultancies that promise the pie in the sky... But when you dig deeper, there is nothing substantial behind those claims. In the first case, they're made by people (looking at you managers) removed from the actual modeling and lacking a proper understanding of it. In the second case, they are simply driven by incentives for making money.  
In the following, we will discuss the shortcomings one by one.


[^1b]: [Here](https://www.linkedin.com/pulse/top-10-challenges-implementing-marketing-mix-models-christopher-doyle/) is another, more concise article on the challenges of implementing MMM. I dwell on many of its points here.


# Statistical issues

Marketing Mix Modeling is founded in statistics. Consequently, one should be confident that the statistical theory behind such models is solid. At least, it should _mostly_ make sense. Why would you trust its results otherwise?  
Unfortunately, when taking a closer look at these models, one can find numerous fundamental flaws. Google, which is invested in the area of MMM, writes in an article[^8c]: "MMM is a valuable tool for measurement, but science alone cannot produce all the right answers. [..] Incorporating business context to shape the MMM is an art, one with implications for the model’s outcomes and final recommendations."[^9a]

{{<figure lightbox="meme" width="60%" caption="MMM expert at Google getting ready for work" attr=imgflip attrlink="https://imgflip.com" src="/img/mmm/meme_clown.jpg">}}

Well, what is it Google, science or art? I prefer my stats to be less artsy and more scientific. Unsurprisingly, so do most Data Scientists. Let's explore that and dive into the statistical shortcomings of Marketing Mix Modeling.


[^8c]: See: [Marketing mix models are based in science, but also need a touch of art](https://www.thinkwithgoogle.com/marketing-strategies/data-and-measurement/art-of-marketing-mix-models/)

[^9a]: Meta, also heavily involved in the MMM space, seems to agree: ["MMM studies are complex, where we don’t believe that a handful of statistical parameters can be used to accurately select a model that reflects the various business contexts and different companies"](https://facebookexperimental.github.io/Robyn/docs/analysts-guide-to-MMM/#applications-of-the-model-)


## Missing causality

For a relationship between inputs and target variable in a MMM to be actionable, it must be causal. Imagine that whenever you spend more on ads you observe higher revenues. If the relationship isn't causal, then it's not the increased spend that leads to additional revenue. Maybe you spent most of your money during christmas, a period where sales increase because of numerous reasons anyway. Or maybe your main competitor went out of business in the meantime. Thus, willingly increasing your ad spending might do nothing for your revenue. It might even hurt your bottom line. You simply won't know.[^3a]

{{<figure lightbox="meme" width="70%" caption="If you think MMM estimates are causal, you\'re probably not a statistician" attr="xkcd.com" attrlink="https://xkcd.com/552" src="/img/mmm/correlation.png">}}

The estimate of a MMM cannot be causal by definition: Because it is derived from observational data, it is based on mere correlation. For estimating causal effects this is not good enough.[^3d] There is no way around randomized experiments for that. This is a very simple and profound truth, but one that is regularly ignored. People will argue things like "it might not _technically_ be causal, but it's so close of an approximation that it's _practically_ causal". Is it though?  
This is only plausible under a **very narrow** set of conditions[^4c]:

1. There is enough data to estimate all parameters in the model
2. There is useful variability in the advertising levels and control variables
3. Model inputs vary independently
4. The model accounts for all important drivers that impact sales
5. The model captures the causal relationship between variables

What are the chances that we can be positive on all those items? Spoiler alert: Practically zero. Skeptical? No worries, we'll examine those one by one.


[^3a]: "But is this causation or merely correlation? [...] How do we attribute Sales increase after an advertising campaign, to the specific marketing effort only, is there any other channel that enhanced the effect of one channel— or this is due to some other impact altogether?" [Pandey, S., Gupta, S., & Chhajed, S. (2021). Marketing Mix Modeling (MMM)-Concepts and Model Interpretation. Sandeep Pandey, Snigdha Gupta, Shubham Chhajed.](https://papers.ssrn.com/sol3/papers.cfm?abstract_id=3877291)

[^3d]: "When we can’t run experiments, one must rely on methods that use observational data such as marketing mix modelling or digital attribution. These measurement techniques are not always good at estimating causal effects [...]" pp. 791 in [Pandey, S., Gupta, S., & Chhajed, S. (2021). Marketing Mix Modeling (MMM)-Concepts and Model Interpretation. Sandeep Pandey, Snigdha Gupta, Shubham Chhajed.](https://papers.ssrn.com/sol3/papers.cfm?abstract_id=3877291)

[^4c]: Taken from p.6 in [Chan, D., & Perry, M. (2017). Challenges and opportunities in media mix modeling.](https://research.google/pubs/pub45998/)


### 1. Limited data

A typical MMM will have two years worth of weekly data. Three, if you're lucky. That makes 156 data points in the best case scenario. That's not bad right? Well, this really depends on the number of parameters you want to estimate. A very broad rule of thumb states that calculating a linear regression you'll need 10-20 observations per parameter you estimate.[^5a] Unfortunately, for MMMs "the more, the merrier" is the name of the game regarding inputs. People will argue that reality is very complex (true) and that a complex model can depict it accurately (not so true).[^2e] In consequence, a typical model for a traditional retail company would include at least the following:

* Factors directly influencing sales, controlled by the company:
    * Assortment (breadth and quality)
    * Stores (number, location, size, features)
    * Prices & promotions
* Observable factors that cannot be controlled:
    * Macro-economic factors (GDP, unemployment, inflation)
    * Competition (quantity and quality)
    * Seasonality (holidays, seasons)
* Media channels used for marketing:
    * Owned channels (social media, newsletter, point of sale, SEO)
    * Classic channels (TV, radio, out of home, print)
    * Online channels (SEA, video, display)

Even when using _very_ aggregated measures for all of the above, it's hard to arrive at less than 20 variables in total.[^4d] And in reality, marketing stakeholders will want to be *way* more detailed: "We follow very different strategies on social media. We should differentiate between TikTok, Instagram, Facebook, Pinterest and MySpace. Also, can we separate each channel into an activation, brand and image part please?".  
Okay, so 20 variables means one needs 200 to 400 observations. Not really close to the 156 one _might_ have.[^4]

{{<figure lightbox="meme" width="50%" caption="Limited data is just one of the problems with MMM" attr=imgflip attrlink="https://imgflip.com" src="/img/mmm/meme_observations.jpg">}}

But don't get your hopes up just yet: Even if one had way more observations, or one believed in a model with only half the covariates, this wouldn't work. That's because the rule of thumb is only meaningful if one had independent observations and variability.


[^4]: "The most challenging data limitations for MMM include three aspects: availability, sparsity/messiness, and limited range and amount. There is no cure for low-quality data [...]" [Marketing Mix Models 102 — the Good, the Bad, and the Ugly](https://towardsdatascience.com/marketing-mix-models-102-the-good-the-bad-and-the-ugly-f5895c86b7c3)

[^5a]: Find a discussion and more references for this claim in this [CrossValidated answer](https://stats.stackexchange.com/a/29624/222007)

[^4d]: "[...] the modeler is expected to produce a MMM often with 20 or more ad channels." p.6 in [Chan, D., & Perry, M. (2017). Challenges and opportunities in media mix modeling.](https://research.google/pubs/pub45998/)

[^2e]: "Large scale simulation studies show that the model can be estimated well on a large data set, but it may produce biased estimates for the typical sample size of a couple of years of weekly national-level data." p.28 in [Jin, Y., Wang, Y., Sun, Y., Chan, D., & Koehler, J. (2017). Bayesian methods for media mix modeling with carryover and shape effects.](https://research.google/pubs/pub46001/)


### 2. Low variability

If a variable in the data is pretty much constant over time, its correlation cannot be measured. Hence, one hopes for all input variables in the Marketing Mix Model to vary. A lot. At different levels and at different points in time. Unfortunately, for most variables this will not happen. The number of stores usually is pretty constant. So are macro-economic factors. But more importantly, this also holds true for at least some of the media spending. Channel budgets might only change on a yearly basis. And for some channels the amount spent is the same, week after week, month after month. No correlation, no chance to estimate an effect on sales!  
Moreover, even high variability itself is not enough to measure correlation. One also needs to achieve some relevant threshold of spending. Your yearly revenue is 100M\$ and you're trying to model a media channel with a spend of 5k\$? Good luck finding _that_ needle.


### 3. Dependent model inputs

Model inputs are **not** independent. There are _so_ many examples for this. Here's a few obvious ones:

* Own media spending will depend on what the competition is doing
* Media spending will be much higher during christmas season
* Marketing campaigns imply that media channels are used in combination
* Promotions are supported by advertising them

I leave it to the avid reader to come up with at least ten more. But it should be clear that model inputs are highly dependent.[^4h]

{{<figure lightbox="meme" width="80%" caption="Company ad spending depends on competition and vice versa" attr=imgflip attrlink="https://imgflip.com" src="/img/mmm/meme_spiderman.jpg">}}

If a company always increases TV spending in response to their main competitor also increasing it, how to disentangle those two effects on sales? You can't! This could only be done if one had truly independent data. In consequence, requiring that one convinces someone in marketing to start spending money on TV ads randomly throughout the year. Let me know how that goes!


[^4h]: "A broader problem is that many important variables may be partially determined by the advertiser’s own ad choices." p.9 in [Chan, D., & Perry, M. (2017). Challenges and opportunities in media mix modeling.](https://research.google/pubs/pub45998/)


### 4. Unobservable / unmeasurable factors that influence sales

Remember statistics 101? Let me bring back some memories: "In statistics, omitted-variable bias (OVB) occurs when a statistical model leaves out one or more relevant variables. The bias results in the model attributing the effect of the missing variables to those that were included."[^16] We better get the specification of our model right, if we want credible results. This means that we need to include all possible factors that affect sales in our model.[^4f] So, one better takes good care while selecting the 20+ variables for the model out of the virtually infinite options. But since MMM believers are also phenomenal Data Scientist, they've already drawn out a [causal graphical model](https://www.stat.cmu.edu/~cshalizi/uADA/12/lectures/ch22.pdf) on a whiteboard. Next, they've deluded themselves into thinking that this is enough for having a "theoretical foundation" for the model.

{{<figure lightbox="meme" width="50%" caption="If omitted variable bias doesn\'t make you cry, you\'re no Data Scientist" attr=imgflip attrlink="https://imgflip.com" src="/img/mmm/meme_crying.gif">}}

So, surely their model specification is now correct beyond a shadow of doubt? Not quite. Remember the list in [1.](#1-limited-data)? I've left out a teeny-tiny detail: There are factors influencing sales that are not measurable. Even worse, sometimes they are not even observable.[^5] Think: cultural trends, implicit preferences, expectations of the future any many many more psychological factors. Still think the model specification is water-proof?


[^16]: Omitted variable bias definition on [Wikipedia](https://en.wikipedia.org/wiki/Omitted-variable_bias)

[^5]: "The conventional marketing mix model is often used to inform the tangible elements, but is lacking into two key aspects. Firstly, it ignores the role of intangibles. [...]" from [Cain, P. M. (2014). Brand management and the marketing mix model. Journal of Marketing Analytics, 2(1), 33-42.](https://link.springer.com/article/10.1057/jma.2014.4)

[^4f]: "Selection bias from MMMs perhaps represents the largest hurdle to MMMs providing valid estimates of advertising effectiveness. Selection bias occurs when an input media variable is correlated with an unobservable demand variable dt, which in turn drives sales. When this variable dt is omitted from the regression, the model has no way to attribute sales between those due to the media channel or to the underlying demand." p.8 in [Chan, D., & Perry, M. (2017). Challenges and opportunities in media mix modeling.](https://research.google/pubs/pub45998/)


### 5. Correlated inputs

Let's go back to that sweet causal graphical model on the whiteboard. Naturally, one has not only mapped out the relationship between inputs and dependent variable but also the relationship between all inputs themselves, right? Sure! Everyone knows about funnel effects in marketing and how this leads to channels influencing each other![^3b] For example, TV ads increase sales directly but also indirectly. This is because they will also increase the number of paid searches (SEA). In statistical terms, multicollinearity describes a situation in which the inputs of a multivariate regression are correlated. While this doesn't affect the predictive power of such a model, "it may not give valid results about any individual predictor, or about which predictors are redundant with respect to others".[^17] Let me rephrase this: Serious correlation between inputs means that you can't trust your model coefficients. And that means **your model sucks at explaining** things.[^4e]  
Instead of coming to terms with this, I see people going: "No biggie, I'll just use a hierarchical model to account for this minor nuisance!".  

{{<figure lightbox="meme" width="80%" caption="It\'s easy to hide flaws behind complexity" attr=imgflip attrlink="https://imgflip.com" src="/img/mmm/meme_bellcurve.jpg">}}

They use this veil of complexity to shield themselves from potential skepticism.[^3e] (Un)luckily, this often works well. Especially, for silencing higher ups that anyway don't care about "details". But deep down they know that you cannot solve fundamental statistical issues by simply adding complexity on top of them. The same problems that hold true for the base model, hold true for an "improved" model. And while the probability that the "theoretically founded" model specification is able to capture reality was already very close to zero that chance just decreased even more.  
Assuming that one _could_ solve the issue of multicollinearity for this specific case: What about all the other cases? Basically, all (media) input variables will be highly correlated. That's a fight one can't win.


[^17]: Multicollinearity on [Wikipedia](https://en.wikipedia.org/wiki/Multicollinearity)

[^3b]: "The biggest challenge in the process of any marketing mix optimization is measuring real-time cross effects and cross channel impact on business" [Pandey, S., Gupta, S., & Chhajed, S. (2021). Marketing Mix Modeling (MMM)-Concepts and Model Interpretation. Sandeep Pandey, Snigdha Gupta, Shubham Chhajed.](https://papers.ssrn.com/sol3/papers.cfm?abstract_id=3877291)

[^4e]: "When fitting a linear regression model, highly correlated input variables can lead to coefficient
estimates with high variance. This in turn, can lead to bad attribution of sales to the ad channel. [...] Another consequence is that the estimated relationship can change radically due to small changes in the data or the addition or subtraction of seemingly unrelated variables in the model." pp.6 in [Chan, D., & Perry, M. (2017). Challenges and opportunities in media mix modeling.](https://research.google/pubs/pub45998/)

[^3e]: "In reality, such fully integrated models that can capture all the effects of advertising are very complex and require substantial data. If researchers want to focus on only a few effects or their data is not rich enough, they might want to simplify the model they use to focus on only the most important effects." p.791 in [Pandey, S., Gupta, S., & Chhajed, S. (2021). Marketing Mix Modeling (MMM)-Concepts and Model Interpretation. Sandeep Pandey, Snigdha Gupta, Shubham Chhajed.](https://papers.ssrn.com/sol3/papers.cfm?abstract_id=3877291)


## Concluding on causality

As I've demonstrated, you can poke a large hole in each argument stating that the conditions needed for a causal interpretation of Marketing Mixed Modeling can be satisfied. Clearly, the source for "it's _practically_ causal" is simply "Trust me, bro"™. 

{{<figure lightbox="meme" width="60%" caption="I\'d rather trust this guy than some MMM consultant" attr=imgflip attrlink="https://imgflip.com" src="/img/mmm/meme_trust.jpg">}}

This could be the end of the story: A model that cannot uncover causal relationships, cannot help us to understand the effect of marketing. Consequently, such a model is missing its purpose. It has no value whatsoever.  
I know that you should not kick someone already lying on the ground. But why stop here when there is _so_ much more wrong with the application of Marketing Mix Modeling in the industry?


## Arbitrary modeling of media inputs

Only the simplest of MMMs assume that the effect of media is linear. Instead, the consensus seems to be that ad spending has to be transformed. But there is no consensus on which transformations to use. This is because nobody really knows. However, many models take diminishing returns + saturation and lags + carry-over effects into account.[^2f] The first, reflecting that the more you spend on a channel the less additional effect you get. This is, until you reach a saturation point where the effect is zero. The second, postulating that ads don't (only) have a direct effect. I might see the ad today but only buy the product on my next trip to the mall in a few days. While the intuition behind this approach seems reasonable, it introduces additional problems from a statistical viewpoint.  
There are numerous potential transformations that could be applied to the media inputs. Each one requires at least one additional parameter. There are two options to decide on which parameter(s) to use:

1. Trust expert opinions (usually some consultancy) or "the literature"™ and use pre-determined values
2. Infer parameters from the data, in addition to the input estimates

The first one should immediately strike you as a bad idea, because it's not very different from just taking a wild guess. Context does matter after all and there is **a lot** of context to account for in an MMM. If you want to guess about this, why don't you just go straight ahead and guess what the model results will be like? At least, you can save yourself some time.  
The second option is not much better, though. Think back to the fundamental problem of [limited data](#1-limited-data): We already have way too few observations for too many parameters. By adding more parameters to the model specification you're just making this worse.[^2d]  
In the end, both options lead to arbitrary parameter values. Thus, resulting in arbitrary media input values. Thus, leading to arbitrary model results.


[^2f]: See first paragraph on p.2 in [Jin, Y., Wang, Y., Sun, Y., Chan, D., & Koehler, J. (2017). Bayesian methods for media mix modeling with carryover and shape effects.](https://research.google/pubs/pub46001/)

[^2d]: "[...] it is usually difficult to estimate the shape and the adstock transformations well if the data consist of only a few hundred observations" p.24 in [Jin, Y., Wang, Y., Sun, Y., Chan, D., & Koehler, J. (2017). Bayesian methods for media mix modeling with carryover and shape effects.](https://research.google/pubs/pub46001/)


# Practical issues

We've seen that the claims of the usefulness of Marketing Mix Modeling break down quickly when looking at the statistical theory. But what if someone would be ignorant enough to look past that? In that case, there also are a whole lot of practical problems that should convince you to not believe the hype.

## Low data quality

There is a saying referring to statistical modeling: "Trash in, trash out". If your input data is trash, so will be your modeling results. Data quality is paramount. So, how good is marketing data on average, you ask? It's different shades of bad. Often, very bad.

{{<figure lightbox="meme" width="60%" caption="Is this a trash can in the Philippines or marketing data?" attr=imgflip attrlink="https://imgflip.com" src="/img/mmm/meme_trash.jpg">}}

Data quality is not a priority for most marketing departments but an afterthought. It simply is nothing the department is being evaluated on. Also, most marketing people are not very technical and data savvy, which further adds to the problem. Expect colorful but ill-formatted Excel files all over the place. Just ask any poor Data Scientist who ever had to gather data for a MMM. You will probably trigger her PTSD, but she will confirm that the data is either:

* Unstructured
* Decentralized
* Inconsistent
* Inaccurate
* Incomplete
* Missing

Often, all of the above. Dealing with this requires a huge effort when collecting the data. In the end, it is still a futile process: While some of those issues can be amended, on average the data set will neither be clean, complete nor correct.[^4a] Do you smell the trash?


[^4a]: "Advertisers may have to go through many separate entities in order to get data on the ad campaigns they have funded. The complexity
of this process can easily lead to missing or misinterpreting some important data" p.5 in [Chan, D., & Perry, M. (2017). Challenges and opportunities in media mix modeling.](https://research.google/pubs/pub45998/)


## Imprecise measurements

As described [above](#1-limited-data), numerous input variables are needed for estimating a MMM. For many of those, there either is no good measure or the measures differ widely between inputs. Let's focus on the marketing channels. Ideally, we want to input how many people have been reached by a specific channel in the model. After all, the more people see our ads, the more likely it is our sales increase. For online channels this is doable. Publishers are good at tracking those metrics. We can export the number of views on YouTube and TikTok and plug those in our model. But a view is not a view. Publishers not only use different definitions of an impression but this definitions might change over time. Hypothetical example: An impression on TikTok might be counted when our ad is shown to someone who immediately skips it. An impression on YouTube might only be counted, if the person doesn't skip the ad for 5 seconds. Now, for classical media, we won't even have a (precise) number for the reach. There is no way to know how many people have seen our out of home ad, and only a vague estimate for how many have listened to our radio ad. Because of this, usually the ad spend is used as a proxy for the reach of channels.[^4b] Admittedly, this gets us closer to comparing apples (\$) with apples (\$) between channels. But it's still just kicking the can down the road. What you get for your spend varies a lot over time. If you decrease your costs per click, you'll get more clicks for the same spend. Also, the quality and type of ads you use is obviously relevant. Spending the same amount on a bad vs. a great creation will have vastly different outcomes. But since these subtleties are lost when using aggregate spend as a model input, your model won't be able to pick up on this. You end up with results that don't reflect reality.


[^4b]: "[...] while the ad dollars spent can be obtained, the ad exposure data can be more difficult to collect and are often estimated in different ways depending on the media and the vendor providing the data. This is particularly true of offline media. For example in print media, while circulation numbers for publication can be provided, it is not always a good proxy for the actual number of people that see the ad" p.5 in [Chan, D., & Perry, M. (2017). Challenges and opportunities in media mix modeling.](https://research.google/pubs/pub45998/)


## Unstable data

As mentioned before, a MMM requires a few years of historic data that is stable. But marketing is fast paced. Things change all the time: You might not have two years of TikTok data, because your company didn't even know about TikTok until last year. The SEA channel used to spend money on Google optimizing on clicks three years ago. Two years ago, it was decided to optimize on impressions. Last year, this decision was reverted back but only for some campaigns. TV was only used to air image campaigns. That is, until the new Head of started to shift focus towards promotional campaigns a few months ago. See where this is going? Marketing is all _but_ stable. Particularly, nowadays when online channels play a large role for all businesses. But not only marketing strategies are volatile. The channels themselves constantly change as well. Young people don't watch linear TV anymore. The only active users on Facebook are boomers. New social media apps come and go (remember Clubhouse?). Marketing data and a requirement for stability don't go well together. But that's not all. A global pandemic hits and turns everything upside down. Hard to argue that data you observed during the first year of Covid-19 is in any way comparable to the year before. Or that you can extrapolate into the next year using it.[^2c] Marketing Mix Modeling requires stable data in a world full of change. Pandey et al. summarize this point as following: "Even tech [..] giants like Facebook has admitted that the old process of basing decisions on many years of data is now outdated. [..] In today’s environment, one of the biggest problems is that stable data doesn’t exist. Today, we have a complex web of interconnected digital channels that are always evolving and changing".[^3c]


[^3c]: "The pace of change in todays transformed marketing world has increased significantly [..]" p.791 [Pandey, S., Gupta, S., & Chhajed, S. (2021). Marketing Mix Modeling (MMM)-Concepts and Model Interpretation. Sandeep Pandey, Snigdha Gupta, Shubham Chhajed.](https://papers.ssrn.com/sol3/papers.cfm?abstract_id=3877291)

[^2c]: "[...] we increase the sample size by including data over a much longer historical period. However, this is not the recommended way to increase sample size in real problems, as other factors such as market demand and macroeconomic factors would have changed significantly over a prolonged time period." p.12 in [Jin, Y., Wang, Y., Sun, Y., Chan, D., & Koehler, J. (2017). Bayesian methods for media mix modeling with carryover and shape effects.](https://research.google/pubs/pub46001/)


## Unrealistic expectations

Implementing a Marketing Mix Model is a long and costly endeavour. Consequently, it comes with great expectations regarding the return on invest of such a project. Leadership anticipates getting a tool to allocate marketing money between channels more efficiently.[^10a] To make it worse, this is accompanied by a lack of understanding for what a model can and can't do. How many of the higher ups demanding a MMM care about the statistical limitations discussed above, let alone understand them? 

{{<figure lightbox="meme" width="80%" caption="Life as a data scientist working on MMM is not chill" attr=imgflip attrlink="https://imgflip.com" src="/img/mmm/meme_kim.jpg">}}

For the Data Scientist responsible for building the model, this inevitably leads to pressure to deliver on those unrealistic expectations. Failure is not an option. Not a great position to be in. In addition, people have strong and widely differing believes about what the model results should look like.[^8a] It's impossible to be immune to this, so it biases the Data Scientist's approach. Hence, instead of being the objective and impartial expert, she will be willing to cut corners. If your career depends on satisfying people's unrealistic expectations, it's easy to ignore obvious methodological deficiencies.[^12a] Clearly, this won't lead to the best model according to the data but rather to the one that is most likely to be accepted by the business. Doing a "good job" means confirming the expectations already held by the most influential people. This takes the undertaking of building a model ad absurdum. 


[^8a]: "But there is more to the measurement strategy than the science behind the model. Incorporating business context to shape the MMM is an art, one with implications for the model’s outcomes and final recommendations." in [Marketing mix models are based in science, but also need a touch of art](https://www.thinkwithgoogle.com/marketing-strategies/data-and-measurement/art-of-marketing-mix-models/)

[^10a]: "MMM usually involves a large amount of investment and a diverse set of stake-holders with whom alignments need to be secured. As such the bar for model interpretability is very high." p.1 in [Ng, E., Wang, Z., & Dai, A. (2021). Bayesian Time Varying Coefficient Model with Applications to Marketing Mix Modeling. arXiv preprint arXiv:2106.03322.](https://arxiv.org/abs/2106.03322)

[^12a]: "In practice, what happens is modelers get pulled in so many different directions that they end up building a franken-model that tries to address all of the different questions but doesn’t answer any of them correctly." from [Why MMM is so hard](https://getrecast.com/mmm-hard/)


## Arbitrary Model selection

As we've seen so far, there is a great ambiguity around Marketing Mix Modeling in theory and practice. Because of this, the only way to come up with a model that _looks_ credible is to run a huge number of iterations. It requires testing endless combinations of inputs and parameters and judging whether the results look "right".[^8b] Sure, one can  also take some statistical measures (e.g. R², Root Mean Squared Error etc.) into account. But those are only meaningful for assessing the predictive power of models and not their explanatory power. Hence, model selection will always be arbitrary.[^4g]

{{<figure lightbox="meme" width="50%" caption="Choosing the right MMM results is easy" attr=imgflip attrlink="https://imgflip.com" src="/img/mmm/meme_darts.jpg">}}

Need more persuasion? Let's see what the documentation of one of the most popular MMM tools has to say: "When facing multiple model candidates, it's common to prefer a model that's more "plausible", meaning one with slightly less fit but better alignment with current spend allocation [...] This is the frequently cited conflict of science and craft.".[^9b]  
I love how brutally honest those recommendations really are, if you think deeply about them. Basically, what counts is that the model results are close enough to whatever was expected beforehand, i.e. they look "plausible". As stated, this has nothing to do with science and a lot with "crafting" your own results.


[^9b]: Meta's Robyn [documentation](https://facebookexperimental.github.io/Robyn/docs/features#multi-objective-hyperparameter-optimization-with-nevergrad)

[^4g]: "The explanatory power of a few input variables–such as seasonal proxies, price and distribution–are often good enough to achieve high predictive power in absence of ad spend variables. This makes it easy to find models with high predictive accuracy by trying different specifications. Concrete examples of specification choices include whether to choose sales volume or log sales volume as the outcome variable, how to control for price and how to capture lag effects and diminishing returns. There are dozens of such choices underlying any single MMM." pp.10 in [Chan, D., & Perry, M. (2017). Challenges and opportunities in media mix modeling.](https://research.google/pubs/pub45998/)

[^8b]: "A team of Google data scientists even published a paper on how MMMs using the same data could report drastically different ROI with the same level of statistical confidence." in [Marketing mix models are based in science, but also need a touch of art](https://www.thinkwithgoogle.com/marketing-strategies/data-and-measurement/art-of-marketing-mix-models/)


# Why is MMM still popular?

If you have made it so far, you should be convinced by now that Marketing Mix Modeling is normally a waste of time and resources (if not, I congratulate you on your bright future in consulting). Why, then, is the believe in the effectiveness of MMM still so widespread?  
Companies, more than ever, strive to be "data-driven". But the inability to asses the contribution of marketing is a huge blind spot for them. Especially, as marketing spending often makes up a big chunk of the total budget. Consequently, companies are very receptive for anyone that claims that they can offer a remedy. In addition, there is also politics at play. In particular, there is pressure on the Chief Marketing Officer (similarly for the marketing channel managers) to prove that the huge amounts spent on marketing are a profitable investment. Obviously, this can be lucrative assignment and consultancies are more than happy to jump on it. They will blatantly make all kinds of overinflated and unrealistic claims on what they can deliver. Doing so, they push at the open door of the CMO eager to prove himself and his organization. Fortunately, this is a mandate sure to succeed in making marketing look good.

{{<figure lightbox="meme" width="80%" caption="Why wouldn\'t you self-asses yourself as stellar?" attr=imgflip attrlink="https://imgflip.com" src="/img/mmm/meme_medal.jpg">}}

But, it's not always just pressure from above that leads to MMM being advocated for. Marketing analytics teams themselves can also have their part in this. Similarly to the marketing department, they also need to give evidence of their contribution to the business. Therefor, contributing to answering the most fundamental question of marketing, namely what is the return of advertisement, is just too compelling. This is further fueled by the lack of honesty in the analytics industry. Critical voices or tales of failure to derive any real value from Marketing Mix Modeling are not to be found in public. To the contrary, marketing conferences and Linkedin posts are littered with success stories that stress the importance of using MMM.[^12b] The fact that the alternative of measuring on a granular user level is becoming increasingly hard (with tracking prevention on the rise), greatly adds to the perceived urgency.[^13] In the last years this narrative has also gained significant support through contributions of Google[^8d] and Meta[^15]. Both of which have a self-serving interest in supporting marketing departments in "quantifying" the contribution of channels, primarily their own.  
The one vindication against those imposters is the ability to cut through their bs by having the right combination of skepticism and methodological knowledge. Unfortunately, most decision makers lack at least the latter. Consequently, the mentioned factors contribute to the FOMO of analytical leaders: "Everyone else is doing it and boasting great results, so we must also do it to not fall behind."  
The ones most likely to realize that Marketing Mix Modeling is a wasted effort are the practitioners working on the model. In the course of a MMM project even the most confident ones will become disillusioned with the feasibility. However, they are unlikely to speak up about it. Their inability or unwillingness to deliver the expected results (see [above](#unrealistic-expectations)) might be construed as personal failure and not as the virtue of speaking up against a vain effort. Leadership will be tempted to see this as the opinion of one, lone data scientist arguing against a multitude of influential public "experts". It's not unreasonable to believe that most practitioners will shy away from that fight.


[^12b]: For example [Meta's MMM summit](https://getrecast.com/meta-mmm-summit/)

[^13]: "At stake is an entire system for measuring digital marketing effectiveness by tracing the digital footprint of the consumer from ad exposure to conversion. The Marketing Mix Model sidesteps this challenge [...]" in [5 reasons why a Marketing Mix Model is relevant today](https://www.gfk.com/blog/5-reasons-why-marketing-mix-model-is-relevant-today)

[^8d]: See: [Marketing mix models are based in science, but also need a touch of art](https://www.thinkwithgoogle.com/marketing-strategies/data-and-measurement/art-of-marketing-mix-models/)

[^15]: See: [Considerations for Creating Modern Marketing Mix Models](https://www.facebook.com/business/news/insights/considerations-for-creating-modern-marketing-mix-models)


## What to do about it?

Ultimately, how should one deal with the conflict between the need to quantify marketing effectiveness and the inability of Marketing Mix Modeling to do so? The simple answer is: Stop waisting resource on MMM and focus on analytical tasks that have a proven value instead. But, as I've argued above, many companies are not following this path due to irrationality.[^19] Then, how can we improve the situation? Obviously, this is only possible if the involved parties are willing to do some introspection. In that case, we can come up with some advice according to the different business roles.


[^19]: Irrational behavior refers to the whole company. The individual behavior of the involved people might well be rational.


### Leadership

Leadership has to be more skeptical when being confronted with MMM success stories. They need to assess the potential motives behind claims. Can a consultancy that promises guaranteed results really be trusted to deliver? Is the speaker at the marketing conference who boasts about the success of MMM in his company interested in sharing his knowledge? Or does he use the stage to promote himself and his business? Are Google, Meta and other publishers really interested in helping companies discover their marketing ROAS? Or do they want to enable decision makers to be able to keep spending money on their channels no matter what? Part of this can be achieved through critical thinking. But the topic is complicated and requires methodological expertise to fully grasp. In consequence, decision makers have to rely on their in-house experts and trust their advice more than their own gut feeling. This requires a culture in which expertise and good arguments trump seniority and title. For critical voices to speak up, leadership needs to foster such an environment. Unfortunately, many places are riddled with politics and people put their own career above what is best for the company. Coincidentally, those will also be the places where you'll find a religious believe in Marketing Mix Modeling.


### Analytical practitioners

It's easy for analytical practitioners to be forced into a corner once higher ups have set their mind on demanding a Marketing Mix Model. If leadership is already convinced that MMM is a good idea, it will be delicate to change the course of things. This is because of the unrealistic expectations mentioned [above](#unrealistic-expectations).

{{<figure lightbox="meme" width="50%" caption="Friends don\'t ask friends how their MMM project went" attr=imgflip attrlink="https://imgflip.com" src="/img/mmm/meme_vince.gif">}}

To avoid that, one needs to be present in the early decision making process and be able to openly share skepticism there. It's paramount to voice concerns in such a way that they can be understood by non-technical folks. Unfortunately, the influence one can assert will often not be enough to avoid the ask for a MMM project. This is sub-optimal. But there is still things one should do in that case. Primarily, Data Scientists must be vocal about the limitations of modeling and what a model can and cannot do from the start.[^4i] Managing stakeholders' expectations will be significant to cover your own ass (CYOA). Part of that strategy can be to frame the project as R&D. One can promise a best effort in modeling but warn that the outcome can't be foreseen. This implies that the project is allowed to fail. In addition, one can stress that results will significantly depend on data quality. Should one be blamed for not fulfilling expectations one can at least deflect that blame towards the data. CYOA.  
Being the naysayer doesn't earn a lot of sympathy. Thus, one also needs to offer alternatives to Marketing Mix Modeling. Fortunately, there are analytical methods out there that are proven to work. No one method will be able to uncover the exact contribution of marketing and the ROAS. Still, they can shed light on some relevant aspects. Experiments like A/B testing, Geo Experiments and other causal impact methods are strong contenders for useful alternatives to offer.[^3f]


[^4i]: "It is important for the modeler to be forthright about the types of questions a MMM can and can’t answer. It is especially important to also acknowledge the uncertainty in the modeling process and the need for transparency between the modeler and the end user of the model results. As part of that transparency, the modeler should be forthright about the capabilities and limitations of MMMs." p.15 in [Chan, D., & Perry, M. (2017). Challenges and opportunities in media mix modeling.](https://research.google/pubs/pub45998/)

[^3f]: "Randomized controlled experiments are by far the most accurate method of measuring causal effects available. But they can typically only test one or two things at a time, they can be difficult to administer [...]" p.792 [Pandey, S., Gupta, S., & Chhajed, S. (2021). Marketing Mix Modeling (MMM)-Concepts and Model Interpretation. Sandeep Pandey, Snigdha Gupta, Shubham Chhajed.](https://papers.ssrn.com/sol3/papers.cfm?abstract_id=3877291)


# Conclusion

Marketing Mix Modeling is believed to be an indispensable tool for companies to asses the return on their marketing spending. Self-serving actors promoting MMM fuel that believe, while opposing standpoints are rare. Nonetheless, in reality numerous statistical and practical arguments show that this believe is wrong or at least overinflated.

> The combination of some data and an aching desire for an answer does not ensure that a reasonable answer can be extracted from a given body of data. – John Tukey

Leadership and analytical practitioners are well-advised to be less gullible and focus on facts instead of self proclaimed "experts". Consequently, they should redirect their efforts from MMM to more worthwhile marketing measurement methods.
