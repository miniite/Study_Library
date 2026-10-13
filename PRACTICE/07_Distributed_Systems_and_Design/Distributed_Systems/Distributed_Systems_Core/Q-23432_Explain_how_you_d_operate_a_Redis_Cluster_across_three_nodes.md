---
type: "practice"
difficulty: "Intermediate"
---

# Explain how you'd operate a Redis Cluster across three nodes with replicas and what cluster-require-full-coverage does to availability when a hash-slot range loses both its master and replica.

**Difficulty:** Intermediate · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Zero Downtime Sharding Migrations](<../../../../07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Distributed_Joins_in_MPP_Systems/Zero_Downtime_Sharding_Migrations.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Summary:

To manage a Redis Cluster with three primary nodes and replicas, you must ensure fault tolerance by replicating data across multiple nodes. The cluster-require-full-coverage setting in Redis clusters affects availability when both master and replica of a hash-slot range fail, potentially leading to degraded service.

#### Triaging the Question

The core of this question revolves around understanding how Redis Clusters are managed with respect to node redundancy (replicas) and how cluster settings like cluster-require-full-coverage impact overall system availability. Specifically, we need to understand what happens in a scenario where a hash-slot range loses both its master and replica.

#### Assumptions
Three Nodes: There are three primary nodes in the Redis Cluster.
Replicas Per Node: Each node has one or more replicas.
Hash-Slot Distribution: Hash slots are evenly distributed across all nodes.
cluster-require-full-coverage Setting: This setting enforces a strict availability requirement.

#### First Principles Solution

Cluster Setup:

A Redis Cluster is divided into 16,384 hash slots (0 to 16383).
Each node in the cluster is responsible for some of these hash slots.
Data distribution ensures that if a node goes down, other nodes can handle requests from its hash slot range.

Replication:

For each primary node, one or more replica nodes exist to ensure redundancy.
This setup allows data to be replicated across different nodes, enhancing fault tolerance and availability.

Cluster-Require-Full-Coverage Setting:

cluster-require-full-coverage is a configuration option that defines the cluster's strictness in requiring all hash slots to have valid masters for the cluster to operate.
When this setting is enabled, Redis enforces that every hash slot must have both a master and a replica.
Scenario Analysis:

When a hash-slot range loses both its master and replica:

Immediate Impact:

If cluster-require-full-coverage is set to yes, the cluster will no longer consider itself operational.
No commands can be executed until at least one of the masters or replicas comes back online.

Fallback Mechanism:

If cluster-require-full-coverage is set to no, the cluster might still allow read and write operations, but with degraded performance.
This setting allows for a more flexible operation in cases where some hash slots are temporarily unavailable.

Recovery Process:

When the master or replica comes back online, Redis will automatically re-configure itself to ensure all hash slots have valid masters.
In case of multiple failures, recovery might take longer and could lead to temporary outages.

#### Closing Summary
+ Common Mistakes/Follow-Ups:

To operate a Redis Cluster effectively with three nodes and replicas:

Ensure Proper Configuration: Set up the cluster with appropriate replication settings.
Monitor Health: Regularly monitor node health and cluster coverage status.
Tune Settings: Use cluster-require-full-coverage judiciously based on service requirements.

#### Common mistakes
include not setting up enough replicas, ignoring monitoring, or incorrectly configuring settings like cluster-require-full-coverage.

Common Follow-Ups:
What if a node is only partially down?
How does network partition tolerance play into this?
What are the performance trade-offs of different settings?

**Comparison:**

```
Setting	Impact on Availability
Yes	Cluster downtime
No	Degraded operation
```

**Code:**

```bash
# Example configuration snippet for Redis Cluster
redis-cli --cluster create 192.168.1.1:6379 192.168.1.2:6379 192.168.1.3:6379 \
    --cluster-replicas 1
```

**Example input / output:**

```text
Input:
- Node 1 (192.168.1.1) goes down.
- Node 2 (192.168.1.2) also goes down.

Output:
- Cluster enters a degraded state if `cluster-require-full-coverage` is set to yes.
- If set to no, the cluster may continue operating but with reduced availability and performance.
```

**Data:**

```json
{
  "nodes": [
    {"ip": "192.168.1.1", "port": 6379, "replicaOf": null},
    {"ip": "192.168.1.2", "port": 6379, "replicaOf": null},
    {"ip": "192.168.1.3", "port": 6379, "replicaOf": 0}
  ]
}
```

**Flow:**

1. Yes
2. No
3. Node 1 Fails
4. Cluster-Require-Full-Coverage?
5. Cluster Downtime
6. Degraded Operation, ContinueFunctioning

