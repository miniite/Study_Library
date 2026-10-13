---
type: "practice"
difficulty: "Deep"
---

# How do you guarantee ordering of remediation actions within a service key when Flink parallelizes processing across tasks?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Exactly Once into Kafka](<../../../../TOOLS/Apache_Flink/Flink_Core/Exactly_Once_into_Kafka.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To ensure ordered remediation actions within a service key in Apache Flink, we need to manage task parallelism carefully and ensure that events with the same service key are processed in order despite being spread across different tasks. The core idea is to use Flink's stateful processing capabilities to maintain a global ordering constraint.

#### Triage

The question asks how to guarantee the correct order of remediation actions (e.g., sending notifications or executing steps) for events within the same service key when Flink parallelizes them across multiple tasks. This involves understanding Flink's task parallelism, state management, and event time processing.

#### Assumptions
Events are timestamped with a service key.
We need to ensure that all events with the same service key are processed in order.
The remediation actions depend on the sequence of these events.

#### First-Principles Solution

To solve this, we can use Flink's stateful processing features and event time semantics. Here’s a step-by-step breakdown:

Event Time Processing: Use event time to control the order of events.
State Management: Maintain an in-memory or RocksDB state table to track the sequence numbers of events for each service key.
Global Ordering Constraint: Ensure that only one task processes events with a given service key at any time.
Step 1: Event Time Processing

Apache Flink supports event time processing, where the order of events is determined by their timestamps rather than the order in which they are received. This means we can ensure that all events for a specific service key are processed in chronological order.

JAVA
StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
DataStream<MyEvent> events = env.addSource(new FlinkKafkaConsumer<>("topic", new MyEventDeserializationSchema(), kafkaConfig));

// Set event time and watermark strategy
events.assignTimestampsAndWatermarks(new EventTimeExtractor<>());
Step 2: State Management with Keyed State

To ensure that all events for the same service key are processed in order, we can use Flink's keyed state to maintain a global view of the sequence numbers.

JAVA
KeyedStream<MyEvent, String> keyedEvents = events.keyBy(event -> event.getServiceKey());

keyedEvents.map(new KeyedStateMapFunction<String, MyEvent>() {
    @Override
    public void map(String key, Iterable<MyEvent> values) throws Exception {
        int seqNum = 0;
        for (MyEvent event : values) {
            if (event.getSequenceNumber() != seqNum) {
                throw new IllegalArgumentException("Out of order: " + event);
            }
            seqNum++;
            // Process the event
        }
    }
});
Step 3: Global Ordering Constraint with Event Time and Watermarks

Flink’s watermarks help to ensure that all elements with a given timestamp have been processed before any element with a later timestamp. By setting up an appropriate watermark strategy, we can ensure that Flink processes events in the correct order.

JAVA
events.assignTimestampsAndWatermarks(new EventTimeExtractor<>()).setProcessingTimewatermarkInterval(Duration.ofSeconds(1));

#### Closing Summary
and Common Mistakes/Follow-Ups

To summarize, ordering remediation actions within a service key when using Apache Flink requires leveraging event time processing and state management. Keyed state is crucial to maintain the global order of events for each service key. The common mistake here is not correctly managing event time or failing to use watermarks effectively.

#### Follow-ups

How do you handle late data in this scenario?
What if the number of tasks changes dynamically?
Can you optimize memory usage while maintaining order?

**Comparison:**

```
Service Key	Sequence Number	Timestamp
key1	0	1594638754000
key1	1	1594638755000
key2	0	1594638756000
key2	1	1594638757000
```

**Code:**

```java
// Example code snippet for setting up Flink with event time processing and watermark strategy
StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
DataStream<MyEvent> events = env.addSource(new FlinkKafkaConsumer<>("topic", new MyEventDeserializationSchema(), kafkaConfig));
events.assignTimestampsAndWatermarks(new EventTimeExtractor<>()).setProcessingTimewatermarkInterval(Duration.ofSeconds(1)).keyBy(event -> event.getServiceKey());
```

**Example input / output:**

```text
Input:
[
  {"serviceKey": "key1", "sequenceNumber": 0, "timestamp": 1594638754000},
  {"serviceKey": "key1", "sequenceNumber": 1, "timestamp": 1594638755000},
  {"serviceKey": "key2", "sequenceNumber": 0, "timestamp": 1594638756000},
  {"serviceKey": "key2", "sequenceNumber": 1, "timestamp": 1594638757000}
]

Output:
[
  {"serviceKey": "key1", "processed": true},
  {"serviceKey": "key1", "processed": true},
  {"serviceKey": "key2", "processed": true},
  {"serviceKey": "key2", "processed": true}
]
```

**Data:**

```json
{
  "topic": "example-topic",
  "serviceKeys": ["key1", "key2"],
  "events": [
    {"serviceKey": "key1", "sequenceNumber": 0, "timestamp": 1594638754000},
    {"serviceKey": "key1", "sequenceNumber": 1, "timestamp": 1594638755000},
    {"serviceKey": "key2", "sequenceNumber": 0, "timestamp": 1594638756000},
    {"serviceKey": "key2", "sequenceNumber": 1, "timestamp": 1594638757000}
  ]
}
```

**Flow:**

1. Yes
2. No
3. Event Source
4. Is Event Time?
5. Assign Watermarks
6. Keyed State Management
7. Handle Late Data
8. Process Events in Order
9. Manage Late Data
10. Output Processed Events

