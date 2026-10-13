---
type: "practice"
difficulty: "Deep"
---

# How do you handle a Kafka topic that needs to preserve only the latest value per key?

**Difficulty:** Deep · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Log Compaction](<../../../../TOOLS/Apache_Kafka/Kafka_Core/Log_Compaction.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To handle a Kafka topic that needs to preserve only the latest value per key, we need to ensure that duplicate messages with the same key are dropped or updated in place without affecting other keys' values. This problem is common in scenarios where you're processing streaming data and want to keep track of the most recent state.

#### Triaging the Question

The question is asking for a strategy to implement the "latest value per key" semantics in Kafka. The interviewer might be probing your understanding of Kafka's internal mechanics, how to configure it properly, and any potential trade-offs involved. Key assumptions include that the incoming messages are timestamped or have a sequence number that can help determine which message is newer.

#### Solution
Overview

The approach involves setting up a KTable in Apache Kafka Streams, using state stores to maintain the latest values, and leveraging Kafka's exactly-once semantics if required. We'll use reduce operations within streams to update the state store with new keys or keep the latest value for existing ones.

Detailed First-Principles Solution
Setting Up Kafka Streams: Create a Kafka Streams application that reads from the input topic.
Defining the State Store: Use a KTable to maintain the latest values by key.
Processing Logic: For each incoming message, update the state store with the new value if it's newer than the current one.
Exactly-Once Semantics (Optional): Ensure that updates are idempotent and use transactional guarantees to avoid duplicates.
Step-by-Step Solution

Define the State Store:

JAVA
KTable<String, Value> stateStore = builder.stateStore(
    Stores.persistentKeyOnly("latestValuesByKey")
        .withLoggingLevel(LoggingLevel.DEBUG)
        .build());

Create a Stream Processor:

JAVA
StreamsBuilder builder = new StreamsBuilder();

KStream<String, Value> inputTopicStream = builder.stream("input-topic");
KTable<String, Value> latestValuesByKey = inputTopicStream
    .groupByKey()
    .reduce((existingValue, newValue) -> newValue); // Update only if the newer value comes in.

stateStore = builder.stateStore(Stores.keyOnlyBuilder(
    "latestValuesByKey",
    Serdes.String()));

Update State Store:

JAVA
latestValuesByKey.toStream().foreach((key, value) -> {
    // Update the state store with the new value.
    // This can be done by storing the new value in a distributed state store.
});

Ensure Exactly-Once Semantics (Optional):

JAVA
StreamsConfig config = new Properties();
config.put(StreamsConfig.APPLICATION_ID_CONFIG, "latest-value-key");
config.put(StreamsConfig.STATE STORES_DIR_CONFIG, "/tmp/state-stores");

// Enable transactional idempotence.
config.put(StreamsConfig.getTransactionTimeoutMs(), 30_000);
config.put(StreamsConfig.DEFAULT KEY SERDE CLASSES_CONFIG, Serdes.String());
config.put(StreamsConfig.DEFAULT VALUE SERDE CLASSES_CONFIG, MyValueSerde.class);

KafkaStreams streams = new KafkaStreams(builder.build(), config);
streams.cleanUp();
streams.start();

Monitoring and Tuning:

Monitor the state stores for unexpected behavior.
Tune the batch window or session timeout if necessary to balance between freshness and latency.

#### Closing Summary

By using a KTable in Kafka Streams, we can efficiently maintain only the latest value per key by updating the state store with new values. The use of reduce ensures that only newer values are stored while avoiding duplicates. Exactly-once semantics provide additional reliability but add complexity, so they should be considered carefully based on the application's requirements.

#### Common Mistakes
and Follow-Ups
Incorrect Assumptions: Assuming that messages come in a specific order can lead to incorrect state updates.
Transaction Handling: Failing to handle transactions correctly might result in data inconsistencies.
Scalability Issues: Not considering how large the state store can grow if there are many unique keys.

By understanding and addressing these points, you ensure robust handling of your Kafka topic.

**Comparison:**

| Time       | Key   | Value |
|------------|-------|-------|
| 10:00 AM   | user1 | 10    |
| 10:05 AM   | user2 | 20    |
| 10:10 AM   | user1 | 30    |
| 10:15 AM   | user3 | 40    |

**Code:**

```java
// Example Serde class for values
public class MyValueSerde implements Serde<Value> {
    @Override
    public Serde<Value> configure(final Map<String, ?> configs, final boolean isKey) {
        return this;
    }

    @Override
    public void close() {}

    @Override
    public Serializer<Value> serializer() {
        return new ValueSerializer();
    }

    @Override
    public Deserializer<Value> deserializer() {
        return new ValueDeserializer();
    }
}

class ValueSerializer implements Serializer<Value> {
    private static final Logger LOG = LoggerFactory.getLogger(ValueSerializer.class);

    @Override
    public void configure(final Map<String, ?> configs, boolean isKey) {}

    @Override
    public byte[] serialize(String topic, Value value) {
        // Implement serialization logic.
        return null;
    }

    @Override
    public void close() {}
}

class ValueDeserializer implements Deserializer<Value> {
    private static final Logger LOG = LoggerFactory.getLogger(ValueDeserializer.class);

    @Override
    public void configure(final Map<String, ?> configs, boolean isKey) {}

    @Override
    public Value deserialize(String topic, byte[] data) {
        // Implement deserialization logic.
        return null;
    }

    @Override
    public void close() {}
}
```

**Example input / output:**

```text
Input:
- Key: user1, Value: 10
- Key: user2, Value: 20
- Key: user1, Value: 30
- Key: user3, Value: 40

Output:
State Store (latestValuesByKey):
- Key: user1, Value: 30
- Key: user2, Value: 20
- Key: user3, Value: 40
```

**Data:**

```json
{
  "input-topic": [
    {"key": "user1", "value": 10},
    {"key": "user2", "value": 20},
    {"key": "user1", "value": 30},
    {"key": "user3", "value": 40}
  ]
}
```

**Flow:**

1. Yes
2. No
3. Input Topic
4. Key Exist?
5. Update State Store with NewValue
6. Add Key to State Store withInitial Value
7. Output Latest Values fromState Store

