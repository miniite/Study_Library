---
type: "practice"
difficulty: "Deep"
---

# How would you design event-time windowing so a delayed upstream batch of events doesn't corrupt already-emitted window results?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Latency Budgets in Streaming Pipelines](<../../../../03_Big_Data_and_Streaming/Distributed_Processing/Big_Data_Processing/Latency_Budgets_in_Streaming_Pipelines.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the challenge of designing event-time windowing in a big data processing system where delayed upstream batches might corrupt already-emitted window results, we need to ensure that the system can handle out-of-order events while maintaining accuracy and consistency. The key idea is to maintain a buffer or state mechanism that allows for late arriving records without affecting the final window aggregation.

#### Triaging the Question

The question is probing our understanding of event-time processing, handling delayed data, and ensuring accurate results in Apache Spark with Kafka as the source. We need to consider assumptions such as the nature of the events, the type of windows (e.g., session, tumbling), and the window's end time policy.

#### First-Principles Solution
Define Event Time: Ensure that each event has a timestamp associated with it.
Window Definition: Choose appropriate window types (tumbling or sliding) based on requirements.
Late Arrival Handling:
Use watermark to define the maximum delay allowed for late events.
Implement stateful processing using Spark's state management capabilities.
Buffer Mechanism: Design a buffer that can hold delayed records until they are processed.
Detailed Solution
Step 1: Event Time and Watermarks

Event time is crucial as it allows us to handle out-of-order data correctly. A watermark defines the maximum delay allowed for late events, ensuring that we do not include events older than this threshold in our current window processing.

PYTHON
# Example Spark Streaming Code with Watermark
from pyspark.sql import SparkSession

spark = SparkSession.builder.appName("EventTimeWindowing").getOrCreate()

# Assuming input is a Kafka topic with structured data
df = spark.readStream.format("kafka") \
              .option("kafka.bootstrap.servers", "localhost:9092") \
              .option("subscribe", "events_topic") \
              .load()

# Define schema for the events
from pyspark.sql.types import StructType, TimestampType

schema = StructType() \
    .add("event_id", IntegerType()) \
    .add("user_id", IntegerType()) \
    .add("timestamp", TimestampType())

df = df.selectExpr("CAST(value AS STRING)").select(from_json(col("value"), schema).alias("data")).select("data.*")

# Apply watermark
windowed_df = df.withWatermark("timestamp", "30 minutes")
Step 2: Window Type and Aggregation Logic

Choose the appropriate window type based on the nature of the events. For example, a tumbling window might be used if we need to process data in fixed time intervals.

PYTHON
from pyspark.sql.functions import window, avg

# Example aggregation logic with tumbling windows
windowed_df = windowed_df.groupBy(window("timestamp", "10 minutes").alias("session"), "user_id") \
                        .agg(avg("event_id").alias("average_event_id"))
Step 3: Late Arrival Handling with Stateful Processing

Use Spark's state management capabilities to buffer and process late events.

PYTHON
from pyspark.sql.streaming import OutputMode

# Define a stateful processing function
def update_state(current_values, new_value):
    if not current_values:
        return [new_value]
    else:
        max_event_id = max(current_values + [new_value])
        return [max_event_id]

# Apply stateful processing with a buffer
windowed_df = windowed_df.groupBy(window("timestamp", "10 minutes").alias("session"), "user_id") \
                        .agg(avg("event_id").alias("average_event_id")) \
                        .writeStream \
                        .outputMode(OutputMode.Update()) \
                        .foreachBatch(lambda batchDF, _: batchDF.write.format("parquet").mode("append").save("/path/to/output")) \
                        .start()

#### Closing Summary

By defining event time and using watermarks, implementing appropriate window types, and leveraging stateful processing, we can handle delayed events without corrupting already-emitted results. The key is to ensure that late data does not affect the integrity of the current window's aggregation.

#### Common Mistakes
and Follow-Ups
Ignoring Watermarks: Not defining watermarks correctly can lead to incorrect handling of out-of-order data.
Inadequate State Management: Failing to buffer delayed events can result in loss of important data.
Incorrect Window Types: Choosing the wrong window type for the use case can lead to inaccurate results.

#### Follow-ups
could involve discussing additional optimizations, such as using micro-batch processing or integrating with external state stores like Kafka Connect.

**Comparison:**

| session          | user_id | average_event_id |
|------------------|---------|-----------------|
| [2023-10-01 14:00, 2023-10-01 15:00] | 1       | 1.5             |

**Code:**

```python
from pyspark.sql import SparkSession

spark = SparkSession.builder.appName("EventTimeWindowing").getOrCreate()

df = spark.readStream.format("kafka") \
              .option("kafka.bootstrap.servers", "localhost:9092") \
              .option("subscribe", "events_topic") \
              .load()

schema = StructType() \
    .add("event_id", IntegerType()) \
    .add("user_id", IntegerType()) \
    .add("timestamp", TimestampType())

df = df.selectExpr("CAST(value AS STRING)").select(from_json(col("value"), schema).alias("data")).select("data.*")

windowed_df = df.withWatermark("timestamp", "30 minutes")
```

**Example input / output:**

```text
# Example input -> output

from pyspark.sql.functions import window, avg

df = spark.createDataFrame([
    (1, 1, "2023-10-01T14:00:00Z"),
    (2, 1, "2023-10-01T14:05:00Z"),
    (3, 1, "2023-10-01T15:05:00Z")
], ["event_id", "user_id", "timestamp"])

df = df.withColumn("timestamp", col("timestamp").cast(TimestampType()))

windowed_df = df.withWatermark("timestamp", "30 minutes")

result = windowed_df.groupBy(window("timestamp", "10 minutes").alias("session"), "user_id") \
                    .agg(avg("event_id").alias("average_event_id"))

result.writeStream.outputMode(OutputMode.Update()).start().awaitTermination()
```

**Data:**

```json
{
  "events_topic": [
    {"event_id": 1, "user_id": 1, "timestamp": "2023-10-01T14:00:00Z"},
    {"event_id": 2, "user_id": 1, "timestamp": "2023-10-01T14:05:00Z"},
    {"event_id": 3, "user_id": 1, "timestamp": "2023-10-01T15:05:00Z"}
  ]
}
```

**Flow:**

1. Yes
2. No
3. Event Time Data Source (Kafka)
4. Spark Streaming
5. Event Time & Watermarks
6. Maintain State Buffer
7. Process Window Aggregations
8. Wait for Late Events
9. Output Results (Parquet)

