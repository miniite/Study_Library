---
type: "practice"
difficulty: "Deep"
---

# What is replica.lag.time.max.ms and how does it decide when a follower falls out of the ISR?

**Difficulty:** Deep · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Durability with acks and ISR](<../../../../TOOLS/Apache_Kafka/Kafka_Core/Durability_with_acks_and_ISR.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To understand replica.lag.time.max.ms in Kafka, we need to dive into the world of Kafka's replication semantics and the concept of an in-sync replica set (ISR). The key idea here is that Kafka ensures data durability by replicating messages across multiple brokers. When a producer sends a message, it waits for acknowledgments from one or more brokers before confirming success. This process helps ensure that if any broker fails, there's still a copy of the data available.

replica.lag.time.max.ms is a configuration parameter that determines how long a follower (a Kafka broker that is lagging behind the leader) can be out of sync with the in-sync replica set before it gets removed from the ISR. This mechanism helps maintain the durability and availability of your Kafka cluster by ensuring that only brokers with recent data are part of the active replication group.

**Code:**

```python
# Example configuration snippet for a Kafka broker
broker_config = {
    "replica.lag.time.max.ms": 10000,  # 10 seconds in milliseconds
}
```

**Example input / output:**

```text
Here’s how this works in practice:
- Suppose we have a Kafka cluster with three brokers (A, B, C) and the leader is broker A.
- Broker B and C are followers. If `replica.lag.time.max.ms` is set to 10000 ms (10 seconds), then any follower that lags more than 10 seconds behind the leader will be considered out of sync.

For example, if a message is sent at time T, and broker B falls more than 10 seconds behind by time T+20s, it will be removed from the ISR.
```

**Flow:**

1. FLOWCHART · MERMAID
2. graph TD
3. A[Message Sent]
4. B[Leader (Broker A)]
5. C[Follower (Brokers B & C)]
6. D{Is replica.lag.time.max.ms exceeded?}
7. A -->|Yes| D
8. D -->|Yes| C1[Remove from ISR, Broker C becomes a follower again]
9. D -->|No| C2[Stay in ISR, continue replication]
10. subgraph ISR
11. B--Replication--C
12. end

