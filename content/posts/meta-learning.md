---
title: "Meta Learning"
date: "2024-06-06"
summary: "Not that meta"
description: "v"
toc: false
readTime: true
autonumber: true
math: true
tags: ["misc", "work"]
showTags: true
hideBackToTop: false
---


## [Meta](https://data-notes.co/meta-learning-is-all-you-need-3bd0bafdf289) [Learning](https://data-notes.co/bayesian-meta-learning-is-all-you-need-1bcff6b889fc) [is](https://data-notes.co/unsupervised-meta-learning-is-all-you-need-71b6dfa29ccd) [all](https://machinelearningmastery.com/meta-learning-in-machine-learning/) [you](https://lilianweng.github.io/lil-log/2018/11/30/meta-learning.html) [need](https://arxiv.org/abs/2004.05439)

Meta-Learning are certainly very useful when large datasets are not available, or when data can have long and weird tails with potentially disastrous outcomes. With designing the algorithm and/or training process to be adaptive, or generalize across task, we can use meta-learning techniques to defend against out of sample tasks, quickly adapt existing algorithms to new tasks, and training towards a new regime given small example of training data.

In the simplest sense, meta-learning is either approaching the results as some meta representation of the problem set, or training a system that learns how to learn. For the former, we can view techniques using results to tune or stack as a form of generalization, which aids in the goal of further generalizing across tasks. For the latter, very basically, we can approach this problem set with model-based, optimization-based or metrics based approaches.

In brief summary, model based methods look to use a model that has "high learning capacity", and looks to quickly retrain the model. If the model can update its parameters with a few training steps, the model is effectively adaptive. The downside being that these methods tend to still require large datasets to perform well. For this type we can look at Memory-Augmented Neural Networks or Meta-Networks type works.

Optimization based methods, on the other hand, looks to tune the optimizer such that the optimization step is more efficient per optimization step for similar problems, and find a optimizer efficient for the specific task. The problem seems to be instability of the optimization, when especially introducing second order effects. Techniques like Model-Agnostic Meta-Learning are part of this family.

Metrics based, or non-parametric, aims to compare train and test through some sort of similarity, if there are similar labels in training data that is of the testing, we update the labels appropriately. If we know how "close" the problem sets are, we can effectively adapt outputs to the new problem without doing too much training. The challenge here of course, is finding then correct metric, and trying to identify the correct relationship between the training problem and testing, target problem. See relation networks and Siamese networks for this type of meta learners.

Anyway, the survey paper gives a bunch of different potential places we can go with meta-learning techniques, with interesting places such as few-shot, active learning (unsupervised meta-learning), adversarial defense, and architecture search and evolution.
