---
type: "practice"
difficulty: "Intermediate"
---

# What does the trigger option (like trigger availableNow or a processing time) control in a streaming query?

**Difficulty:** Intermediate · **Levels:** Architect/Principal, Fresher, SDE-1/2, SDE-2/3 · [↑ Streaming Ingest Tuning](<../../../../TOOLS/Databricks/Databricks_Platform/Streaming_Ingest_Tuning.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR: The trigger option in Databricks controls when and how often data is processed based on specific timing conditions, such as "available now" or "processing time." For a Data Engineer with intermediate experience, understanding these triggers is crucial for optimizing the performance and efficiency of streaming queries.

#### Triage
In Databricks, particularly within Structured Streaming queries, the trigger option determines how data events are processed. This involves specifying when to start processing, and whether the processing should be done on a batch basis or continuously. The common triggers include "available now," which processes as soon as data is available, and "processing time," which processes at specific intervals based on the system clock.

#### First-Principles Solution

Understanding Triggers:

trigger.availableNow: This option immediately processes any incoming data events as soon as they become available in the source. It's useful when you want to ensure that all new data is processed without delay.

trigger.processingTime('interval'): Here, 'interval' specifies a fixed time period (e.g., "1 minute", "5 seconds"). Data processing occurs at regular intervals based on the system clock. This is suitable for scenarios where you need to batch process data and perform aggregations over a certain window.

#### Example

Suppose we are building a real-time dashboard that shows the number of users visiting our website every minute.
PYTHON
from pyspark.sql import SparkSession

spark = SparkSession.builder.appName("Streaming Example").getOrCreate()

df = spark.readStream.format("kafka") \
    .option("kafka.bootstrap.servers", "localhost:9092") \
    .option("subscribe", "user-visits") \
    .load()

user_visits = df.selectExpr("CAST(value AS STRING)")

# Define the processing logic
aggregated_data = user_visits.groupBy("window").count()

query = aggregated_data.writeStream \
    .outputMode("complete") \
    .trigger(processingTime="1 minute") \
    .format("console") \
    .start()

query.awaitTermination()

Advantages and Disadvantages:

Using availableNow can lead to higher resource utilization if data is streaming in frequently, but it ensures immediate processing.
processingTime helps manage resources better by batching data, making the system more efficient in terms of CPU and memory usage.

#### Common Mistakes
and Follow-Ups:

Incorrectly setting the trigger can lead to unnecessary delays or resource wastage.
Failing to consider the trade-offs between immediate processing and resource management.
Not understanding how windowing works with processingTime can result in incorrect aggregation results.

#### Closing Summary
The choice of trigger in Databricks' Structured Streaming is critical for performance optimization. Intermediate Data Engineers should be aware that trigger.availableNow processes data as soon as it becomes available, while trigger.processingTime uses regular intervals based on the system clock. Both options have their trade-offs and are best chosen based on specific use cases.

**Comparison:**

| Window End Time | Count |
|-----------------|-------|
| 2023-10-01T00:01:00Z | 5000 |
| 2023-10-01T00:02:00Z | 6000 |

**Code:**

```python
from pyspark.sql import SparkSession

spark = SparkSession.builder.appName("Streaming Example").getOrCreate()

df = spark.readStream.format("kafka") \
    .option("kafka.bootstrap.servers", "localhost:9092") \
    .option("subscribe", "user-visits") \
    .load()

user_visits = df.selectExpr("CAST(value AS STRING)")

# Define the processing logic
aggregated_data = user_visits.groupBy("window").count()

query = aggregated_data.writeStream \
    .outputMode("complete") \
    .trigger(processingTime="1 minute") \
    .format("console") \
    .start()

query.awaitTermination()
```

**Example input / output:**

```
Input: Real-time events from Kafka topic user-visits.
Output: Aggregated user visit counts printed to console every minute.
Example Dry-Run:
CODE
Copy
Window End Time | Count
-----------------------------------
2023-10-01T00:01:00Z | 5000
2023-10-01T00:02:00Z | 6000
```

**Data:**

```json
{
  "kafka_config": {
    "bootstrap.servers": "localhost:9092",
    "subscribe": "user-visits"
  }
}
```

**Flow:**

1. availableNow
2. processingTime
3. Input Kafka Topic
4. Trigger Option?
5. Process Data As Soon AsAvailable
6. Process At Regular IntervalsBased On System Clock
7. Perform Aggregation

