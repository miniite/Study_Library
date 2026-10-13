---
type: "practice"
difficulty: "Deep"
---

# A rolling broker upgrade caused producers to see transient NotLeaderForPartition errors. Is this expected, and what client settings make it invisible to your service?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Leader Election and Unclean Failover](<../../../../TOOLS/Apache_Kafka/Kafka_Core/Leader_Election_and_Unclean_Failover.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Rolling upgrades of Apache Kafka brokers can introduce temporary partition leader changes that might cause producers to encounter NotLeaderForPartition errors. This is expected behavior because during an upgrade, broker leadership reassignment happens as part of the high availability (HA) mechanism, and this can lead to partitions becoming temporarily unleadered or having a different leader.

To understand why these errors are seen and how we can make them invisible to our service, let's break down the key concepts and considerations. We'll start by defining what NotLeaderForPartition is, then discuss the impact of rolling upgrades on brokers, and finally explore client settings that mitigate visibility of these errors.

**Comparison:**

```
Client Setting	Description	Impact on NotLeaderForPartition Errors
retries	Number of times to retry sending a message.	Reduces the frequency of transient errors seen by producers.
retry_backoff_ms	Time between retries, in milliseconds.	Helps in managing backpressure and resource utilization during upgrades.
request_timeout_ms	Timeout for request, in milliseconds.	Ensures that requests do not hang indefinitely due to network issues or broker state changes.
max_in_flight_requests_per_connection	Maximum number of unacknowledged requests per connection.	Limits the number of messages being sent concurrently and helps manage load during upgrades.
```

**Code:**

```python
# Example Python code snippet to illustrate handling NotLeaderForPartition
from kafka import KafkaProducer

producer = KafkaProducer(bootstrap_servers=['kafka1:9092', 'kafka2:9092'])
try:
    # Attempt to send a message
    producer.send('my-topic', b'Hello, World!')
except Exception as e:
    if isinstance(e, NotLeaderForPartitionError):
        print("NotLeaderForPartition error caught")
```

**Example input / output:**

```text
# Example input-output dry-run
producer = KafkaProducer(bootstrap_servers=['kafka1:9092', 'kafka2:9092'])
producer.send('my-topic', b'Hello, World!')
if producer.future().get(timeout=10).exception() and isinstance(
        producer.future().exception(), NotLeaderForPartitionError
):
    print("NotLeaderForPartition error handled")
```

**Data:**

```json
{
  "bootstrap_servers": ["kafka1:9092", "kafka2:9092"],
  "topic": "my-topic"
}
```

**Flow:**

1. Yes
2. No
3. Produce Message
4. Is Leader Known?
5. Send Request to Leader
6. Receive Acknowledgment orError
7. HandleNotLeaderForPartitionError
8. Retry with Increased Delay

