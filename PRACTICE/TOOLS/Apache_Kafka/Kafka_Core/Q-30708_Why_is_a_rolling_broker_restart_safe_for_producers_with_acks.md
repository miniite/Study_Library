---
type: "practice"
difficulty: "Deep"
---

# Why is a rolling broker restart safe for producers with acks=all, and what config makes clients ride through it?

**Difficulty:** Deep · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Durability with acks and ISR](<../../../../TOOLS/Apache_Kafka/Kafka_Core/Durability_with_acks_and_ISR.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Rolling broker restarts in Kafka can be safe for producers configured with acks=all, under specific conditions. This is because the producer's acknowledgment model ensures that all brokers acknowledge receipt of the message before considering the send operation successful. Here, we'll explore why this approach works and what Kafka configuration helps clients ride through these restarts seamlessly.

#### Triaging the Question

The core of the question involves understanding:

Producer Acknowledgment Model: Specifically, how acks=all impacts producer behavior.
Broker Restart Safety: Under what conditions rolling broker restarts are safe for producers using acks=all.
Client Resilience Configurations: Which Kafka client configurations ensure that clients can handle broker restarts without data loss.

#### First-Principles Solution

To understand the safety of rolling broker restarts, let's break down the acknowledgment model and client-side configurations.

Producer Acknowledgment Model (acks=all)

When a producer sends a message with acks=all, it means the send operation is considered successful only when:

The leader partition replica has stored the message in its log.
All in-sync replicas (ISR) acknowledge receipt of the message.

This ensures that even if one broker fails, there are enough replicas to guarantee durability.

Broker Restart Safety

A rolling broker restart should not impact producers with acks=all because:

Leader Election: When a broker restarts, the leader for each partition is elected from the ISR. Since acks=all guarantees all in-sync replicas acknowledge receipt, messages are stored in the leader’s log before the producer considers success.
Replication Factor: As long as the replication factor is greater than one (e.g., 3 or more), there will always be enough replicas to ensure message durability even if a broker fails.
Client Resilience Configurations

To ensure that clients can handle rolling restarts without data loss:

Session Timeout and Retry Mechanisms: Configure session timeouts and retry intervals to allow for transient failures.
Metadata Refresh Interval: Ensure the client metadata refresh interval is appropriate, allowing it to detect broker changes during a restart.

#### Closing Summary
and Common Mistakes

Rolling broker restarts are safe for producers with acks=all because the acknowledgment model ensures durability across multiple replicas. However, proper client configurations, such as session timeouts and retry intervals, are crucial to maintain resilience. Common mistakes include:

Ignoring session timeout settings, leading to unresponsive clients.
Underestimating the impact of metadata refresh intervals on recovery times.
Further Considerations
Failure Scenarios: Understand how a broker failure during a restart impacts ongoing sends and retries.
Best Practices: Implement best practices such as using topic-based routing for more fault-tolerant systems.

**Comparison:**

| Configuration | Impact                           |
|---------------|----------------------------------|
| `acks=all`    | Ensures all replicas acknowledge receipt.  |
| Session Timeout | Configures retry behavior after broker failures. |
| Metadata Refresh Interval | Detects broker changes during restarts. |

**Code:**

```python
# Example producer configuration with acks=all
producer_config = {
    'bootstrap.servers': 'broker1:9092,broker2:9092',
    'acks': 'all',  # Ensure all in-sync replicas acknowledge receipt
    'retries': 5,   # Number of retries before failing
}
```

**Example input / output:**

```text
Input:
- Producer sends message to topic with `acks=all`.

Output:
- Message is written to the leader's log.
- ISR acknowledges receipt of the message.
- Producer receives acknowledgment, considering send successful.

Dry-run Request:
- What happens if a broker restarts while a producer is sending messages?

Dry-run Response:
- Messages are stored in the leader's log before ACK is sent.
- Upon restart, new leaders are elected from ISR, ensuring durability.
```

**Data:**

```json
{
  "brokers": [
    {"id": 1, "host": "broker1", "state": "active"},
    {"id": 2, "host": "broker2", "state": "active"}
  ],
  "partitions": {
    "topic1": [
      {"id": 0, "leader": 1, "replicas": [1, 2], "isr": [1]}
    ]
  }
}
```

**Flow:**

1. Yes
2. No
3. Producer sends message
4. Broker Restart?
5. Message stored in leader's log
6. Leader election occurs, ISRmaintains durability
7. Message processed normally

