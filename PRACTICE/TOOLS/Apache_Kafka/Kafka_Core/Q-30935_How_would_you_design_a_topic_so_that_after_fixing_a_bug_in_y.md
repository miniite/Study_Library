---
type: "practice"
difficulty: "Deep"
---

# How would you design a topic so that after fixing a bug in your detection model, you can reprocess telemetry and produce the exact same corrected anomalies deterministically?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Idempotent and Transactional Producers](<../../../../TOOLS/Apache_Kafka/Kafka_Core/Idempotent_and_Transactional_Producers.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address this problem, we need to ensure that Kafka can be used as a reliable system for reprocessing old telemetry data after fixing a bug in our detection model. The goal is to produce exactly the same anomalies deterministically, ensuring consistency and reliability.

Firstly, let's clarify what the question is really asking:

Reproducibility: We need to ensure that any anomaly detected can be traced back to its original state.
Determinism: Any given piece of data should yield the same results every time it is processed.
Efficiency: The solution should be scalable and performant.

#### Triage

The core issue here is to design a Kafka topic in such a way that we can reprocess all historical telemetry data, fix bugs, and ensure that the anomalies detected are exactly the same as they were originally. This requires understanding how Kafka works with respect to message durability, partitioning, and offsets.

#### Assumptions
The detection model uses specific fields or metadata from the telemetry data.
We need to support both reading old and new data in a consistent manner.
The bug is fixed and we have a reliable method for reprocessing data.

#### Solution

To achieve this, we will leverage Kafka's features such as partitioning, offsets, and message durability. Here’s a step-by-step approach:

Partition Design: Ensure each telemetry record is uniquely identifiable through its timestamp or unique identifier.
Message Durability: Use exactly-once processing guarantees provided by Apache Kafka Streams or KStreams to ensure each message is processed only once per topic partition.
Offset Management: Use offset management strategies to control the reprocessing flow, ensuring we can replay messages from a specific point in time.
Step 1: Partitioning and Unique Identifiers

We need to design our topic partitions such that each telemetry record has a unique identifier or key. This will help us precisely locate and process individual records. For example, if we are dealing with device telemetry data, we might use the combination of device_id and timestamp.

PLAINTEXT
Partition Key: <device_id>_<timestamp>
Step 2: Exactly-Once Processing

To ensure exactly-once processing, we can leverage Kafka Streams or KStreams which provide built-in support for exactly-once semantics. This involves setting up proper checkpointing mechanisms to store offsets and state.

JAVA
KafkaStreams streams = new KafkaStreams(builder.build(), props);
streams.start();
Step 3: Replaying Messages

When fixing a bug, we can replay messages from the beginning of the topic or from a specific offset. This ensures that all historical data is processed again with the updated model.

JAVA
// Example of replaying messages starting from a specific offset
KafkaConsumer<String, String> consumer = new KafkaConsumer<>(props);

#### cons
umer.assign(Collections.singletonList(new TopicPartition("telemetry-topic", 0)));

// Set the initial offset to reprocess from
long startOffset = 12345L; // This could be any valid offset

while (true) {

#### Cons
umerRecords<String, String> records = consumer.poll(Duration.ofMillis(100));
    for (ConsumerRecord<String, String> record : records) {
        if (record.offset() < startOffset) continue;

        processRecord(record.value());
    }
}
Step 4: Deterministic Processing

Ensure that the processing logic in your detection model is deterministic. This means that given the same input and state, it should always produce the same output.

#### Closing Summary

By designing a Kafka topic with unique partition keys, using exactly-once processing guarantees, managing offsets effectively, and ensuring deterministic processing, we can reliably reprocess historical telemetry data and fix bugs while producing exact anomalies deterministically.

#### Common Mistakes/Follow-Ups
Not Using Exactly-Once Semantics: Failing to use proper mechanisms like Kafka Streams for exactly-once processing.
Ignoring Offset Management: Not handling offsets correctly during replay, leading to potential data loss or duplication.
Non-Deterministic Processing Logic: Ensuring that the detection model logic does not introduce any non-determinism.

**Comparison:**

| Partition Key | Device ID   | Timestamp            | Value | Metadata       |
|---------------|-------------|----------------------|-------|----------------|
| D001_2023-10-01T12:00:00Z | 85         | {"sensor_type": "temperature"} |

**Code:**

```java
// Example of Kafka Streams setup for exactly-once processing
Properties props = new Properties();
props.put(StreamsConfig.APPLICATION_ID_CONFIG, "telemetry-streaming-app");
props.put(StreamsConfig.BOOTSTRAP_SERVERS_CONFIG, "localhost:9092");

StreamsBuilder builder = new StreamsBuilder();

KStream<String, String> source = builder.stream("telemetry-topic");

// Process the records with exactly-once semantics
source.foreach((key, value) -> {
    // Process record deterministically here
});

KafkaStreams streams = new KafkaStreams(builder.build(), props);
streams.start();
```

**Example input / output:**

```text
Input: Telemetry records in Kafka topic with specific partition keys.
Output: Processed anomalies stored in a state store, ensuring deterministic results.
```

**Data:**

```json
{
  "device_id": "D001",
  "timestamp": "2023-10-01T12:00:00Z",
  "value": 85,
  "metadata": {
    "sensor_type": "temperature"
  }
}
```

**Flow:**

1. No
2. Yes
3. Replay Messages
4. Is Fixed Bug Applied?
5. Process Historical Data
6. Update State Store with NewAnomalies
7. Ensure DeterministicProcessing Logic

