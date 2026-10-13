---
type: "practice"
difficulty: "Deep"
---

# How would you detect and safely recover from a stuck consumer that holds a partition assignment but has silently stopped making progress mid-poll?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Consumer Groups and Rebalancing](<../../../../TOOLS/Apache_Kafka/Kafka_Core/Consumer_Groups_and_Rebalancing.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the issue of detecting and safely recovering from a stuck Kafka consumer, we need to understand how consumers operate in Kafka and what signals or mechanisms can be leveraged for detection and recovery. This problem is crucial as it directly impacts the reliability and availability of our distributed system.

Kafka consumers periodically poll for messages and process them. A stuck consumer, which has obtained a partition assignment but stopped making progress during a poll() call, poses a risk to the overall system's integrity and data processing workflow. We need to design a mechanism that can detect this state and ensure safe recovery without compromising other consumer groups or partitions.

#### Triage
What is Being Probed

The question asks for a method to detect a stuck consumer and recover from it safely. The interviewer is likely probing your understanding of Kafka's architecture, the behavior of consumers during poll() calls, and how you can implement robust monitoring and recovery strategies without introducing unnecessary complexity or performance overhead.

#### Assumptions
We have multiple partitions in our topic.
There are multiple consumer groups consuming from these partitions.
The stuck state is detected when a consumer fails to process messages within a certain time frame relative to the poll interval.

#### First Principles Solution

Polling and Heartbeating:

#### Cons
umers should periodically send heartbeats to the Kafka cluster indicating their presence and progress on each partition they are assigned.
These heartbeats can be timestamped and logged to ensure reliable tracking of consumer activity.

Heartbeat Monitoring:

A monitoring service or a broker-side mechanism should track these heartbeats and detect anomalies, such as missing heartbeats within a certain window.
Configurable thresholds can define how long a consumer is allowed to be idle before being considered stuck.

#### Cons
umer Group Coordinator Elections:

When a stuck consumer is detected, the coordinator for that consumer group should attempt to reassigned the partitions it was previously holding back to other healthy consumers.
This reassignment ensures that no messages are lost and the system remains resilient.

Graceful Recovery:

During recovery, the new consumer taking over the partition should be allowed a grace period to catch up with any pending messages before fully resuming normal operations.
This grace period can help prevent abrupt restarts that might disrupt the overall processing flow.

Logging and Alerting:

Detailed logs and alert mechanisms should be in place to notify operators of stuck consumers, allowing for manual intervention if necessary.
Alerts can also trigger automated recovery procedures if configured.

#### Closing Summary

In summary, detecting a stuck Kafka consumer involves implementing heartbeating, heartbeat monitoring, reassignment of partitions, graceful recovery, and robust logging and alerting. These steps ensure that the system remains resilient to partial failures while minimizing disruptions to data processing workflows.

#### Common Mistakes/Follow-Ups
Ignoring Graceful Recovery: Failing to allow a grace period for recovering consumers can lead to abrupt restarts and loss of data consistency.
Overly Complicated Heartbeat Mechanisms: Using overly complex heartbeat mechanisms can introduce unnecessary overhead and reduce performance.
Insufficient Logging and Alerting: Without proper logging and alerting, issues might go unnoticed until they affect the entire system.

Would you like a deeper dive into any part of this solution or how to implement these features in practice?

**Comparison:**

| Partition | Consumer Group | Last Heartbeat | Status |
|-----------|---------------|----------------|--------|
| 0         | group1        | -              | Stuck  |
| 1         | group2        | 3 minutes ago  | Healthy|

**Code:**

```python
# Example heartbeat mechanism using Kafka producer
from kafka import KafkaProducer

producer = KafkaProducer(bootstrap_servers=['localhost:9092'])

def send_heartbeat(partition, timestamp):
    producer.send('heartbeats', key=partition, value=str(timestamp).encode())
```

**Example input / output:**

```text
Input:
- A consumer group with a stuck partition (e.g., partition 0 on broker-1)

Output:
- Heartbeat sent every 5 seconds for each partition assignment.
- Monitor detects missing heartbeat after 30 seconds and triggers reassignment of partition 0 to another healthy consumer.
- New consumer starts catching up with pending messages within a grace period before resuming normal operations.
```

**Data:**

```json
{
  "topic": "user_events",
  "partitions": [
    {"id": 0, "leader": "broker-1"},
    {"id": 1, "leader": "broker-2"}
  ]
}
```

**Flow:**

1. Yes
2. No
3. Consumer Polls for Messages
4. Heartbeat Sent?
5. Monitor Detects Progress
6. Reassign Partition to NewConsumer
7. Grace Period for Catch-Up
8. Resume Normal Operations
9. Notify Operators

