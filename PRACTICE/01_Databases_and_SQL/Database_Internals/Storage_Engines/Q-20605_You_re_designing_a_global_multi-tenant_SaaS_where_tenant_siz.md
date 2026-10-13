---
type: "practice"
difficulty: "Expert"
---

# You're designing a global multi-tenant SaaS where tenant sizes span 6 orders of magnitude. Decide how storage-engine choice and per-tenant isolation (one big LSM vs DB-per-tenant vs column families) should vary by tenant tier, and how you'd avoid noisy-neighbor compaction across tiers.

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ B-tree vs LSM Storage Engines](<../../../../01_Databases_and_SQL/Database_Internals/Storage_Engines/Tiered_Storage_in_LSM_Engines/B-tree_vs_LSM_Storage_Engines.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When designing a global multi-tenant SaaS that supports tenants of vastly different sizes (spanning 6 orders of magnitude), the choice between using one big LSM, DB-per-tenant, or column families needs to be carefully considered based on tenant tier. The primary goal is to ensure efficient storage and performance while maintaining per-tenant isolation. To address the issue of noisy-neighbor compaction across tiers, we need a scalable solution that can handle varying workloads without impacting others.

#### Triage

The question asks us to determine how different database engines (LSM, DB-per-tenant, column families) should be used based on tenant tier size and how to avoid noisy-neighbor effects. We will consider the following assumptions:

Each tenant has a unique set of data needs.
Performance and storage efficiency are critical factors.
Tenant isolation is paramount.

#### First Principles Solution
Tenant Tier Considerations

Small Tenants (10s or 100s of MB):

LSM Engine: A single LSM could handle small tenants efficiently due to its logarithmic performance and compact storage. However, as the number of tenants grows, the compaction overhead might become significant.

Medium Tenants (10s GB):

Column Families: For medium-sized tenants, column families can provide a good balance between flexibility and efficiency. They offer better query performance and are more manageable than large LSMs.

Large Tenants (10s of TB or more):

DB-per-Tenant: Large tenants might benefit from having their own dedicated database systems for isolation and performance reasons. This approach ensures that the tenant’s data is isolated, and compaction and other maintenance operations do not affect others.
Avoiding Noisy-Neighbor Compaction

To avoid noisy-neighbor effects across tiers, we can implement a tiered storage strategy:

Tier-Based Compaction Policies: Use different compaction policies based on tenant size to balance performance and resource utilization.
Horizontal Scaling: Distribute tenants across multiple nodes to ensure that the workload is spread evenly.
Partitioning Strategies: Implement partitioning strategies like range or hash sharding to distribute data logically, reducing contention.

#### Example
Compaction Policies

For small tenants:

Small Tenants
Compaction Policy
Logarithmic

For medium tenants:

Medium Tenants
Compaction Policy
Hybrid

For large tenants:

Large Tenants
Compaction Policy
Independent

#### Closing Summary
+ Common Mistakes/Follow-Ups

In summary, the choice of storage engine and per-tenant isolation strategy should be tier-based. Small tenants can use LSM engines with logarithmic compaction policies, medium tenants can benefit from column families, and large tenants should have their own dedicated databases for isolation.

#### Common mistakes
include:

Ignoring Tenant Isolation: Not ensuring that data from different tenants is properly isolated can lead to noisy-neighbor effects.
Uniform Compaction Policies: Using the same compaction policies across all tenant tiers without considering tier-specific needs.
Inadequate Scalability Planning: Failing to plan for horizontal scaling and partitioning strategies can limit performance as the number of tenants grows.

#### Follow-ups
could include:

How data access patterns affect the choice of storage engine?
What are the implications of using a single database vs multiple databases in terms of maintenance and backup?
How do transactional requirements influence the decision?

**Comparison:**

```
Tenant Tier	Storage Engine	Compaction Policy	Isolation Strategy
Small	LSM	Logarithmic	No dedicated DB
Medium	Column Families	Hybrid	Shared but isolated within
Large	DB-per-Tenant	Independent	Separate database instance
```

**Code:**

```python
# Example Python code to simulate compaction policies
def compact_data(tenant_size, policy):
    if tenant_size <= 1024:  # Small Tenant
        return "logarithmic"
    elif 1024 < tenant_size <= 1048576:  # Medium Tenant
        return "hybrid"
    else:  # Large Tenant
        return "independent"

# Example usage
tenant_sizes = [128, 512, 1024, 2048]
for size in tenant_sizes:
    print(f"Tenant Size {size}: Compaction Policy -> {compact_data(size, None)}")
```

**Example input / output:**

| Tenant Size (MB) | Compaction Policy |
|------------------|-------------------|
| 128              | Logarithmic       |
| 512              | Hybrid            |
| 1024             | Independent       |
| 2048             | Independent       |

**Data:**

```json
{
  "tenant_sizes": [128, 512, 1024, 2048],
  "compaction_policies": ["logarithmic", "hybrid", "independent"]
}
```

**Flow:**

1. Small Tenants
2. Compaction Policy
3. Logarithmic
4. Medium Tenants
5. Compaction Policy
6. Hybrid
7. Large Tenants
8. Compaction Policy
9. Independent

