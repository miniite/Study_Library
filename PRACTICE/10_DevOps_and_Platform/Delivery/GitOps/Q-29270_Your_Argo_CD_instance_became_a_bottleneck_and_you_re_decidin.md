---
type: "practice"
difficulty: "Expert"
---

# Your Argo CD instance became a bottleneck and you're deciding between one big instance versus sharded instances per cluster — what drives that architecture choice?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Scaling Argo CD](<../../../../10_DevOps_and_Platform/Delivery/GitOps/Argo_CD_vs_Flux/Scaling_Argo_CD.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To decide whether to use one large Argo CD instance or multiple sharded instances per cluster, we need to consider several factors including performance, reliability, security, and maintenance complexity. The core decision involves evaluating how resource usage and load balancing can be optimized while ensuring high availability.

Argo CD is a powerful tool for implementing GitOps by keeping application manifests up-to-date with desired states stored in version control systems like Git. When an Argo CD instance becomes a bottleneck, it usually means that the single point of truth is struggling to manage updates efficiently across multiple clusters. The choice between one large instance or sharding instances per cluster depends on various trade-offs.

First, let's consider the current situation where a single Argo CD instance handles all clusters. This setup might be simpler to maintain and monitor but can become a single point of failure if not properly architected for high availability. Performance bottlenecks could occur due to excessive load on this central instance during updates or in case of unexpected spikes.

On the other hand, sharding instances across multiple clusters can distribute both the load and the risk. Each cluster has its own Argo CD instance, which reduces dependency and potential failure points. However, managing multiple instances introduces complexity around coordination, consistency, and synchronization between clusters.

To reason through this from first principles:

Scalability: How many clusters do we have? If there are only a few clusters, one large instance might be sufficient. For larger setups or clusters with varying workloads, sharding can help manage load more effectively.
Resource Utilization: A single instance may underutilize resources during light loads but overwork during heavy updates. Sharding ensures that each cluster has the necessary compute capacity available at any time.
Availability and Resilience: In a sharded setup, if one Argo CD instance fails, others can still function independently. This improves overall reliability compared to a single point of failure.
Operational Complexity: Managing multiple instances requires more operational effort for deployment, monitoring, and maintenance. However, this can be mitigated with proper automation and tooling.

In summary, the decision should balance between centralized control and distributed management based on the specific requirements of the infrastructure. A hybrid approach might also work where a primary instance coordinates updates while sharded instances handle local operations, ensuring both efficiency and reliability.

**Comparison:**

| Approach           | Single Large Instance     | Sharded Instances per Cluster |
|--------------------|--------------------------|------------------------------|
| Scalability        | Centralized control      | Distributed management       |
| Resource Utilization | High during peak loads   | Balanced load distribution   |
| Availability & Resilience | Single point of failure  | Reduced risk from SPOF       |
| Operational Complexity | Lower                    | Higher, but with automation |

**Code:**

```
MERMAID
Copy
graph TD;
    A[Single Large Instance] --> B[Centralized Control]
    A --> C[High Resource Utilization During Peak Load]
    A --> D[Single Point of Failure]

    E[Sharded Instances per Cluster] --> F[Distributed Management]
    E --> G[Reduced Risk from Single Point of Failure]
    E --> H[Increased Operational Complexity]
```

**Example input / output:**

```text
# Example input and output for monitoring load distribution

Input:
- Cluster1: Argo CD Instance is under heavy load (90% CPU usage)
- Cluster2: Argo CD Instance has moderate load (45% CPU usage)

Output:
- Cluster1 sharded instance deployed to handle high load
- Cluster2 sharding not necessary as current utilization is within limits

Request for Response:
- Is it feasible to maintain a single large instance with load balancing?
  - Yes, if the number of clusters and updates are manageable.
- Should we prioritize operational simplicity over distributed management?
  - No, unless the benefits significantly outweigh increased complexity.

Dry Run:
- Simulate updating multiple clusters using both approaches
```

**Data:**

```yaml
# Example YAML for cluster configuration with sharding

clusters:
  - name: cluster1
    argo-cd-instance: instance1
  - name: cluster2
    argo-cd-instance: instance2
  - name: cluster3
    argo-cd-instance: instance3
```

**Flow:**

1. Number of Clusters
2. <5 - Use single instance
3. <10 - Consider sharding forredundancy
4. >10 - Definitely use shardedinstances
5. Resource Utilization DuringPeak Load
6. High - Optimize usingsharding
7. Low - Single instance sufficient
8. Operational Complexity
9. Acceptable - Use singleinstance
10. Not Acceptable - Shard forsimplicity and reliability

