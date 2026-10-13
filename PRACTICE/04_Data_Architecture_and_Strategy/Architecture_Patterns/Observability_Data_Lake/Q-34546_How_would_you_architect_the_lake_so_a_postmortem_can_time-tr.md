---
type: "practice"
difficulty: "Deep"
---

# How would you architect the lake so a postmortem can time-travel to the exact table state at the incident minute while a compaction job ran hours ago?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Exactly Once into Observability Lakes](<../../../../04_Data_Architecture_and_Strategy/Architecture_Patterns/Observability_Data_Lake/Exactly_Once_into_Observability_Lakes.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To architected an Observability Data Lake that supports "time-travel" back to the exact table state during an incident, despite ongoing compactions, we need to focus on several key principles: event-time semantics, incremental data processing, and the use of watermarks. This ensures we can pinpoint the state of our tables at any given moment, even when compaction jobs are running.

The question is probing how to design a robust data lake architecture that provides granular temporal accuracy, enabling postmortem analysis with high fidelity. The core assumptions include ongoing data ingestion, periodic compactions to optimize storage and query performance, and the need for immutable historical data.

Let's break down the solution step-by-step:

Event-Time Semantics: Ensure all events are timestamped correctly using event time instead of processing time. This is crucial because it allows us to process records in any order while maintaining temporal correctness.
Watermarks for Compaction Control: Implement watermarks that track the maximum event-time seen so far. Watermarks help in skipping outdated data during compactions, ensuring only new and relevant data gets compacted.
Incremental Processing with Timestamps: Use incremental processing frameworks like Apache Flink or Kafka Streams to process events based on their timestamps. This ensures we can replay exactly once semantics for each event.
Immutable Historical Stores: Maintain immutable historical snapshots of tables at specific points in time. This could be achieved using technologies like Apache Iceberg, which supports both transactional and snapshot-based storage.
Compaction with Time-Windowing: Configure compactions to run over predefined time windows, ensuring they do not overwrite recent data that may still be relevant for the incident analysis.

#### Triaging the Question

The core challenge is balancing the need for efficient data processing and storage optimization (through compaction) while maintaining temporal accuracy for postmortem analysis. The solution must account for ongoing event ingestion, periodic compactions, and the requirement to access historical states accurately at any point in time.

Step-by-Step Solution from First Principles

Event-Time Semantics:

Ensure all events have a timestamp that reflects when the event occurred.
Use @timestamp fields in JSON or appropriate metadata in your data schema.

Watermarks for Compaction Control:

Define watermarks using the maximum event time seen so far. Watermarks can be set up in Flink jobs to control compactions and ensure only recent data is compacted.
JAVA
DataStream<MyEvent> stream = env.addSource(source);
KeyedStream<MyEvent, String> keyedStream = stream.keyBy(event -> event.getUserId());
SingleOutputStreamOperator<MyEvent> watermarkedStream = keyedStream.assignTimestampsAndWatermarks(new EventTimeAssigner<>(), new EventTimeExtractor<>());

Incremental Processing with Timestamps:

Configure Flink or Kafka Streams to process events in the order of their timestamps.
JAVA
stream.process(new ProcessFunction<MyEvent, MyResult>() {
    @Override
    public void processElement(MyEvent value, Context ctx, Collector<MyResult> out) throws Exception {
        // Process event based on timestamp and watermark
    }
});

Immutable Historical Stores:

Use a storage layer that supports snapshots, such as Apache Iceberg.
JAVA
Table table = HiveCatalog.createTable("my_table", schema);
table.createSnapshot();

Compaction with Time-Windowing:

Define compactions to run over specific time windows (e.g., hourly, daily) without affecting recent data.
JAVA
stream.process(new CompactionFunction<> {
    @Override
    public void processElement(MyEvent value, Context ctx, Collector<MyResult> out) throws Exception {
        // Compaction logic based on time window and watermark
    }
});

#### Closing Summary

By implementing these principles, we can ensure that the data lake supports precise temporal analysis even during compactions. Using event-time semantics, watermarks, incremental processing, immutable historical stores, and judicious compaction strategies, we can achieve a robust architecture for postmortem analysis.

#### Common Mistakes/Follow-Ups
Mistake 1: Ignoring event time and relying solely on processing time, leading to incorrect temporal analysis.
Mistake 2: Overcompacting data without proper watermarks, potentially losing relevant historical states.

#### Follow-Up
Discussing specific scenarios where the compaction strategy might need adjustment or additional controls.

**Comparison:**

| userId | eventTime        | metricValue |
|--------|------------------|-------------|
| user123 | 2023-10-01T14:30:00Z | 5.2         |

**Code:**

```java
// Example Flink code for event-time processing and watermarking
DataStream<MyEvent> stream = env.addSource(source);
KeyedStream<MyEvent, String> keyedStream = stream.keyBy(event -> event.getUserId());
SingleOutputStreamOperator<MyEvent> watermarkedStream = keyedStream.assignTimestampsAndWatermarks(new EventTimeAssigner<>(), new EventTimeExtractor<>());

// Compaction function
class CompactionFunction implements ProcessFunction<MyEvent, MyResult> {
    @Override
    public void processElement(MyEvent value, Context ctx, Collector<MyResult> out) throws Exception {
        // Compaction logic based on time window and watermark
    }
}
```

**Example input / output:**

| Input Event             | Watermark     | Output State |
|-------------------------|---------------|--------------|
| {"userId":"user123", ...} | 2023-10-01T14:30:00Z | Updated state with event |

**Data:**

```json
{
  "userId": "user123",
  "eventTime": "2023-10-01T14:30:00Z",
  "metricValue": 5.2
}
```

**Flow:**

1. No
2. Yes
3. Yes
4. No
5. Compaction Needed
6. No
7. Event Ingestion
8. Is Event Time > Watermark?
9. Process Event
10. Skip Event
11. Record Event in State Store?
12. Update State
13. Compaction Check
14. Compact Data

