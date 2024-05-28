---
layout: post
date: 2024-05-26
title: Would you hire GPT as an equity analyst?
giscus_comments: true
related_posts: false
tags:
  - llm
---
This week I am looking at a new paper "[Financial Statement Analysis with LLMs](https://papers.ssrn.com/sol3/papers.cfm?abstract_id=4835311)" from a Chicago Booth team. The paper tries to rigorously test whether LLMs can do fundamental analysis. Before we get into the meat of the paper, we note that the context matters here. The human analysts historically have a 53% accuracy of predicting correctly. In other words, a machine needs to do be 3% better than random to beat the human specialists. Naturally one wonders how come the specialists only have 53% accuracy. So we note that 53% is the accuracy of the median of the analysts. I don't have the raw data but I imagine the best analysts score meaningfully above 53%. In any case this means that in any given moment about half of the analysts are right and the other half are wrong, which reminds me of the old advertisement saying "only half of my advertisement budget works, problem is I don't know which half" (attributed to [J. Wanamaker](https://en.wikipedia.org/wiki/John_Wanamaker)). Anyway, let's leave aside the absolute accuracy for the remaining of this post, and let's focus on the relative accuracy of machine versus human. Let's dive into the specifics of the paper. 

First the results, the paper finds that the LLMs are worse than humans out of the box, but better than humans if tweaked on a specific task. To be specific, LLMs, in this paper, mean GPT-4-Turbo. To prove this, the authors set up an experiment. Used historical data of financial statements, without any additional context, and removing names and years from the dataset. This was done to minimize [ML leakage](https://en.wikipedia.org/wiki/Leakage_(machine_learning)), i.e. a form of model cheating where contextual information from the future leaks into the training set that allows the model to cheat when making a prediction. The authors wanted to test the LLMs ability to analyze the pure numbers from a financial statement. *The target variable is annual directional change of earnings*, i.e. predicting whether the next year's earnings will go up or down relative to the current year. The dataset comes from [Compustat](https://www.marketplace.spglobal.com/en/datasets/compustat-financials-(8)) which "provides standardized North American and global financial statements and market data for over 80,000 active and inactive publicly traded companies that financial professionals have relied on for over 50 years." The paper filtered the data to the years 1983-2021 with about 40,000 observations for about 3,000 distinct companies.

They compared 2 LLM methods against the human's 53% accuracy. The first was a simple prompt method which performed at 49%. The second was a chain-of-thought setup (CoT) which performed at 60%. **So we can conclude that as far as financial statement analysis goes, LLMs + CoT prompting outperforms human consensus.** We note again the inclusion of the word "consensus", which means that the LLMs are not better than the best humans necessarily, but are better than the average. This begs the question "is the consensus representative of the humans' ability in this space". I am not sure about this, since there is no discussion in the paper about the criteria used to pick the sample for this paper. But one can imagine that the 53% figure is sensitive to the choice of the analysts that were used in the data pool. It would be interesting to see more specifics around this in order to understand not only what is the accuracy of the average analyst but also how do the machines fare against the best humans, not just the average. 

There are some additional interesting results in the paper that put the human consensus accuracy in the context of bias variance tradeoff. The authors created 3 more forecasting methods with the corresponding accuracy scores (in parenthesis):
 - a logistic regression trained on an expanded set of features (*53%*)
 - a NN trained on the smaller set of features extracted from the documents fed to the LLM (*59%*)
 - an NN trained on the expanded set of features (*63%*)

The conclusion of this is:
 1) the average humans are comparable to a logistic regression which is consistent with prior [literature](https://papers.ssrn.com/sol3/papers.cfm?abstract_id=1528987) 
 2) more sophisticated algorithms outperform the average human by a considerable margin, but we don't know if they beat the best humans
 4) LLMs are general purpose learners that can be adapted to perform on par with specialized algorithms


## References

 - Kim, Alex, Maximilian Muhn, and Valeri V. Nikolaev. "Financial Statement Analysis with Large Language Models." _Chicago Booth Research Paper Forthcoming, Fama-Miller Working Paper_ (2024). [PDF](https://bfi.uchicago.edu/wp-content/uploads/2024/05/BFI_WP_2024-65.pdf)
 - Bradshaw, Mark T and Drake, Michael S. and Myers, James N. and Myers, Linda A., A Re-Examination of Analysts’ Superiority over Time-Series Forecasts of Annual Earnings (2012). Review of Accounting Studies, Vol. 17, No. 4, 2012, [PDF](https://care-mendoza.nd.edu/assets/152185/bradshawpaper.pdf)
 