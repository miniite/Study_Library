---
type: "practice"
difficulty: "Deep"
---

# How would you design a multi-tenant lakehouse so one noisy tenant cannot degrade queries for others?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Lakehouse Fundamentals](<../../../../02_Data_Engineering/Storage_and_Lakehouse/Lakehouse/Lakehouse_Fundamentals.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To design a multi-tenant lakehouse where no single noisy tenant degrades queries for others, we need to focus on data isolation, resource management, and efficient query execution strategies. At its core, this involves creating logical and physical partitions that ensure each tenant’s workload is isolated from others while still allowing for shared storage and processing resources.

#### Triaging the Question

The key aspects here are:

Isolation: Ensuring that one tenant's noisy queries do not impact another tenant.
Resource Management: Efficiently allocating compute and storage resources without bottlenecks.
Query Optimization: Ensuring that query performance is optimized for each tenant, even in a shared environment.

#### Assumptions
We are using a cloud-based data lakehouse platform (e.g., AWS Lake Formation, Databricks Delta Live Tables).
We have multiple tenants with varying levels of data needs and SLAs.
Each tenant has its own set of queries that can be noisy or benign.

#### First Principles Solution

Data Partitioning

Logical Partitions: Use logical partitions to group related datasets by tenant. For example, each tenant’s raw data can reside in separate directories or databases.
Physical Partitions: Implement physical partitioning within tables to ensure that queries only access relevant partitions. This reduces the I/O overhead and query execution time.

Resource Allocation

Use dynamic resource allocation policies based on tenant SLAs. For example, high-priority tenants can get more compute resources during peak times.
Leverage auto-scaling capabilities of cloud services to dynamically allocate resources as needed.

Query Optimization

Implement query caching mechanisms to store the results of frequently executed queries and serve them from cache instead of running them again.
Use query profiling tools to identify performance bottlenecks and optimize execution plans accordingly.
Employ partition pruning techniques to limit the amount of data scanned during query execution.

Monitoring and Alerts

Set up monitoring for resource usage, query performance, and tenant SLA violations.
Implement alerting mechanisms to notify administrators when a noisy query is detected or when resources are being overutilized by one tenant.

#### Closing Summary

By implementing logical and physical partitions, efficient resource management policies, robust query optimization strategies, and comprehensive monitoring, we can design a multi-tenant lakehouse that ensures each tenant’s workload remains isolated from others. This approach not only prevents noisy queries but also optimizes overall performance and resource utilization.

#### Common Mistakes/Follow-Ups
Mistake: Not properly isolating data or queries, leading to cross-tenant performance issues.

#### Follow-Up
Discussing how to implement role-based access controls (RBAC) and tenant-specific schemas to enforce strict isolation.
Mistake: Over-provisioning resources for a single tenant, which can lead to resource waste.

#### Follow-Up
Exploring strategies like preemptive resource allocation and using spot instances in the cloud.

**Comparison:**

```
Tenant	Raw Data Path	High Priority?
Tenant1	/data/raw/tenant_1	Yes
Tenant2	/data/raw/tenant_2	No
```

**Code:**

```python
# Example of query optimization with caching
def optimize_query(tenant_id):
    cache_key = f"tenant_{tenant_id}_query_cache"
    if cached_result := get_from_cache(cache_key):
        return cached_result
    else:
        result = execute_query(tenant_id)
        store_in_cache(cache_key, result)
        return result

# Example of resource allocation policy
def allocate_resources(tenant_id, priority_level):
    if is_high_priority(tenant_id):
        # Allocate more resources to high-priority tenants
        allocate_more_cpu_and_memory()
    else:
        # Allocate standard resources
        allocate_standard_resources()

# Example of monitoring and alerting
def monitor_resource_usage():
    resource_usage = collect_resource_metrics()
    if exceeds_threshold(resource_usage):
        send_alert("Resource usage exceeded threshold")

# Example of query execution with partition pruning
def execute_query(tenant_id, table_name):
    # Apply partition pruning logic here
    partitions_to_scan = get_partitions_for_tenant(tenant_id)
    results = run_sql_query(table_name, partitions_to_scan)
    return results
```

**Example input / output:**

```text
Input:
Tenant 1 submits a noisy query that scans large partitions.

Output:
The system identifies the noisy query and applies query optimization techniques to limit I/O, ensuring no degradation in performance for Tenant 2.
```

**Data:**

```json
{
  "tenant_data": {
    "tenant_1": {
      "raw_data_path": "/data/raw/tenant_1",
      "queries": ["query_a", "query_b"]
    },
    "tenant_2": {
      "raw_data_path": "/data/raw/tenant_2",
      "queries": ["query_c", "query_d"]
    }
  },
  "resource_allocation_policy": {
    "high_priority_tenant_ids": [1],
    "standard_resource_levels": 0.8
  }
}
```

**Flow:**

1. Yes
2. No
3. Query Execution
4. Is Tenant High Priority?
5. Allocate More Resources
6. Allocate Standard Resources
7. Execute Query with PartitionPruning
8. Cache Result

