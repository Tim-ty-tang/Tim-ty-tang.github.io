---
title: "Notes on Inference risks for AI systems"
date: "2025-03-13"
summary: "Some Notes on Inference risks for AI systems"
description: "Some Notes on Inference risks for AI systems"
toc: false
readTime: true
autonumber: true
math: true
tags: ["ML", "security"]
showTags: true
hideBackToTop: false
---

## Intro on Inference risks for AI systems

Sharing data and models across boundaries enable groups of stakeholders to rapidly create integrated and optimized system solutions. This is especially true for digital twin style systems, where components modelling is key. These system solutions need to be able to confidently share data and outputs from models whilst preventing IP leakage, so that they can understand the impact their decisions have on the whole system. This knowledge will lead to better system level decisions and lead to better outcomes for all.

To facilitate this knowledge sharing, the model provider will need to facilitate the IP protection and orchestration of model components into system level models that can be run to assess impact and optimism subsystem variables on whole system outputs. This generally means model or data will be accessed by users (technical or UI) through a secured API.

From the perspective of the users, interacting with a model API is effectively interacting with a blackbox, especially in the case machine learning models[^1]. As such, this presents a new genre of security risks and failure modes associated with adversarial usage, unintended outputs, and system attacks on data and algorithm specifically, which are separate from traditional software security concerns[^2]. As such, A new set of considerations and approaches would need to be considered.

[^1]: This is highly dependent on the model, but generally as the complexity of a model grows, the less interpretable and explainable it is. This is related to the bias-variance tradeoff, but also impacts the various risks when deploying machine learning systems into production. For example, for rule based or linear systems, the risk of unintended outputs is much less compared to a large language model, but conversely, it is likely easier to reverse engineer the former compared to the latter.

[^2]: Of course failure to address traditional security concerns will enhance the AI/model system specific risks. This especially applies to access risks and supply chain risks. In a sense, for AI/ML driven systems, AI/ML specific risks are of a secondary concern, but covers a larger part of its life cycle compared to traditional infosec in software engineering.

In general, we can separate model specific failure modes into intentional and non-intentional, and also into security risks during inference or training. Non-intentional and training risks, such as data poisoning, distributional shifts, or supply chain attacks, comes in many flavours and types, but generally should be considered the responsibility of the model owners[^3]. As such from the perspective of a information exchange an API marketplace, we will focus in further describing the risks that are intentional and from an inference point of view.

[^3]: This in and of itself is a large topic for data science in a production/live environment. In general these types of issues can be approached from two angles, either from the training side, or from the operations side. Training side mitigation include data verification techniques, masking and tokenization for storage and training, or business rules based filters to safeguard model outputs. Operations mitigation include relevant model telemetry, pipeline security, or input validations.

Here are some model specific, machine learning or otherwise, intentional failure modes specifically for inference:

### Perturbation Attacks

Perturbation attacks are generally a malicious user manipulates the input query to get a desired output form the production system. This can be used to either avoid safeguards, such as avoiding censor systems in language models, or avoiding the system entirely, such as avoiding facial detection in ML driven security systems.

On a base level, this can be achieved by effectively either querying the system until the needed perturbation is found, or finding incorrect examples in the training set to exploit the resulting models. More advanced techniques include adding noise to diffuse samples into desired labels, or even using an reinforcement learning adversarial system.

### Model/membership Inversion

Features or training data used in models could be recovered by repeated querying and observing the resulted outputs. This can be problematic, due to privacy or intellectual property issues, such as using proprietary information for training.

This type of attack is in general part of a family of reverse engineering techniques called hill-climbing attacks, where adversaries iteratively submit synthetic data and exploit the corresponding outputs to guide the iteration process until a false acceptance is attained. This is relatively common in biometrics, where a users biometrics can be recovered through this technique.

If sensitive data is used for training purposes, the models trained can potentially "memorizes" the data verbatim instead of the desired generalized form. This can become a privacy risk, such as leaking whether user data was part of the training set. Generally this is of greater concern for high complexity models such as deep neural networks.

### Model Theft

 Given sufficient access to query the model, there is a risk to fully or partially replicate the model. The victim model can be queried to form the training and testing set for a surrogate. A fully replicated local model can be used maliciously, or expose IP contained within the model.

 More sophisticated attacks can include using stolen models to craft adversarial examples, or access IP embedded inside the model, such as proprietary data. In extreme cases, the adversary can combine the extracted information with their own knowledge to supersede the victim model.

Model theft difficulty is correlated with two main factors: the availability of the data input and output, and the model complexity. For example, unmodified inputs and outputs available without pre and post processing is more at risk compared to masked inputs and truncated outputs. A large language model likely has a data and processing moat compared to a linear regression, which prevents model theft using iterative methods.

In particular, we are interested in exploring the limits and indicators of model theft and model inversion, and furthermore design possible countermeasures and controls against this particular failure mode. In abstract, if a model is given unlimited access, it is likely the model can be replicated[^4] and further developed. As noted above, generally the difficulty of model theft is correlated with complexity[^5], and therefore any investigation will likely involve looking at model theft at different types of model complexity, to determine whether there is a difference in difficulty or strategy involved. From there, we can determine the correct mitigation that would fit the model in question.

