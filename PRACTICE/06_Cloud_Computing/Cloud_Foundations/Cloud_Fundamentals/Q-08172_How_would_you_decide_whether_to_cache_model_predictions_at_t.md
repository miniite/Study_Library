---
type: "practice"
difficulty: "Deep"
---

# How would you decide whether to cache model predictions at the edge or gateway, given that some inputs are highly repeated and others are unique per user?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Separating Compute and Storage](<../../../../06_Cloud_Computing/Cloud_Foundations/Cloud_Fundamentals/Separating_Compute_and_Storage.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To decide whether to cache model predictions at the edge or gateway, we need to balance between reducing latency, minimizing bandwidth usage, and ensuring data freshness. The key factors include prediction reuse patterns, data size, network conditions, and computational resources. Let's break this down step by step.

#### Triage

The question is essentially asking us to evaluate trade-offs between caching at the edge (e.g., a mobile app or IoT device) versus the gateway (e.g., a server closer to the user). We need to consider factors like:

Prediction Reuse: How often are predictions repeated for different users?
Data Size and Latency Sensitivity: Are predictions large, and do they require low latency?
Network Conditions: What is the typical network quality between the edge device and the gateway?

#### First Principles Solution

Identify Prediction Patterns:

If many inputs are highly repeated, caching at the edge can be beneficial.
For unique per-user predictions, the gateway might be more suitable.

Evaluate Data Size and Latency Sensitivity:

Larger model outputs may require less frequent caching to avoid data bloat on edge devices.
Low-latency applications (e.g., real-time decision-making) favor minimal latency overhead, so edge caching is advantageous.

Network Conditions:

Poor network conditions can make gateway caching more reliable and faster.
High-bandwidth networks might allow for more frequent updates at the edge.

Compute Resources:

Edge devices typically have limited compute power compared to gateways.
If computing resources are scarce, offloading computation to a gateway can be necessary.
Step-by-Step Solution

Assess Reuse Patterns:

Collect usage logs and analyze input patterns. If more than 50% of inputs are repeated across users, caching at the edge could significantly reduce requests to the model server.

Model Output Size and Latency Requirements:

Determine the size of predictions (e.g., a simple classification might be small while complex regression models can produce larger outputs).
Evaluate latency requirements for each use case.

Network Analysis:

Measure network latency between edge devices and gateways.

#### Cons
ider the reliability and consistency of network connections.

Compute Resource Assessment:

Assess available compute resources at both edges and gateways.
Ensure that gateway can handle increased load if it's taking on more computation.

Implement a Hybrid Strategy:

Use an edge cache for highly repeated predictions to reduce latency.
Implement a global cache or database at the gateway for unique per-user predictions to ensure data freshness and consistency.

#### Closing Summary

In summary, caching model predictions can be effectively leveraged by considering prediction patterns, data size, network conditions, and compute resources. A hybrid approach combining edge and gateway caches can optimize performance while ensuring data freshness and minimizing resource usage.

#### Common Mistakes
and Follow-Ups
Overlooking Network Conditions: Failing to consider the reliability of local networks might lead to inconsistent cache performance.
Ignoring Edge Device Resources: Underestimating the compute capabilities of edge devices may result in suboptimal use cases where edge caching could have been beneficial.
Neglecting Data Freshness: Over-caching unique per-user predictions can lead to outdated data, compromising the accuracy and relevance of results.

**Comparison:**

```
Prediction Reuse Rate	Decision
>50%	Edge Cache
<30%	Gateway Cache
30-50%	Hybrid Strategy
```

**Code:**

```python
# Example pseudo-code for assessing prediction reuse patterns
def assess_reuse_patterns(user_inputs):
    # Analyze input patterns using statistical methods or clustering algorithms
    repeated_count = 0
    total_count = len(user_inputs)

    for i in range(len(user_inputs) - 1):
        for j in range(i + 1, len(user_inputs)):
            if are_similar(user_inputs[i], user_inputs[j]):
                repeated_count += 1

    reuse_rate = (repeated_count / (total_count * (total_count - 1))) * 100
    return reuse_rate

# Function to check similarity between inputs
def are_similar(input1, input2):
    # Implement a simple or complex comparison logic based on the nature of inputs
    pass
```

**Example input / output:**

```text
# Example dry-run of assessing reuse patterns
User inputs: [[1,2,3], [4,5,6], [1,2,3]]
Reuse rate = (7 + 4) / ((3 * 2) - 1) * 100 = 86.67%
```

**Data:**

```json
{
  "network_latencies": {
    "edge_to_gateway": 50ms,
    "gateway_to_model_server": 300ms
  },
  "user_inputs": [
    {"input": [1,2,3], "repeat_count": 7},
    {"input": [4,5,6], "repeat_count": 3},
    {"input": [1,2,3], "repeat_count": 4}
  ]
}
```

**Flow:**

1. Yes
2. No
3. Yes
4. No
5. Start
6. Are inputs highly repeated?
7. Cache at Edge
8. Is network reliable?
9. Cache at Gateway
10. Hybrid Strategy: Cache at Edgeand Gateway

