---
type: "practice"
difficulty: "Intermediate"
---

# What ordering guarantee does Kafka give within a single partition and why can ordering across partitions surprise you?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Topic Partitioning and Ordering](<../../../../03_Big_Data_and_Streaming/Distributed_Processing/Big_Data_Processing/Topic_Partitioning_and_Ordering.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In a Kafka topic, messages are stored in partitions. Within a single partition, Kafka guarantees exactly-once delivery of messages, meaning that each message is processed at most once. However, when dealing with multiple partitions, this guarantee is no longer applicable, leading to potential ordering issues.

#### Triage

The question asks for two main points: the ordering guarantee within a single Kafka partition and why ordering can be surprising across different partitions. The key concept here involves understanding how Kafka handles message sequencing and delivery semantics in both scenarios.

#### Assumptions
We assume you are familiar with basic Kafka concepts such as topics, partitions, and brokers.
We consider a typical use case where data processing pipelines rely on ordering guarantees for correct operations.
Detailed Solution
Within a Single Partition

When messages are written to a single partition in Kafka, they are processed in the order they were produced. This is because Kafka ensures that within a partition, messages are sequentially appended and consumed. The guaranteed ordering within a single partition means that no matter how many consumers you have, each consumer will process the messages in the same order.

To understand this better, consider a scenario where a producer sends three messages (A), (B), and (C) to a single Kafka partition. The broker appends these messages sequentially as they arrive. When a consumer group consumes from this partition, it will always receive the messages in the exact order: (A \rightarrow B \rightarrow C). This guarantee is crucial for maintaining consistency in applications that depend on message order.

Across Partitions

The ordering guarantee within a single partition does not extend to multiple partitions. In Kafka, each partition operates independently, and there's no built-in mechanism to ensure global ordering across different partitions. This can lead to unexpected behavior when processing data from multiple partitions in parallel or in sequence.

To illustrate this, imagine two partitions: Partition 1 contains messages (A), (B), and (C) with timestamps t1, t2, and t3 respectively (where t1 < t2 < t3). Similarly, Partition 2 has messages (D), (E), and (F) with timestamps t4, t5, and t6 where t4 < t5 < t6. If you have a consumer group that subscribes to both partitions, it could potentially consume the following sequence: (A \rightarrow D \rightarrow B \rightarrow E \rightarrow C \rightarrow F).

The surprising part here is that even though messages in each partition are ordered by their timestamps, there's no guarantee that they will be processed in the same global order. This can cause issues in applications where maintaining a consistent state or processing sequence across partitions is essential.

#### Closing Summary
and Common Mistakes/Follow-Ups

To summarize, Kafka guarantees exact-order delivery within a single partition but not between different partitions. Developers must account for this when designing their applications to ensure that messages are processed correctly and consistently.

#### Common Mistakes
Ignoring Partition Boundaries: Assuming that ordering is consistent across partitions without understanding the implications.
Overlooking Global Consistency: Failing to consider global consistency requirements, especially in distributed systems.

#### Follow-Ups
How can you handle global ordering in Kafka?
What are some use cases where maintaining global order is critical?

**Comparison:**

```
Message	Time
A	t1
B	t2
C	t3
D	t4
E	t5
F	t6
```

**Code:**

```python
# Example of a simple consumer setup in Python using kafka-python library
from kafka import KafkaConsumer

consumer = KafkaConsumer('my-topic',
                         bootstrap_servers=['localhost:9092'],
                         auto_offset_reset='earliest')
for message in consumer:
    print(f"Received message: {message.value.decode()}")
```

**Example input / output:**

```text
- **Input**: Producer sends messages \(A\), \(B\), \(C\) to Partition 1 and \(D\), \(E\), \(F\) to Partition 2.
- **Output** (Consumer group): Receives messages in a non-deterministic order, e.g., \(A \rightarrow D \rightarrow B \rightarrow E \rightarrow C \rightarrow F\).
```

**Data:**

```json
{
  "partitions": [
    {"id": 1, "messages": ["A", "B", "C"]},
    {"id": 2, "messages": ["D", "E", "F"]}
  ]
}
```

**Flow:**

1. Message A
2. Message B
3. Message C
4. Message D
5. Message E
6. Message F