[^4]: In fact explainability techniques uses similar techniques to map out the marginal contributions for each feature. See SHAP and LIME, or surrogate model techniques. Furthermore in a sense, this is a form of self supervision; you are using a machine learning model on unlabeled data to generate labels for downstream use, with the distinction being the downstream is not improving an existing model, but generating a surrogate.

[^5]: Though newest research indicates that new large general models are very capable of generating self supervised training sets!

## References and Resources

See below for some interesting resources generally surrounding inference attacks and risks.

### White papers and Presentations

Here are some industry presentation and whitepapers on machine learning specific threat vectors and model theft methods. This forms the core of our understanding and planning.

[Microsoft whitepaper](https://learn.microsoft.com/en-us/security/engineering/failure-modes-in-machine-learning)

A good overview of failure modes in machine learning, along with some reasonable mitigation methods. Provides a rich taxonomy to identify potential weakness and vulnerabilities, and ties well with traditional infosec concerns.

[Huawei whitepaper](https://www-file.huawei.com/-/media/corporate/pdf/trust-center/ai-security-whitepaper.pdf)

A white paper with a focus on 4 more general failure modes focused on perturbing results (Evasion, Poisoning, Backdoor, and Model theft.)

[Kubecon slides](https://docs.google.com/presentation/d/1Etn50JQdOL9Lsa065sngUmIZ0yk63NKwlXDNiHR0Ta4/edit#slide=id.g9f7867562c_0_0) and [Presentation](https://www.youtube.com/watch?v=a3_nPe3pJ8I)

Introductory talk for MLOps and security from a MLOps perspective. Specifically for Kubeflow which is a MLOps platform. Raises thoughts on equivalency of source code and models.

[HITB Seccon Reverse Engineering Models](https://conference.hitb.org/hitbsecconf2018dxb/materials/D1T1%20-%20AI%20Model%20Security%20-%20Reverse%20Engineering%20Machine%20Learning%20Models%20-%20Kang%20Li.pdf) and [Presentation](https://www.youtube.com/watch?v=Dn3jb2BBBCE)

A more holistic overview on model security topics. More from a security researcher's perspective, but worth considering to understand further vulnerabilities in systems with machine learning components.

[Stealing Machine Learning Models via Prediction APIs](https://arxiv.org/abs/1609.02943) and [Presentation](https://www.youtube.com/watch?v=BD7RcRLkk_0)

This is one of the original papers on model theft. The authors specifically look at the context MLaaS in the cloud as a target (AWS). The available talk is a great summary of the paper, which should inform of the general approach we will be taking.

### Papers and Research

Some more supplemental papers on more interesting techniques in model theft/model inversion.

[A Survey of Privacy Attacks in Machine Learning](https://www.semanticscholar.org/paper/A-Survey-of-Privacy-Attacks-in-Machine-Learning-Rigaki-Garc%C3%ADa/4e5c8648cc363a341eca2020d2238ab4e8e2a3ba)

[A Survey on Security Threats and Defensive Techniques of Machine Learning: A Data Driven View](https://www.semanticscholar.org/paper/A-Survey-on-Security-Threats-and-Defensive-of-A-Liu-Li/4a8c332b09bb99333a8bce6a4640a20c1352aa63)

[AUTOLYCUS: Exploiting Explainable AI (XAI) for Model Extraction Attacks against Decision Tree Models](https://arxiv.org/abs/2302.02162)

[Efficiently Stealing your Machine Learning Models](https://www.semanticscholar.org/paper/Efficiently-Stealing-your-Machine-Learning-Models-Reith-Schneider/304ac5d62f2888e94078715413c40cf00b58ac4f)

[Practical Black-Box Attacks against Machine Learning](https://arxiv.org/abs/1602.02697)

[Membership Inference Attacks Against Machine Learning Models](https://www.semanticscholar.org/paper/Membership-Inference-Attacks-Against-Machine-Models-Shokri-Stronati/f0dcc9aa31dc9b31b836bcac1b140c8c94a2982d)

[Towards Data-Free Model Stealing in a Hard Label Setting](https://www.semanticscholar.org/paper/Towards-Data-Free-Model-Stealing-in-a-Hard-Label-Sanyal-Addepalli/35ade8553de7259a5e8105bd20a160f045f9d112)

[Stealing Hyperparameters in Machine Learning](https://www.semanticscholar.org/paper/Stealing-Hyperparameters-in-Machine-Learning-Wang-Gong/0f5476c9629f8093e8ba8c6a41868415c6a7f2f1)

[Model Extraction Warning in MLaaS Paradigm](https://www.semanticscholar.org/paper/Model-Extraction-Warning-in-MLaaS-Paradigm-Kesarwani-Mukhoty/181fe8787937dbf7abf886042855be1bc6149f80)

[Generic Black-Box End-to-End Attack Against State of the Art API Call Based Malware Classifiers](https://www.semanticscholar.org/paper/Generic-Black-Box-End-to-End-Attack-Against-State-Rosenberg-Shabtai/29ef90762761693ff585452d8ed1f1b36933692e)

[Data-Free Model Extraction](https://www.semanticscholar.org/paper/Data-Free-Model-Extraction-Truong-Maini/287e82b08cc5b8c8aae03825b466fcb73860b0c4)
