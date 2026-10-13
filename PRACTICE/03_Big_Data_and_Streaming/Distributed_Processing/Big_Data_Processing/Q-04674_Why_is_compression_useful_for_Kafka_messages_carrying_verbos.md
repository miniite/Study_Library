---
type: "practice"
difficulty: "Intermediate"
---

# Why is compression useful for Kafka messages carrying verbose logs and what is the trade-off?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Kafka Retention and Storage](<../../../../03_Big_Data_and_Streaming/Distributed_Processing/Big_Data_Processing/Kafka_Retention_and_Storage.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Compression is a crucial feature in Kafka for handling verbose log data, as it significantly reduces storage requirements and improves performance by reducing the amount of data that needs to be transferred over the network. In essence, compression helps in making efficient use of resources while maintaining the integrity of the logs.

To understand why compression is useful and what trade-offs come with it, let's break down the problem from first principles. We'll start by defining the key concepts and then move on to a detailed explanation.

**Code:**

```python
# Example Python code snippet for reading compressed Kafka messages (not executable)
from kafka import KafkaConsumer

consumer = KafkaConsumer(bootstrap_servers='localhost:9092',
                         value_deserializer=lambda m: m.decode('utf-8'))
```

**Data:**

```json
{
  "topic": "error_logs",
  "partitions": [
    { "id": 0, "replicas": [1, 3] },
    { "id": 1, "replicas": [2, 4] }
  ]
}
```

