---
type: "practice"
difficulty: "Basic"
---

# How does Kafka provide durability so that telemetry events are not lost if a consumer is temporarily down?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ End to End Exactly Once](<../../../../03_Big_Data_and_Streaming/Distributed_Processing/Big_Data_Processing/End_to_End_Exactly_Once.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To ensure telemetry events are not lost when a Kafka consumer is temporarily down, Kafka provides several mechanisms to maintain data durability and availability. TL;DR: Kafka achieves this through its publish-subscribe model, persistent storage, and the ability to recover from failures.

Kafka operates on the principle of a publish-subscribe pattern where producers send messages (telemetry events) to topics, and consumers subscribe to these topics to process the messages. The key to understanding how Kafka ensures durability lies in its internal architecture: it stores all data persistently, uses a log compaction strategy for reliability, and handles consumer failures through offset management.

First, let's break down what the question is really asking:

What are we trying to achieve? Ensure that telemetry events are not lost if a consumer goes down temporarily.
What does the interviewer want us to explain? The mechanisms Kafka uses to maintain data integrity and availability in such scenarios.

#### Assumptions
We can assume basic familiarity with Kafka's key concepts like topics, brokers, producers, consumers, and offsets.

Now, let's reason from first principles:

Persistent Storage:

Each message published to a Kafka topic is stored on disk by the Kafka broker. This ensures that even if one or more brokers fail, messages will still be available.
Kafka supports multi-replica storage across multiple brokers. By default, each message is replicated to multiple brokers to ensure high availability and durability.

Topic Partitioning:

Topics in Kafka are divided into partitions. Each partition can have multiple replicas (configurable via replication factor).
When a consumer goes down, the consumer group manager maintains a record of which messages the consumers have processed using offsets.
If a consumer crashes and then recovers later, it can resume processing from where it left off by fetching the last committed offset.

Offset Management:

Kafka keeps track of each message’s consumption status using offsets. Offsets are stored in the __consumer_offsets topic managed by Kafka.
When a consumer restarts, it can retrieve its previously committed offsets and resume processing from that point.
If multiple consumers subscribe to a topic, they will share the load across partitions, ensuring that no messages are lost.

Log Compaction (Optional but Recommended):

For certain types of telemetry events where older data may not be needed immediately (e.g., logs), Kafka provides log compaction.
With log compaction enabled, Kafka retains only the latest value for each unique key in a topic, reducing storage requirements and ensuring that even if multiple messages with the same key are produced, only the last one is retained.

In summary, when a consumer goes down temporarily:

The messages remain stored on disk across multiple brokers.
Upon recovery, the consumer can resume processing from its last committed offset.
If log compaction is enabled, older redundant data can be discarded to optimize storage usage and ensure that no valuable data is lost due to temporary failures.

