---
type: "practice"
difficulty: "Deep"
---

# Explain ordering guarantees in Kafka: what's ordered within a partition vs across partitions, and how partition-key choice affects both ordering and load balance. What breaks if you repartition?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Ordering and Partition Keys](<../../../../07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Ordering_and_Partition_Keys.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Ordering guarantees in Apache Kafka are crucial for ensuring that messages from a specific consumer group follow a certain order when consumed, which is vital for many distributed applications. In this answer, we'll explore what ordering means within and across partitions, how partition keys impact both ordering and load balance, and the implications of repartitioning topics.

First, let's clarify the basic structure: Kafka organizes messages into partitions, each of which can be consumed independently by a consumer group. Messages within a single partition are always delivered in the order they were produced. However, when considering multiple partitions, the ordering is only guaranteed for those that share the same key (i.e., belong to the same message set).

To illustrate this, consider a logging system where each log entry has a timestamp as its key; messages with the same timestamp would be ordered within their respective partitions but not necessarily across different timestamps.

Now, let's delve into the specifics:

Ordered Within Partitions:

Kafka guarantees that messages are delivered in the order they were produced for any single partition.
This means if you have a producer sending messages to a topic with 3 partitions, and all messages share the same key (e.g., user ID), those messages will always be ordered within their respective partitions.

Ordered Across Partitions:

Messages are not guaranteed to be in order across different partitions unless they share the same key.
If you have a topic with 3 partitions and send messages with keys user1, user2, and user3 respectively, there is no guarantee that these will arrive in any particular order.

Partition Key Choice:

The choice of partition key directly influences the ordering and load balance.
A well-chosen partition key can ensure ordered processing within partitions while also balancing the load across multiple consumers.
For example, using a user ID as the key would result in logs from the same user being processed together, which is useful for analytics or tracing user behavior.

Repartitioning:

Repartitioning involves changing how messages are distributed among partitions, potentially breaking existing ordering guarantees if not managed carefully.
If you repartition a topic and change the partition key, messages that were previously ordered might now be reordered due to their new distribution across partitions.
This can lead to inconsistencies in order for consumers depending on the new key scheme.

To summarize:

Within Partitions: Messages are always delivered in order if they share the same key.
Across Partitions: Ordering is not guaranteed unless messages share a common key.
Partition Key Choice: Affects both ordering and load balance; choose wisely to avoid issues during repartitioning.

#### Common mistakes
include:

Assuming that Kafka guarantees global message ordering without considering the partition keys.
Not carefully managing the impact of repartitioning on existing order dependencies, leading to inconsistent data processing.

**Comparison:**

```
Partition	Key	Message
0	user1	Log 0
0	user2	Log 1
1	user3	Log 0
1	user4	Log 1
```

**Code:**

```python
# Example code snippet for producing messages with a specific key
from kafka import KafkaProducer

producer = KafkaProducer(bootstrap_servers='localhost:9092')
key = b'user1'
for i in range(5):
    producer.send('logs', key=key, value=f'Log {i}'.encode())
```

**Example input / output:**

```
Input: Produce messages with keys user1, user2 to the topic logs.
Output:
JSON
Copy
{
  "partition_0": [
    {"key": "user1", "value": "Log 0"},
    {"key": "user1", "value": "Log 1"}
  ],
  "partition_1": [
    {"key": "user2", "value": "Log 0"},
    {"key": "user2", "value": "Log 1"}
  ]
}
Note: Messages with the same key (user1) are ordered within their respective partitions.
```

**Data:**

```json
{
  "topic": "logs",
  "partitions": [
    {
      "partition_id": 0,
      "messages": [
        {"key": "user1", "value": "Log 0"},
        {"key": "user2", "value": "Log 1"}
      ]
    },
    {
      "partition_id": 1,
      "messages": [
        {"key": "user3", "value": "Log 0"},
        {"key": "user4", "value": "Log 1"}
      ]
    }
  ]
}
```

**Flow:**

1. Yes
2. No
3. Produce Message with Key
4. Key Match?
5. Send to Same Partition
6. Send to Different Partition,Unordered
7. Receive Ordered Messages inPartition
8. Receive Unordered MessagesAcross Partitions

