---
type: "practice"
difficulty: "Basic"
---

# How does a producer decide which partition a message goes to?

**Difficulty:** Basic · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Partitions and Ordering](<../../../../TOOLS/Apache_Kafka/Kafka_Core/Partitions_and_Ordering.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When using Apache Kafka, producers must determine which partition within a topic receives each message. This decision is crucial for ensuring efficient data distribution and balancing load across partitions. The key concept here is that producers can control this behavior through various strategies.

The producer must understand the underlying mechanism of Kafka partitions to make an informed decision on how messages are distributed. Typically, this involves leveraging the partitioner’s API or using a custom strategy based on message metadata.

**Code:**

```python
from kafka import KafkaProducer

producer = KafkaProducer(bootstrap_servers='localhost:9092')
```

