---
type: "practice"
difficulty: "Deep"
---

# How would you architect a Flink pipeline that fans one raw event stream out into separate metrics, warehouse-load, and archival streams?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Flink vs Spark Streaming](<../../../../TOOLS/Apache_Flink/Flink_Core/Flink_vs_Spark_Streaming.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

#### Overview

When designing a Flink pipeline to fan an event stream into multiple streams—metrics, warehouse load, and archival—a key objective is to ensure efficient data processing with minimal latency while maintaining robustness and flexibility. This architecture leverages Flink's streaming capabilities to distribute the events based on specific business logic.

#### Triage

The question requires us to design a multi-stream Flink pipeline that processes raw event data and distributes it into three separate streams: metrics, warehouse load, and archival storage. The interviewer is probing your understanding of Flink's state management, parallelism, checkpointing, and fault tolerance mechanisms.

#### First Principles Solution
Event Source: Assume the events are continuously arriving in a stream.
Parallelism: Use high parallelism to handle large volumes efficiently.
Data Transformation & Distribution:
Apply business logic to categorize events into metrics, warehouse load, and archival streams.
State Management & Checkpointing: Ensure state consistency with checkpointing.
Fault Tolerance: Implement failover strategies using Flink's recovery mechanisms.
Step-by-Step Solution

Event Source

The raw event stream can be read from Kafka or a similar source using Flink’s DataStreamSource API.

Parallelism

Configure the Flink job to run with high parallelism, e.g., 64 tasks, to handle concurrent processing efficiently.

Data Transformation & Distribution

Use flatMap or map transformations to apply business logic and route events:
JAVA
// Example: Routing Events Based on Business Logic
DataStream<Event> rawEvents = env.addSource(new MyEventSource());

DataStream<Metric> metrics = rawEvents.flatMap(new MetricsExtractor()).name("metrics-extraction");
DataStream<LoadData> warehouseLoad = rawEvents.filter(event -> shouldGoToWarehouse(event)).map(new WarehouseLoader()).name("warehouse-load");
DataStream<ArchiveData> archivalStorage = rawEvents.filter(event -> shouldGoToArchival(event)).map(new Archiver()).name("archival-storage");

// Combine streams (optional)
SingleOutputStreamOperator<MergedEvent> mergedEvents = metrics.union(warehouseLoad, archivalStorage).name("merged-events");

State Management & Checkpointing

Set up checkpointing to ensure state consistency during recovery.
JAVA
env.setStateBackend(new FsStateBackend(checkpointsPath));
env.enableCheckpointing(5000); // every 5 seconds
env.getCheckpointConfig().setMinPauseBetweenCheckpoints(1000);
env.getCheckpointConfig().setMaxConcurrentCheckpoints(2);
env.getCheckpointConfig().setCheckpointingMode(CheckpointingMode.EXACTLY_ONCE);

Fault Tolerance

Configure Flink to recover from failures using a recovery strategy.
JAVA
env.setRestartStrategy(RestartStrategies.fixedDelayRestart(
  3, // number of restart attempts
  Time.of(10, TimeUnit.SECONDS) // delay between restarts
));

Sink Configuration

Configure sinks for each stream:
JAVA
metrics.addSink(new MetricsWriter()).name("metrics-writer");
warehouseLoad.addSink(new WarehouseLoader()).name("warehouse-loader");
archivalStorage.addSink(new Archiver()).name("archival-sink");

#### Closing Summary

The key components of the solution include high parallelism, data routing based on business logic, state management with checkpointing, and fault tolerance mechanisms. Each step ensures that events are processed efficiently and distributed correctly.

#### Common Mistakes/Follow-Ups
Mistake: Failing to configure enough parallelism can lead to bottlenecks.

#### Follow-Up
What happens if the event stream volume increases?
Mistake: Not setting up checkpointing can result in data loss during failures.

#### Follow-Up
How do you ensure state consistency with more complex business logic?

**Comparison:**

```
Archiver
Archiver
EventSource
Archiver
Archiver
EventSource
Event
Metric
LoadData (Filtered)
LoadData
ArchiveData (Filtered)
ArchiveData
```

**Code:**

```java
// Example of a MetricsExtractor
public class MetricsExtractor implements FlatMapFunction<Event, Metric> {
  @Override
  public void flatMap(Event value, Collector<Metric> out) throws Exception {
    // Extract metrics and emit them
    out.collect(new Metric(value));
  }
}

// Example of a WarehouseLoader
public class WarehouseLoader implements MapFunction<Event, LoadData> {
  @Override
  public LoadData map(Event value) throws Exception {
    if (shouldGoToWarehouse(value)) {
      return new LoadData(value);
    } else {
      throw new SkipException("Not going to warehouse");
    }
  }
}

// Example of a Archiver
public class Archiver implements MapFunction<Event, ArchiveData> {
  @Override
  public ArchiveData map(Event value) throws Exception {
    if (shouldGoToArchival(value)) {
      return new ArchiveData(value);
    } else {
      throw new SkipException("Not going to archival");
    }
  }
}
```

**Example input / output:**

```text
Input:
{
  "id": "12345",
  "type": "metric",
  "value": 98.6,
  "timestamp": "2023-10-01T12:00:00Z"
}

Output (Metrics Stream):
{
  "id": "12345",
  "type": "temperature",
  "value": 98.6
}

Output (Warehouse Load Stream):
{
  "id": "12345",
  "warehouse_id": "W1001",
  "timestamp": "2023-10-01T12:00:00Z"
}

Output (Archival Storage Stream):
{
  "id": "12345",
  "archive_type": "daily",
  "timestamp": "2023-10-01T12:00:00Z"
}
```

**Data:**

```json
{
  "Event": {
    "id": "12345",
    "type": "metric",
    "value": 98.6,
    "timestamp": "2023-10-01T12:00:00Z"
  },
  "Metric": {
    "id": "12345",
    "type": "temperature",
    "value": 98.6
  },
  "LoadData": {
    "id": "12345",
    "warehouse_id": "W1001",
    "timestamp": "2023-10-01T12:00:00Z"
  },
  "ArchiveData": {
    "id": "12345",
    "archive_type": "daily",
    "timestamp": "2023-10-01T12:00:00Z"
  }
}
```

**Flow:**

1. Extraction
2. Filtered Events
3. Filtered Events
4. Event Source
5. MetricsExtractor
6. MetricWriter
7. WarehouseLoader
8. WarehouseLoaderSink
9. Archiver
10. ArchiveDataWriter

