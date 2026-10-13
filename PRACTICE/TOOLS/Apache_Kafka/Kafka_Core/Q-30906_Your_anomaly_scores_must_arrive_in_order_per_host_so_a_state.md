---
type: "practice"
difficulty: "Deep"
---

# Your anomaly scores must arrive in order per host so a state-machine detector works correctly. How does keying by host_id guarantee this, and what breaks it when you add partitions?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Partitions and Ordering](<../../../../TOOLS/Apache_Kafka/Kafka_Core/Partitions_and_Ordering.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To ensure that anomaly scores arrive in order per host, we need to use Kafka's key-based partitioning mechanism effectively. Keying by host_id guarantees ordered delivery within each partition because Kafka maintains a consistent ordering of messages with the same key across all partitions. However, adding more partitions introduces complexity and potential issues. Let’s break this down step-by-step.

First, understand that in Kafka, when you produce messages with a specific key (in our case, host_id), these messages are assigned to the same partition based on the hash of the key. This ensures that all messages for a given host_id go to the same partition and thus arrive in order as they leave the producer.

**Comparison:**

| Partition | Messages (Keyed by host_id) |
|-----------|----------------------------|
| 0         | [h1, 0.8], [h1, 0.9]       |
| 1         | [h2, 0.3]                  |

**Code:**

```python
# Example Kafka producer code snippet with key-based partitioning
from kafka import KafkaProducer

producer = KafkaProducer(bootstrap_servers=['localhost:9092'],
                         value_serializer=lambda v: v.encode('utf-8'),
                         key_serializer=str.encode)

for host_id, anomaly_score in data:
    message = f"host_id={host_id},anomaly_score={anomaly_score}"
    producer.send(topic='anomaly_scores', key=host_id, value=message)
```

**Example input / output:**

```text
Input:
- Topic: anomaly_scores
- Data Points:
  - host_id = h1, anomaly_score = 0.8
  - host_id = h2, anomaly_score = 0.3
  - host_id = h1, anomaly_score = 0.9

Output (with key-based partitioning):
- Partition 1: [host_id=h1,anomaly_score=0.8, host_id=h1,anomaly_score=0.9]
- Partition 2: [host_id=h2,anomaly_score=0.3]

Explanation:
- Both h1 anomaly scores are in the same partition and maintain order.
- h2's score is in a different partition as expected due to key-based assignment.
```

**Data:**

```json
{
  "data": [
    {"host_id": "h1", "anomaly_score": 0.8},
    {"host_id": "h2", "anomaly_score": 0.3},
    {"host_id": "h1", "anomaly_score": 0.9}
  ]
}
```

**Flow:**

1. Yes
2. No
3. Producer sends message withhost_id key
4. Key matches existing partition?
5. Message goes to correctpartition
6. Partition state-machinedetector processes messagesin order
7. Correct processing of anomalyscores per host
8. New partition created for thishost_id
9. Message sent to new partition
10. Partition state-machinedetector processes messagesin order

