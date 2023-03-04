---
title: Is the sex of siblings independent?
author: ''
date: '2018-01-23'
slug: is-the-sex-of-siblings-independent
categories: []
tags:
  - realworld
subtitle: ''
output:
  html_document:
    toc: true
    number_sections: true
    fig_caption: true
bibliography: ./sexratio.bib
---

In this post I want to answer the question "Does the gender of the first birth affect the gender of subsequent births?".

Someone told me that if two parents have a baby boy then their future babies are more likely to be boys than girls. This sounds plausible, and generally a useful thing to know. But how exactly can we test this? This question led me to study the gender ratio in general.

$$\mathbf{Pr}( \text{boy} > 0.5) \text{ ?}$$

It's interesting that the gender ratio, the ratio of male births over female births, is systematically above 50%, across the world and through history.  This is already surprising. Even more interestingly, this ratio fluctuates over time and appears to be affected by socioeconomic effects such as wars and biological ones, such as the parental age. 

But first, before delving further into the topic, let's make sure we define the research question. The question refers to the gender at birth. This is different from the number of males over females in existence, which is affected by more factors, such as the life expectancy of each sex. 

Wikipedia [lists](https://en.wikipedia.org/wiki/List_of_countries_by_sex_ratio) the sex ratio for all countries broken down by age groups, including at birth. The common trend across the world is that males are born more often but die younger. So the effect of this discrepancy between the genders at birth to the world population at large, is washed away by the fact that men last less.


# History

Many people were interested in this question, among whom we find some mathematical royalty. As @Fellman2011 says

"John Graunt (1620–1674) was the first person to compile data showing an excess of male births to female births and to note spatial and temporal variation in the SR. John Arbuthnot (1667–1735) demonstrated that the excess of males was statistically significant and asserted that the SR is uniform over time and space (Campbell 2001). Referring to christenings in London in the 82 years up to 1710, Arbuthnot suggested that the regularity in the SR and the dominance of males over females could not be attributed to chance and must be an indication of divine providence. Nicholas Bernoulli’s (1695–1726) counter-argument was that Arbuthnot’s model was too restrictive. Instead of a fair coin model, the model should be based on an asymmetric coin. Based on the generalized model, chance could give uniform dominance of males over females. Later, Daniel Bernoulli (1700–1782), Pierre Simon de Laplace (1749–1827) and Siméon-Denis Poisson (1781–1840) also contributed to this discussion (David 1962; Hacking 1975)."


## General Points
"Some general features of the SR can be noted. Stillbirth rates are usually higher among males than females, and the SR among stillborn infants is markedly higher than normal values, but the excess of males has decreased during the last decades. Hence, the SR among liveborn infants is slightly lower than among all births, but this difference is today very minute. Further, the SR among multiple maternities is lower than among singletons. In addition to these general findings, the SR shows marked regional and temporal variations."

## Notes

There are whole books written about this topic @armitage2005, and a recent article  @Fellman2011.

God's hand: from @chahnazarian1988determinants
  "Sussmilch invoked the
wisdom of the Creator in ensuring that
"4 to 5 per cent more boys than girls are
born, thus compensating for the higher
male losses due to the recklessness of
boys, to exhaustion and to dangerous
tasks, to war, to sailing, to emigration,
and Who thus maintains the balance between
the two sexes so that everyone
can find a spouse at the appropriate time for marriage. (see Sussmilch, 1979. t.
II, p. 505)" from (SUSSMILCH, J. P. 1979. L'Ordre divin aux origines
de la demographic Traduction originale,
avec des etudes et commentaires rassembles
par J. Hecht. Institut National
d'Etudes Demographiques, Paris.)




Reverse question : "gage the effect of a treatment on the population by measuring the sex ratio, e.g. @davis1998reduced

Possible determinants:

* review @chahnazarian1988determinants
* wars 
* natural disaster @fukuda1998
* parental behavior and natural selection @trivers1973natural

* Paternal Age and Birth Order increase chance of male
* Maternal Age does not seem to affect it
* Higher Socioeconomic status increase chances of male birth
* Whites birth more males than Blacks
* Mortality rate at different stages can affect the ratio
* "In fact,
one hypothesis suggests that the sex ratio
at birth is a direct reflection of the sex
ratio at conception and is determined by
the levels of maternal hormones circulating
at that time. This hypothesis
would explain the racial determinant and fluctations around war" from @chahnazarian1988determinants. 
* However, if war effect is explained by above then how can we explain the natural disasters effect? 

Parental preferences and continuing giving birth until families get sons @park1995consequences

Things to investigate:  

* European decline of ration @grech2000declining and @martuzzi2001declining
* Can ratio be predicted by variation of factors across different countries?

data sources: 

* tables at @mathews2005trend 
* https://catalog.data.gov/dataset/health-data-interactive-hdi
* https://www.cdc.gov/nchs/data_access/vitalstatsonline.htm
* The CIA, yes the CIA, publishes [information]( https://www.cia.gov/library/publications/the-world-factbook/fields/2018.html)
on this. 


# References

