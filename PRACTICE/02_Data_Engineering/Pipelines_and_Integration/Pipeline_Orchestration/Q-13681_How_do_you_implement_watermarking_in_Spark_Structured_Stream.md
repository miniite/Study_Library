---
type: "practice"
difficulty: "Deep"
---

# How do you implement watermarking in Spark Structured Streaming to bound state for a windowed aggregation and what happens to events that arrive past the watermark?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Batch vs Streaming Features](<../../../../02_Data_Engineering/Pipelines_and_Integration/Pipeline_Orchestration/Batch_vs_Streaming_Features/_index.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To ensure bounded state in Spark Structured Streaming when implementing windowed aggregations, we can use watermarking. Watermarking allows us to handle out-of-order data and define a tolerance level beyond which late-arriving events are discarded. In this explanation, I'll cover how to implement watermarking using Apache Spark's structured streaming API with an example that demonstrates the concept.

Firstly, let’s clarify what we’re dealing with: we want to aggregate data within a sliding window while ensuring that any event arriving after a certain point (the watermark) is ignored. This approach helps in managing the state and avoiding unnecessary computations for late events.

Watermarking involves setting up a timestamp column in your DataFrame, defining the processing time of events (processingTime) to determine when the watermark will advance, and specifying how much later an event can arrive relative to its timestamp before it is considered “late” and thus discarded.

Here's a step-by-step guide on how to implement watermarking for windowed aggregations:

Setup Environment: Ensure you have Apache Spark installed with structured streaming enabled.
Create DataFrame: Load your data into a DataFrame and add a timestamp column, if not already present.
Define Watermark Duration: Set the watermark duration based on your business requirements (e.g., 5 minutes).
Use Window Aggregation: Define window specifications using the watermark to exclude late events from computations.

Let's dive into a practical example:

SCALA
import org.apache.spark.sql.functions._
import org.apache.spark.sql.SparkSession

val spark = SparkSession.builder()
  .appName("Watermarking Example")
  .getOrCreate()

// Sample data schema: (event_id, event_value, event_timestamp)
val data = Seq(
  (1L, 10.0, "2023-04-01 10:00:00"),
  (2L, 15.0, "2023-04-01 10:10:00"),
  (3L, 20.0, "2023-04-01 10:15:00"),
  (4L, 25.0, "2023-04-01 10:20:00"), // Late event
  (5L, 30.0, "2023-04-01 10:30:00")
).toDF("event_id", "event_value", "event_timestamp")

data.writeStream
  .outputMode("complete") // or "append" if you want to append new rows only
  .format("console")      // use any sink like Kafka, Console, etc.
  .option("checkpointLocation", "/path/to/checkpoint")
  .trigger(processingTime = "1 minute")
  .start()

To introduce watermarking:

SCALA
val dfWithWatermark = data.withColumn("event_timestamp", to_timestamp($"event_timestamp"))
                          .withWatermark("event_timestamp", "5 minutes") // 5-minute watermark

// Define the window for aggregation and apply watermarking
val aggregatedDF = dfWithWatermark.groupBy(
  window($"event_timestamp", "2 minutes", "1 minute"),
  $"value"
).agg(sum($"event_value").alias("sum_of_values"))

aggregatedDF.writeStream
  .outputMode("complete")
  .format("console")
  .option("checkpointLocation", "/path/to/checkpoint")
  .trigger(processingTime = "1 minute")
  .start()

In this example, events with a timestamp more than 5 minutes past the current watermark will be ignored. The aggregation is performed over a sliding window of 2 minutes with a slide interval of 1 minute.

By setting up the watermark correctly, you ensure that only relevant data within the specified time bounds are processed, thus managing state and avoiding unnecessary computations for late events.

**Comparison:**

| event_id | event_value | event_timestamp        |
|---------:|------------:|-----------------------:|
|         1 |          10 | 2023-04-01 10:00:00    |
|         2 |          15 | 2023-04-01 10:10:00    |
|         3 |          20 | 2023-04-01 10:15:00    |
|         4 |          25 | 2023-04-01 10:20:00    |
|         5 |          30 | 2023-04-01 10:30:00    |

**Code:**

```scala
import org.apache.spark.sql.functions._
import org.apache.spark.sql.SparkSession

val spark = SparkSession.builder()
  .appName("Watermarking Example")
  .getOrCreate()

// Sample data schema: (event_id, event_value, event_timestamp)
val data = Seq(
  (1L, 10.0, "2023-04-01 10:00:00"),
  (2L, 15.0, "2023-04-01 10:10:00"),
  (3L, 20.0, "2023-04-01 10:15:00"),
  (4L, 25.0, "2023-04-01 10:20:00"), // Late event
  (5L, 30.0, "2023-04-01 10:30:00")
).toDF("event_id", "event_value", "event_timestamp")

data.writeStream
  .outputMode("complete") // or "append" if you want to append new rows only
  .format("console")      // use any sink like Kafka, Console, etc.
  .option("checkpointLocation", "/path/to/checkpoint")
  .trigger(processingTime = "1 minute")
  .start()

// Define the DataFrame with watermark and perform windowed aggregation

val dfWithWatermark = data.withColumn("event_timestamp", to_timestamp($"event_timestamp"))
                          .withWatermark("event_timestamp", "5 minutes") // 5-minute watermark

// Define the window for aggregation
val aggregatedDF = dfWithWatermark.groupBy(
  window($"event_timestamp", "2 minutes", "1 minute"),
  $"value"
).agg(sum($"event_value").alias("sum_of_values"))

aggregatedDF.writeStream
  .outputMode("complete")
  .format("console")
  .option("checkpointLocation", "/path/to/checkpoint")
  .trigger(processingTime = "1 minute")
  .start()
```

**Example input / output:**

```text
Console Output:
-------------------------------------------
+-------------------------------+----------+
|           window_start         |sum_of_values|
-------------------------------------------
+-------------------------------+----------+
|[2023-04-01 10:05:00.0,2023-04-...|    30.0   |
+-------------------------------+----------+
```

**Data:**

```json
{
  "event_id": [1, 2, 3, 4, 5],
  "event_value": [10.0, 15.0, 20.0, 25.0, 30.0],
  "event_timestamp": ["2023-04-01 10:00:00", "2023-04-01 10:10:00", "2023-04-01 10:15:00", "2023-04-01 10:20:00", "2023-04-01 10:30:00"]
}
```

**Flow:**

1. No
2. Yes
3. Read Data
4. Is Event Timestamp >Watermark?
5. Compute Aggregation
6. Late Event Discarded
7. Write Result to Output

