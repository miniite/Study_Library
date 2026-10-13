---
type: "practice"
difficulty: "Intermediate"
---

# What is the difference between log retention by time and log compaction?

**Difficulty:** Intermediate · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Log Compaction](<../../../../TOOLS/Apache_Kafka/Kafka_Core/Log_Compaction.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In Apache Kafka, two key mechanisms control how logs are managed: log retention by time and log compaction. Log retention by time ensures that messages older than a certain duration are removed. Log compaction, on the other hand, retains exactly one message per unique key across all partitions of a topic.

To understand these concepts, let's first break down what each mechanism entails:

Log Retention by Time:

This feature guarantees that messages in Kafka topics will not be kept indefinitely.
It works by setting a retention period (e.g., 7 days). After this duration, older messages are automatically deleted from the log to free up storage space.
The key advantage is straightforward management and predictable resource usage since you know how long data is retained.

Log Compaction:

This mechanism ensures that for a given topic partition, only one message with the latest timestamp per unique key is stored.
If there are multiple messages with the same key in different partitions (due to replica synchronization or other factors), Kafka retains the most recent message based on its timestamp.
Log compaction can significantly reduce the size of data stored and improve efficiency for certain types of analytics.

Now, let's dive into a step-by-step explanation from first principles:

#### Triage
What is being Probed

The interviewer is probing your understanding of Kafka's core concepts related to log management. They want to know if you grasp the differences between these retention strategies and how they impact data storage and consumption patterns in Kafka.

#### Assumptions
We assume a basic understanding of Kafka topics, partitions, and key-value pairs.
The question doesn't specify specific scenarios; therefore, we need to cover general use cases.

#### First Principles Solution
Log Retention by Time
Configuration:
Defined via log.retention.hours (or similar) in the Kafka broker configuration.
Functionality:
All messages older than the retention period are deleted from the log, regardless of their key or value.
Use Case:
Suitable for applications where data needs to be retained for a fixed duration but can be discarded after that time.
Log Compaction
Configuration:
Enabled by setting log.cleanup.policy=compact in the Kafka topic configuration.
Functionality:
Keeps only one message per unique key, based on the latest timestamp across all partitions.
Reduces storage consumption and improves efficiency for certain queries that rely on maintaining a single most recent value.
Use Case:
Ideal for use cases requiring low-latency access to the latest data (e.g., real-time analytics, leaderboards).
Step-by-Step Reasoning

Retention by Time:

Set up a topic with a retention period of 7 days.
Kafka will retain messages until they reach this age and then delete them.
Useful for applications where historical data isn't needed beyond a certain point.

Log Compaction:

Enable log compaction on the desired topic.
Messages are retained based on key uniqueness, ensuring only one message per key is stored at any given time.
This can significantly reduce storage and improve query performance for applications like leaderboards or aggregations that require the most recent data.

#### Closing Summary
+ Common Mistakes/Follow-Ups

#### Closing Summary

Both log retention by time and log compaction serve crucial roles in managing Kafka logs but operate differently. Retention by time ensures predictable data retention, while log compaction optimizes storage and query efficiency by retaining only the latest message per key.

#### Common Mistakes/Follow-Ups
Misunderstanding Time-Based Retention: Failing to realize that messages are removed regardless of their value or importance.
Overlooking Key Uniqueness in Compaction: Not recognizing that log compaction retains exactly one message per unique key, which can lead to incorrect assumptions about data availability.

If I were asked a follow-up question, it might be around specific scenarios where these mechanisms could conflict or how they interact with other Kafka features such as offsets and replicas. For example:

How would you design a scenario that uses both retention by time and log compaction?
What are the implications of enabling both policies on the same topic?

**Comparison:**

| Mechanism     | Retention by Time          | Log Compaction         |
|---------------|----------------------------|------------------------|
| Definition    | Message age based deletion | Unique keys retention  |
| Key Factors   | `log.retention.hours`      | `log.cleanup.policy=compact` |
| Use Case      | Predictable data lifecycle | Efficient real-time queries |

**Code:**

```python
# Example configuration for Kafka topics
topic_config = {
    "retention.ms": 604800000,  # 7 days in milliseconds
    "cleanup.policy": "delete",  # No compaction by default
}
```

**Example input / output:**

```text
Input:
- Create a Kafka topic with retention policy of 7 days and no compaction.
- Produce messages to the topic with keys like 'user_1', 'user_2'.

Output:
- Messages will be retained for up to 7 days, after which they are deleted.

Compaction Example:
- Enable log compaction on the same topic.
- Produce multiple messages with the same key (e.g., 'user_1') in different partitions.

Output:
- Only the latest message per unique key ('user_1', 'user_2') will be retained.
```

**Data:**

```json
{
  "topic_name": "user_activity",
  "retention_policy": {
    "type": "time",
    "duration": 7,  // 7 days
    "cleanup_policy": "delete"
  }
}
```

**Flow:**

1. Log Retention by Time
2. Set retention period
3. Delete messages older thanthe period
4. Log Compaction
5. Enable compaction policy
6. Retain one message perunique key across partitions

