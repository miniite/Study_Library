---
type: "practice"
difficulty: "Deep"
---

# What happens to in-flight messages if a broker crashes mid-produce?

**Difficulty:** Deep · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Idempotent and Transactional Producers](<../../../../TOOLS/Apache_Kafka/Kafka_Core/Idempotent_and_Transactional_Producers.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When a Kafka broker crashes during message production, several things can happen depending on how the producer and broker handle transactions. This scenario is critical because it involves durability, consistency, and the reliability of messages in a distributed system.

#### Triage

The core of the question revolves around understanding how Kafka's transactional semantics ensure data integrity when a broker fails mid-produce. The key points to consider include:

Producer State: How does the producer know which messages were successfully written before the crash?
Transaction Logs: Do these logs help in recovering lost messages or ensuring that only committed messages are sent?
Recovery Mechanisms: What happens after a broker recovers from a crash?

#### First-Principles Solution

Kafka's design handles such scenarios through its transactional semantics and log compaction features. Here’s the step-by-step reasoning:

Producer State Tracking:

Producers maintain their own state in the form of offsets, which track which messages have been sent and acknowledged by brokers.
If a producer crashes mid-produce, it may not be aware of what was actually written to the broker.

Log Compaction:

Kafka can configure log compaction settings that ensure only the latest version of each message is retained in the topic logs.
This helps in recovering from partial writes by ensuring that any in-flight messages are lost if they weren't committed before the crash.

Producer Retry Mechanism:

Producers implement a retry mechanism to resend unacknowledged or partially written messages after the broker recovers.
The producer can use transactional semantics, where it commits offsets only when all intended messages are successfully produced and acknowledged by brokers.

Partition Replication:

Kafka partitions are replicated across multiple brokers (replicas).
If a partition leader fails, another replica takes over leadership, ensuring that messages are not lost.

Log Compaction and Transaction Logs:

For transactional producers, transactions are logged in special topic __transaction_state to ensure consistency.
In-flight messages are marked as part of an ongoing transaction until the producer commits or aborts it.

#### Closing Summary

In summary, if a broker crashes mid-produce, Kafka's mechanisms ensure that:

Unacknowledged messages may be lost depending on log compaction settings and producer state tracking.
Producers can handle retries to recover from such failures.
Replication across partitions helps in maintaining data durability.
Transactional semantics provide a way to commit or abort writes consistently.

#### Common Mistakes/Follow-Ups
Overreliance on Acknowledgments: Failing to understand that even with multiple acknowledgments, unacknowledged messages can still be lost if the broker crashes.
Ignoring Replication: Assuming that single-node failures are sufficient for handling all potential issues without considering partition replication.
Misconfiguring Log Compaction: Misunderstanding how log compaction works and its impact on in-flight transactions.

**Comparison:**

```
Kafka Broker State	Producer Offset	Log Compaction Setting	Outcome
Crashed	Unacked	Enabled	Message loss possible but recoverable
Crashed	Unacked	Disabled	Message lost, no recovery
```

**Code:**

```python
# Example of a Kafka producer using transactional semantics
from kafka import KafkaProducer, KafkaAdminClient

producer = KafkaProducer(bootstrap_servers='localhost:9092',
                         transactional_id='my-transaction-id')

try:
    with producer.transaction():
        # Produce messages within the transaction block
        producer.send('topic-name', b'message1')
        producer.send('topic-name', b'message2')
finally:
    producer.flush()
    producer.close()
```

**Example input / output:**

```text
Input:
Producer sends messages to 'topic-name' while broker-1 crashes mid-produce.

Output:
- Messages sent before the crash might be lost depending on log compaction and producer state.
- Retries will ensure that unacknowledged messages are resent after recovery.
- Replicas will take over leadership, ensuring data is not lost.
```

**Data:**

```json
{
  "topics": [
    {
      "name": "topic-name",
      "partitions": 3,
      "replication-factor": 3,
      "logs": [
        {"log-id": "0", "leader": "broker-1"},
        {"log-id": "1", "leader": "broker-2"},
        {"log-id": "2", "leader": "broker-3"}
      ]
    }
  ],
  "brokers": [
    {"id": 1, "host": "localhost", "port": 9092}
  ]
}
```

**Flow:**

1. Producer Sends Messages
2. Broker Crashes
3. Produce Failed
4. Retry on Broker Recovery
5. Message Committed
6. Partition Leader Replicates
7. Unacked Message Lost
8. Recovery Mechanism Activated

