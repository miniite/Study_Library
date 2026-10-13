---
type: "practice"
difficulty: "Expert"
---

# You must design admission control for a multi-tenant platform where one tenant's concurrent burst must never starve others (noisy neighbor). Compare per-tenant bulkheads, weighted fair queuing, and concurrency limits. How do you set and dynamically tune the limits without constant manual intervention?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Bulkheads and Thread Pools](<../../../../07_Distributed_Systems_and_Design/Distributed_Systems/Concurrency_Patterns/Bulkheads_and_Thread_Pools.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR

In a multi-tenant platform, ensuring that one tenant's burst of concurrent requests does not starve others is crucial to maintain system stability and fair usage. This can be achieved through various concurrency control patterns such as per-tenant bulkheads, weighted fair queuing (WFQ), and concurrency limits. Each method has its own trade-offs in terms of fairness, performance, and complexity. To dynamically tune these limits without constant manual intervention, we need to implement monitoring and feedback mechanisms.

#### Triaging the Question

The core of this question revolves around designing admission control for a multi-tenant platform where tenant A's burst of concurrent requests should not starve tenants B, C, or D. This problem is common in cloud-native environments where shared resources must be managed efficiently across multiple users. The interviewer is probing your understanding of concurrency patterns and race conditions.

#### Assumptions
The system has a limited number of available threads or CPU cores.
Requests from each tenant are independent and can be prioritized based on some criteria.
We need to ensure that no single tenant can monopolize the resources, thus ensuring fair usage for all tenants.

#### First Principles Solution
1. Per-Tenant Bulkheads

Per-tenant bulkheads limit the maximum number of concurrent requests from each tenant. This ensures that even if a tenant has a burst of requests, it cannot overwhelm the system. However, this approach does not provide fairness across tenants since a single tenant can still starve others.

Tenant 1
Bulkhead
Concurrent Requests
Other Tenants
Bulkhead
Concurrent Requests

#### Pros

Simple to implement.
Guarantees that no single tenant can starve the system.

#### Cons

No fairness mechanism across tenants.
Can still lead to uneven resource distribution.
2. Weighted Fair Queuing (WFQ)

Weighted fair queuing ensures that all tenants get a fair share of resources, but the allocation is not based on absolute limits. Instead, it allocates resources proportionally according to predefined weights or priorities.

Tenant 1
Queue
Weighted Resource Allocation
Other Tenants
Queue
Weighted Resource Allocation

#### Pros

Ensures fair usage across tenants.
Can be adjusted dynamically based on tenant needs.

#### Cons

Complexity in implementation and tuning.
May not handle bursts well since it is more about long-term fairness.
3. Concurrency Limits

Concurrency limits allow setting a global limit for all tenants combined, which can be dynamically adjusted based on observed usage patterns. This approach strikes a balance between fairness and resource management.

Global Concurrency Limit
Tenant Requests
Monitor Usage
Adjust Limits

#### Pros

Flexibility in setting global limits.
Can be dynamically adjusted based on real-time monitoring.
Ensures that bursts from one tenant do not starve others.

#### Cons

Requires sophisticated monitoring and feedback mechanisms.
Potential for over-provisioning or under-provisioning resources.
Dynamic Tuning Without Constant Manual Intervention

To set and dynamically tune the limits, we need to implement a robust monitoring and feedback loop. This involves:

Real-Time Monitoring: Continuously monitor the system's resource usage (CPU, memory, network) for each tenant.
Feedback Mechanism: Use historical data and current metrics to adjust concurrency limits in real-time.
Machine Learning Models: Employ machine learning algorithms to predict and optimize resource allocation based on past patterns.

#### Example
Workflow
Monitoring: Collect usage data from each tenant every 5 minutes.
Analysis: Analyze the data to identify trends, bursts, and long-term usage patterns.
Adjustment: Adjust concurrency limits for tenants with high burst usage while ensuring fairness.

#### Closing Summary
+ Common Mistakes/Follow-Ups

#### Common Mistakes

Overly complex solutions without considering simplicity.
Ignoring real-time monitoring in favor of static configurations.
Failing to account for edge cases such as extreme bursts or sudden traffic changes.

#### Follow-Ups

How would you implement the machine learning model?
What are some metrics that should be collected and analyzed?
How do you balance fairness with resource efficiency?

**Comparison:**

```
Tenant ID	Usage %	Concurrency Limit
TENANT_001	75	20
TENANT_002	90	15
```

**Code:**

```python
# Example of a simple monitoring script
import time

def monitor_tenant_usage(tenant_id, interval=5):
    while True:
        # Simulate fetching usage data
        usage = fetch_usage_data(tenant_id)
        print(f"Tenant {tenant_id} usage: {usage}")

        # Adjust concurrency limits based on usage
        adjust_concurrency_limits(usage)

        time.sleep(interval)

# Example of adjusting concurrency limits
def adjust_concurrency_limits(usage):
    if usage > 80:
        set_concurrency_limit(tenant_id, 10)
    else:
        set_concurrency_limit(tenant_id, 20)
```

**Example input / output:**

```text
**Input:**
- Tenant `TENANT_001` has a current usage percentage of 75%.

**Output:**
- The concurrency limit for `TENANT_001` is set to 20.

---
```

**Data:**

```json
{
  "tenant_id": "TENANT_001",
  "usage_percentage": 75,
  "concurrency_limit": 20
}
```

**Flow:**

1. Yes
2. No
3. Monitor Usage
4. Usage > Threshold?
5. Adjust Concurrency Limits
6. Continue Monitoring

