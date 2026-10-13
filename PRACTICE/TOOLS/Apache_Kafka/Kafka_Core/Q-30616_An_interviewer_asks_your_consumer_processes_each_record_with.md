---
type: "practice"
difficulty: "Deep"
---

# An interviewer asks: your consumer processes each record with a 30-second external call and you keep getting kicked out of the group. What single setting is likely misconfigured?

**Difficulty:** Deep · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Pipeline Lag Monitoring](<../../../../TOOLS/Apache_Kafka/Kafka_Core/Pipeline_Lag_Monitoring.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When dealing with Apache Kafka, if a consumer keeps being kicked out of its group due to processing each record involving a 30-second external call, it's crucial to understand that this issue stems from how Kafka manages session timeouts and rebalancing. The key setting involved here is the session.timeout.ms property. Let’s break down the problem and solution step-by-step.

Triaging the Problem

The interviewer is probing your understanding of Kafka consumer group dynamics and the role of configuration settings in maintaining stable and efficient consumer operations. Specifically, the issue of being kicked out of a consumer group usually indicates that the consumer failed to send heartbeats within the session timeout period, which triggers a rebalance and potentially leads to other consumers taking over the partitions.

Key Assumptions
The external call is synchronous and time-consuming.
The consumer is using auto-offset committing.
Kafka version and configuration are standard unless otherwise specified.

#### First Principles Solution

Understanding the Session Timeout:

session.timeout.ms is a crucial setting that determines how long a consumer has to send heartbeats before being marked as failed by the coordinator in the group.

Heartbeat Mechanics:

#### Cons
umers send periodic heartbeats (every heartbeat.interval.ms) to the Kafka broker, which keeps them alive within the session timeout period.

Calculating Heartbeat Interval:

The heartbeat interval is typically half of the session timeout value: heartbeat.interval.ms = session.timeout.ms / 2.

Rebalance Mechanism:

If a consumer fails to send heartbeats for more than the session timeout, it will be removed from the group and rebalanced.

Given that each record processing involves a 30-second external call, we need to ensure this time is well within the heartbeat interval plus some buffer. Let's calculate:

Heartbeat Interval: [ \text{heartbeat.interval.ms} = \frac{\text{session.timeout.ms}}{2} ]

To avoid being kicked out, session.timeout.ms should be greater than or equal to twice the processing time (30 seconds).

#### Solution

The single setting likely misconfigured is session.timeout.ms. Setting it too low can lead to frequent rebalances and group instability.

To resolve this:

Increase session.timeout.ms to a value that covers the 30-second external call plus some buffer time (e.g., 60000 ms = 1 minute).

#### Closing Summary

By adjusting the session.timeout.ms setting, you can ensure that consumers have enough time to process each record without being forcibly removed from the group due to untimely heartbeats.

#### Common Mistakes
and Follow-Ups

Mistake: Assuming the issue is related to other settings like max.poll.interval.ms.

#### Follow-Up
Understanding the difference between heartbeat interval and session timeout, and how they interact.

Advanced Question: How does this setting impact the overall throughput of your consumer group under different load conditions?

**Comparison:**

```
Setting	Default Value	Purpose
session.timeout.ms	45000	Time before consumer is marked as failed if no heartbeat is received.
heartbeat.interval.ms	15000	Frequency of sending heartbeats to the coordinator.
```

**Code:**

```python
# Example configuration in Kafka consumer properties
props = {
    'bootstrap.servers': 'kafka-broker1:9092,kafka-broker2:9092',
    'group.id': 'my-consumer-group',
    'session.timeout.ms': 60000,  # Adjusted for a 30-second external call
    'heartbeat.interval.ms': 30000,
    ...
}
```

