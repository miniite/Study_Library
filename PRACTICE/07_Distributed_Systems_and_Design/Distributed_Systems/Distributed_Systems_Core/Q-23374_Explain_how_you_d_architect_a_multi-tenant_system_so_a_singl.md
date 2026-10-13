---
type: "practice"
difficulty: "Expert"
---

# Explain how you'd architect a multi-tenant system so a single abusive or buggy tenant cannot exhaust a shared resource (connections, threads, DB rows, hot partitions) and take down neighbors. Cover quota enforcement, hot-partition splitting, and the tension between isolation cost and tenant density.

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Multi-tenant vs Dedicated Clusters](<../../../../07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Mixed_Version_Fleets/Multi-tenant_vs_Dedicated_Clusters.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To architect a multi-tenant system where individual tenants cannot exhaust shared resources like connections or DB rows, we need to focus on enforcing quotas, managing hot partitions, and balancing the trade-offs between isolation cost and tenant density. The key is to ensure that no single tenant can monopolize resources in such a way that it degrades the experience for other tenants.

#### Triage
Understanding the Question

The core of this question is about designing a system that ensures resource fairness and resilience among multiple tenants, especially when dealing with potential abusive or buggy behavior. The interviewer might be probing your understanding of resource management, concurrency control, and distributed systems principles. We'll assume the system handles web services with varying degrees of tenant density and that hot partitions can occur due to uneven data distribution.

#### Solution
From First Principles
Quota Enforcement: Implementing strict quotas for each tenant on shared resources is crucial. This ensures that no single tenant can overuse resources, leading to resource starvation or degradation of service for others.
Resource Allocation Strategy: Use a combination of static and dynamic allocation strategies based on historical usage patterns and current load conditions.
Hot Partition Handling: Detect and split hot partitions in real-time to prevent concentration of requests that could overwhelm shared resources.
Isolation vs. Tenant Density Trade-offs: Balance the cost of isolation mechanisms with the need for high tenant density, ensuring both scalability and fair resource distribution.
Detailed Solution

Quota Enforcement:

Define per-tenant quotas based on observed usage patterns and expected maximums.
Use rate limiting to enforce these quotas at the API gateway level (e.g., NGINX or Traefik).
Implement monitoring and alerting for quota breaches, enabling quick corrective actions.

Resource Allocation Strategy:

Static Allocation: Assign a fixed number of resources per tenant based on their historical usage.
Dynamic Allocation: Adjust resource allocations in real-time based on current load conditions using techniques like auto-scaling or adaptive capacity planning.

Hot Partition Handling:

Implement intelligent sharding and data replication strategies to distribute load evenly across multiple nodes.
Use advanced caching mechanisms (e.g., Redis) to reduce the burden on backend databases.
Employ real-time monitoring tools to detect hot partitions and trigger automatic partition splitting or rebalancing.

Isolation vs. Tenant Density Trade-offs:

Evaluate different isolation strategies such as virtual machines, containers, or function-as-a-service (FaaS).

#### Cons
ider the overhead of these isolation mechanisms versus the benefits they provide in terms of tenant isolation.
Implement a strategy that dynamically adjusts isolation boundaries based on resource utilization metrics.

#### Closing Summary
and Common Mistakes

To summarize, we need to implement robust quota enforcement, dynamic resource allocation, intelligent partition handling, and balanced isolation strategies. Common mistakes include over- or under-provisioning resources, neglecting real-time monitoring and alerting, and failing to account for the trade-offs between isolation cost and tenant density.

**Comparison:**

```
Tenant ID	Current Usage	Max Quota
A	50	100
B	40	80
```

**Code:**

```python
# Example of quota enforcement in a rate limiter
from typing import Dict

class RateLimiter:
    def __init__(self, max_requests_per_minute: int):
        self.max_requests = max_requests_per_minute
        self.requests_in_window = 0
        self.reset_time = time.time()

    def allow_request(self) -> bool:
        if time.time() - self.reset_time >= 60:
            # Reset the window after one minute
            self.requests_in_window = 0
            self.reset_time = time.time()

        if self.requests_in_window < self.max_requests:
            self.requests_in_window += 1
            return True
        else:
            return False
```

**Example input / output:**

```text
Input:
- Tenant A sends 10 requests in the first minute.
- Tenant B sends 5 requests in the same minute.

Output:
- Allow request from Tenant A (current usage = 40).
- Allow request from Tenant B (current usage = 30).

Alert: Quota breach detected for Tenant A, investigate and adjust quotas.
```

**Data:**

```json
{
  "tenantA": {
    "quota_limit": 50,
    "current_usage": 30
  },
  "tenantB": {
    "quota_limit": 40,
    "current_usage": 25
  }
}
```

**Flow:**

1. Allow
2. Deny
3. Receive Request
4. Rate Limiter
5. Allow/Deny
6. Log and Serve Request
7. Alert Admin

