---
title: "Pure and Applied"
date: "2024-12-11"
summary: "Some thoughts on the two cultures"
description: "and some libs to look at"
toc: false
readTime: true
---

## [Pure](https://www.nature.com/articles/s41586-021-04086-x?s=09) and [Applied](https://www.technologyreview.com/2020/10/30/1011435/ai-fourier-neural-network-cracks-navier-stokes-and-partial-differential-equations)

How I wish to be a better writer and coder back then! I think the two sides form two very interesting distinct use cases of machine learning technique in maths (in my uneducated views). The former is using efficient universal function approximators to generate observable data and its associated approximate supervised model, and through those patterns and results we can reformulate our hypothesis, forming a positive feedback cycle in understanding the underlying problem. The paper provided examples in knot theory and representation theory, where divergent in fields produces interesting datasets to form supervised targets.

The latter, I think is closer to a traditional applied maths topic, is the use of some new DL layer to better solve some classes of PDEs. The key interests here are two-fold: Better representations of PDEs through a new neural operator, but also practically, a machine learning based solver that's more efficient and accurate. This speaks to me greatly in particular, because I do think representation of the problem space is likely the most important thing in DL for generalization and speed. In essence, we can view this as a better simulator.

Broadly speaking, this also represents what I see as two cultures in machine learning in practise, understanding driven and results driven. In most given problem set, we either want to know the answers quickly and accurately, or we want to know an explanation for the problem. I do not think they are too far apart, but it does show the diverse ways machine learning techniques can be applied in conjunction with human in the domain of "knowledge generation".

Which leads me to two new libs I have looked at (at least for understanding driven):

## [Interpret.ml](https://interpret.ml/) and [ACV00](https://github.com/salimamoukou/acv00)

I complain about interpretability and explainability tools a lot, but here are two promising libraries I have been playing with:

Interpret ML is an interesting "wrap-all" library for explainability techniques and contains most of our favourite explainability techniques. It already brings more value that something like [Shapash](https://github.com/MAIF/shapash), mostly because it contains less opiniated usage, and it's more flexible. I think the "Explainable Boosted Machine" implementation is for sure the most interesting part of it all. Very briefly, it's building Generalized Additive Models through pairwise interaction and bagging and boosting. The boosting is restricted to one feature at a time, which lowers training speed, but allows feature functions to be isolated. Individual predictions are made through each individual predictor, so is transparent and independently attributable. For sure an interesting method to add, though I am a bit sceptical with the SOTA performances, especially compared to extreme boosted machines.

ACV is another method to derive **Shapley Values** from tree models. From what I can see the idea of probabilistic sufficient explanations is choosing a minimal subset of features that explains the results, hence forming the sufficient explanation (or sufficient reason or prime implicant). In particular to ACV, the conditional probability (or Same-Decision Probability) of maintaining this same prediction can be estimated via random forests. Unlike SHAP or LIME, the subset of features are guaranteed to be maintain he same predictions; it further considers feature interactions and data distribution in a meaningful way. For sure worth a test drive, along with the other technique to build a good explainability product.
