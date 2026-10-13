---
type: "practice"
difficulty: "Expert"
---

# Explain watermarking in Spark Structured Streaming from Python: how does it bound state for late data, and what happens to events that arrive after the watermark?

**Difficulty:** Expert · **Levels:** SDE-1/2, SDE-2/3 · [↑ Large Data Migrations in Python](<../../../../TOOLS/Python/Python_Language/Large_Data_Migrations_in_Python.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Watermarking in Apache Spark's Structured Streaming is a mechanism used to handle event time processing and manage out-of-order (OOO) events effectively. When you use watermarking, you define a timestamp column that represents when an event occurred, and you specify how far behind the current time stream of events can be processed before they are considered late. This allows for accurate computation of aggregates and joins even with delays in data.

In this context, "late data" refers to events that arrive after their expected timestamps. The watermarking mechanism ensures that Spark does not process an event until it is confident that all earlier events have arrived. Here’s a detailed explanation:

#### Triaging the Question

The core of this question involves understanding how watermarking in Structured Streaming manages late arriving events and how state is bounded to handle such scenarios from a Python perspective.

Timestamp Column: First, you need to define a timestamp column that represents when each event occurred.
Watermark Definition: You specify a watermark using the withWatermark method on your DataFrame or Dataset. This defines the maximum amount of time an OOO event can be late before it is considered as late data and dropped.

#### First-Principles Solution
Defining Timestamp Column: The timestamp column is critical because it tells Spark when each event occurred. For example, if you have a log stream with timestamps, this becomes your reference for processing events.
Specifying Watermark Duration: You use the withWatermark method on your DataFrame to specify how late an event can be before it is considered as out-of-order. This duration is defined in seconds or milliseconds.

#### Example
Code
PYTHON
from pyspark.sql import SparkSession
from pyspark.sql.functions import col, window

spark = SparkSession.builder.appName("WatermarkingExample").getOrCreate()

# Assume you have a DataFrame `df` with columns `value` and `timestamp`
df = spark.readStream.format('kafka').option('kafka.bootstrap.servers', 'localhost:9092').option('subscribe', 'input-topic').load().selectExpr("CAST(value AS STRING)").withColumn("timestamp", col("timestamp"))

# Define the watermark
windowSpec = window(col("timestamp"), "5 minutes", "1 minute")
watermarked_df = df.withWatermark("timestamp", "10 minutes")

# Now, you can perform operations like aggregations or joins on `watermarked_df`
Handling Late Data: When an event arrives late (i.e., after the watermark), Spark will only process it if all earlier events have arrived by that point. If not, it drops the late event to ensure consistency.

#### Closing Summary

In summary, watermarking in Structured Streaming allows you to handle late data by bounding state and ensuring that only relevant events are processed. This is crucial for maintaining correctness in applications dealing with real-time streams where events can be delayed.

#### Common Mistakes/Follow-Ups
Incorrect Watermark Duration: Choosing an incorrect watermark duration can lead to either too many or too few events being treated as late.
Failing to Define a Timestamp Column: Without a proper timestamp column, Spark cannot determine the order of events and will not apply watermarking correctly.
Overlooking Event Time Processing: Focusing solely on processing data in the order it arrives without considering event time can lead to incorrect results.

To further explore this concept, you might want to look into scenarios where there are multiple watermarks or handling specific failure modes like retries of late events.

**Comparison:**

| Event | Timestamp |
|-------|-----------|
| event1| 1632945600|
| event2| 1632945700|
| event3| 1632945800|

**Code:**

```python
from pyspark.sql import SparkSession
from pyspark.sql.functions import col, window

# Example code snippet
spark = SparkSession.builder.appName("WatermarkingExample").getOrCreate()

df = spark.readStream.format('kafka').option('kafka.bootstrap.servers', 'localhost:9092').option('subscribe', 'input-topic').load().selectExpr("CAST(value AS STRING)").withColumn("timestamp", col("timestamp"))

windowSpec = window(col("timestamp"), "5 minutes", "1 minute")
watermarked_df = df.withWatermark("timestamp", "10 minutes")

# Perform operations
```

**Example input / output:**

```text
# Example input and output for a simple aggregation with watermarking
watermarked_df.groupBy(window("timestamp", "5 minutes", "1 minute")).count().show()
# Output:
# +-----------------------+-----+
# |          start         | count|
# +-----------------------+-----+
# |[2021-10-01 06:00:00,2021-10-01 06:05:00)|     2|
# +-----------------------+-----+
```

**Data:**

```json
{
  "input": [
    {"value": "event1", "timestamp": 1632945600},
    {"value": "event2", "timestamp": 1632945700},
    {"value": "event3", "timestamp": 1632945800}
  ]
}
```

**Flow:**

1. Yes
2. No
3. Event arrives
4. Is within watermark?
5. Process event
6. Discard late event
7. Compute aggregation/join
8. No action

