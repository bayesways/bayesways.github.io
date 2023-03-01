---
title: How many German tanks did the Allies not see?
author: ''
date: '2021-07-17'
slug: how-many-german-tanks-did-the-allies-not-see
categories: []
tags:
  - realworld
subtitle: ''
---

This is about a math triumph during World War II. While preparations for D-Day are under way, the allies' plan is threatened by news from Italy. The Germans have rolled out a new tank model Panzer V, which threatens the superiority of allied tank thus far. This model was seen before only in small numbers but now rumors of a large fleet of them are spreading. In order to decide whether to go ahead or not with the invasion plans, the allies desperately need to know how many of these new tanks do the Germans have. Presumably if the enemy has a lot then the invasion plan has to be cancelled and redesigned. Otherwise, the plan has to be expedited before the Germans have a chance to build more. 

The allies had capture a few of these tanks and were looking for any evidence that can help them estimate how many of them could there be, or how many are being produced. Obviously counting them all is not possible. Allied mechanics found production numbers printed on the motors and chasis of the captured tanks. Perhaps each model was numbered according to the order that it was made in, they thought. Under that assumption, what's the right estimate of the total population of tanks? The assumption turned out to be right, and the statistcal estimate the war mathematicians came up with turned out to be highly accurate. Wikipedia says *"According to conventional Allied intelligence estimates, the Germans were producing around 1,400 tanks a month between June 1940 and September 1942. Applying the formula below to the serial numbers of captured tanks, the number was calculated to be 246 a month. After the war, captured German production figures from the ministry of Albert Speer showed the actual number to be 245."*

Reading about this problem, coined the German tank problem, I came across many solutions. Wikipedia even makes a distinction between the frequentist and the Bayesian answer, claiming the Bayesian answer is questionable. I had to investigate. 

If the captured sample has size $S$ with numbers $x_1< x_2< ...< x_S$ what is the true population size $N$? Of course $N \geq x_S$, but can we do better? One answer [2] claims to be $\frac{(x_S-1)(S+1)}{S}$. Another formula [3] claims to be $x_S + m$ where $m$ is the average distance between all $x_{i+1} - x_i$. Another formulat [4] claims that the MLE is $x_S$.

The same question arises in estimating the prevelance of a disease in a population, or the size of an animal population [3].

Is Bayesian really different? Check [7]. 

Read [6]

1. [The German Tank Problem: Math/Stats at War!](https://web.williams.edu/Mathematics/sjmiller/public_html/math/talks/GermanTankProblem_Talk_Hampshire2019.pdf)

2. [How a statistical formula won the war](https://www.theguardian.com/world/2006/jul/20/secondworldwar.tvandradio?utm_source=pocket_mylist)

3. [Data sleuths go to war](https://web.archive.org/web/20010418025817/http://www.newscientist.com/ns/980523/features.html#data)

4. [How many tanks? MC testing the GTP](https://statisticsblog.com/2010/05/25/how-many-tanks-gtp-gets-put-to-the-test/)

5. [The German Tank Problem](https://www.statisticalconsultants.co.nz/blog/the-german-tank-problem.html)

6. [An Empirical Approach to Economic Intelligence in World War II](https://www.tandfonline.com/doi/abs/10.1080/01621459.1947.10501915)

7. [Bayesian Estimation of the Size of a Population](https://epub.ub.uni-muenchen.de/2094/1/paper_499.pdf)