---
type: "practice"
difficulty: "Deep"
---

# You're getting inconsistent metric values between your batch feature job (Spark) and a streaming version (Flink). How do you make them reconcile by construction?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Streaming-first Data Architecture](<../../../../04_Data_Architecture_and_Strategy/Architecture_Patterns/Data_Architecture/Tiered_Lake_Storage/Streaming-first_Data_Architecture.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To resolve inconsistent metric values between your batch feature job (using Spark) and a streaming version (using Flink), we need to ensure that both systems are processing the same data in a consistent manner, and that they both handle incremental updates correctly. This involves understanding the differences between batch and stream processing, ensuring data consistency, and implementing robust reconciliation mechanisms.

First, let's clarify what we mean by "inconsistent metric values." Metrics could be out of sync due to issues such as data lag, different processing windows, or how each system handles state and time-based operations. Our goal is to reconcile these differences so that both systems produce consistent results.

#### Triage
Understanding the Problem

The question probes your understanding of batch vs. streaming architectures, state management in distributed systems, and the challenges of maintaining consistency across data processing pipelines. We'll assume:

The Spark job processes historical data in batches.
The Flink job is a real-time system that processes incoming data streams.
Both systems calculate metrics over overlapping windows or time periods.

#### First Principles Solution

Understand the Data Processing Workflow:

Batch Job (Spark): Processes historical data, possibly using hourly, daily, or custom windows to compute metrics.
Streaming Job (Flink): Continuously processes incoming data streams and computes rolling metrics based on sliding or tumbling windows.

Data Consistency and Windowing:

Ensure both systems use the same windowing strategy (e.g., tumbling, sliding) with identical boundaries.
For example, if your Spark job aggregates data every hour from midnight to 1 AM, ensure Flink does the same.

State Management and Out-of-Order Events:

Both systems need to handle out-of-order events gracefully. This is critical in streaming systems but may also apply to batch jobs if they are processing late data.
Implement state stores (in Flink) or checkpoints (in Spark) to maintain consistent state across the pipeline.

Incremental Updates and Delta Processing:

For real-time updates, consider using incremental processing techniques where only new or updated records are processed.
In batch systems, use data versioning or timestamp-based filtering to ensure only relevant data is considered for metric calculation.

Data Reconciliation Mechanism:

Implement a reconciliation mechanism that compares and adjusts the metrics between the two systems periodically.
Use a common reference system (e.g., a central database) to store consistent state, which both Spark and Flink can query to ensure consistency.

Monitoring and Logging:

Continuously monitor the data flows, processing times, and output values from both systems.
Set up logging mechanisms to capture discrepancies and trace their root causes.

#### Example
Flow

Spark Batch Job:

PYTHON
# Assuming a Spark job that processes hourly data windows
spark = SparkSession.builder.appName("BatchFeatureJob").getOrCreate()

def process_data(start_time, end_time):
    df = spark.read.parquet(f"data/{start_time}-{end_time}")
    metrics = df.groupBy("key").agg(avg("value"), count("value"))
    return metrics

# Process data for the current hour
start_hour = datetime.now().replace(minute=0, second=0, microsecond=0)
end_hour = (datetime.now() + timedelta(hours=1)).replace(minute=0, second=0, microsecond=0)

metrics_df = process_data(start_hour, end_hour)

Flink Streaming Job:

JAVA
final StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();

env.setParallelism(1);

DataStream<String> stream = env.addSource(new SocketInputSource("localhost", 9876));

DataStream<Row> processedStream = stream.map(line -> parseLine(line)).keyBy("key").timeWindow(Time.minutes(5))
    .reduce((a, b) -> new Row(Arrays.asList(a.getValue0() + b.getValue0(), a.getValue1() + b.getValue1())));

processedStream.print();
env.execute("Flink Streaming Job");

#### Closing Summary
and Common Mistakes

By ensuring consistent windowing strategies, robust state management, incremental updates, and implementing a reconciliation mechanism, we can align the metrics from batch and streaming jobs. Common mistakes include:

Mismatched window boundaries between batch and stream processing.
Not handling out-of-order events correctly in either system.
Failing to implement adequate monitoring and logging mechanisms.

**Comparison:**

 | Batch Job Metrics | Streaming Job Metrics |
|--------------------|----------------------|
| Key: A, Avg Value: 1.5, Count: 2 | Key: A, Avg Value: 1.5, Count: 2 |
| Key: B, Avg Value: 2.0, Count: 2 | Key: B, Avg Value: 2.0, Count: 2 |
| Key: C, Avg Value: 3.0, Count: 2 | Key: C, Avg Value: 3.0, Count: 2 |

**Code:**

```
Spark example

spark = SparkSession.builder.appName("BatchFeatureJob").getOrCreate() def process_data(start_time, end_time): df = spark.read.parquet(f"data/{start_time}-{end_time}") metrics = df.groupBy("key").agg(avg("value"), count("value")) return metrics

start_hour = datetime.now().replace(minute=0, second=0, microsecond=0) end_hour = (datetime.now() + timedelta(hours=1)).replace(minute=0, second=0, microsecond=0)

metrics_df = process_data(start_hour, end_hour)

CODE
Copy

```java
// Flink example
final StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
env.setParallelism(1);

DataStream<String> stream = env.addSource(new SocketInputSource("localhost", 9876));

DataStream<Row> processedStream = stream.map(line -> parseLine(line)).keyBy("key").timeWindow(Time.minutes(5))
    .reduce((a, b) -> new Row(Arrays.asList(a.getValue0() + b.getValue0(), a.getValue1() + b.getValue1())));

processedStream.print();
env.execute("Flink Streaming Job");
```

**Example input / output:**

```text
Input:
 - Spark batch job processes data from midnight to 1 AM.
 - Flink streaming job receives a stream of events.

 Output:
 - Both systems compute metrics over the same time window (e.g., 0-1 AM).
 - Periodic reconciliation ensures that discrepancies are detected and resolved.
```

**Data:**

```json
{
   "data": [
     { "key": "A", "value": 1 },
     { "key": "B", "value": 2 },
     { "key": "C", "value": 3 }
   ],
   "metrics": [
     { "key": "A", "avg_value": 1.5, "count": 2 },
     { "key": "B", "avg_value": 2.0, "count": 2 },
     { "key": "C", "avg_value": 3.0, "count": 2 }
   ]
 }
```

**Flow:**

1. Time Boundary
2. True
3. False
4. Process Event
5. True
6. False
7. Batch Job
8. Process Data in Window
9. Incremental Update
10. Compute Metrics
11. Wait for Next Window
12. Store/Update State
13. Flink Streaming Job
14. Receive Data Stream
15. Process Incrementally
16. Update Metrics
17. Wait for Next Event
18. Store/Update State
19. Periodic ReconciliationCheckpoints

