---
type: "practice"
difficulty: "Deep"
---

# How would you handle out-of-order CDC events (an update arriving before its insert) when materializing the current state of a table in Spark?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Topic and Schema Design for Pipelines](<../../../../03_Big_Data_and_Streaming/Distributed_Processing/Big_Data_Processing/Topic_and_Schema_Design_for_Pipelines.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To handle out-of-order Change Data Capture (CDC) events, we need to ensure that our data processing pipeline correctly reconstructs the current state of the table despite updates arriving before their corresponding inserts. This is a common challenge in real-time big data processing using systems like Apache Kafka and Apache Spark.

#### Triage

The question essentially asks how to maintain temporal consistency when processing CDC events with out-of-order updates. The interviewer may be probing for an understanding of event time processing, watermarking, session windows, and ensuring that the current state is correctly materialized in Spark.

#### Assumptions
and Key Ideas
We are using a distributed system (Kafka) to capture changes.
Our goal is to maintain a consistent view of the data over time.
Out-of-order events can lead to incorrect state reconstruction if not handled properly.

#### First Principles Solution

Understand Event Time vs. Processing Time:

In real-time systems, event time refers to when an event actually occurred.
Processing time is when the event is processed by the system.

Event Timestamps and Watermarking:

Assign a timestamp to each CDC event in Kafka.
Set up watermarking to manage out-of-order events: if an update has a timestamp earlier than the current watermark, it is considered late and dropped or stored for later processing.

Session Windows (Optional but Useful):

Use session windows in Spark to group events that are close in time. This can help in handling out-of-order updates by aggregating them within a certain timeframe.

Fault Tolerance and State Management:

Ensure that state changes are checkpointed regularly.
Use reliable state stores (like Kafka offsets, RocksDB) to maintain the state of the materialized view.
Step-by-Step Solution

Capture Event Time in Kafka:

JSON
{
  "id": 123,
  "action": "UPDATE",
  "timestamp": 1640995200, // Unix timestamp
  "data": {"name": "John Doe", "age": 30}
}

Process Events in Spark:

Use a FileStreamSource or KafkaConsumer to read events from Kafka.
Assign timestamps and watermarks using the withWatermark method.
SCALA
val df = spark.readStream.format("kafka")
  .option("kafka.bootstrap.servers", "localhost:9092")
  .option("subscribe", "cdc-topic")
  .load()

// Convert to DataFrame with columns 'key', 'value', and inferred event time column from Kafka message
val eventsDf = df.selectExpr("CAST(key AS STRING)", "CAST(value AS STRING)")

// Assign timestamps to the events (assuming value contains a JSON with a timestamp)
val eventsWithTimestamps = eventsDf.withColumn("timestamp", from_json($"value".cast(StringType), Map("timestamp" -> LongType)).cast("long"))

// Create a stream with watermark
val stream = eventsWithTimestamps.writeStream.format("console")
  .option("checkpointLocation", "/path/to/checkpoint")
  .trigger(ProcessingTime("1 second"))
  .outputMode("update")

stream.start().awaitTermination()

Handle Out-of-Order Events with Watermark:

Define a watermark to drop late events.
SCALA
.withWatermark("timestamp", "30 seconds")

Aggregate and Materialize Current State:

Use groupByKey or reduceByKey for aggregating updates.
SCALA
val currentState = stream.groupByKey("id").mapGroups { (key, events) =>
  // Implement logic to handle out-of-order updates and maintain current state
  key -> processEvents(events)
}

Persist Current State:

Write the materialized view to a reliable storage like Kafka or a database.
SCALA
currentState.writeStream.outputMode("complete").format("kafka")
  .option("kafka.bootstrap.servers", "localhost:9092")
  .option("topic", "materialized-view-topic")
  .start().awaitTermination()

#### Closing Summary
and Common Mistakes/Follow-Ups
Closing: The key to handling out-of-order CDC events is to use proper event time processing, watermarking, and reliable state management. This ensures that the current state of the table can be correctly reconstructed even with delayed or out-of-order updates.

#### Common Mistakes
Forgetting to set up watermarks, leading to late event processing issues.
Not checkpointing state changes, resulting in data loss during failures.
Mismanaging session windows, causing incorrect aggregation of events.
Common Follow-Ups
How do you handle out-of-order events with high latency?
What happens if the watermark is too low or too high?
How would you optimize performance while ensuring correctness?

**Comparison:**

```
FLOWCHART · MERMAID
erDiagram
    CUSTOMER ||--o{ ORDER : places
    ORDER }|--{ LINE_ITEM : contains
    CUSTOMER ||--|{ ADDRESS : ships to
```

**Code:**

```scala
val df = spark.readStream.format("kafka")
  .option("kafka.bootstrap.servers", "localhost:9092")
  .option("subscribe", "cdc-topic")
  .load()

val eventsDf = df.selectExpr("CAST(key AS STRING)", "CAST(value AS STRING)")

val eventsWithTimestamps = eventsDf.withColumn("timestamp", from_json($"value".cast(StringType), Map("timestamp" -> LongType)).cast("long"))

val stream = eventsWithTimestamps.writeStream.format("console")
  .option("checkpointLocation", "/path/to/checkpoint")
  .trigger(ProcessingTime("1 second"))
  .outputMode("update")

stream.start().awaitTermination()
```

**Example input / output:**

| Input Event                | Timestamp        | Action   |
|-----------------------------|-------------------|----------|
| {id:123, action:"UPDATE", timestamp:1640995200} | 1640995200       | UPDATE   |
| Output Event               | Materialized State |
|----------------------------|--------------------|
| {id:123, name: "John Doe", age: 30} | Updated state      |

**Data:**

```json
{
  "id": 123,
  "action": "UPDATE",
  "timestamp": 1640995200, // Unix timestamp
  "data": {"name": "John Doe", "age": 30}
}
```

**Flow:**

1. Event Source (Kafka)
2. Spark Streaming withWatermarks
3. Checkpointing State
4. Maintain Current State
5. Write Materialized View toKafka

