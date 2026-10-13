---
type: "practice"
difficulty: "Expert"
---

# A director asks whether to precompute features into a store nightly or compute on-request at serving, as a platform-wide default. How would you build the decision framework across latency, freshness, and cost, and where do you allow exceptions?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Estimating GenAI Cost and Latency](<../../../../TOOLS/Python/Python_Language/Python_Tail_Latency/Estimating_GenAI_Cost_and_Latency.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To decide between precomputing features into a store nightly or computing them on-request at serving, we need to carefully balance three key factors: latency, freshness, and cost. This decision will impact the overall performance and efficiency of our machine learning platform.

The core question is whether it's better to:

Precompute features daily (or hourly) and store them in a persistent database.
Compute features on-demand as they are requested during model serving.

Let’s break down these factors:

Latency
Precomputed Features: Lower latency since the data is readily available from the store.
On-request Computation: Higher latency, as computation needs to be performed at runtime.
Freshness
Precomputed Features: Risk of staleness if not updated frequently enough. However, nightly updates can provide a good balance.
On-request Computation: Real-time freshness, but may introduce delays or need more complex caching strategies.
Cost
Precomputed Features: Initial storage costs for the computed features and potential costs of maintaining the database. Could also involve ongoing compute costs if features are updated frequently.
On-request Computation: Lower initial cost since no persistent storage is required. However, this could lead to higher operational costs due to increased computation overhead.

To build a robust decision framework, we need to consider both static and dynamic dimensions of our platform:

Static Considerations:

System constraints: How many features are being computed? What is the expected volume of requests?
Storage constraints: What is the available storage capacity for precomputed features?

Dynamic Considerations:

User experience: Does the application have real-time requirements that necessitate on-demand computation?
Data dynamics: Are the features highly volatile, requiring frequent updates? Or are they relatively static with less change over time?
Step-by-Step Solution

Latency Analysis:

Identify critical paths where low latency is essential.
For non-critical paths, precomputation can be more efficient.

Freshness Evaluation:

Determine the acceptable staleness period for each feature.
Ensure that nightly or hourly updates are sufficient to maintain freshness without excessive overhead.

Cost Optimization:

Evaluate storage costs versus compute costs over time.
Implement cost-effective strategies such as incremental precomputation, caching, and dynamic scaling of resources.
Exceptions
For features with extremely high volatility or real-time requirements, on-request computation is essential.
Certain use cases might justify a hybrid approach where some features are precomputed while others are computed on-demand based on context.

#### Closing Summary

In summary, the decision between precomputation and on-demand feature computation depends on a nuanced balance of latency, freshness, and cost. By analyzing these factors within both static and dynamic contexts, we can optimize our platform to meet user needs efficiently. Exceptions should be reserved for scenarios where real-time or high-frequency updates are critical.

#### Common Mistakes
Ignoring the trade-offs between latency and freshness.
Failing to consider cost implications of storage versus compute resources.
Overlooking the importance of dynamic context in decision-making, such as user experience and data dynamics.

**Comparison:**

| Feature      | Name           | Latency Tolerance (ms) | Freshness Required? |
|--------------|----------------|-----------------------|--------------------|
| 1            | user_profile   | 10                    | No                 |
| 2            | transaction_history | 2                  | Yes                |

**Code:**

```python
# Example code snippet for evaluating feature computation strategy

def evaluate_computation_strategy(features, latency_tolerances, freshness_requirements):
    results = []

    for feature in features:
        precompute_latency = calculate_precompute_latency(feature)
        on_request_latency = calculate_on_request_latency(feature)

        if precompute_latency <= latency_tolerances[feature] and \
           (not freshness_requirements[feature] or is_freshness_acceptable(precompute_latency, feature)):
            results.append("Precompute")
        else:
            results.append("On-demand")

    return results
```

**Example input / output:**

```text
Input:
- Features = [
    {"name": "user_profile", "latency_tolerances": 10, "freshness_requirements": false},
    {"name": "transaction_history", "latency_tolerances": 2, "freshness_requirements": true}
]
- Latency Tolerances = [10, 2]
- Freshness Requirements = [false, true]

Output:
- Computation Strategy = ["Precompute", "On-demand"]
```

**Data:**

```json
{
  "features": [
    {"name": "user_profile", "latency_tolerances": 10, "freshness_requirements": false},
    {"name": "transaction_history", "latency_tolerances": 2, "freshness_requirements": true}
  ]
}
```

**Flow:**

1. Feature Evaluation
2. Static Analysis
3. Latency, Freshness, Cost
4. Dynamic Context
5. On-demand vs Precomputed
6. Final Strategy

