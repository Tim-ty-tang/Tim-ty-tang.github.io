---
title: "Confidence, Decisions and Uncertainty"
date: "2024-07-14"
summary: "Some thoughts on confidence, decisions and uncertainty"
description: "It's all about the confidence"
toc: false
readTime: true
---

## [Confidence](https://nbviewer.org/github/sdcastillo/Prior-Weights-with-XGboost/blob/master/xgb_with_prior.html), [Decisions](https://netflixtechblog.com/building-confidence-in-a-decision-8705834e6fd8) and [Uncertainty](https://towardsdatascience.com/modeling-uncertainty-in-neural-networks-with-tensorflow-probability-part-1-an-introduction-2bb564c67d6)

One gap in my modelling is approaching problems in a probabilistic manner. It's all good in a open setting to produce a score of n% accuracy, or other one number metrics, but more often than not, this is not what could translate to actionable results. From a action perspective, it seems far more important to convey how confident the model is towards a score, as humans in the loop would like to have a risk evaluation to come with any kind of machine prediction.

In the posts on using priors with XGBoost and tensorflow probability of in building known distributions or imparting confidence inside the model. For the XGBoost, it can be used with prior belief to avoid predictions that make do not make sense in aggregate, and the tensorflow probability lets a layer be replaced by a distribution (which if using `DenseVariational` we can even define the prior and posteriors!). Both can help push modelling towards a probabilistic manner.

In any case, pyro.ai is a PyTorch version of tensorflow probability, and I will take a look at it in the following weeks.

Another important factor to consider in thinking probabilistic is the meta aspect: this obviously includes our own decision making, for example, release decisions, uncertainty in A/B testing, etc. To build a data-driven case we need to consider:

- Do the results align with the hypothesis?
- Does the metric story hang together?
- Is there additional supporting or refuting evidence?
- Do the results repeat?
- Can we construct a decision utility?
- Can we run backtesting? what macro decisions might impact the backtesting results?

Food for thought
