---
type: "practice"
difficulty: "Deep"
---

# Your model-serving fleet holds warm GPU replicas at low utilization to meet a p99 latency SLA on bursty traffic. Design the cost-versus-latency governance that decides how much idle accelerator you pay for, and defend the SLA tier where you'd deliberately accept cold starts to save money.

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ GPU Capacity Economics](<../../../../06_Cloud_Computing/Cloud_Cost/FinOps/Serverless_Cost_Visibility/GPU_Capacity_Economics.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To design a cost-versus-latency governance strategy for maintaining warm GPU replicas in a model-serving fleet, we need to balance the trade-offs between cost and latency. The key is to ensure that we maintain sufficient resources to meet our p99 latency SLA while minimizing unnecessary costs during periods of low traffic.

#### Triaging the Question

The question primarily probes your understanding of:

Cost Management: How to efficiently manage cloud resources to minimize expenses.
Performance Optimization: Ensuring a consistent user experience despite varying traffic patterns.
Decision-Making Models: Establishing rules or algorithms that govern resource allocation and deallocation based on usage patterns.

#### Assumptions
The fleet operates in a cloud environment with dynamic scaling capabilities.
We aim to maintain p99 latency at acceptable levels without frequent cold starts, which are expensive due to the time taken for initial boot-up.
Usage follows a bursty pattern, meaning there are periods of high and low traffic.
Step-by-Step Solution
Define Latency SLA: The primary goal is to ensure p99 latency remains within acceptable bounds (e.g., 50ms).
Monitor Traffic Patterns: Use historical data and real-time monitoring to understand the bursty nature of the traffic.
Cost Model: Develop a cost model that factors in both warm replicas and cold starts, weighted by their respective costs.
Resource Allocation Strategy:
Warm Replicas: Keep a number of GPUs always active to ensure low latency during bursts.
Cold Starts: Accept occasional cold starts if the cost savings outweigh the temporary impact on p99 latency.
Cost Model

Let's define some variables:

( C_w ): Cost per hour for warm GPU replicas ($1/hour).
( C_c ): Cost per hour for a single cold start (assume $50 for an average cold start).

The cost model can be defined as follows:

[ \text{Total Cost} = n \times C_w + m \times C_c ]

Where:

( n ) is the number of warm GPU replicas.
( m ) is the number of cold starts.

To optimize costs, we need to balance ( n ) and ( m ). We can use a threshold-based approach where if traffic exceeds a certain level (e.g., 90% utilization), additional replicas are spun up; otherwise, idle replicas may be shut down or kept in a warm state for quick reactivation.

#### Example Scenario

Assume we have the following data:

Warm replica cost: $1/hour.
Cold start cost: $50.
Traffic levels over time (in % of capacity): 20%, 40%, 60%, 80%, 95%.

For each traffic level, decide whether to scale up or down based on the cost model:

At 20% utilization, keep ( n = 1 ) warm replica.
At 40% utilization, consider maintaining ( n = 2 ) warm replicas.
At 60%, 80%, and 95% utilization, scale up to more warm replicas or accept cold starts.

#### Implementation

We can implement this logic using a simple script that monitors traffic and adjusts the number of warm replicas accordingly. Here is a simplified Python code snippet:

PYTHON
def adjust_replicas(traffic_utilization, cost_per_hour_warm, cost_per_cold_start):
    if traffic_utilization < 40:
        return 1  # Maintain one warm replica
    elif traffic_utilization >= 60 and traffic_utilization < 95:
        return 2  # Maintain two warm replicas
    else:
        # Accept cold starts for higher utilizations
        warm_replicas = min(traffic_utilization // 30, 3)
        cold_starts = (traffic_utilization - 120) / 40 if traffic_utilization > 95 else 0
        return warm_replicas + int(cold_starts)

# Example usage
utilizations = [20, 40, 60, 80, 95]
for util in utilizations:
    print(f"Utilization: {util}%, Replicas Needed: {adjust_replicas(util, 1, 50)}")

#### Closing Summary
and Common Mistakes

This approach ensures that we maintain a balance between cost and latency. By accepting cold starts during high traffic periods, we can significantly reduce costs while still meeting our SLA requirements.

#### Common mistakes
include:

Over-provisioning resources to avoid any chance of cold starts.
Ignoring the long-term cost implications of maintaining warm replicas unnecessarily.
Not considering edge cases like very low or extremely high traffic patterns.

#### Follow-Up
How would you handle scenarios with varying types of workloads?
Can you integrate machine learning models to predict future traffic and optimize resource allocation further?

**Code:**

```python
def adjust_replicas(traffic_utilization, cost_per_hour_warm, cost_per_cold_start):
    if traffic_utilization < 40:
        return 1
    elif traffic_utilization >= 60 and traffic_utilization < 95:
        return 2
    else:
        warm_replicas = min(traffic_utilization // 30, 3)
        cold_starts = (traffic_utilization - 120) / 40 if traffic_utilization > 95 else 0
        return warm_replicas + int(cold_starts)

# Example usage
utilizations = [20, 40, 60, 80, 95]
for util in utilizations:
    print(f"Utilization: {util}%, Replicas Needed: {adjust_replicas(util, 1, 50)}")
```

**Example input / output:**

```text
Utilization: 20%, Replicas Needed: 1
Utilization: 40%, Replicas Needed: 1
Utilization: 60%, Replicas Needed: 2
Utilization: 80%, Replicas Needed: 2
Utilization: 95%, Replicas Needed: 3
```

