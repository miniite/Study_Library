---
type: "practice"
difficulty: "Intermediate"
---

# A consumer joins a group and immediately gets assigned zero partitions while others are overloaded. What assignment-strategy or membership issue explains this?

**Difficulty:** Intermediate · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Consumer Groups and Rebalancing](<../../../../TOOLS/Apache_Kafka/Kafka_Core/Consumer_Groups_and_Rebalancing.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In this scenario, we face an interesting challenge with Apache Kafka where a new consumer in a consumer group is not receiving any assigned partitions, whereas other consumers are handling a larger load. This situation can be explained through several potential issues related to the consumer group's assignment strategy and membership state.

To break it down from first principles:

#### Cons
umer Group Membership and Assignment: In Kafka, each consumer in a group must agree on which partitions should be assigned to each member of the group. This is crucial for load balancing.
Rebalance Mechanism: When a new consumer joins or an existing one leaves, Kafka triggers a rebalance process where the remaining consumers reassign partitions among themselves.

Now let's explore the possible causes and solutions:

#### Triage
Restating the Question

The question essentially asks us to identify what could cause a newly joined consumer to be assigned no partitions while others in the group are handling a significant load. We need to consider potential rebalance issues, configuration settings, or state-related anomalies.

#### First-Principles Solution
Check Rebalance Strategy:
Kafka uses round-robin by default for partition assignment among consumers within a group. However, if this strategy is misconfigured or the consumer's metadata (like its consumer group state) is corrupted, it can lead to improper rebalancing.
Review Consumer Group State and Metadata:
Each consumer maintains its own metadata about the consumer group. If the new consumer has stale or incorrect metadata, it might not be recognized by Kafka as part of the group during a rebalance.
Monitor Partition Assignment Logs:
Review the logs for partition assignment events to understand why the new consumer is not receiving any partitions.
Step-by-Step Solution
Verify Metadata and Group Membership:
BASH
kafka-consumer-groups --bootstrap-server <kafka-broker> --describe --group <consumer-group>
Ensure that the new consumer’s metadata is correctly updated in Kafka.
Inspect Broker Logs for Rebalance Events:
Check broker logs to see if there are any messages indicating a rebalance was initiated but did not properly reassign partitions due to some issue.

#### Common Mistakes
and Follow-Ups
Incorrect Metadata: The new consumer's metadata might be outdated or corrupted.
Broker Configuration: The Kafka brokers might have misconfigured settings that affect partition assignment policies.
Network Issues: Network latency or connectivity issues could delay the propagation of rebalance events.

#### Cons
umer Code Implementation: The consumer’s implementation might have bugs, such as not correctly handling rejoin signals.

#### Closing Summary

To resolve this issue, you should verify and ensure proper metadata management for the consumer group. Monitor logs to trace any potential miscommunications or errors during the rebalance process. Additionally, review Kafka broker configurations related to partition assignment strategies.

**Code:**

```bash
kafka-consumer-groups --bootstrap-server localhost:9092 --describe --group my-group
```

**Example input / output:**

```text
Consumer group 'my-group' has 4 members:
- Member member1 from broker <broker-ip>:9092 [NOT_CONNECTED] -> not receiving any partitions
- Member member2 from broker <broker-ip>:9093 [ASSIGNED] -> assigned to partition 1, 3
- Member member3 from broker <broker-ip>:9094 [ASSIGNED] -> assigned to partition 2, 4

[Broker logs]
<timestamp> [Rebalance LeaderElection] - New leader elected for topic-partition: my-topic-0
<timestamp> [Rebalance Assignment] - Partition assignment failed due to stale metadata
```

**Flow:**

1. New Consumer Joins Group
2. No Partitions Assigned
3. Misconfigured Metadata/State
4. Incorrect Rebalance Strategy
5. Review Metadata and Logs
6. Tweak RebalanceConfigurations

