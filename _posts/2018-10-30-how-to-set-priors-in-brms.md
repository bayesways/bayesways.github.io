---
layout: post
title: How to set priors in `brms`
date: '2018-10-30'
categories: statistics R
tags: brms
giscus_comments: true
related_posts: false
---


{% highlight R %}
knitr::opts_chunk$set(echo = TRUE)
{% endhighlight %}

{% highlight R %}
library(MASS)
library(brms)
library(ggplot2)
{% endhighlight %}

## Generate simple data
Let's assume that our data $y$ is real valued and our predictor is "Age".

{% highlight R %}
y <- rnorm(30, mean = 1, sd = 2)
dat1 <- data.frame(y)
dat1$age <- rnorm(30, mean = 40, sd = 10)
head(dat1)
{% endhighlight %}

## Understanding the priors
Let's imagine we want to create a simple model of the form $$y \sim N(\mu, \sigma)$$ where $$\mu = a + b \cdot Age$$. 

{% highlight R %}
model_formula<- bf(y ~ 1 + age)
{% endhighlight %}

To understand the model we need to use `get_prior` and `make_stancode`. We need to explore the output of the two to learn how to set custom priors. 

{% highlight R %}
get_prior(model_formula,data = dat1, family = gaussian())
{% endhighlight %}

The best way to enter priors is to save the output dataframe of `get_priors` and edit it directly. Let's first explore the resulting *Stan* code to understand where the priors will be set
{% highlight R %}
make_stancode(model_formula,data = dat1, family = gaussian())
{% endhighlight %}


We see that parameters are `b;  // population-level effects` , `real temp_Intercept;  // temporary intercept`, and `real<lower=0> sigma  // residual SD`. Let's match that to the dataframe of priors

{% highlight R %}
priors <- get_prior(model_formula,
            data = dat1, family = gaussian())
priors

{% endhighlight %}
Let's give normal priors to the 
{% highlight R %}
priors$prior[2] <- "normal(0,10)"
priors$prior[3] <- "normal(0,1000)"
priors$prior[4] <- "cauchy(0,10)"

priors
{% endhighlight %}
And check that the generated Stan code reflects the change
{% highlight R %}
make_stancode(bf(y ~  age),data = dat1, family = gaussian(), prior = priors)
{% endhighlight %}

Note that if we had made a typo or passed a distribution that is not part of Stan we would get an error. The following code is **not** evaluated, if it was it would raise an error.  

{% highlight R %}
priors$prior[4] <- "clichy(0,10)"
make_stancode(bf(y ~  age),data = dat1, family = gaussian(), prior = priors)
{% endhighlight %}

### References

* See documenation for [`get_prior`](https://www.rdocumentation.org/packages/brms/versions/2.4.0/topics/get_prior)