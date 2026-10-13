---
type: "practice"
difficulty: "Expert"
---

# Defend your choice of min.insync.replicas and acks settings for a critical alerting topic where losing events is worse than a few extra ms of latency.

**Difficulty:** Expert · **Levels:** SDE-1/2, SDE-2/3 · [↑ End to End Exactly Once](<../../../../03_Big_Data_and_Streaming/Distributed_Processing/Big_Data_Processing/End_to_End_Exactly_Once.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address this question, we need to understand that min.insync.replicas and acks are crucial parameters in Kafka's configuration to ensure fault tolerance and data availability, especially for a critical alerting topic where losing events can have severe consequences. I will justify why setting these configurations appropriately is essential, explain the trade-offs involved, and provide a step-by-step reasoning from first principles.

#### Triage
What Are We Probing?

The question is probing our understanding of Kafka's fault tolerance mechanisms and how to configure them effectively for mission-critical topics. It also tests our ability to balance between data durability and latency in real-time systems. The assumptions are that we have a critical alerting topic where losing events (missed alerts) can lead to severe consequences, whereas a few extra milliseconds of latency is acceptable.

Reasoning from First Principles

Understanding min.insync.replicas:

min.insync.replicas specifies the minimum number of in-sync replicas required for a partition to be considered safe.
Setting this value too high increases durability but can lead to higher latency and resource usage.

Understanding acks:

acks controls how many acknowledgments (confirms) the producer receives before considering a write operation as successful.
Setting it to all ensures that all in-sync replicas receive the message, enhancing data durability but increasing latency.

#### Trade-offs
and Configuration Choices:

For a critical alerting topic where losing events is worse than having slight increases in latency, we need to prioritize data durability.
We would set both parameters appropriately: min.insync.replicas to ensure enough replicas are available for fault tolerance, and acks to confirm that all necessary replicas have the message.
Step-by-Step Solution

Determine min.insync.replicas:

For a critical alerting topic, let's set min.insync.replicas = 2.
This ensures that even if one replica goes down, there is still another copy available for data retrieval.
We can increase this to 3 or more depending on the number of partitions and the expected failure rate.

Set acks:

For high durability, set acks = "all".
This ensures that all in-sync replicas acknowledge receipt of the message before the producer considers it successful.
Alternatively, we can use acks = 1 for slightly lower latency but still good fault tolerance.

#### Closing Summary
and Common Mistakes/Follow-Ups

In summary, setting min.insync.replicas to at least 2 ensures that there are enough replicas available even if one fails. Setting acks = "all" guarantees that all necessary replicas receive the message, enhancing data durability but increasing latency slightly.

#### Common Mistakes
and Follow-Ups:

Mistake: Ignoring min.insync.replicas can lead to data loss in case of node failures.

#### Follow-Up
Discuss potential scaling trade-offs and how to dynamically adjust these settings based on the number of replicas and failure rates.
Mistake: Setting acks = "all" without considering the impact on latency, especially in real-time systems.

