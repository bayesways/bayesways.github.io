---
layout: post
date: 2024-05-26
title: Would you hire GPT as an equity analyst?
giscus_comments: true
related_posts: false
tags: 
---
This week I am looking at a new paper "Financial Statement Analysis with LLMs" from a Chicago Booth team. The paper tries to rigorously test whether LLMs can do financial fundamental analysis. Before we get into the meat of the paper, we note that the context matters here. The human analysts historically have a 53% accuracy of predicting correctly. In other words, a machine needs to do be 3% better than random to beat the human specialists. Naturally one wonders how does the profession justify its existence with 53% accuracy. This requires a whole other post, but for now we can note that 53% is the accuracy of the consensus view, which means that in any given moment about half of the analysts are right and the other half are wrong. This reminds me of the old advertisement saying "only half of my advertisement budget works, problem is I don't know which half". Anyway, let's leave aside the absolute accuracy for the remaining of this post, and let's focus on the relative accuracy of machine versus human. 

First the results, the paper finds that the LLMs are worse than humans out of the box, but better than humans if tweaked on a specific task. To prove this, the authors set up an experiment. Used historical data of financial statements, without any additional context, and removing names and years from the dataset. This was done to minimize [ML leakage](https://en.wikipedia.org/wiki/Leakage_(machine_learning)), i.e. a form of model cheating where contextual information from the future leaks into the training set that allows the model to cheat when making a prediction. The authors wanted to test the LLMs ability to analyze the pure numbers from a financial statement. *The target variable is directional change of earnings.* The dataset 1983-2021 about 40,000 observations for about 3,000 distinct companies.

Second, the authors compared 2 LLM methods against the human's 53% accuracy. The first was a simple prompt method which performed at 49%. The second was a chain-of-thought setup (CoT) which performed at 60%. **So we can conclude that as far as financial statement analysis goes, the LLMs + prompting outperforms human consensus.** Note the word "consensus", which means that the LLMs are not better than the best humans necessarily, but are better than the average. There is a deeper question here, which is the consensus representative of the humans' ability in this space. I am not sure about this, since there are a lot of analysts out there and I am not sure what criteria were used to pick the sample for this paper. But one can imagine that the 53% figure is sensitive to the choice of the analysts that were used. 

There are some additional interesting results in the paper that put the human consensus accuracy in the context of bias variance tradeoff. The authors created 3. more forecasts:
 - a logistic regression trained an expanded set of features (*53%*)
 - a NN trained on the smaller set of features extracted from the documents fed to the LLM (*59%*)
 - an NN trained on the expanded set of features (*63%*)
The conclusion of this is:
 1) humans are comparable to a logistic regression which is consistent with prior literature as well. 
 2) more sophisticated algorithms outperform humans by comparable
 3) LLMs are general purpose machines that can be adapted to perform on par with specialized algorithms



## References

 - Kim, Alex, Maximilian Muhn, and Valeri V. Nikolaev. "Financial Statement Analysis with Large Language Models." _Chicago Booth Research Paper Forthcoming, Fama-Miller Working Paper_ (2024). [PDF](https://bfi.uchicago.edu/wp-content/uploads/2024/05/BFI_WP_2024-65.pdf)