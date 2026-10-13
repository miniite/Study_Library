---
type: "practice"
difficulty: "Intermediate"
---

# You inherit a pipeline with no monitoring. What are the first three things you would add to know it is healthy?

**Difficulty:** Intermediate · **Levels:** Architect/Principal, Fresher, SDE-1/2, SDE-2/3 · [↑ Pipeline Lag Monitoring](<../../../../TOOLS/Apache_Kafka/Kafka_Core/Pipeline_Lag_Monitoring.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When inheriting a Kafka-based data pipeline without any monitoring, the initial focus should be on understanding the current state and ensuring basic health checks are in place before delving into more complex metrics. The first three things I would add to ensure the pipeline is healthy are:

Producer Latency: Measure how long it takes for messages to be produced.

#### Cons
umer Lag: Monitor how much data has been consumed compared to what was produced, ensuring no backlogs or stalls.
Topic Partitions Health: Check if all partitions in a topic are being evenly consumed and no single partition is lagging behind.

These steps will provide foundational insights into the pipeline's performance and alert us to potential issues early on.

**Comparison:**

```
Topic	Partition	Lag
topic1	0	5
	1	3
topic2	0	8
	1	7
```

**Code:**

```python
# Example of how to monitor producer latency using KafkaConsumer statistics
from kafka import KafkaProducer

producer = KafkaProducer(bootstrap_servers='localhost:9092')
for _ in range(10):
    start_time = time.time()
    future = producer.send('test_topic', b'key')
    record_metadata = future.get(timeout=10)
    latency = (time.time() - start_time) * 1000
    print(f"Latency: {latency} ms")
```

**Example input / output:**

```text
# Sample input and output for monitoring consumer lag
import json

consumer_lag = {
    "topic1": {"partition0": 5, "partition1": 3},
    "topic2": {"partition0": 8, "partition1": 7}
}

print("Consumer Lag:", json.dumps(consumer_lag))
```

**Flow:**

1. Yes
2. No
3. Yes
4. No
5. Produce Message
6. Send to Kafka
7. Is Latency OK?
8. Move to Next Step
9. Alert on High Latency andInvestigate
10. Consume Message
11. Subscribe to Topic
12. Is Lag OK?
13. Continue Processing
14. Identify Backlogged Partitionsand Alert

