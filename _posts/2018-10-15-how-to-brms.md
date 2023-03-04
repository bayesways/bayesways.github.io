---
layout: post
title: How to brms?
author: ''
date: '2018-10-15'
categories: []
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

Let's assume that our data $$y$$ is real valued and our predictor is "Age" and a group number. 
{% highlight R %}
y <- c(rnorm(15, mean = 1, sd = 2), rnorm(15, mean = 10, sd = 2.5))
dat1 <- data.frame(y)
dat1$age <- rnorm(30, mean = 40, sd = 10)
dat1$grp <- c(rep(1,15),rep(2,15))
head(dat1)
{% endhighlight %}
We would like to model this data according to the real-valued predictor `age`, as well as the categorical predictor `grp`. What can go wrong? 


# Encode `as.factor` categorical features
Let's imagine we want to create a simple model of the form $$y \sim N(\mu, \sigma)$$ where $$\mu = a_i + b \cdot Age$$, where $$a_i$$ is the intercept for i-th group. That can be easily encoded in the brms formula

{% highlight R %}
model_formula<- bf(y ~ 0 +  grp + age)
{% endhighlight %}

but it's crucial that the data is encoded accordingly
{% highlight R %}
dat1$grp <- as.factor(dat1$grp)
{% endhighlight %}
and then fit
{% highlight R %}
fit0 <- brm(model_formula,
data = dat1, family = gaussian())
{% endhighlight %}

# Examine the posterior
Let's say that we fit our model without caring much about enconding the data
{% highlight R %}
y <- c(rnorm(15, mean = 1, sd = 2), rnorm(15, mean = 10, sd = 2.5))
dat1 <- data.frame(y)
dat1$age <- rnorm(30, mean = 40, sd = 10)
dat1$grp <- c(rep(1,15),rep(2,15))
fit0 <- brm(model_formula,
data = dat1, family = gaussian())
{% endhighlight %}
and examine the posterior samples with `posterior_summary`
{% highlight R %}
posterior_summary(fit0)
{% endhighlight %}
The estimate for the effect of group is wrong. We need to refit the model using the encoded data. 

# Fit model to new data?
How to fit an existing model to new data? Use ` update`. 
{% highlight R %}
dat1$grp <- as.factor(dat1$grp)
fit1<- update(fit0,newdata = dat1) # avoids re-compiling
{% endhighlight %}

{% highlight R %}
posterior_summary(fit1)
{% endhighlight %}

# Generate Stan code
This is a great feature that helps understand the model and automate a lot of coding
{% highlight R %}
make_stancode(model_formula,
                   data = dat1, family = gaussian())
{% endhighlight %}


# How to define the model?
Let's imagine we want to create a simple model of the form $$y \sim N(\mu, \sigma)$$ where $$\mu = a + b \cdot Age$$. 
That model can be defined in two ways

{% highlight R %}
# case 1
model_formula<- bf(y ~ 1 + age)
{% endhighlight %}

The number `1` can be ommitted in this case. To confirm it run the following code, does it output `TRUE`?
{% highlight R %}
# case 1
c1<- make_stancode(bf(y ~  age),
                   data = dat1, family = gaussian())
# case 2
c2<- make_stancode(bf(y ~ 1+ age),
                   data = dat1, family = gaussian())
identical(c1,c2)
{% endhighlight %}

If instead we wanted $$\mu = b \cdot Age$$ we would write `y ~  0 + age`, to indicate that the constant term is 0. 

{% highlight R %}
# case 3
c3<- make_stancode(bf(y ~ 0 + age),
                   data = dat1, family = gaussian())
identical(c1,c3)
{% endhighlight %}


# References

* See documentation for [`update`](https://www.rdocumentation.org/packages/brms/versions/2.4.0/topics/update.brmsfit)
* See documentation for [`posterior_summary`](https://www.rdocumentation.org/packages/brms/versions/2.4.0/topics/posterior_summary.brmsfit)
* See documentation for [`brmsformula`](https://www.rdocumentation.org/packages/brms/versions/2.4.0/topics/brmsformula)
