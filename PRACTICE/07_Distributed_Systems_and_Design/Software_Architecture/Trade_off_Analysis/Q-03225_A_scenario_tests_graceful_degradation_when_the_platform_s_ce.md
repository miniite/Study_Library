---
type: "practice"
difficulty: "Deep"
---

# A scenario tests graceful degradation when the platform's central logging/metrics pipeline is saturated during an incident. Walk through analyzing whether the platform degrades gracefully (drops sampling) or fails closed (blocks workloads), and what sensitivity points that exposes.

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Sensitivity Points](<../../../../07_Distributed_Systems_and_Design/Software_Architecture/Trade_off_Analysis/Sensitivity_Points.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When a platform's central logging/metrics pipeline becomes saturated, the decision to degrade gracefully by dropping samples or fail closed by blocking incoming workloads involves critical trade-offs in terms of system resilience, performance, and observability. This scenario highlights the importance of understanding the underlying architecture, its constraints, and the implications of each approach.

#### Triaging the Question

The core of this question is about evaluating the architectural trade-offs between graceful degradation (sampling) and fail closed policies. The key assumptions include:

Pipeline Saturation: The logging/metrics pipeline is overwhelmed with too many data points.
Impact on Critical Services: Whether the failure in the pipeline affects critical services that depend on real-time metrics or logs for decision-making.
System Sensitivity: How sensitive the system is to latency and throughput in the logging/metrics pipeline.
First-Principles Analysis
Graceful Degradation (Sampling)

Graceful degradation involves dropping samples from incoming data before it reaches the pipeline. This approach aims to maintain system stability by reducing the load on the saturated pipeline:

Reducing Data Volume: Dropping a portion of the logs/metrics can help prevent overwhelming downstream services.
Maintaining Observability: Partial data loss is better than complete failure, allowing for some level of monitoring and debugging.

However, sampling introduces several trade-offs:

Data Loss: There's a risk that important events might be missed or delayed.
Latency Increase: Sampling can introduce additional latency as the system decides which data to drop.
Fail Closed (Blocking)

Fail closed involves blocking new workloads until the logging/metrics pipeline is back within acceptable limits. This approach ensures no loss of critical services but comes with its own set of challenges:

No Data Loss: Critical services remain operational, ensuring that all data collection processes are halted only as a last resort.
Potential for Cascading Failures: Blocking new workloads can cascade into other parts of the system, leading to reduced overall throughput.

However, fail closed also has significant drawbacks:

Service Degradation: Services dependent on real-time metrics or logs might experience performance degradation due to blocked requests.
Increased Complexity: Managing complex states and retries in a blocking scenario adds complexity to the architecture.
Detailed Trade-off Analysis

Latency and Throughput:

Graceful Degradation: Introduces minimal latency but can lead to increased throughput by reducing pipeline load.
Fail Closed: Blocks new workloads entirely, leading to potential service degradation.

Data Integrity vs. Service Availability:

Graceful Degradation: Sacrifices some data integrity for system stability.
Fail Closed: Ensures all data is collected but may sacrifice service availability during the blocking period.

Resource Utilization:

Graceful Degradation: Efficient use of resources by processing less data, potentially allowing critical services to continue operating.
Fail Closed: More resource-intensive as it involves managing blocked workloads and retry mechanisms.

Operational Complexity:

Graceful Degradation: Requires fine-tuning sampling rates and monitoring mechanisms.
Fail Closed: More complex to manage blocking states and retries, but simpler in terms of data flow control.

#### Closing Summary

In summary, the decision between graceful degradation (sampling) and fail closed policies depends on the criticality of real-time metrics and logs for your system. Graceful degradation offers a more lightweight solution that minimizes data loss while maintaining overall service availability. However, it requires careful tuning to balance load effectively without missing important events. In contrast, fail closed ensures no loss of data but risks cascading failures in downstream services. Both approaches have their trade-offs and should be evaluated based on the specific requirements and constraints of your platform.

#### Common Mistakes
and Follow-Ups
Overlooking Edge Cases: Forgetting to consider scenarios where critical events are missed due to sampling.
Ignoring Latency Impacts: Failing to account for the additional latency introduced by sampling mechanisms.
Failure Mode Analysis: Not thoroughly assessing how a fail closed state might affect system resilience and recovery strategies.
Triaging Follow-Ups
How would you implement such a graceful degradation mechanism?
What are the potential failure modes of your chosen approach, and how would you handle them?
Are there any specific metrics or thresholds that you would use to trigger fail closed vs. graceful degradation?

**Comparison:**

```
Approach	Data Loss	Latency Impact	Service Degradation
Sampling	Low	High	Low
Fail Closed	None	High	High
```

**Code:**

```python
# Example code for implementing a simple sampling mechanism

import random

def sample_logs(logs, threshold=0.1):
    sampled_logs = []
    for log in logs:
        if random.random() < threshold:
            sampled_logs.append(log)
    return sampled_logs
```

**Example input / output:**

```text
Input:
logs = [
  {"timestamp": "2023-10-01T12:00:00Z", "message": "User login success"},
  {"timestamp": "2023-10-01T12:01:00Z", "message": "API request failure"}
]

Output:
sampled_logs = [
  {"timestamp": "2023-10-01T12:00:00Z", "message": "User login success"}  # Sampled based on threshold
]
```

**Data:**

```json
{
  "logs": [
    {"timestamp": "2023-10-01T12:00:00Z", "message": "User login success"},
    {"timestamp": "2023-10-01T12:01:00Z", "message": "API request failure"},
    // ...
  ]
}
```

**Flow:**

1. Yes
2. No
3. Log Received
4. Sampling Decision
5. Sampled?
6. Store Sampled Log
7. Drop Log
8. Send to Pipeline
9. Block Workload
10. Fall Back to Fail Closed

