---
type: "practice"
difficulty: "Deep"
---

# A team wants to run analytics directly against the LSM primary using long-range iterators, and it keeps evicting the block cache and hurting OLTP latency. As principal, design the org-wide policy that prevents this pattern rather than fixing it per incident, and the enforcement mechanism.

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ B-tree vs LSM Storage Engines](<../../../../01_Databases_and_SQL/Database_Internals/Storage_Engines/Tiered_Storage_in_LSM_Engines/B-tree_vs_LSM_Storage_Engines.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the issue of long-range iterators evicting the block cache in an LSM database and impacting OLTP latency, we need to design a policy that enforces strict usage rules around analytics workloads while maintaining optimal performance for transactional operations. The solution involves understanding how LSM databases operate, particularly regarding their use of write-ahead logs (WAL) and block caching strategies.

#### Triage

The question is essentially asking us to implement an organizational policy and enforcement mechanism that will prevent the long-range iterators from overwriting the block cache during analytics workloads. This is a critical concern because OLTP operations rely heavily on cache hits for performance, while analytics queries can be more batch-oriented and less sensitive to latency. The key assumptions are:

The database environment supports both transactional (OLTP) and analytical workloads.
Long-range iterators are being used inappropriately by analytics teams, leading to cache evictions.

#### First Principles Solution
Isolate Analytics Workloads: Separate the analytics team from the OLTP team to ensure that the workloads do not interfere with each other. This can be achieved through resource allocation policies.
Enforce Time-Based Scheduling: Schedule analytics queries during off-peak hours when transactional operations are less likely to require cache hits.
Resource Quotas and Limits: Implement quotas on resources such as memory, CPU, and disk I/O for both OLTP and analytics workloads to prevent resource contention.
Monitoring and Alerting: Set up monitoring and alerting systems that trigger when cache eviction thresholds are exceeded by either workload.
Optimize Analytics Queries: Encourage the use of optimized query patterns that minimize the need for long-range iterators, or at least reduce their frequency.
Enforcement Mechanism

To enforce these policies, we will:

Implement a Multi-tenant Database Architecture: Use a multi-tenant database setup where each tenant (analytics team and OLTP team) has its own isolated namespace.
Use a Policy Engine: Develop a policy engine that can dynamically adjust resource allocation based on real-time workload metrics.
Integrate with Monitoring Tools: Integrate the policy engine with existing monitoring tools to detect and respond to cache eviction issues proactively.

#### Closing Summary

The proposed solution is to implement an organizational policy that enforces strict usage rules for both transactional and analytical workloads, ensuring that analytics queries do not interfere with OLTP operations. This involves isolating workloads, enforcing time-based scheduling, setting resource quotas, and using monitoring tools to enforce these policies.

#### Common Mistakes/Follow-Ups
Misunderstanding the Database Model: Not fully understanding how LSM databases operate can lead to ineffective policies.
Overlooking Long-Term Effects: Policies that only address short-term issues without considering long-term performance impacts may not be sustainable.
Insufficient Monitoring and Feedback Loops: Relying solely on post-failure resolution mechanisms can lead to a reactive rather than proactive approach.

**Comparison:**

```
Policy	Description
Isolate Tenants	Use separate namespaces for each tenant
Time-Based Scheduling	Schedule analytics queries during off-peak times
Resource Quotas	Limit memory, CPU, and I/O resources for both workloads
```

**Code:**

```python
# Example of policy enforcement in Python (simplified)
def enforce_workload_policy(usage_data):
    if usage_data['cache_eviction'] > threshold:
        trigger_alert('Cache eviction threshold exceeded')
        adjust_resource_allocation('OLTP', 'reduce')
        if is_analytics_time_window():
            limit_analytics_queries()
```

**Example input / output:**

```text
Input: OLTP transaction request (high cache hit ratio)
Output: Cache hit -> High response time
Input: Analytics query with long-range iterator (peak hours)
Output: Cache eviction -> Reduced cache hit ratio in OLTP transactions

Request for policy adjustment during analytics peak hours:
- Schedule: Off-peak hours
- Resource allocation: Reduce allocations to analytics workloads
```

**Data:**

```json
{
  "tenant_config": {
    "analytics": {
      "schedule": "off-peak",
      "resource_limits": {"memory": 1024, "cpu": 50},
      "query_patterns": ["optimized"]
    },
    "oltp": {
      "schedule": "always",
      "resource_limits": {"memory": 2048, "cpu": 75}
    }
  }
}
```

**Flow:**

1. Yes
2. No
3. Yes
4. No
5. OLTP Transaction
6. Cache Hit?
7. High Response Time
8. Analytics Query
9. Is Off-Peak?
10. Optimized Query
11. Evict Cache, Alert

