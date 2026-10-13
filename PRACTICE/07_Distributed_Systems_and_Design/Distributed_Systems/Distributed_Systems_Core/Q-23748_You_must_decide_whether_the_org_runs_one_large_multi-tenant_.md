---
type: "practice"
difficulty: "Expert"
---

# You must decide whether the org runs one large multi-tenant GPU serving cluster or many isolated per-team clusters. Frame the trade-off through GPU-utilization efficiency, noisy-neighbor blast radius, and control-plane failure domain, and state the tenancy density where your answer flips.

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Multi-tenant vs Dedicated Clusters](<../../../../07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Mixed_Version_Fleets/Multi-tenant_vs_Dedicated_Clusters.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In deciding between running a single large multi-tenant GPU serving cluster versus multiple isolated per-team clusters, we need to carefully weigh several critical factors: GPU utilization efficiency, noisy-neighbor blast radius, and control-plane failure domain. These considerations will help us determine the optimal solution based on the tenancy density of teams in the organization.

#### Triaging the Question

The question is fundamentally about balancing resource management with isolation and risk management across multiple tenants. The key trade-offs include:

GPU Utilization Efficiency: Maximizing the use of available GPU resources.
Noisy-Neighbor Blast Radius: Minimizing the impact when one tenant misbehaves or fails, affecting others.
Control-Plane Failure Domain: Ensuring that a failure in managing clusters does not cascade to other teams.

#### Assumptions
and Initial Considerations
Tenancy Density: The number of tenants (teams) sharing resources within a cluster.
Resource Variability: GPUs have different capabilities, usage patterns, and performance requirements.
Failure Rate: Probability and impact of failures in the clusters.
Operational Complexity: Ease of managing one large cluster versus multiple smaller ones.

#### First-Principles Solution
GPU Utilization Efficiency

In a single large multi-tenant cluster:

#### Pros
Higher overall GPU utilization due to better resource sharing and load balancing.
Easier management of common resources like software stacks, updates, and maintenance.

#### Cons
Potential underutilization during off-peak times when not all tenants are active.
More complex scheduling algorithms required to balance tenant workloads.

In multiple isolated per-team clusters:

#### Pros
Each team can have its own dedicated resources, leading to better utilization efficiency.
Simplified workload management and resource allocation.

#### Cons
Lower overall GPU utilization due to smaller cluster sizes.
Increased operational overhead for managing multiple control planes.
Noisy-Neighbor Blast Radius

In a single large multi-tenant cluster:

#### Pros
Isolating failures is more challenging, potentially affecting other tenants if one misbehaves or fails.

#### Cons
Lower risk of noisy neighbors impacting others due to resource isolation.

In multiple isolated per-team clusters:

#### Pros
Each team has its own isolated environment, reducing the blast radius of failures.

#### Cons
Higher risk of noisy neighbors affecting their own cluster if misbehaving or failing.
Control-Plane Failure Domain

In a single large multi-tenant cluster:

#### Pros
Single control plane for management and monitoring, simplifying operations.

#### Cons
A failure in the central control plane can cascade to all tenants.

In multiple isolated per-team clusters:

#### Pros
Each tenant has its own control plane, reducing the risk of a single point of failure.

#### Cons
Higher complexity and operational overhead for managing multiple control planes.

#### Closing Summary

The optimal solution depends on the tenancy density. At low tenancy densities (e.g., fewer than 10 teams), the benefits of isolation in per-team clusters are more significant, reducing the blast radius and simplifying operations. However, as tenancy density increases, the advantages of a single large cluster in terms of resource efficiency become more pronounced.

#### Common Mistakes/Follow-Ups
Overlooking Resource Variability: Failing to consider different GPU types and usage patterns.
Neglecting Operational Complexity: Ignoring the additional overhead of managing multiple control planes.
Ignoring Failure Probabilities: Not accounting for the likelihood and impact of failures in different cluster configurations.
Triaging Follow-Ups
Resource Variability: What are the specific GPU models and their resource demands?
Operational Complexity: How many teams will be involved, and what is the expected growth rate?
Failure Probabilities: What is the historical failure rate of each cluster configuration?

**Comparison:**

```
Low Tenancy Density
Isolated per-team clusters
Single large multi-tenantcluster
Utilization Efficiency
Blast Radius
Utilization Efficiency
Blast Radius
```

**Code:**

```python
# Example code to simulate GPU utilization in both scenarios
def gpu_utilization(tenancy_density):
    if tenancy_density < 10:
        return "Isolated per-team clusters"
    else:
        return "Single large multi-tenant cluster"

print(gpu_utilization(5))  # Output: Isolated per-team clusters
print(gpu_utilization(20)) # Output: Single large multi-tenant cluster
```

**Example input / output:**

| Tenancy Density | Cluster Type                | Utilization Efficiency | Blast Radius |
|-----------------|----------------------------|-----------------------|--------------|
| Low             | Isolated per-team clusters  | Medium-High           | Low-Medium   |
| Medium          | Single large multi-tenant   | High                  | Medium-High  |
| High            | Single large multi-tenant   | High                  | Medium-High  |

**Data:**

```json
{
  "tenancy_density": {
    "low": 1,
    "medium": 10,
    "high": 20
  },
  "cluster_types": [
    {"type": "Isolated per-team clusters", "utilization_efficiency": "Medium-High", "blast_radius": "Low-Medium"},
    {"type": "Single large multi-tenant cluster", "utilization_efficiency": "High", "blast_radius": "Medium-High"}
  ]
}
```

**Flow:**

1. Low Tenancy Density
2. Isolated per-team clusters
3. Single large multi-tenantcluster
4. Utilize Efficiently?
5. Yes, High
6. Recommend Isolated Clusters
7. No, Medium-High
8. Recommend Large Cluster
9. Utilize Efficiently?
10. Yes, High
11. Recommend Isolated Clusters
12. No, High
13. Recommend Large Cluster

