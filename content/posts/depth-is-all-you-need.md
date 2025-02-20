---
title: "Depth is All You Need"
date: "2025-01-03"
summary: "A short post for the essential reading exploring the basics of LLMs and VLMs"
description: "Exploring the basics of LLMs and VLMs"
toc: false
readTime: true
---

Exploring the basic depths of LLMs and VLMs, A review of LLM and VLM content. I have been working on the basics, but now is a good time to explore the depths and refresh.

Here is a list of resources I have gone through:

The top 3 gets us up to date in data manipulation, basic architecture of attention, and tokenisation.

- makemore by [Andrej Karpathy](https://github.com/karpathy/makemore)
- minbpe by [karpathy](https://github.com/karpathy/minbpe)
- attention? attention! [Lilian Weng](https://lilianweng.github.io/posts/2018-06-24-attention/)

Then we get into actually training a model. Inccluding llm.c, which is worth the deepdive

- gpt-2 again by [karpathy](https://www.youtube.com/watch?v=l8pRSuU81PU)
- llama3 from scratch by [naklecha](https://github.com/naklecha/llama3-from-scratch)
- llm training in simple, raw by c/cuda [karpathy](github.com/karpathy/llm.c)

And some extra bits we can get into once the basics is complete, esepcially surrounding modern techniques, such as Q/Lora, quantisation, evals, MoE, ViTs, and Flash Attention.

- decoding strategies in large language models [mlabonne](https://huggingface.co/blog/mlabonne/decoding-strategies)
- how to make llms go fast by [vgel](https://vgel.me/posts/faster-inference/)
- a visual guide to quantization [maarten](https://newsletter.maartengrootendorst.com/p/a-visual-guide-to-quantization)
- the novice's llm training guide by [alpin](https://gist.github.com/btbytes/cf845f9ade1cb34348110c14c8c49cea)
- a survey on evaluation of large language models [paper](arxiv.org/abs/2307.03109)
- mixture of experts explained [huggingface](huggingface.co/blog/moe)

Some notes on ViTs, CLIP, and Paligemma

- vision transformer by [aman-arora](https://amaarora.github.io/posts/2021-01-18-ViT.html)
- clip, siglip and paligemma by [umar-jamil](https://www.youtube.com/watch?v=vAmKB7iPkWw)

Some further extra reading, (less truly evergreen, but still useful)

- extending the RoPE by [eleutherai](blog.eleuther.ai/yarn/)
- Flash attention by [Alesia Gorkic](https://gordicaleksa.medium.com/eli5-flash-attention-5c44017022ad)
