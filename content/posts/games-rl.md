---
title: "Games and Reinforcement Learning"
date: "2024-10-01"
summary: "RL in games are cool"
description: "RL in games are cool"
toc: false
readTime: true
---

## [Atari](https://arxiv.org/abs/2111.00210), [pokemon](https://www.youtube.com/watch?v=rhvj7CmTRkg), [Monoply](https://www.youtube.com/watch?v=dkvFcYBznPI) and [Dark Souls](https://www.youtube.com/watch?v=eJ6J_2zThC8) (or, I really want to get back to RL for games)

I think the coolest type of ML personal projects are the type where we are doing game strategy discovery through AI, or RL to train bots. I think if resources allow, there's no better bug checker than a RL system. Atari gyms (Atari 100k and its ilk) are well known "benchmarks", but in this particular case, the codebase is great and the training efficiency is great. It's also a semi survey paper to ge into the history of techniques used for this benchmark.

The Pokemon and Monoply examples are super fresh. They represent the two directions I have stated: in the pokemon one we are building a human level agent and in the monoply one, we are (re) discovering strategies. I think this is the absolutely best part of these game RL agents: both create insights and create interaction, much more so than the standard supervised and unsupervised projects.

The dark souls one seems a bit dead, but let me see whether I can find some code for it! Worse come to worse we always have [this tutorial](https://www.youtube.com/watch?v=ks4MPfMq8aQ&list=PLQVvvaa0QuDeETZEOy4VdocT7TOjfSA8a) to work on. Sentdex is an absolute legend.

Best place to start: [stable retro](https://github.com/Farama-Foundation/stable-retro). It's Gym, but with retro games
