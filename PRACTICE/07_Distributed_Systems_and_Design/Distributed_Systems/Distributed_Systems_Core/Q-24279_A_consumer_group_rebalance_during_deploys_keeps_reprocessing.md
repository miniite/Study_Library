---
type: "practice"
difficulty: "Deep"
---

# A consumer group rebalance during deploys keeps reprocessing and stalling your ingestion. Explain the rebalance protocol why a slow consumer triggers it and how static membership and cooperative rebalancing reduce the stop-the-world.

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Partition Lag and Hot Keys](<../../../../07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Partition_Lag_and_Hot_Keys/_index.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the issue of a consumer group rebalance during deploys causing reprocessing and stalling your ingestion, we first need to understand the underlying mechanisms of Kafka's rebalance protocol. A slow consumer triggers this because it causes unbalanced load distribution among the consumer group members. The core solution involves static membership and cooperative rebalancing to mitigate these issues.

The rebalance protocol in Apache Kafka is triggered when any member of a consumer group leaves or joins, leading to an uneven distribution of partitions across consumers. In such scenarios, slow consumers can exacerbate this imbalance by not processing messages as quickly as their peers, thereby causing additional delays and reprocessing. Static membership and cooperative rebalancing are designed to mitigate these problems.

Firstly, let's delve into the mechanics of static membership. This approach involves assigning specific partitions to each consumer in a consumer group at configuration time. By doing so, we ensure that the workload is distributed more evenly across all consumers, reducing the likelihood of bottlenecks caused by slow consumers. If a consumer goes down or restarts, its assigned partitions are immediately reassigned to other active members, minimizing downtime and reprocessing.

Cooperative rebalancing is another key mechanism. It aims to achieve a better balance among consumers without forcing an immediate global stop-the-world (STW) event. Cooperative rebalancing works by having consumers negotiate the transfer of partitions among themselves rather than relying on the Kafka broker to forcibly reassign them. This negotiation process can be more efficient and less disruptive, especially in large-scale distributed systems.

To further illustrate these concepts, let's consider an example where we have a consumer group with three members: Consumer A, B, and C. Initially, they are assigned partitions 1, 2, and 3 respectively. If Consumer B goes down, static membership ensures that the partitions it was handling (say partition 2) are reassigned to other active consumers immediately. Cooperative rebalancing would involve Consumer A and C negotiating to take on the tasks of Consumer B without a global reassignment.

In terms of implementation, we can use Kafka's consumer properties to configure static membership using group.instance.id or similar parameters. For cooperative rebalancing, leveraging features like max.poll.interval.ms and implementing custom offset management strategies can help consumers handle partition transfers more gracefully.

#### Common mistakes
in addressing this issue include not properly configuring static membership and relying solely on the default Kafka behavior for reassignment, which often leads to unbalanced loads and unnecessary reprocessing. Additionally, failing to implement cooperative rebalancing can result in frequent and disruptive STW events, impacting overall system performance.

**Comparison:**

| Partitions | Initial Assignment | After Rebalance (Consumer B Down) |
|------------|--------------------|----------------------------------|
| 0          | A                  | A                                |
| 1          | B                  | -                                |
| 2          | C                  | A                                |
| 3          | -                  | C                                |

**Code:**

```python
# Example of setting up static membership in a Python Kafka consumer
from kafka import KafkaConsumer

consumer = KafkaConsumer(
    'topic_name',
    bootstrap_servers=['localhost:9092'],
    group_id='my_group',
    # Assign partitions statically
    auto_offset_reset='earliest',
    enable_auto_commit=False,
    group_instance_id="instance_1"  # Static membership configuration
)
```

**Example input / output:**

| Input (Consumer Group State) | Output (Rebalance Strategy) |
|-----------------------------|----------------------------|
| Consumer A: Active, Consumers B and C are down | Reassign partitions held by Consumer B to Consumer A. Use cooperative rebalancing for graceful negotiation of partition transfers. |
| Consumer C restarts but others are active | Assign any new partitions to Consumer C as part of static membership configuration. |

**Data:**

```json
{
  "group": {
    "name": "my_group",
    "members": [
      {"id": "consumer_a", "partitionAssignments": [0, 2]},
      {"id": "consumer_b", "partitionAssignments": [1]},
      {"id": "consumer_c", "partitionAssignments": [3]}
    ]
  }
}
```

**Flow:**

1. Receives message
2. Down
3. Active
4. Consumes messages
5. Notifies Kafka Broker
6. Receives notification
7. Transfers partitions
8. Updates state
9. Consumer A
10. MessageHandling
11. Consumer B
12. DownState
13. Consumer C
14. ActiveState
15. ConsumerLogic
16. RebalanceNotification
17. CooperativeNegotiation
18. PartitionTransfer
19. NewAssignment

