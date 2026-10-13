---
type: "practice"
difficulty: "Intermediate"
---

# What is replication in Kafka and what are leader and follower replicas?

**Difficulty:** Intermediate · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Durability with acks and ISR](<../../../../TOOLS/Apache_Kafka/Kafka_Core/Durability_with_acks_and_ISR.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Replication in Apache Kafka ensures data durability by distributing a topic's partitions across multiple brokers (servers). Each partition has one leader replica and zero or more follower replicas. The leader is responsible for handling all read/write operations, while followers replicate the leader’s data to maintain redundancy.

To thoroughly understand replication in Kafka, we need to delve into its architecture, roles of leaders and followers, and how this design ensures reliability and fault tolerance.

**Comparison:**

```
Partition ID	Leader Broker	Follower Brokers
0	broker-1	broker-2
1	broker-3
```

**Example input / output:**

```text
Input:
- A topic named 'example-topic' with two partitions.

Output:
Partition 0 - Leader on broker-1, followers are broker-2.
Partition 1 - Leader on broker-3 (with no followers).
```

**Data:**

```json
{
  "topic": {
    "name": "example-topic",
    "partitions": [
      {"id": 0, "leader": "broker-1", "replicas": ["broker-1", "broker-2"]},
      {"id": 1, "leader": "broker-3", "replicas": []}
    ]
  }
}
```

**Flow:**

1. Yes
2. No
3. Yes
4. No
5. Producer Sends Message
6. Message Sent to Leader
7. Leader Processes
8. Message Replicated byFollowers
9. Message Stored on LeaderOnly
10. Consumer Reads Message
11. Follower Has Latest Data?
12. Follower Returns Message toConsumer
13. Leader Sends Message toFollower, Then to Consumer

