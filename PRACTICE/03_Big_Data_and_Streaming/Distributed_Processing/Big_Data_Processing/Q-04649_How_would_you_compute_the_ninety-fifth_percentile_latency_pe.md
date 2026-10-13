---
type: "practice"
difficulty: "Intermediate"
---

# How would you compute the ninety-fifth percentile latency per endpoint from a stream of request logs?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Latency Budgets in Streaming Pipelines](<../../../../03_Big_Data_and_Streaming/Distributed_Processing/Big_Data_Processing/Latency_Budgets_in_Streaming_Pipelines.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To compute the ninety-fifth percentile latency per endpoint from a stream of request logs, we need to process real-time data effectively using big data tools like Apache Kafka and Apache Spark. The core idea is to first consume and aggregate the logs in near-real time, then calculate percentiles for each endpoint based on the latencies. Here’s how you would approach this:

Understand the Problem: We are tasked with computing the 95th percentile latency per endpoint from incoming request logs.
Setup Data Pipeline: Use Kafka to handle the streaming data and Spark for processing the aggregated data.
Process Streams in Real Time: Use Spark Streaming or Structured Streaming to process the data as it arrives.

Let's break this down step by step:

#### Triage
What is the Question Asking?
The question asks us to calculate the 95th percentile of latency per endpoint from a stream of request logs.
This involves handling real-time streaming data, ensuring fault tolerance, and performing complex aggregations like percentiles.
Key Assumptions:
We have incoming request logs in a format such as {"endpoint": "E1", "latency": 100ms}.
Logs are ingested into Kafka for processing.
We need to maintain state per endpoint across multiple partitions and batches of data.

#### First-Principles Solution
Step 1: Setting Up the Data Pipeline
Kafka Topic: Create a Kafka topic where request logs are published by some ingestion system (like an HTTP server or another Kafka producer).
Spark Streaming/Structured Streaming Setup: Initialize Spark streaming to consume from the Kafka topic.
SCALA
// Scala example for setting up Spark Structured Streaming with Kafka
import org.apache.spark.sql.SparkSession

val spark = SparkSession.builder()
  .appName("LatencyPercentileAggregator")
  .getOrCreate()

val kafkaParams = Map(
  "bootstrap.servers" -> "<KAFKA_BROKER_LIST>",
  "group.id" -> "<GROUP_ID>"
)

val topics = Seq("<TOPIC_NAME>")
val df = spark.readStream
  .format("kafka")
  .options(kafkaParams)
  .option("subscribe", topics.mkString(","))
  .load()
Step 2: Processing the Stream
Parse Log Data: Convert the Kafka message to a structured DataFrame or Dataset.
Group by Endpoint and Calculate Latencies:
Use groupBy to aggregate by endpoint.
Use a window function like percentile_approx to compute the 95th percentile.
SCALA
// Parse JSON logs into a DataFrame
val parsedLogs = df.selectExpr("CAST(value AS STRING)")
                   .select(from_json($"value", schema).as("log"))
                   .select($"log.endpoint", $"log.latency")

// Group by endpoint and calculate percentiles
val latencyPercentileDF = parsedLogs.groupBy("endpoint")
                                    .agg(percentile_approx($"latency", 0.95).alias("p95_latency"))

// Write the result to another Kafka topic or a console sink for monitoring
val query = latencyPercentileDF.writeStream
  .format("console") // or "kafka"
  .outputMode(OutputMode.Update())
  .start()
Step 3: Monitoring and Fault Tolerance
Monitoring: Use Spark’s built-in monitoring tools (e.g., Grafana, Prometheus) to monitor the streaming application.
Fault Tolerance: Ensure that Spark jobs are resilient by setting up recovery mechanisms like checkpointing.

#### Closing Summary
+ Common Mistakes/Follow-Ups:

This solution leverages Kafka for real-time data ingestion and Spark’s structured streaming capabilities for efficient processing. Key steps include setting up a robust data pipeline, parsing logs correctly, performing group-wise aggregations, and using window functions to compute percentiles. Common mistakes might include incorrect schema definitions, not handling state management properly, or failing to set up adequate monitoring and fault tolerance mechanisms.

For follow-ups:

What measures would you take to ensure the accuracy of latency measurements?
How would you handle high-latency outliers in your calculations?

**Comparison:**

```
Endpoint	p95_Latency
E1	140ms
```

**Code:**

```scala
// Example code for setting up Spark Streaming with Kafka
import org.apache.spark.sql.SparkSession

val spark = SparkSession.builder()
  .appName("LatencyPercentileAggregator")
  .getOrCreate()

val kafkaParams = Map(
  "bootstrap.servers" -> "<KAFKA_BROKER_LIST>",
  "group.id" -> "<GROUP_ID>"
)

val topics = Seq("<TOPIC_NAME>")
val df = spark.readStream
  .format("kafka")
  .options(kafkaParams)
  .option("subscribe", topics.mkString(","))
  .load()

// Parse JSON logs into a DataFrame
val parsedLogs = df.selectExpr("CAST(value AS STRING)")
                   .select(from_json($"value", schema).as("log"))
                   .select($"log.endpoint", $"log.latency")

// Group by endpoint and calculate percentiles
val latencyPercentileDF = parsedLogs.groupBy("endpoint")
                                    .agg(percentile_approx($"latency", 0.95).alias("p95_latency"))

// Write the result to another Kafka topic or a console sink for monitoring
val query = latencyPercentileDF.writeStream
  .format("console") // or "kafka"
  .outputMode(OutputMode.Update())
  .start()
```

**Example input / output:**

```text
Input:
{"endpoint": "E1", "latency": "120ms"}

Output (after processing):
p95_latency: 140ms
```

**Data:**

```json
{
  "endpoint": "E1",
  "latency": "100ms"
}
```

**Flow:**

1. Yes
2. No
3. Receive Log
4. Kafka
5. Parse JSON
6. Group by Endpoint &Compute P95
7. Handle Error
8. Write Result to Sink

