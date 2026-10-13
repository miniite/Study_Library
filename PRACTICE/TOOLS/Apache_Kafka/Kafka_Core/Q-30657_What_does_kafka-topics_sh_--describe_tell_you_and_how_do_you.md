---
type: "practice"
difficulty: "Intermediate"
---

# What does kafka-topics.sh --describe tell you, and how do you read the Isr and Replicas fields to spot an under-replicated partition?

**Difficulty:** Intermediate · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Topic Design for Re-sync](<../../../../TOOLS/Apache_Kafka/Kafka_Core/Topic_Design_for_Re-sync.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To understand what kafka-topics.sh --describe tells us, we need to first grasp its utility in Kafka clusters. This command provides detailed information about topics, including partition details and replication configurations. Specifically, the Isr (In-Sync Replicas) and Replicas fields are crucial for identifying under-replicated partitions.

#### Triaging the Question

The question is asking you to explain what specific information these two fields provide in a Kafka topic description output, focusing on how they indicate whether a partition has enough replicas that are in sync. The interviewer might be probing your understanding of Kafka's replication semantics and how to diagnose potential issues with data durability.

#### First-Principles Solution
Understanding Replicas: This field lists all the brokers (replica nodes) assigned to each partition of a topic. Each replica is essential for ensuring that the data is stored in multiple places, thereby providing fault tolerance.
Understanding Isr: The In-Sync Replicas list contains only those replicas from the Replicas list that are currently healthy and able to participate in replication and fetch operations. A partition is under-replicated if there are not enough Isr nodes relative to the desired number of replicas.
Step-by-Step Solution
Identify Replicas: First, look at the Replicas field for a specific partition. This list tells us all the brokers that are assigned as replicas for this partition.
Compare with Isr: Next, check the Isr field. It should only contain those nodes from the Replicas list that are currently in sync and available to replicate and fetch data. If the number of Isr nodes is less than the minimum required number (which can be set by configuration properties like min.insync.replicas), then this partition is under-replicated.
Under-Replication Scenario: An example would be if a topic has 3 replicas, but only 2 of those are in sync as per the Isr. This means one node might have gone down or lagged behind significantly.

#### Closing Summary
and Common Mistakes/Follow-Ups

In summary, kafka-topics.sh --describe is an essential tool for monitoring Kafka cluster health. By examining both the Replicas and Isr fields, you can identify partitions that are under-replicated, which could lead to data loss if a node fails.

#### Common mistakes
include ignoring the min.insync.replicas configuration or misunderstanding how these fields interact. Be sure to account for dynamic rebalancing scenarios where nodes might join or leave the Isr.

#### Example
PLAINTEXT
Topic: my_topic
PartitionCount: 3
ReplicaAssignment:
    [0] -> [198|1]
    [1] -> [201|2]
    [2] -> [204|3]
Isr:
    [0] -> [201,204]
    [1] -> [201,204]
    [2] -> [201,204]

In this example, all partitions have two in-sync replicas (Isr), so the topic is well-replicated.

Edge Cases and Follow-Ups
Partition Loss: If a node is removed from Isr, but still appears in Replicas, you might need to investigate why it was taken out of sync.
Manual Replication Management: Sometimes, manual intervention is required to adjust the replication setup if certain nodes are being excluded due to network issues or hardware problems.

**Code:**

```bash
kafka-topics.sh --describe --topic my_topic --zookeeper localhost:2181
```

**Example input / output:**

```

Example output:

PLAINTEXT
Copy
Topic: my_topic
PartitionCount: 3
ReplicaAssignment:
    [0] -> [198|1]
    [1] -> [201|2]
    [2] -> [204|3]
Isr:
    [0] -> [201,204]
    [1] -> [201,204]
    [2] -> [201,204]
```

