---
type: "practice"
difficulty: "Expert"
---

# You're designing shuffle-sharding as the org's default multi-tenant isolation. Derive why shuffle sharding reduces the probability that any two tenants share a full failure domain, and how you'd size virtual shards for 50k tenants so a poison tenant can't blast more than a bounded fraction.

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Sharding Vector Indexes](<../../../../07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Distributed_Joins_in_MPP_Systems/Sharding_Vector_Indexes.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In distributed systems, shuffle sharding is a technique that enhances multi-tenant isolation by reducing the probability that any two tenants share a full failure domain. This technique involves dividing data and processing tasks into smaller, independent shards or partitions. The goal is to ensure that even if one shard fails, it does not impact other tenants significantly.

To understand why shuffle sharding works in this context, let's break down the problem and then derive a solution step-by-step.

#### Triage
Understanding the Question

The question requires us to design a system where 50k tenants can operate with minimal risk of full failure domains overlapping. We need to ensure that no single tenant can cause a catastrophic failure by overwhelming resources, while still maintaining efficient data processing and isolation.

#### Assumptions

Each tenant has an equal probability of being affected by a failure.
The system needs to be robust against failures from individual tenants.
We want to limit the impact of any one tenant’s resource consumption or failure to a manageable level.

#### First-Principles Solution

To achieve this, we need to:

Understand the Failure Domains: A full failure domain is typically defined by regions, data centers, or zones that could go down together.
Shard Design: Ensure that each tenant’s data and processing tasks are distributed across multiple failure domains.
Virtual Shard Sizing: Determine the optimal number of virtual shards for 50k tenants to limit the impact of any single tenant.
Step-by-Step Solution

Failure Domain Identification:

Assume there are ( n ) failure domains available in our system.
To reduce the risk, we want each tenant to be sharded across multiple failure domains.

Shard Distribution:

Each tenant should have its data and processing tasks distributed among at least ( m ) failure domains, where ( m > 1 ).
This ensures that even if one failure domain fails, only a fraction of tenants are impacted.

Virtual Shard Sizing:

We need to determine the number of virtual shards per tenant.
Each virtual shard will be assigned to a specific failure domain.

Impact Bounding:

Define a threshold ( T ) that limits the maximum resources (e.g., CPU, memory) any single tenant can consume in a given failure domain.
Ensure that no tenant can exceed this threshold to prevent overwhelming any one failure domain.
Detailed Solution

Given 50k tenants and assuming we have enough failure domains available:

Determine ( m ):

Let's assume each tenant is sharded across 3 failure domains (( m = 3 )). This means each tenant’s data and processing tasks are distributed among three different failure domains.

Shard Count:

For 50k tenants, we need to create ( 50k \times 3 = 150k ) virtual shards (since each tenant is sharded across 3 failure domains).

Impact Bounding:

Define the threshold ( T ). For example, if each virtual shard can consume up to 2% of a failure domain’s resources in any given period, we need to ensure that no single tenant exceeds this limit.
This means each tenant should not use more than ( 3 \times 2% = 6% ) of the total resources across all its virtual shards.

#### Closing Summary
and Common Mistakes

To summarize:

By sharding tenants across multiple failure domains, we reduce the risk that any single failure will impact a significant number of tenants.
Properly sizing virtual shards ensures that no tenant can overwhelm a single failure domain, maintaining system stability and resilience.

#### Common mistakes
to avoid:

Not distributing data and processing tasks widely enough among failure domains.
Failing to set appropriate resource thresholds for individual tenants.
Ignoring the impact of edge cases where tenants might attempt to circumvent the sharding mechanism.

#### Follow-Ups
How would you handle scenarios with varying tenant workloads?
What measures can be taken to further enhance fault tolerance and isolation?

**Comparison:**

| Failure Domain | Tenants in Shard 0 | Tenants in Shard 1 |
|----------------|--------------------|--------------------|
| 0              | 5000               | 6000               |
| 1              | 7000               | 8000               |

**Code:**

```python
# Example code for distributing tenants across failure domains
def distribute_tenants(tenants, num_failure_domains, shards_per_tenant):
    import random

    virtual_shards = []
    shard_distribution = {i: [] for i in range(num_failure_domains)}

    for tenant_id in range(tenants):
        for _ in range(shards_per_tenant):
            # Randomly assign a failure domain to each virtual shard
            shard_distribution[random.randint(0, num_failure_domains - 1)].append(tenant_id)

    return shard_distribution

# Example usage
num_tenants = 50000
failure_domains = 20
shards_per_tenant = 3

shard_distribution = distribute_tenants(num_tenants, failure_domains, shards_per_tenant)
print(shard_distribution)
```

**Example input / output:**

```

Input:

Tenants: 50,000
Failure Domains: 20
Shards Per Tenant: 3

Output (Example Shard Distribution):

JSON
Copy
{
  "failure_domain_0": [0, 1667, 3334],
  "failure_domain_1": [50001, 6668, 8332],
  ...
}

This shows how tenants are distributed across failure domains to ensure isolation.
```

**Data:**

```json
{
  "tenants": 50000,
  "failure_domains": 20,
  "shards_per_tenant": 3,
  "threshold": 6% // of total resources per failure domain
}
```

**Flow:**

1. Yes
2. No
3. Start
4. Is Each Tenant Sharded AcrossMultiple Failure Domains?
5. Distribute Tenants Evenly
6. Define Resource Thresholds forEach Shard
7. Monitor and Adjust ifNecessary
8. Redesign Shard Strategy toEnsure Isolation
9. End
10. This flowchart outlines the steps involved in designing a robust shuffle sharding strategy.

