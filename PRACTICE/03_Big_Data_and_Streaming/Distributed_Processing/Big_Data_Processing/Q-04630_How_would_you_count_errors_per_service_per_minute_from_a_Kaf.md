---
type: "practice"
difficulty: "Intermediate"
---

# How would you count errors per service per minute from a Kafka log stream using Spark?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Computing SLIs with Spark](<../../../../03_Big_Data_and_Streaming/Distributed_Processing/Big_Data_Processing/Computing_SLIs_with_Spark.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To count errors per service per minute from a Kafka log stream using Spark, we need to process the incoming data in real-time while maintaining stateful operations. The core idea is to use Apache Kafka as the input source and Apache Spark Streaming (or Structured Streaming) for processing. Here’s how you can approach this problem:

TL;DR: We will set up a pipeline where we consume logs from Kafka, filter out error messages, categorize them by service, and aggregate their counts on a per-minute basis.

#### Triaging the Question

The question is asking for real-time data processing to count errors for different services.
This involves understanding how Apache Kafka works as a distributed messaging system, and how Spark Streaming can be used to process this stream of events in real time.
Key assumptions include that the log messages are structured (e.g., JSON) with fields such as service and error.

#### First Principles Solution

Step 1: Kafka Setup: Kafka acts as a reliable messaging system for ingesting logs from various sources.
Log Source
Kafka Topic
Step 2: Spark Streaming Setup: Use Apache Spark to process the stream of events. We will use Structured Streaming, which allows us to write SQL-like queries on streaming data.
Kafka Topic
Spark Streaming Application
countErrorsPerService
Step 3: Data Processing: In Spark Streaming, we define a schema for our incoming log messages and use foreachBatch to perform the aggregation.
PYTHON
from pyspark.sql import SparkSession
from pyspark.sql.functions import col, window, group_by, count

spark = SparkSession.builder.appName("ErrorCountApp").getOrCreate()

# Read data from Kafka topic with structured streaming
df = spark.readStream.format("kafka") \
    .option("kafka.bootstrap.servers", "localhost:9092") \
    .option("subscribe", "log_stream") \
    .load() \
    .selectExpr("CAST(value AS STRING) as log")

# Parse the JSON logs
parsed_df = df.select(from_json(col("log"), schema).alias("data")) \
    .select("data.*")

# Filter errors and group by service
error_counts = parsed_df.filter(col("message").contains("ERROR")) \
    .withColumn("timestamp", col("ts")).window(window_spec) \
    .groupBy("service", window("timestamp", "1 minute")) \
    .agg(count("*").alias("error_count"))

# Write the results to a console for testing
query = error_counts.writeStream.format("console").start()

query.awaitTermination()
Step 4: Output: The output will be streamed errors per service per minute.

#### Closing Summary

We've set up a pipeline that uses Kafka to ingest logs and Spark Streaming to process them in real-time.
This approach ensures we can handle large volumes of data efficiently and maintain state for aggregations.

#### Common Mistakes/Follow-Ups

#### Common mistakes
include not properly handling schema evolution, incorrect windowing, or failure to manage offsets correctly.

#### Follow-ups
could involve discussing how to handle complex error messages, ensuring fault tolerance, and optimizing performance with caching or co-location strategies.

**Comparison:**

| service | timestamp         | error_count |
|---------|------------------|-------------|
| web     | 2023-10-05 14:30  | 1           |
| web     | 2023-10-05 14:31  | 2           |

**Code:**

```python
from pyspark.sql import SparkSession
from pyspark.sql.functions import col, window, group_by, count

spark = SparkSession.builder.appName("ErrorCountApp").getOrCreate()

# Define the schema for our log messages (assuming JSON format)
schema = "service STRING, message STRING, ts TIMESTAMP"

# Read data from Kafka topic with structured streaming
df = spark.readStream.format("kafka") \
    .option("kafka.bootstrap.servers", "localhost:9092") \
    .option("subscribe", "log_stream") \
    .load() \
    .selectExpr("CAST(value AS STRING) as log")

# Parse the JSON logs
parsed_df = df.select(from_json(col("log"), schema).alias("data")) \
    .select("data.*")

# Filter errors and group by service
window_spec = window("ts", "1 minute")
error_counts = parsed_df.filter(col("message").contains("ERROR")) \
    .withColumn("timestamp", col("ts")).window(window_spec) \
    .groupBy("service", window("timestamp", "1 minute")) \
    .agg(count("*").alias("error_count"))

# Write the results to a console for testing
query = error_counts.writeStream.format("console").start()

query.awaitTermination()
```

**Example input / output:**

```

Input:

JSON
Copy
[
    {"service": "web", "message": "ERROR: failed to handle request", "ts": "2023-10-05 14:30:00"},
    {"service": "api", "message": "INFO: successful request", "ts": "2023-10-05 14:31:00"},
    {"service": "web", "message": "ERROR: failed to handle request", "ts": "2023-10-05 14:31:00"}
]

Output:

PLAINTEXT
Copy
+-------+-------------------+----------+
| service|          timestamp |error_count|
+-------+-------------------+----------+
|   web  |[2023-10-05 14:30] |        1  |
|   web  |[2023-10-05 14:31] |        2  |
+-------+-------------------+----------+
```

**Data:**

```json
{
  "service": "web",
  "message": "ERROR: failed to handle request",
  "ts": "2023-10-05T14:30:00Z"
}
```

**Flow:**

1. Log Source
2. Kafka Topic
3. Spark Streaming Application
4. countErrorsPerService
5. Console Output

