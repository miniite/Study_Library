---
type: "practice"
difficulty: "Expert"
---

# Adjudicate a cross-team dispute: the fraud team wants exactly-once, the analytics team wants max throughput, both consume the same topic. Design a topology that satisfies both without duplicating the ingest infrastructure.

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Dedup Contracts](<../../../../03_Big_Data_and_Streaming/Streaming_and_Messaging/Delivery_Semantics/Dedup_Contracts.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To design a topology that satisfies both the fraud team's requirement of exactly-once delivery and the analytics team's need for maximum throughput, we must understand the underlying message semantics and leverage Kafka's distributed streams framework effectively. The key challenge is to ensure that messages are processed at least once by each consumer but only once in total across all consumers.

#### Triage
Understanding Requirements
Fraud Team Requirement: Exactly-once delivery ensures no duplicate or missed fraud alerts, critical for compliance.
Analytics Team Requirement: Max throughput means high data processing speed and volume to support real-time analytics.

#### First-Principles Solution

We can achieve this by designing a topology that uses Kafka Streams for at-least-once processing with checkpointing, combined with Kafka Connect for exactly-once delivery through transactional semantics. This approach leverages Kafka’s distributed stateful stream-processing capabilities.

Step-by-Step Solution:
Topic Design: Use the same topic for both teams to minimize complexity.
Exactly-once Semantics: Enable transactional semantics in Kafka using ProducerTransactionState and ConsumerRecordMetadata.
At-least-once Processing: Utilize Kafka Streams to process messages at least once by each consumer but ensure global uniqueness.
Detailed Workflow:
Producer Side:
Use a Kafka producer with transactional semantics enabled.
JAVA
Properties props = new Properties();
props.put("bootstrap.servers", "localhost:9092");
props.put("acks", "all"); // Ensure all replicas receive the message
props.put("transaction.timeout.ms", 60000);
KafkaProducer<String, String> producer = new KafkaProducer<>(props);
Kafka Connect for Exactly-once:
Use Kafka Connect to manage transactions at a lower level.
Kafka Streams for At-least-once Processing:
Create a KafkaStreams instance with a global state store and stateful processing topology.
Code Example:
JAVA
Properties streamsProps = new Properties();
streamsProps.put(StreamsConfig.APPLICATION_ID_CONFIG, "at-most-once-processing-topology");
streamsProps.put(StreamsConfig.BOOTSTRAP_SERVERS_CONFIG, "localhost:9092");
streamsProps.put(ProducerConfig.TRANSACTIONAL_ID_CONFIG, "transactional-id");

StreamsBuilder builder = new StreamsBuilder();

KStream<String, String> sourceTopic = builder.stream("common-topic");
sourceTopic.groupByKey()
           .reduce((value1, value2) -> value1); // Ensure unique processing

// Add additional state stores and processors as needed
KTable<Bytes, Long> stateStore = ...;

StreamsConfig streamsConfig = new StreamsConfig(streamsProps);
final KafkaStreams streams = new KafkaStreams(builder.build(), streamsConfig);
streams.start();
Failure Modes:
Message Loss: If a producer transaction fails, messages may be lost.
Duplicate Processing: Ensuring only one consumer processes each message is critical to avoid duplication.

#### Closing Summary

By leveraging Kafka's built-in exactly-once processing semantics and stateful stream-processing capabilities, we can design a topology that satisfies both the fraud team’s requirement of exactly-once delivery and the analytics team’s need for maximum throughput. This approach ensures data integrity while maximizing performance.

#### Common Mistakes/Follow-ups
Ignoring Transactional Semantics: Not enabling transactional semantics in Kafka producers or consumers.
Misunderstanding State Management: Failing to correctly manage state stores and transactions.
Incorrect Topology Design: Not properly configuring the stream topology for at-least-once processing.

**Comparison:**

| Partition | Consumer Group A | Consumer Group B |
|-----------|------------------|------------------|
| 0         | Yes              | No               |
| 1         | Yes              | No               |
| 2         | Yes              | No               |

**Code:**

```java
// Example producer configuration
Properties props = new Properties();
props.put("bootstrap.servers", "localhost:9092");
props.put("acks", "all"); // Ensure all replicas receive the message
props.put("transaction.timeout.ms", 60000);
KafkaProducer<String, String> producer = new KafkaProducer<>(props);

// Example stream configuration and topology creation
Properties streamsProps = new Properties();
streamsProps.put(StreamsConfig.APPLICATION_ID_CONFIG, "at-most-once-processing-topology");
streamsProps.put(StreamsConfig.BOOTSTRAP_SERVERS_CONFIG, "localhost:9092");
streamsProps.put(ProducerConfig.TRANSACTIONAL_ID_CONFIG, "transactional-id");

StreamsBuilder builder = new StreamsBuilder();
KStream<String, String> sourceTopic = builder.stream("common-topic");
sourceTopic.groupByKey()
           .reduce((value1, value2) -> value1); // Ensure unique processing
// Add additional state stores and processors as needed
KTable<Bytes, Long> stateStore = ...;

StreamsConfig streamsConfig = new StreamsConfig(streamsProps);
final KafkaStreams streams = new KafkaStreams(builder.build(), streamsConfig);
streams.start();
```

**Data:**

```json
{
  "common-topic": {
    "partitions": [
      {"partition": 0},
      {"partition": 1},
      {"partition": 2}
    ]
  }
}
```

**Flow:**

1. Producer
2. Start Transaction
3. Send Message
4. Commit Transaction
5. Consumer Group A
6. Fetch and Process Messages
7. Commit or Rollback
8. At-least-once Processing
9. Consumer Group B (Optional)
10. Fetch and Process Messages (IfNeeded)

