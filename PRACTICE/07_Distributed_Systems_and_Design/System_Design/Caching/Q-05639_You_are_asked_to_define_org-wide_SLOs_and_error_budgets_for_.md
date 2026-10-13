---
type: "practice"
difficulty: "Expert"
---

# You are asked to define org-wide SLOs and error budgets for a shared cache platform. What SLIs do you commit to, how do you keep tenants from consuming each other's error budget, and how do you handle a tenant that repeatedly breaches?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Caching and Service Independence](<../../../../07_Distributed_Systems_and_Design/System_Design/Caching/Cache_Invalidation/Caching_and_Service_Independence.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR

For an org-wide shared cache platform, we need to define Service Level Indicators (SLIs) based on critical metrics like hit rate, cache-to-db ratio, and cache-eviction frequency. We'll implement tenant isolation strategies such as namespace partitioning and quota management to prevent tenants from consuming each other's error budgets. Handling a tenant that repeatedly breaches involves monitoring, alerting, and leveraging adaptive policies for error budget recovery.

#### Triage

The question asks us to define Service Level Objectives (SLOs) and error budgets for a shared cache platform used across multiple organizational units or tenants. The key aspects include committing to specific SLIs, ensuring tenant isolation to avoid mutual resource consumption, and strategies for dealing with repeated breaches.

#### Assumptions

The cache is used by multiple independent tenants.
We are using a managed service that supports error budgets.
SLOs are defined based on business priorities and operational constraints.

#### First-Principles Solution
Service Level Indicators (SLIs)
Hit Rate: Measure the percentage of requests served directly from cache versus those that bypassed it due to cache misses or evictions.
Cache-to-DB Ratio: Track the number of cache hits compared to database accesses to ensure efficient use of cache resources.
Eviction Frequency: Monitor how often items are evicted, indicating potential issues with cache size or hit patterns.
Tenant Isolation

To prevent tenants from consuming each other's error budgets:

Namespace Partitioning: Each tenant gets a unique namespace for their data and metadata in the cache. This ensures that even if one tenant experiences high rates of errors or cache evictions, it won't affect others.
Quota Management: Implement per-tenant quotas on cache usage metrics like maximum cache size, eviction threshold, and error rate. Use tiered quotas to allow some flexibility while maintaining control.
Error Budget Management
Error Budget Allocation: Allocate a certain percentage of the overall SLO as an error budget for each tenant.
Alerting Mechanisms: Set up alerting thresholds that notify operations teams if a tenant's error rate exceeds its allocated budget.
Adaptive Policies: Implement adaptive policies to recover from breaches, such as temporarily reducing write operations or scaling out read capacity.
Handling Repeated Breaches
Monitoring and Alerting: Continuously monitor tenant performance metrics and trigger alerts for significant deviations from the SLIs.
Root Cause Analysis: Conduct thorough root cause analysis when a breach is detected to understand underlying issues (e.g., inefficient cache policies, high traffic patterns).
Policy Adjustments: Dynamically adjust error budget allocations or apply more stringent quotas based on historical performance and future forecasts.

#### Closing Summary

By defining clear SLIs and implementing robust tenant isolation strategies, we ensure that a shared cache platform can support multiple tenants while maintaining service quality and resource management. Handling repeated breaches involves proactive monitoring, detailed analysis, and adaptive policy adjustments to maintain system stability and tenant satisfaction.

#### Common Mistakes/Follow-Ups
Overcommitting on SLIs: Focusing too narrowly on hit rate at the expense of other important metrics like cache-to-db ratio.
Lack of Tenant Isolation: Not implementing proper namespace partitioning or quota management, leading to resource contention and performance degradation.
Ignoring Adaptive Policies: Relying solely on static error budget allocations without flexibility for recovery mechanisms.

To address these common pitfalls, it’s essential to have a flexible and adaptive approach to SLOs and error budgets. Regularly revisiting and refining the policies based on real-world usage patterns can help maintain optimal system performance.

**Code:**

```python
# Example of configuring cache quotas in Python

def configure_cache_quotas(tenant_id, max_size_mb, eviction_threshold):
    tenant_config = {
        "tenant_id": tenant_id,
        "cache_size_limit": max_size_mb,
        "eviction_policy": "LRU",  # Least Recently Used
        "max_evictions_per_minute": eviction_threshold
    }
    return tenant_config

# Example usage
tenant_1_config = configure_cache_quotas("tenant-001", 512, 5)
tenant_2_config = configure_cache_quotas("tenant-002", 768, 10)
```

**Example input / output:**

| Tenant ID | Cache Size Limit (MB) | Max Evictions Per Minute |
|-----------|----------------------|-------------------------|
| tenant-001 | 512                  | 5                       |
| tenant-002 | 768                  | 10                      |

**Data:**

```json
{
    "tenants": [
        {
            "id": "tenant-001",
            "cache_size_limit_mb": 512,
            "max_evictions_per_minute": 5
        },
        {
            "id": "tenant-002",
            "cache_size_limit_mb": 768,
            "max_evictions_per_minute": 10
        }
    ]
}
```

