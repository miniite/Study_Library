---
type: "practice"
difficulty: "Deep"
---

# What is min.insync.replicas and how does it interact with acks=all?

**Difficulty:** Deep · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Durability with acks and ISR](<../../../../TOOLS/Apache_Kafka/Kafka_Core/Durability_with_acks_and_ISR.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Min.insync.replicas (minISR) in Apache Kafka is a configuration parameter that defines the minimum number of replicas within a partition that must acknowledge an event before it can be considered successfully written to the topic. It works closely with acks=all to ensure strong consistency and durability, making sure data integrity and reliability are maintained.

When acks=all is set, every write operation waits for acknowledgment from all in-sync replicas (ISR) before returning a success response to the producer. This setting ensures that no event is lost even if one or more replicas fail during the write process. However, this can introduce latency issues due to the additional round trips required for acknowledgments.

#### Triage

The key aspect of min.insync.replicas in interaction with acks=all revolves around ensuring fault tolerance and consistency while managing trade-offs between performance and reliability. The interviewer is probing your understanding of how these configurations impact Kafka's behavior, particularly in scenarios involving partition failures or network partitions.

#### Assumptions
We are discussing a Kafka cluster configured to use acks=all.
MinISR is set to a non-zero value.
There is at least one replica for each partition.

#### First Principles Solution

To solve the question, let's break it down step by step:

Understanding min.insync.replicas:

min.insync.replicas (minISR) is a configuration that guarantees data durability. It ensures that only after a sufficient number of in-sync replicas acknowledge a write operation will the event be considered successfully written to the topic.
For example, if minISR is set to 3 and there are 5 replicas for a partition, at least 3 out of these 5 must confirm receipt of the message.

Understanding acks=all:

When acks=all, every write operation waits for acknowledgment from all in-sync replicas before returning a success response.
This setting ensures strong consistency by making sure that all in-sync replicas have received and processed the event, reducing the risk of data loss.

Interaction Between minISR and acks=all:

Setting acks=all with minISR guarantees that only after the leader sends the message to all in-sync replicas (as determined by minISR) will it return a success response.
This means the producer waits for acknowledgments from all in-sync replicas, ensuring high reliability but potentially introducing latency.
Detailed Solution

To illustrate this interaction, let's consider a scenario where we have three replicas per partition and min.insync.replicas is set to 2. The configuration looks like this:

PLAINTEXT
Leader (Replica 1) -> ISR: Replica 1, Replica 2, Replica 3

If the producer sends a message with acks=all, it will wait for acknowledgments from all in-sync replicas before returning success.

If Replica 1 receives the message and forwards it to Replicas 2 and 3:
Acknowledgment from Replica 2 and Replica 3 are required.
Once both replicas acknowledge, the leader returns a success response to the producer.

#### Closing Summary

In summary, min.insync.replicas and acks=all work together to ensure that only after a sufficient number of in-sync replicas confirm receipt of an event will it be considered successfully written. This setup guarantees high reliability but can introduce latency due to additional round trips for acknowledgments. The trade-off is between strong consistency and performance, and the choice depends on the specific requirements of the application.

#### Common Mistakes/Follow-Ups

Mistake: Setting acks=all without considering the impact on write performance.

#### Follow-up
Discuss how to balance between high reliability and low latency using strategies like adjusting minISR or implementing retries with backoff.

Mistake: Ignoring partition failures when setting minISR.

#### Follow-up
Explain how to design a fault-tolerant Kafka setup that accounts for potential leader elections and partition rebalancing.

**Comparison:**

```
Config Parameter	Description
acks	Acknowledgment mode, can be set to all, leader, or none.
min.insync.replicas	Minimum number of in-sync replicas required for a write operation to succeed.
```

**Code:**

```java
// Example of setting up a producer with acks=all and min.insync.replicas in Java
Properties props = new Properties();
props.put("acks", "all"); // Ensure all replicas acknowledge the write
props.put("min.insync.replicas", "2"); // Set minimum number of in-sync replicas required
Producer<String, String> producer = new KafkaProducer<>(props);
```

**Example input / output:**

```text
Producer sends a message -> Leader (Replica 1) receives the message and forwards it to ISR.
Leader waits for acknowledgments from all in-sync replicas (Replica 2, Replica 3).
Both replicas acknowledge -> Leader returns success response to producer.

Scenario with failure of Replica 2:
Leader still needs acknowledgment from Replica 3 before returning success.
```

**Data:**

```yaml
# Sample configuration for a Kafka broker
broker:
  - node1:9092
  - node2:9092
  - node3:9092

topic:
  name: example-topic
  partitions: 3
  replication-factor: 3
  min.insync.replicas: 2
```

**Flow:**

1. Yes
2. Yes
3. No
4. Producer sends message
5. Leader receives and forwards
6. Replica 2 acknowledges?
7. Replica 3 acknowledges?
8. Leader returns success
9. Leader waits for Replica 3 toacknowledge

