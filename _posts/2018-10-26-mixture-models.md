---
layout: post
title: Mixture models
author: ''
date: '2018-10-26'
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
library(dplyr)
{% endhighlight %}


Let's say we want to find a symple model where $y \sim N(\mu_i, \sigma_i^2)$ where the probability of i-th point to belong to cluster 1 is denoted by $\eta_i$ and the cluster variable is denoted by $\alpha_i$. Simulate some data


{% highlight R %}
set.seed(1234)
dat <- data.frame(
  y = c(rnorm(200), rnorm(100, 6))) #alpha = 0 for i:0->200, alpha=1 for i:201->300
head(dat)
{% endhighlight %}

Fit a simple normal mixture mode

{% highlight R %}
mix <- mixture(gaussian, gaussian)
prior <- c(
  prior(normal(0, 7), Intercept, dpar = mu1),
  prior(normal(5, 7), Intercept, dpar = mu2)
)
fit1 <- brm(bf(y ~ 1), dat, family = mix,
            prior = prior, chains = 2)
{% endhighlight %}


{% highlight R %}
summary(fit1)
{% endhighlight %}


{% highlight R %}
plot(fit1, N = 2)
{% endhighlight %}

{% highlight R %}
y0<- data.frame(y = c(rnorm(5), rnorm(5, 6)))
pp_check(fit1, newdata = y0, type = 'intervals')
{% endhighlight %}


## Classify data points

Mixture models can also be used for classification tasks. Given a point $y$ we can ask $P(\alpha_i = 1)$ ?

{% highlight R %}
y0<- data.frame(y = c(rnorm(5), rnorm(5, 6)))
ppm<- pp_mixture(fit1,
         newdata = y0)
head(ppm[, 1, ])
apply(ppm[, 1, ], 1, which.max)

{% endhighlight %}

What's our predictive distribution? It's a weighted average of the means from each cluster. 


{% highlight R %}
y0<- data.frame(y = rep(0,10)) # dummy data
predict(fit1,newdata = y0)
{% endhighlight %}

we can also extract the probabilities $\eta_i$ for each point. 

{% highlight R %}
fitted(fit1,newdata = y0,
         dpar = 'theta')
{% endhighlight %}


## Model the latent variable

The probability $eta_i$ is usually a function of the covariates. So the observable model is the same, $y \sim N(\mu_i, \sigma_i^2)$, but $\text{logit}(p) = \eta(x)$. 


{% highlight R %}
set.seed(1234)
x <- c(rnorm(50,0),rnorm(50,2))
alpha <- ifelse(x<0.5, 0, 5)
dat <- data.frame(
  x = x,
  y = rnorm(mean=alpha,n=100))
head(dat)
{% endhighlight %}
Create a model where $\eta = A + B \cdot x$ 

{% highlight R %}
fit3 <- brm(bf(y ~ 1, theta1 ~ x),
            dat, family = mix, prior = prior,
            inits = 0, chains = 2)

{% endhighlight %}


{% highlight R %}
summary(fit3)
pp_check(fit3, type='intervals')
{% endhighlight %}
create 

{% highlight R %}
y0<- data.frame(x = c(rnorm(5 , 0), rnorm(5,2)),
                y = rep(0,10)) # dummy data
{% endhighlight %}


Find the probability of belonging to each cluster 


{% highlight R %}
df <- fitted(fit3,newdata = y0,
         dpar = 'theta1')
print(df, digits = 2)

{% endhighlight %}


{% highlight R %}
ppm<- pp_mixture(fit3, newdata = y0)
apply(ppm[, 1, ], 1, which.max)

{% endhighlight %}
and plot the marginal effect of $x$ on the probability of cluster

{% highlight R %}
marginal_effects(fit3, effects = 'x', dpar='theta1')
{% endhighlight %}

## References

* small [tutorial](https://rdrr.io/cran/brms/man/mixture.html) on mixtures with brms
