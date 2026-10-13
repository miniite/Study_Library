---
type: "practice"
difficulty: "Deep"
---

# A single tenant on your multi-tenant LLM platform sends a burst of very long-context requests that consume most of the KV-cache, starving other tenants. Explain how you'd detect this noisy neighbor and design fair sharing that accounts for token cost, not request count.

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Load Balancing Stateful Inference](<../../../../07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Partition_Lag_and_Hot_Keys/Load_Balancing_Stateful_Inference.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the issue of a single tenant consuming excessive resources on a multi-tenant LLM platform, we need to detect when one tenant is "noisy" and ensure fair sharing among tenants based on token cost rather than just request count. This involves monitoring resource usage, detecting anomalies, and implementing a dynamic allocation mechanism.

First, let's triage the question:

What the Question is Asking: The key here is designing a system to detect when one tenant is consuming too many resources, such as a KV-cache, and ensuring fair distribution of these resources among all tenants.
What the Interviewer is Probing: The interviewer is looking for your understanding of distributed systems, monitoring mechanisms, resource allocation strategies, and fairness in multi-tenant environments.

#### Assumptions
We are dealing with an LLM (Large Language Model) platform supporting multiple tenants.
Each tenant's requests can be charged based on token usage rather than just the number of requests.
The system has access to monitoring tools that track resource consumption and request patterns.

#### First-Principles Solution

Resource Monitoring:

Implement a robust telemetry system to monitor each tenant’s resource utilization, particularly focusing on key metrics like cache hits/misses, token usage, and response times.
Use time-series databases to store these metrics for historical analysis.

Anomaly Detection:

Utilize statistical methods or machine learning models to detect anomalies in resource consumption patterns. For example, a single tenant consuming an unusually high percentage of the cache might indicate noisy behavior.
Define thresholds and baseline behaviors for each tenant to identify when they are deviating from normal usage.

Fair Sharing Mechanism:

Design a dynamic allocation algorithm that adjusts the amount of resources allocated to each tenant based on their token consumption, not just request count.
Ensure that resource allocation is proportional to the actual cost (token usage) incurred by the tenant, ensuring fairness even when some tenants generate more traffic.

Dynamic Resource Management:

Integrate this fair sharing mechanism into the LLM’s infrastructure layer where resources like cache and tokens are managed dynamically.
Use load balancers or virtual machines that can scale up or down based on real-time resource demand from different tenants.

Feedback Loop:

Implement a feedback loop to continuously adjust the allocation strategy based on historical data and current resource consumption patterns.
Regularly review and fine-tune the thresholds and algorithms to improve fairness and efficiency.

#### Closing Summary

By implementing a combination of robust monitoring, anomaly detection, dynamic resource management, and fair sharing mechanisms, we can ensure that a single tenant does not disproportionately consume resources, thereby maintaining a fair distribution among all tenants. This approach leverages real-time telemetry data and machine learning to adapt dynamically to changing workloads.

#### Common Mistakes/Follow-Ups
Mistake: Focusing solely on request count without considering token usage.

#### Follow-Up
Discuss the trade-offs between simplicity (request count) vs. accuracy (token cost). Highlight how monitoring token usage provides a more nuanced and fair metric for resource allocation.

**Comparison:**

```
Tenant ID	Token Usage History	Baseline Threshold	Moving Average	Noisy Neighbor Detection Result
T001	[150, 200, 250]	30	200	False
```

**Code:**

```python
# Example pseudo-code for anomaly detection using moving average threshold
def detect_noisy_neighbor(usage_data, baseline_threshold):
    moving_average = calculate_moving_average(usage_data)
    if usage_data > (moving_average + 2 * baseline_threshold):
        return True
    return False

# Example pseudo-code for dynamic resource allocation based on token cost
def allocate_resources(tenant_id, token_usage_history):
    # Calculate proportional share of resources based on token usage history
    total_tokens_used = sum(token_usage_history)
    tenant_share = (tenant_id.token_usage / total_tokens_used) * 100
    return tenant_share
```

**Example input / output:**

```text
Input:
- Tenant T001 with token usage history [150, 200, 250]
- Baseline threshold of 30

Output:
- Moving average: 200 (calculated as the average of historical data)
- Detection result for noisy neighbor: False (since current usage is within thresholds)
- Resource allocation percentage: 75% (tenant T001's share based on token usage history)

Dry-run results suggest that tenant T001’s resource usage does not exceed the baseline threshold and thus no action is required.
```

**Data:**

```json
{
  "tenant_id": "T001",
  "token_usage_history": [150, 200, 250],
  "baseline_threshold": 30,
  "moving_average": 200
}
```

**Flow:**

1. Monitor Tenant ResourceUsage
2. Determine Baseline Thresholds
3. Calculate Moving Average
4. Detect Noisy Neighbor UsingAnomaly Detection
5. Adjust Resource AllocationBased on Token Cost
6. Re-evaluate and Fine-tuneStrategy

