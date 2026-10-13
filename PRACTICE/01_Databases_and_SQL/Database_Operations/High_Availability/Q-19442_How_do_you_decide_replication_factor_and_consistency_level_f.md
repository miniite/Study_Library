---
type: "practice"
difficulty: "Deep"
---

# How do you decide replication factor and consistency level for a globally-replicated NoSQL store balancing HA, latency, and storage cost across regions? Walk through a concrete tuning example.

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ HA Failover Design](<../../../../01_Databases_and_SQL/Database_Operations/High_Availability/HA_Failover_Design.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When designing a highly available (HA) database architecture for a globally replicated NoSQL store, deciding on the replication factor and consistency level is crucial to balance high availability, low latency, and storage costs across regions. For an experienced Database Developer Architect/Principal, this involves understanding the trade-offs between these factors and making informed decisions based on specific requirements.

#### Triaging the Question

The key elements of this question are:

Replication Factor: Determines how many copies of the data are stored in different nodes or regions.

#### Cons
istency Level: Defines the required level of agreement among replicas before a write operation is considered successful.
Globally Replicated NoSQL Store: Ensures data availability across multiple regions, which adds complexity due to network latency and cost.

#### First-Principles Solution

Let's consider a concrete example: A multi-regional NoSQL store that needs to balance high availability with low latency and storage costs. We'll use a distributed database like Cassandra as the basis for this discussion.

Step 1: Define Business Requirements
High Availability: Ensuring minimal downtime by having multiple replicas.
Low Latency: Minimizing response times for global users.
Cost Management: Balancing the cost of storage and network bandwidth across regions.
Step 2: Determine Replication Factor

The replication factor (RF) is set to ensure that data is available in multiple nodes or regions. For a globally distributed system, we need to balance RF with network latency and consistency level.

Initial Assumption: Let's assume the global region consists of three main regions (Region A, Region B, and Region C).
Replication Strategy: To achieve high availability, we might choose an RF of 3. This means each piece of data will be stored in 3 replicas across different nodes or regions.
Step 3: Define Consistency Level

The consistency level determines how many replicas must acknowledge a write before it is considered successful. Common choices are ONE, QUORUM, and ALL.

Choosing Consistency Level: Given the need for low latency, we might opt for LOCAL_QUORUM or EACH_QUORUM. These ensure that a majority of nodes in the region acknowledge the write, providing good performance while maintaining strong consistency.
Step 4: Balancing Availability and Cost
Network Latency Considerations: Higher RF increases network bandwidth usage. For example, if each node sends data to all replicas, the total network traffic can be significant.
Storage Cost Management: More replicas mean higher storage costs. We need to optimize by ensuring that regions with lower write activity or higher latency have fewer replicas.
Step 5: Example Configuration

Let's configure our NoSQL store for a specific use case:

Region A (Primary Region): RF = 3, Consistency Level = EACH_QUORUM.
Region B and C (Secondary Regions): RF = 2, Consistency Level = LOCAL_QUORUM.

This configuration ensures that the primary region has strong consistency and availability, while secondary regions have lower costs and moderate latency impacts.

#### Closing Summary

In summary, for a globally distributed NoSQL store, setting an appropriate replication factor and consistency level involves balancing high availability with low latency and storage costs. By carefully choosing these parameters based on specific business requirements and regional characteristics, we can achieve optimal performance and cost-efficiency.

#### Common Mistakes/Follow-Ups
Ignoring Latency: Failing to consider network latency when setting RF can lead to unnecessary bandwidth usage.
Over-Optimizing Costs: Reducing the replication factor too much might compromise availability during failures.
Insufficient Consistency Level: Choosing a consistency level that is too low can result in data inconsistencies.

#### Follow-ups
could involve discussing advanced features like cross-region read repair, global tables, and custom consistency strategies based on application needs.

