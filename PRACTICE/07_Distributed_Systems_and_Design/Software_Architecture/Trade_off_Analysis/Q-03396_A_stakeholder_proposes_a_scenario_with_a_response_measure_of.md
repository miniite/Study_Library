---
type: "practice"
difficulty: "Deep"
---

# A stakeholder proposes a scenario with a response measure of 'the model never hallucinates.' How do you convert that unrealistic absolute into a measurable groundedness/citation-coverage/error-budget target the GenAI architecture can actually be evaluated against?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Regulatory Requirements in Utility Trees](<../../../../07_Distributed_Systems_and_Design/Software_Architecture/Trade_off_Analysis/Regulatory_Requirements_in_Utility_Trees.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In this scenario, we need to break down the concept of "the model never hallucinates" into concrete metrics and targets that can guide our architectural design and evaluation processes. This involves a deep dive into understanding what hallucination means in an AI context, identifying relevant trade-offs, and establishing measurable performance indicators.

First, let's define hallucination: it refers to instances where the AI produces information that is not supported by any input data or known facts. Hallucinations can be harmful as they may lead to incorrect decisions based on false information. Our goal is to transform this abstract concept into tangible metrics that we can implement and monitor.

#### Triaging the Question

The core of this question is about converting an idealistic requirement (no hallucination) into a practical, measurable target. We need to consider the following:

Scope: What types of hallucinations are we concerned with?
Contextual Understanding: How do different use cases define and handle hallucinations differently?

#### Trade-offs
Implementing strict controls might reduce flexibility; how do we balance these factors?

#### First-Principles Solution
Step 1: Define Hallucination Types

Hallucinations can be categorized into types such as:

Factual Errors: Incorrect data that contradicts known facts.
Imaginative Errors: Information generated from imagination, not based on any input or fact.
Synthesis Errors: Combining facts in a way that creates new information inaccurately.
Step 2: Establish Metrics

We need to establish metrics for each type of hallucination:

Factual Error Rate (FER): Percentage of predictions that contain factual errors.
Imaginative Error Rate (IER): Frequency of generated content that is purely imaginative.
Synthesis Error Rate (SER): Proportion of synthesized information that introduces inaccuracies.
Step 3: Set Performance Targets

Based on the use case, set realistic targets for each metric:

For a medical application, an FER below 1% might be acceptable.
For legal applications, both IER and SER should ideally approach zero.
For creative applications, higher imagination tolerance might be acceptable.
Step 4: Build Error Budgets

Implement an error budget that tracks these metrics over time. The error budget will help us understand when we are approaching our defined thresholds for hallucinations.

#### Closing Summary
+ Common Mistakes/Follow-Ups

By breaking down "never hallucinates" into specific, measurable targets, we can create a robust evaluation framework. However, common pitfalls include:

Over-engineering: Striving for absolute perfection might lead to overly complex solutions.
Ignoring edge cases: Failing to account for rare but critical scenarios can leave significant gaps in performance.

#### Follow-ups
could involve discussing how these metrics interact with other aspects of the architecture such as latency, resource usage, and user experience.

**Comparison:**

```
Hallucination Type	Definition	Target Metric
Factual Error	Contradicts known facts	<1%
Imaginative Error	Purely imaginative content	<5%
Synthesis Error	Inaccurate synthesis of existing information	<30%
```

**Code:**

```python
# Example code snippet to track hallucination rates

def evaluate_hallucination(model_output, ground_truth):
    factual_errors = 0
    imaginative_errors = 0
    synthesis_errors = 0

    for prediction, truth in zip(model_output, ground_truth):
        if is_factual_error(prediction, truth):
            factual_errors += 1
        elif is_imaginative_error(prediction):
            imaginative_errors += 1
        else:
            synthesis_errors += 1

    total_predictions = len(model_output)

    fér = factual_errors / total_predictions * 100
    ier = imaginative_errors / total_predictions * 100
    ser = synthesis_errors / total_predictions * 100

    return {"FER": fér, "IER": ier, "SER": ser}

def is_factual_error(prediction, truth):
    # Implement logic to check factual errors
    pass

def is_imaginative_error(prediction):
    # Implement logic to check imaginative errors
    pass
```

**Example input / output:**

| Input                                                                 | Output                                                                                                                                                                |
|-----------------------------------------------------------------------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| Ground Truth: ["The Earth is flat", "Humans evolved from apes"]<br>Model Output: ["The Earth is round", "Humans did not evolve from apes"] | {"FER": 0, "IER": 100, "SER": 0}                                                                                                                                     |

**Data:**

```json
{
  "ground_truth": ["The Earth is flat", "Humans evolved from apes"],
  "model_output": [
    "The Earth is round",
    "Humans did not evolve from apes"
  ]
}
```

**Flow:**

1. Yes
2. Initiate Model Evaluation
3. Is Ground Truth Available?
4. Collect Model Output andGround Truth
5. Evaluate Hallucination RatesUsing Metrics
6. Monitor Error Budgets
7. Adjust Architectural Design ifNeeded
8. Iterate Until Targets Met

