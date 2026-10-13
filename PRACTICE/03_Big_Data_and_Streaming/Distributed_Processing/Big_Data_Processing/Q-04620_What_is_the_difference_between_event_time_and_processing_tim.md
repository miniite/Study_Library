---
type: "practice"
difficulty: "Basic"
---

# What is the difference between event time and processing time and why does it matter for late-arriving logs?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Structured Streaming Micro-batches](<../../../../03_Big_Data_and_Streaming/Distributed_Processing/Big_Data_Processing/Structured_Streaming_Micro-batches.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In big data processing, especially when dealing with streams of events like log entries, two fundamental concepts—event time and processing time—are crucial to understanding how systems handle incoming data. Event time refers to the actual moment an event occurs in the real world, whereas processing time is the moment at which a system processes that event. The difference between these two times can significantly impact how late-arriving logs are handled, making it essential for any AIOps fresher or mid-level engineer to grasp this concept.

To begin with, let's restate what the question is probing: the core of this question lies in understanding the distinction between event time and processing time and why this distinction matters in scenarios involving late-arriving logs. We will assume a basic familiarity with Kafka for stream processing and Apache Spark for batch processing, as these are common tools used in big data environments.

Now, let's break down the solution step by step:

Understanding Event Time vs Processing Time:

Event Time: This is the time at which an event actually occurred. For example, if a log entry records a user login on a web application, the event time would be when the user first accessed the server.
Processing Time: This is the time at which a system processes and ingests this event. In real-time systems like Kafka or Spark Streaming, processing can occur either in parallel with the actual event occurrence (event time) or after the data has been collected (processing time).

Why It Matters for Late-Arriving Logs:

Late-arriving Data: Logs might arrive late due to network delays, server issues, or other factors. If these logs are processed strictly in processing time without considering their actual event times, the resulting analysis could be skewed because the events would appear to have occurred much later than they actually did.
Impact on Analysis: Consider a scenario where user sessions need to be accurately tracked over multiple minutes or hours. If late-arriving logs are not handled properly, the session durations might be underestimated or misinterpreted.

Handling Late-Arriving Logs:

Watermarking and Event Time Processing (EDR): Using Apache Spark with its Structured Streaming framework allows setting up watermarking to handle late data by specifying a window of allowed lateness for events.
Exactly Once Semantics: Kafka ensures at-least-once delivery, meaning that even if a message is late, it will be processed. This can help in ensuring that all logs are eventually accounted for.

#### Example Scenario
Suppose we have a system processing user logins to detect unusual login patterns:

An event time-based approach would track the actual moments of each login.
A processing time-based approach might group logins by timestamp buckets, leading to potential inaccuracies if some logs arrive late.

#### Closing Summary
and Common Mistakes:

The key takeaway is that understanding both event time and processing time helps in designing robust data pipelines capable of handling late-arriving data gracefully.
A common mistake is assuming all events are processed immediately, which can lead to inaccurate analyses or misinterpretations.

**Comparison:**

| Time | User ID | Event Time (UTC) |
|------|---------|------------------|
| 14:00:00Z | u1      | 2023-09-15T14:00:00Z |
| 14:05:00Z | u2      | 2023-09-15T14:05:00Z |  # Late arrival
| 14:10:00Z | u3      | 2023-09-15T14:10:00Z |

**Code:**

```python
# Example Python code for setting up a Spark Streaming job with event-time processing
from pyspark.sql import SparkSession
from pyspark.sql.functions import from_json, col, window

spark = SparkSession.builder \
    .appName("EventTimeProcessingExample") \
    .getOrCreate()

logs_df = spark.readStream.format('kafka') \
    .option('kafka.bootstrap.servers', 'localhost:9092') \
    .option('subscribe', 'user_logs') \
    .load() \
    .selectExpr("CAST(value AS STRING)")  # Assuming logs are JSON

parsed_logs = logs_df.withColumn('data', from_json(col('value'), schema)).selectExpr("data.*")

# Setting up a watermark to handle late-arriving data
output = parsed_logs.groupBy(window(col('timestamp'), '10 minutes')).agg(count('*').alias('login_count'))

query = output.writeStream.format('console').start()
query.awaitTermination()
```

**Example input / output:**

```text
Input Logs:
{"user_id": "u1", "timestamp": "2023-09-15T14:00:00Z"}
{"user_id": "u2", "timestamp": "2023-09-15T14:05:00Z"}  # Late arrival
{"user_id": "u3", "timestamp": "2023-09-15T14:10:00Z"}

Output (Processing Time):
Window [2023-09-15 14:00:00, 2023-09-15 14:10:00):
u1: 1
u2: 1

Correct Output (Event Time):
Window [2023-09-15 14:00:00, 2023-09-15 14:10:00):
u1: 1
u2: 0
```

**Data:**

```json
{
  "logs": [
    {"user_id": "u1", "timestamp": "2023-09-15T14:00:00Z"},
    {"user_id": "u2", "timestamp": "2023-09-15T14:05:00Z"},  # Late arrival
    {"user_id": "u3", "timestamp": "2023-09-15T14:10:00Z"}
  ]
}
```

**Flow:**

1. Yes
2. No
3. Log Entry
4. Is Late?
5. Handle Late Arrival
6. Process Normally
7. Update State with DelayedData
8. Process Data in Real Time

