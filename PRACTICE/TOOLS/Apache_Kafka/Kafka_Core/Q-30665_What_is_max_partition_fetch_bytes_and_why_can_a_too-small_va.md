---
type: "practice"
difficulty: "Deep"
---

# What is max.partition.fetch.bytes and why can a too-small value stall a consumer on a topic with large messages?

**Difficulty:** Deep · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Consumer Tuning per Workload](<../../../../TOOLS/Apache_Kafka/Kafka_Core/Consumer_Tuning_per_Workload.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Max.partition.fetch.bytes in Apache Kafka controls the maximum amount of data (in bytes) that a consumer group can fetch from a single partition during each poll() call. A too-small value can indeed stall a consumer on a topic with large messages, as it might need to make multiple fetch requests to fully consume a message. This scenario highlights an important trade-off in Kafka's design and the importance of setting this parameter appropriately.

To understand why a small max.partition.fetch.bytes value can lead to stalling, let's start from first principles.

**Comparison:**

```
Partition	Message Size (MB)	Fetch Bytes Setting (MB)	Fetch Requests Required
P1	2	1	2
P2	5	1.5	4
P3	8	2.5	3
```

**Code:**

```python
# Example consumer code snippet for illustration
from kafka import KafkaConsumer

consumer = KafkaConsumer('my-topic', bootstrap_servers=['localhost:9092'],
                         max_partition_fetch_bytes=1048576)  # 1MB as an example
```

**Example input / output:**

```text
Given a topic with large messages and a max.partition.fetch.bytes setting of 1 MB, the consumer will need to make multiple fetch requests to fully consume one message. If this results in more than one partition fetch request per poll(), it can lead to stalling.
```

**Data:**

```json
{
  "message_size": "large",
  "partition_fetch_limit": "small"
}
```

**Flow:**

1. No
2. Yes
3. Consumer poll()
4. Message Size >max.partition.fetch.bytes?
5. Fetch smaller chunk
6. Fully fetch message and end
7. Poll for next messages

