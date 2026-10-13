---
type: "practice"
difficulty: "Intermediate"
---

# How does a consumer fetch from multiple partitions in one request, and how are records returned across those partitions?

**Difficulty:** Intermediate · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Consumer Groups and Rebalancing](<../../../../TOOLS/Apache_Kafka/Kafka_Core/Consumer_Groups_and_Rebalancing.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To understand how a Kafka consumer fetches from multiple partitions in one request, it’s essential to grasp the underlying mechanism of Kafka partitioning and the role of offsets. A key point is that each message within a topic is assigned to exactly one partition. When a consumer reads messages, it operates on these partitions independently but can coordinate with other consumers through group management.

#### Triage

The question probes your understanding of Kafka's architecture, specifically how consumers manage multiple partitions and the role of offsets in this process. You need to explain how consumers can fetch data from all available partitions within a topic in one request (or as close to it as possible), and how they ensure that records are returned consistently across these partitions.

#### First-Principles Solution

Kafka operates on a partitioned model where each partition is an ordered, immutable sequence of messages. Consumers read from specific partitions using offsets, which represent the position in the log. To fetch data from multiple partitions, consumers must first join into a consumer group and subscribe to the topic. The key here is that a single consumer instance can only consume one partition at a time, but Kafka allows multiple instances within a consumer group to work together.

Partition Assignment: When a consumer joins a consumer group, Kafka assigns it specific partitions from the subscribed topics based on the group's configuration and the current state of the cluster.
Offset Management: Each message in a partition has an offset, which is used by consumers to track their position. Consumers can fetch messages within their assigned partitions using their respective offsets.

#### Cons
umer Group Coordination: Multiple consumer instances in a single group coordinate through Kafka’s group coordinator and metadata service (Zookeeper or KRaft). They share state and ensure that no two consumers from the same group read the same message, maintaining a consistent flow of data.

#### Example
Fetching Data Across Partitions

To illustrate, let's assume we have a topic orders with three partitions. A consumer group has multiple instances subscribing to this topic. Each instance will be assigned one partition and can fetch messages from its own partition independently. If the consumer group has more than one instance, they collectively manage fetching data across all partitions.

Here’s an example:

Partitions: P0, P1, P2

#### Cons
umer Group Instances: C1 (P0), C2 (P1), C3 (P2)

When a request is made to fetch messages from the orders topic:

PLAINTEXT
C1: Fetches messages from partition P0.
C2: Fetches messages from partition P1.
C3: Fetches messages from partition P2.

This allows for parallel processing and efficient data consumption, as each consumer instance can handle its own partition independently.

#### Closing Summary
+ Common Mistakes/Follow-Ups

In summary, Kafka ensures that multiple consumers in a group can fetch messages across partitions by leveraging partition assignment and offset management. The key is understanding how the consumer group manages state and coordination to ensure no overlap and efficient data processing.

#### Common mistakes
include:

Not configuring the correct number of consumer instances for an optimal balance between parallelism and resource utilization.
Misunderstanding the role of offsets in maintaining consistency across partitions.
Overlooking the importance of proper consumer group management to avoid issues like rebalancing.

#### Follow-up
questions might delve into specific configuration options, best practices for scaling consumer groups, or handling failure scenarios.

**Comparison:**

| Partition | Leader Broker | Offset Range |
| --- | --- | --- |
| P0 | broker1 | 100 - 300 |
| P1 | broker2 | 50 - 250 |
| P2 | broker3 | 75 - 275 |
| Consumer Instance | Assigned Partition |
| --- | --- |
| C1 | P0 |
| C2 | P1 |
| C3 | P2 |

**Code:**

```python
# Example Python Kafka Consumer Code
from kafka import KafkaConsumer

consumer = KafkaConsumer('orders',
                         bootstrap_servers=['localhost:9092'],
                         group_id='my-consumer-group')
for message in consumer:
    print(f"Received {message.value} from partition {message.partition}")
```

**Example input / output:**

```text
Input:
- Subscription to topic 'orders' by consumer group 'my-consumer-group'
Output:
- C1: [Message value from partition P0, offset 100], [Message value from partition P0, offset 200], [Message value from partition P0, offset 300]
- C2: [Message value from partition P1, offset 50], [Message value from partition P1, offset 150], [Message value from partition P1, offset 250]
- C3: [Message value from partition P2, offset 75], [Message value from partition P2, offset 175], [Message value from partition P2, offset 275]
```

**Data:**

```json
{
  "topic": "orders",
  "partitions": [
    {"id": 0, "leader": "broker1", "offsets": [100, 200, 300]},
    {"id": 1, "leader": "broker2", "offsets": [50, 150, 250]},
    {"id": 2, "leader": "broker3", "offsets": [75, 175, 275]}
  ],
  "consumer_group_instances": [
    {"instance_id": "C1", "assigned_partitions": [0]},
    {"instance_id": "C2", "assigned_partitions": [1]},
    {"instance_id": "C3", "assigned_partitions": [2]}
  ]
}
```

**Flow:**

1. Yes
2. No
3. Consumer Joins Group
4. Assign Partitions?
5. Fetch Messages from AssignedPartition
6. Return Fetched Messages toClient
7. Wait for Reassignment

