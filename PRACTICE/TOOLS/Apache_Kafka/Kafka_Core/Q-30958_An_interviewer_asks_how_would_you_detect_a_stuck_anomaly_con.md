---
type: "practice"
difficulty: "Deep"
---

# An interviewer asks: how would you detect a stuck anomaly consumer that holds a partition assignment but silently stopped scoring mid-poll?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Consumer Lag Monitoring](<../../../../TOOLS/Apache_Kafka/Kafka_Core/Consumer_Lag_Monitoring.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Summary:

To detect a stuck anomaly consumer in Kafka, we need to monitor consumer behavior by tracking its offsets and timing. The key idea is to leverage Kafka's built-in metrics and configuration settings to set up alerts when a consumer falls behind or stops processing messages.

#### Triaging the Question

The interviewer is probing your ability to identify issues with Kafka consumers that might not be immediately obvious, such as a stuck consumer that is holding onto its partition assignment but isn't processing any new messages. This can happen if the consumer logic has bugs or external factors are preventing it from scoring (processing) mid-poll.

#### First-Principles Solution

#### Cons
umer Offsets and Timing:

The primary metric to monitor here is the offset lag between the consumer's committed offsets and the current leader's offsets in Kafka.
If a consumer stops processing, its committed offsets will remain unchanged while new messages are being produced.

Kafka Consumer Group Metrics:

Use kafka-consumer-groups tool or Kafka Manager to check group metrics like lag (the difference between the latest offset and the member's assigned offset).
Set up a monitoring dashboard that tracks these metrics over time. If the lag suddenly increases, it might indicate an anomaly.

Configuration Settings:

Configure the session.timeout.ms property in Kafka to ensure that if a consumer doesn't send heartbeats within this interval, the group coordinator will consider it failed and rebalance.
Set up heartbeat.interval.ms to control how frequently the consumer sends heartbeats to keep its membership alive.

Custom Monitoring Scripts:

Write custom scripts or use Kafka's built-in tools to monitor consumer health. These can log messages when a consumer's lag exceeds a certain threshold or when no new offsets are committed for an extended period.
For example, you could write a script that checks the last polled offset and compares it with the current offset of the leader.

Handling Silent Stops:

If a consumer is stuck processing a message (silent stop), ensure there's some backoff mechanism in place to retry or fail over to another instance.
Use Kafka's max.poll.interval.ms configuration to limit how long a consumer can be idle before it gets marked as failed.

Rebalance Handling:

Implement custom rebalance logic to handle scenarios where the consumer group might need to reassign partitions due to the stuck consumer.
Ensure that your application has failover mechanisms in place, such as retrying with another instance or handling the partition being reassigned to a different consumer.

#### Closing Summary

In summary, detecting a stuck anomaly consumer involves monitoring offsets and timing, leveraging Kafka's built-in tools for metrics, setting up appropriate configuration settings, and implementing custom monitoring scripts. By doing so, you can proactively identify and handle issues before they cause serious disruptions in your system.

#### Common Mistakes/Follow-Ups
Mistake: Not setting up proper heartbeat mechanisms or session.timeout.ms to detect inactive consumers.
Mistake: Failing to monitor offsets lag comprehensively using tools like Kafka Manager.

#### Follow-Up
Discussing how to integrate custom monitoring scripts with your CI/CD pipeline for automated alerting and handling.

**Comparison:**

```
Topic	Partition	Offset Lags
topic1	0	5
topic1	1	8
topic2	0	3
topic2	1	7
```

**Code:**

```python
# Example of a simple script to check consumer health
import json

def get_consumer_group_offsets(kafka_consumer, group_id):
    command = f"KAFKA_CLIENT_ID=consumer-id KAFKA_GROUP={group_id} kafka-consumer-groups.sh --bootstrap-server <kafka_broker> --describe"
    output = subprocess.check_output(command, shell=True)
    topic_partitions = json.loads(output.decode('utf-8'))
    return {topic: partition['lag'] for topic, partitions in topic_partitions.items() for partition in partitions if partition['consumer_id'].startswith(group_id)}
```

**Example input / output:**

```

Input:

Kafka broker address: localhost:9092
Consumer group ID: my-consumer-group

Output:

Lag for topic1 partition 0: 5
Lag for topic1 partition 1: 8
SHELL
Copy
$ kafka-consumer-groups.sh --bootstrap-server localhost:9092 --describe --group my-consumer-group
```

**Data:**

```json
{
  "group_offsets": {
    "topic1": [0, 5],
    "topic2": [3, 8]
  }
}
```

**Flow:**

1. Yes
2. No
3. Start
4. Is consumer lagging?
5. Log alert and trigger rebalance
6. Continue monitoring
7. Handle partition reassignmentor failover

