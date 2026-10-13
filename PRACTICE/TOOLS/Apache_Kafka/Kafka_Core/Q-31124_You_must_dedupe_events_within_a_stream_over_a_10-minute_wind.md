---
type: "practice"
difficulty: "Deep"
---

# You must dedupe events within a stream over a 10-minute window before writing to the lake; how do you bound the state store size and evict expired keys?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Kafka Streams and Event Time](<../../../../TOOLS/Apache_Kafka/Kafka_Core/Kafka_Streams_and_Event_Time/_index.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the requirement of deduplicating events within a stream over a 10-minute window, we need to design a system that can efficiently manage the state store while ensuring that only unique events are written to the data lake. The key challenge is to bound the size of the state store and evict expired keys effectively.

The question revolves around how to manage state in Apache Kafka Streams or KSQL to ensure efficient deduplication within a sliding window, while also efficiently managing the state store size. We will break this down step by step from first principles.

#### Triaging the Question
Understanding the Requirement: The goal is to remove duplicate events based on certain keys within a 10-minute window before writing to the data lake.
Key Probes:
How to implement deduplication in Kafka Streams or KSQL.
Managing state store size and expiration of expired keys.

#### Assumptions
The key by which events need to be deduplicated can vary (e.g., event_id, user_id).
The data lake is a downstream destination that will receive unique, non-duplicate records.

#### Solution
Key Selection: Identify the key(s) by which you want to deduplicate events.
State Store Configuration:
Use KafkaStreams or KSQLDB for managing state stores.
Window-Based Deduplication:
Utilize windowing in Kafka Streams or KSQL to create a sliding window of 10 minutes.
State Management:
Implement state stores such as ValueStore, MapStore, or SessionWindows depending on the use case.
Eviction Policies:
Configure eviction policies for state stores to manage the size and ensure that only relevant data is retained.

#### First-Principles Solution
Step 1: Define Deduplication Logic

We will define a logic where events are processed, checked against a state store, and only unique events (within the specified window) are written to the data lake.

JAVA
KStream<String, String> input = builder.stream("inputTopic");

// Define the state store for deduplication
ValueJoiner<String, String> joiner = (event, existingEvent) -> event; // Use new event if it exists

// Create a session window of 10 minutes with a gap of 1 minute
KTable<Windowed<String>, String> deduplicatedEvents =
    input
        .groupByKey()
        .windowedBy(TimeWindows.ofDuration(Duration.ofMinutes(10)).advanceBy(Duration.ofMinutes(1)))
        .reduce(joiner, Materialized.as("deduplication-store").withValueSerde(SpecificAvroSerde.class));

// Write the deduplicated events to the data lake topic
deduplicatedEvents.toStream().to("outputTopic", Produced.with(Serdes.WindowedString(), Serdes.String()));
Step 2: State Store Configuration

We need to configure the state store to manage the size and expiration of keys.

JAVA
Properties streamProps = new Properties();
streamProps.put(StreamsConfig.STATE_DIR_CONFIG, "path/to/state/dir");
streamProps.put(StreamsConfig.APPLICATION_ID_CONFIG, "deduplication-app");

KafkaStreams streams = new KafkaStreams(builder.build(), streamProps);
streams.start();

// Handle state store expiration
streams.setStateListener((metadata, metadata1) -> {
    if (metadata != null && metadata.state().isRestoreComplete()) {
        // Evict expired keys from the state store
        ((SessionStore<?>) streams.getStateStores().get("deduplication-store")).evictExpiredKeys();
    }
});
Step 3: Manage State Store Size and Expiration

To manage the size of the state store, we need to configure appropriate settings for maxBytes and replicationFactor.

JAVA
// Configure state store with max bytes and replication factor
Properties props = new Properties();
props.put(StreamsConfig.STATE_DIR_CONFIG, "path/to/state/dir");
props.put(StreamsConfig.NUMBERS_OF_REPLICAS, 1); // Adjust based on the requirement
props.put(StreamsConfig.APPLICATION_ID_CONFIG, "deduplication-app");

KafkaStreams streams = new KafkaStreams(builder.build(), props);
streams.start();

#### Closing Summary
and Common Mistakes/Follow-Ups
Mistake: Not configuring state stores properly can lead to excessive memory usage or loss of critical data.

#### Follow-Up
Discuss trade-offs between using in-memory storage versus persistent storage for state management.
Triangulation and Confirmation

This solution covers the deduplication logic, state store configuration, and key management. Ensuring that state stores are properly managed is crucial for maintaining system performance and reliability.

**Comparison:**

| State Store Configuration | Description |
|---------------------------|-------------|
| `state.dir`               | Path to state directory for storing state data. |
| `replication.factor`      | Number of replicas for the state store.       |
| `application.id`          | Unique application ID for the Kafka Streams.  |

**Code:**

```

KStream<String, String> input = builder.stream("inputTopic");

ValueJoiner<String, String> joiner = (event, existingEvent) -> event;

KTable<Windowed<String>, String> deduplicatedEvents = input .groupByKey() .windowedBy(TimeWindows.ofDuration(Duration.ofMinutes(10)).advanceBy(Duration.ofMinutes(1))) .reduce(joiner, Materialized.as("deduplication-store").withValueSerde(SpecificAvroSerde.class));

deduplicatedEvents.toStream().to("outputTopic", Produced.with(Serdes.WindowedString(), Serdes.String()));

CODE
Copy
```java
Properties streamProps = new Properties();
streamProps.put(StreamsConfig.STATE_DIR_CONFIG, "path/to/state/dir");
streamProps.put(StreamsConfig.APPLICATION_ID_CONFIG, "deduplication-app");

KafkaStreams streams = new KafkaStreams(builder.build(), streamProps);
streams.start();

streams.setStateListener((metadata, metadata1) -> {
    if (metadata != null && metadata.state().isRestoreComplete()) {
        ((SessionStore<?>) streams.getStateStores().get("deduplication-store")).evictExpiredKeys();
    }
});
JAVA
Copy
Properties props = new Properties();
props.put(StreamsConfig.STATE_DIR_CONFIG, "path/to/state/dir");
props.put(StreamsConfig.NUMBERS_OF_REPLICAS, 1);
props.put(StreamsConfig.APPLICATION_ID_CONFIG, "deduplication-app");

KafkaStreams streams = new KafkaStreams(builder.build(), props);
streams.start();
```

**Example input / output:**

```text
Input to Kafka Topic (inputTopic):
{"key": "user_1", "value": "event_1"}

Output from Kafka Streams:
{"windowedKey": {"key": "user_1", "windowStart": 0, "windowEnd": 600}, "value": "event_1"}
```

**Data:**

```json
{
  "inputTopic": {
    "partitionCount": 3,
    "replicationFactor": 1
  },
  "outputTopic": {
    "partitionCount": 3,
    "replicationFactor": 1
  }
}
```

**Flow:**

1. Input Topic (inputTopic)
2. Deduplication Logic
3. Session Window of 10 Minutes
4. Reduce by Key and Event Time
5. State Store(deduplication-store)
6. Deduplicated Events to DataLake
7. Output Topic (outputTopic)

