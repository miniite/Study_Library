---
type: "practice"
difficulty: "Expert"
---

# What is unclean leader election and why is it risky to enable?

**Difficulty:** Expert · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Leader Election and Unclean Failover](<../../../../TOOLS/Apache_Kafka/Kafka_Core/Leader_Election_and_Unclean_Failover.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Unclean leader election in Apache Kafka refers to a situation where an unclean leader becomes the active leader due to node failures or network partitions, bypassing nodes that are still considered leaders according to the majority rule. This can lead to data loss and inconsistencies. In this answer, we'll delve into the concept of unclean leader elections, why they pose risks, and explore how to mitigate these risks.

To start, let's define what an unclean leader election is: It occurs when a Kafka cluster experiences network partitions or node failures that disrupt the normal leadership election process. Normally, in a ZooKeeper-based Kafka cluster, leaders are elected based on a majority rule; however, if there is a partition where one side has more nodes than the other and a minority of nodes still claim to be leaders, an unclean leader election can happen.

The key risk here is data loss or inconsistency because the unclean leader might not have all the latest updates. Imagine a scenario where Node A and Node B are in a network partition; Node A becomes the leader despite having only half the required nodes. If Node C on the other side still claims to be a leader, it will be bypassed even though it has more recent data. This can lead to conflicting versions of the same topic being consumed by different consumers.

Next, let's explore why enabling unclean leader elections is risky:

Data Inconsistency: As mentioned, unclean leaders might not have the latest data, leading to potential data loss or outdated information.

#### Cons
umer Confusion: Consumers connected to the unclean leader will receive stale or incorrect data, which can cause downstream systems to malfunction or produce inconsistent results.
Operational Complexity: Managing and debugging issues related to unclean leaders can be challenging due to the complex nature of Kafka's internal mechanisms.

Mitigating these risks involves disabling unclean leader elections:

SHELL
# In Kafka server properties file (server.properties)
unclean.leader.election.enable=false

Disabling this feature ensures that only nodes with a majority in their respective partitions can become leaders, thus maintaining data integrity and consistency. However, it's important to note that this comes at the cost of availability; if there is an unpartitioned failure, no leader will be available until all nodes are back online.

In conclusion, while enabling unclean leader elections might seem like a good idea in certain edge cases, the potential risks outweigh the benefits in most scenarios. Disabling it ensures data integrity but may lead to temporary service disruptions during node failures or partitions.

**Code:**

```shell
# Example of setting server.properties
unclean.leader.election.enable=false
```

