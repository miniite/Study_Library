---
type: "practice"
difficulty: "Deep"
---

# How would you cap the memory a single tenant can consume in a shared multi-tenant cache and evict fairly so a noisy tenant cannot starve others?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Tail Latency with Caching](<../../../../07_Distributed_Systems_and_Design/System_Design/Caching/Tail_Latency_with_Caching.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the challenge of capping memory consumption for a single tenant in a shared multi-tenant cache while ensuring fair eviction, we need to design a mechanism that limits the amount of memory each tenant can consume and ensures that evictions are handled fairly. This involves setting up a quota system and implementing an eviction strategy that respects these quotas.

#### Triage

The question is essentially about managing resource allocation in a multi-tenant environment where you want to ensure fair usage while preventing any single tenant from monopolizing the cache space, leading to starvation of other tenants. We need to consider several factors: memory capacity, quota enforcement, and eviction fairness.

#### First Principles Solution
Define Quotas: Each tenant should have a defined memory quota.
Monitor Usage: Track each tenant's current memory usage.
Enforce Quotas: Ensure that no tenant exceeds their allocated quota.
Fair Eviction Strategy: Implement an eviction strategy to handle overflow, ensuring all tenants get a fair chance.
Step-by-Step Solution
1. Define Quotas

Each tenant should be assigned a specific memory limit based on the cache's total available space and the number of tenants. For example, if you have 1 GB of cache space for 100 tenants, each tenant could be allocated 10 MB.

Define Tenant Quotas
Memory Limit per Tenant
2. Monitor Usage

Implement a system to track the current memory usage for each tenant. This can be done using key-value pairs where the keys are tenant IDs and the values are their respective usage.

Monitor Usage
Hashmap: Tenant ID ->Memory Usage
3. Enforce Quotas

Create a mechanism to check if any tenant exceeds their allocated quota. If a tenant's memory usage surpasses their limit, they should not be allowed to store new data.

Monitor Usage
Check Memory Limit Exceeded
4. Fair Eviction Strategy

When the cache exceeds its total capacity, we need to evict some data in a fair manner. There are several strategies you can use:

LRU (Least Recently Used): Evict the least recently used items first.
LFU (Least Frequently Used): Evict the least frequently accessed items first.
Random: Evict items randomly, ensuring that no single tenant is targeted disproportionately.

To implement fair eviction, you can use a combination of these strategies and ensure that each eviction event respects all tenants' quotas. For example, you could prioritize evictions from tenants who have exceeded their quota but also balance it with others to avoid starvation.

Deny New Data
Fair Eviction Strategy
Log and Warn
Log Exceeded Quota
Evict Least Used/QuotaExceeded Items First

#### Closing Summary

To cap the memory a single tenant can consume in a shared multi-tenant cache, we need to implement quotas for each tenant, monitor their usage, enforce these quotas, and use a fair eviction strategy when overflow occurs. This ensures that no single tenant monopolizes resources, leading to fair and efficient resource management.

#### Common Mistakes/Follow-Ups
Neglecting Fairness: Failing to ensure that evictions are handled fairly can lead to some tenants being starved.
Not Enforcing Quotas: Without strict enforcement of quotas, noisy or malicious tenants could abuse the system.
Ignoring Edge Cases: Not considering scenarios like sudden spikes in usage or changes in tenant behavior.

To further explore this solution, we could discuss implementation details such as:

How to dynamically adjust quotas based on cache load and tenant activity.
Using machine learning models to predict memory usage patterns.
Integrating with other systems (e.g., billing) for more granular control over resource allocation.

**Comparison:**

```
FLOWCHART · MERMAID
gantt
    title Tenant Quota Enforcement Example
    dateFormat XYYYY-MM-DD
    section Setup
    Init: 2023-10-01
    Define quotas: done, 1d
    Monitor usage: open, 5d

    section Execution
    Check memory limit exceeded: pending, 3d
    Deny new data: open, 2d
    Log and warn: open, 1d
```

**Code:**

```python
# Example code snippet for quota enforcement
class CacheManager:
    def __init__(self, total_memory, num_tenants):
        self.total_memory = total_memory
        self.num_tenants = num_tenants
        self.quota_per_tenant = total_memory // num_tenants
        self.usage_map = {}

    def add_data(self, tenant_id, data_size):
        current_usage = self.usage_map.get(tenant_id, 0)
        new_usage = current_usage + data_size

        if new_usage > self.quota_per_tenant:
            print(f"Tenant {tenant_id} exceeded quota. New usage: {new_usage}, Quota: {self.quota_per_tenant}")
            return False
        else:
            self.usage_map[tenant_id] = new_usage
            return True

    def evict_data(self):
        # Implement fair eviction strategy here
        pass
```

**Example input / output:**

```text
| Tenant ID | Current Usage (B) | Quota Exceeded? |
|-----------|------------------|----------------|
| tenant_1  | 8192             | No             |
| tenant_2  | 7680             | No             |
| tenant_3  | 9500             | Yes            |

For `tenant_3`, we would deny the addition of new data and log this event.
```

**Data:**

```json
{
  "total_memory": 1024 * 1024, // 1 GB in bytes
  "num_tenants": 100,
  "quota_per_tenant": 10240,   // 10 MB per tenant
  "usage_map": {
    "tenant_1": 8192,
    "tenant_2": 7680
  }
}
```

**Flow:**

1. Exceeded
2. Within Limits
3. Add Data Request
4. Tenant ID
5. Check Quota Map
6. Deny New Data
7. Update Usage Map

