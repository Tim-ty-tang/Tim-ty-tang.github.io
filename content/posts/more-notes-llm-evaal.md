---
title: "More notes on LLM evals"
date: "2024-09-25"
summary: "Adapted from a presentation"
description: "Adapted from a presentation"
toc: false
readTime: true
---

## More notes on LLM evals

### Building Effective Evaluation Datasets

Creating a good evaluation dataset is crucial for meaningful LLM assessment. Here are some tips for building effective datasets:

1. **Consistent Outputs**: Aim for structured outputs when possible. For example, multiple-choice questions provide a clear framework for both prompting and evaluation.

2. **Step-by-Step Reasoning**: Design tasks that encourage the model to break down its thinking process. This approach facilitates easier self-critique and analysis.

3. **Tool Integration**: Consider incorporating tool use (e.g., search engines, calculators) into your evaluation tasks to assess the model's ability to leverage external resources.

4. **Evaluation-Centric Design**: When creating your dataset, keep in mind how you'll evaluate the responses. This foresight can greatly simplify the assessment process.

5. **Fuzzy Matching**: Develop strategies for fuzzy matching between the model's outputs and target responses. This approach allows for more flexible and nuanced evaluation.

6. **Rich Metadata**: Include detailed metadata about input types, variations, and other relevant factors. This information can be invaluable for in-depth analysis and iterative improvement.

### Evaluating Plausibility of Synthetic Data

In some scenarios, particularly when sharing data between parties, there's a need to generate synthetic data that closely mimics real data. This synthetic data can be used for querying and analysis while protecting the privacy of the original dataset. LLMs can be powerful tools for generating such synthetic data, but how do we ensure its plausibility?

Here's a step-by-step approach to evaluating and improving synthetic data generation:

Step 1: **Define the Problem and Criteria**: Create a structured system prompt that includes

- Context: Background information about the data domain

- Example/data format: Clear specifications for the desired output

- Criteria: Specific requirements for plausibility and accuracy

Step 2: **Generate Sample Responses**: Use your initial prompt to create a small set of synthetic data examples

Step 3: **Develop an Evaluation Strategy**: Based on the sample responses, decide on an evaluation approach. This could involve human review, automated checks, or a combination of both

Step 4: **Iterate and Refine**: Generate a larger sample set and use your evaluation strategy to identify areas for improvement. Tweak your prompting and context as needed

This iterative process not only helps in creating better synthetic data but also establishes a framework that can be adapted for different use cases by modifying the system prompt.

#### Prompting Guide for Synthetic Data Generation

To combat common issues like hallucination and bias towards popular but potentially inaccurate information, consider the following strategies:

1. **Provide Real Data Samples**: Offer the model access to a prepared CSV file or API containing real (but non-sensitive) data examples. This can help ground the model's outputs in reality.

2. **Multi-Stage Prompting**: Implement a two-step process where the model first generates a sample, then self-evaluates using additional data or search capabilities. This approach can help overcome context window limitations and incorporate richer data.

3. **External Scaffolding**: Use techniques like geo-boxing (providing geographical boundaries) to constrain and guide the model's outputs.

4. **Avoid Generic Prompts**: Highly specific and contextual prompts tend to yield better results than broad, generic instructions.

### Scaling Up: LLM-Driven Evaluations

While manual evaluation is often necessary in the early stages of development, scaling up requires more automated approaches. LLM-driven evaluations can be surprisingly effective when properly structured. Here's how to implement this approach:

1. **Create Structured Evaluation Prompts**: Design prompts that clearly outline the evaluation criteria and expected format of the assessment.

2. **Use a Separate Evaluation Agent**: To maintain objectivity, use a different LLM instance or model for evaluation than the one being tested.

3. **Provide Context**: Give the evaluation model access to relevant background information or data to inform its assessment.

4. **Implement a Scoring System**: Develop a clear, quantifiable scoring method that the evaluation model can use to rate responses.

5. **Human Verification**: Regularly sample and manually review a subset of the LLM-driven evaluations to ensure their accuracy and identify any systematic biases.

Remember that while LLM-driven evaluations can greatly increase the scale of your assessment, they should be used in conjunction with, not as a replacement for, human oversight.
