---
type: "practice"
difficulty: "Deep"
---

# How would you compute a rolling 7-day alert-rate-per-service feature efficiently in Spark without reprocessing the full history every night?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Batch vs Streaming Features](<../../../../02_Data_Engineering/Pipelines_and_Integration/Pipeline_Orchestration/Batch_vs_Streaming_Features/_index.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To compute a rolling 7-day alert-rate-per-service feature efficiently in Spark, we need to avoid reprocessing the entire history of data every night and instead maintain state across batches. This involves using window functions, caching, and possibly leveraging external storage for metadata.

The key idea is to process the latest batch while retaining historical alerts, which allows us to compute the rolling 7-day alert rate without full reprocessing. The solution will involve setting up a Spark pipeline that updates an in-memory or persistent store of recent alerts to ensure we always have the necessary data window available.

**Comparison:**

| rolling_window           | service      | alert_rate |
|--------------------------|--------------|------------|
| [2023-10-07, 2023-10-14] | ServiceA     | 0.5        |
| [2023-10-07, 2023-10-14] | ServiceB     | 0.66       |

**Code:**

```scala
import org.apache.spark.sql.SparkSession
import org.apache.spark.sql.functions._
import org.apache.spark.sql.streaming.Trigger

val spark = SparkSession.builder()
  .appName("Rolling Alert Rate")
  .getOrCreate()

// Assume we have a DataFrame `alerts` with columns: service, timestamp, alert
val alerts = ... // Load data from source

// Define the window size (7 days) and slide interval (1 day)
val windowSizeInDays = 7
val slideIntervalInDays = 1

val rollingWindowSpec = window(
  col("timestamp"), s"${windowSizeInDays} days", s"${slideIntervalInDays} days"
).as("rolling_window")

// Compute the alert rate per service within the rolling window
val alertRateDf = alerts.withWatermark("timestamp", "7 days")
  .groupBy(
    window(col("timestamp"), s"${windowSizeInDays} days"),
    col("service")
  )
  .agg(
    count(when(col("alert").eqNullSafe(true), true)).alias("num_alerts"),
    count("*").as("total_events")
  )
  .withColumn("alert_rate", col("num_alerts") / col("total_events"))

// Cache the result to avoid recomputation
alertRateDf.cache()
```

**Example input / output:**

```text
Input:
+-------+-------------------+--------+
|service|      timestamp    | alert  |
+-------+-------------------+--------+
|ServiceA|2023-10-01T12:00:00Z|   true  |
|ServiceB|2023-10-02T15:00:00Z| false  |
|ServiceA|2023-10-07T18:00:00Z|   true  |
|ServiceB|2023-10-14T09:00:00Z|   true  |
+-------+-------------------+--------+

Output:
+------------+-------------+----------+
|rolling_window|service      |alert_rate|
+------------+-------------+----------+
|[2023-10-07,2023-10-14]|[ServiceA]     |    0.5   |
|[2023-10-07,2023-10-14]|[ServiceB]     |    0.66  |
+------------+-------------+----------+
```

**Data:**

```json
{
  "service": ["ServiceA", "ServiceB"],
  "timestamp": [
    "2023-10-01T12:00:00Z",
    "2023-10-02T15:00:00Z",
    "2023-10-07T18:00:00Z",
    "2023-10-14T09:00:00Z"
  ],
  "alert": [true, false, true, true]
}
```

**Flow:**

1. No
2. Yes
3. Start Batch Job
4. Is Window Complete?
5. Process Latest Batch
6. Cached Alerts Updated
7. Compute Rolling Alert Rate
8. Cache Results
9. End Batch Job

