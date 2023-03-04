---
layout: post
title: Model fit checks
date: 2019-01-29
tags: brms
categories: statistics R
giscus_comments: true
related_posts: false
---


{% highlight R %}

library(MASS)
library(brms)
library(ggplot2)
{% endhighlight %}


Let's assume that our data $$y$$ is real valued and our predictor is "Age" and a group number. 
{% highlight R %}

y <- c(rnorm(15, mean = 1, sd = 2), rnorm(15, mean = 10, sd = 2.5))
dat1 <- data.frame(y)
dat1$age <- rnorm(30, mean = 40, sd = 10)
dat1$grp <- as.factor(c(rep(1,15),rep(2,15)))
head(dat1)
{% endhighlight %}

We would like to model this data according to the real-valued predictor `age`, as well as the categorical predictor `grp`. Let's imagine we want to create a simple model of the form $$y \sim N(\mu, \sigma)$$ where $$\mu = a_i + b \cdot Age$$, where $a_i$ is the intercept for i-th group. That can be easily encoded in the brms formula

{% highlight R %}

model_formula<- bf(y ~ 0 +  grp + age)
{% endhighlight %}


{% highlight R %}

fit0 <- brm(model_formula,
data = dat1, family = gaussian())
{% endhighlight %}


# Posterior summary
examine the posterior samples with `posterior_summary`
{% highlight R %}

posterior_summary(fit0)
{% endhighlight %}


we can also plot the posterior samples for each parameter
{% highlight R %}

plot(fit0)
{% endhighlight %}


# Posterior predictive checks
`pp_check` is a handy function for visualizing the predictions of the model. 
{% highlight R %}

pp_check(fit0)
{% endhighlight %}

By default it ouputs an overlayed density plot. Another handy visualization is the predictive intervals for each datapoint against it's true value. 
{% highlight R %}

pp_check(fit0,
         type = 'intervals')
{% endhighlight %}


Type `pp_check(type = xyz)` for a full list of options.

Let's assume we have 10 new datapoints, together with their true values,
{% highlight R %}

age <- rnorm(10, mean = 40, sd = 10)
dat_new <- data.frame(age)
dat_new$grp <- as.factor(c(rep(1,5),rep(2,5)))
dat_new$y <- c(rnorm(5, mean = 1, sd = 2), rnorm(5, mean = 10, sd = 2.5))
{% endhighlight %}


we can get redictions from our model
{% highlight R %}

pp_check(fit0,
         type='intervals',
         newdata = dat_new)
{% endhighlight %}


# Predictions

Each data point gets a predictive distribution that is implied by the posterior samples.
Let's say we have one data point 
{% highlight R %}

y0 <- expand.grid(age = 30 ,
                         grp = 2)
{% endhighlight %}

Our model predicts the following value
{% highlight R %}

predict(fit0, newdata = y0)
{% endhighlight %}

To see the full values of values type
{% highlight R %}

predict(fit0, newdata = y0, summary=FALSE)

# or equivalently
posterior_predict(fit0, newdata = y0)
{% endhighlight %}


Recall that the model is $$y \sim N(\mu, \sigma)$$ and hence the predictions incorporate the uncertainty that stems from the variance $$\sigma$$. There is a special function, called `fitted` for the prediction of the average $$\mu = a + b * Age$$
{% highlight R %}

fitted(fit0, newdata = y0)
{% endhighlight %}

Note how the estimate is very close to the value proivded by `predict`. The difference lies in the predicted percintile intervals, which are much narrower for the `fitted` function, as they should be.

Finally there is `marginal_effects` which visualizes the effect of the values of the features on the observed outcome $y$. For example we can see the effect of the group as follows
{% highlight R %}

marginal_effects(fit0, effects = 'grp')
{% endhighlight %}

there is mode options to visualize interactions or other details
{% highlight R %}

marginal_effects(fit0, effects = 'grp:age')
{% endhighlight %}


# Model comparison
Let's say we want to examine if ignoring the age effect makes for a better model. We would model it as follows
{% highlight R %}

model_formula2<- bf(y ~ 0 +  grp )
fit2 <- brm(model_formula2,
data = dat1, family = gaussian())
{% endhighlight %}


One method to compare the models is to compute the approximate leave-one-out cross-validation. The smaller the value the better. Hence we see that model 2 wins here.
{% highlight R %}

loo(fit0,fit2)
{% endhighlight %}

Another way to compare models is to compute their weights if they were to be combined
{% highlight R %}

loo_model_weights(fit0, fit2)   
{% endhighlight %}


Another method is WAIC, again the smaller the better. 
{% highlight R %}

waic(fit0,fit2)
{% endhighlight %}


There is also a handy method for k-fold cross validation.
{% highlight R %}

kfold(fit2, K = 3)
{% endhighlight %}



# Model Averaging 
If we wanted to extract posterior samples from a weighted average of the two models we can use 
{% highlight R %}

posterior_average(fit0, fit2, weights = "loo")
{% endhighlight %}


And to retrieve direct predictions 
{% highlight R %}

pp_average(fit0, fit2, method = "fitted", newdata = y0)
{% endhighlight %}



# References

* See documentation of [pp_check](https://www.rdocumentation.org/packages/brms/versions/2.4.0/topics/pp_check.brmsfit)
* For examples of [bayesplot](http://mc-stan.org/bayesplot/articles/), the package behind `pp_check`
* See documentation of [predict](https://www.rdocumentation.org/packages/brms/versions/2.4.0/topics/predict.brmsfit)
* See documentation of [fitted](https://www.rdocumentation.org/packages/brms/versions/2.4.0/topics/fitted.brmsfit)
* See documentation of [marginal_effects](https://www.rdocumentation.org/packages/brms/versions/2.4.0/topics/marginal_effects.brmsfit)
* See documentation of [loo](https://www.rdocumentation.org/packages/brms/versions/2.4.0/topics/loo.brmsfit)
* See documentation of [kfold](https://www.rdocumentation.org/packages/brms/versions/2.4.0/topics/kfold.brmsfit)
* See documentation of [posterior_average](https://www.rdocumentation.org/packages/brms/versions/2.4.0/topics/posterior_average.brmsfit)
* See documentation of [pp_average](https://www.rdocumentation.org/packages/brms/versions/2.4.0/topics/pp_average.brmsfit)



