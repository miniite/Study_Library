---
type: "practice"
difficulty: "Intermediate"
---

# How would you enrich raw logs in Spark by joining them with a lookup table of service metadata?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Computing SLIs with Spark](<../../../../03_Big_Data_and_Streaming/Distributed_Processing/Big_Data_Processing/Computing_SLIs_with_Spark.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To enrich raw logs in Spark, we'll join them with a lookup table containing service metadata. This process involves several steps to ensure efficient and accurate data processing. We’ll start by understanding the problem, then proceed through the design and implementation.

Firstly, the raw logs and the service metadata are typically stored in separate sources such as Kafka topics or HDFS files. The goal is to join these datasets to add context and meaning to the log entries, enabling better analysis and insights.

Now let's break down the solution step by step:

#### Triaging the Question

The interviewer is probing your understanding of big data processing frameworks like Apache Spark, how to handle large volumes of data in real-time or batch, and efficient ways to join datasets.

#### Assumptions
The raw logs are generated continuously and stream into a Kafka topic.
Service metadata is stored in an external HDFS file or another Kafka topic.
We use Spark Streaming for real-time processing.
The log entries contain a unique identifier (e.g., service ID) that can be used to join with the metadata.
Step-by-Step Solution
Setup Environment: Ensure you have Apache Kafka and Apache Spark setup in your environment.
Read Raw Logs: Read the raw logs from Kafka using DStream or Structured Streaming API in Spark.
Read Metadata: Load the service metadata into Spark as a DataFrame.
Join Operations: Perform an efficient join operation to enrich the log data with service metadata.
Output: Write the enriched logs back to another Kafka topic or HDFS for further processing.

#### Implementation

Here’s how we can implement this using Spark Structured Streaming:

SCALA
// Step 1: Setup Spark session and read raw logs from Kafka
val spark = SparkSession.builder().appName("LogEnrichment").getOrCreate()
import spark.implicits._

val kafkaParams = Map(
  "bootstrap.servers" -> "localhost:9092",
  "group.id" -> "log-enrichment-group"
)

// Read the raw logs from Kafka
val logsDF = spark.readStream.format("kafka")
  .option("kafka.bootstrap.servers", "localhost:9092")
  .option("subscribe", "raw-logs-topic")
  .load()
  .selectExpr("CAST(value AS STRING) as value")

// Step 2: Read the metadata from HDFS or another Kafka topic
val metadataDF = spark.readStream.format("csv")
  .option("header", "true")
  .load("/path/to/service-metadata.csv")

// Step 3: Join operations
val enrichedLogsDF = logsDF.join(
  metadataDF,
  $"value.serviceId" === metadataDF.col("serviceId"),
  "left"
)

// Step 4: Output the result back to Kafka or HDFS
val query = enrichedLogsDF.writeStream
  .format("kafka")
  .option("kafka.bootstrap.servers", "localhost:9092")
  .option("topic", "enriched-logs-topic")
  .start()

query.awaitTermination()

#### Closing Summary
and Common Mistakes

This approach ensures that we efficiently enrich our raw logs with service metadata. By leveraging Spark's Structured Streaming, we can process real-time data streams effectively.

#### Common Mistakes

Inefficient Join Operations: Failing to optimize the join operation can lead to performance issues.
Incorrect Data Schemas: Mismatching schemas between the raw logs and metadata can cause failures during joins.
Resource Management: Not managing resources properly might lead to out-of-memory errors or slow processing.

By following these steps, you ensure a robust and scalable solution for enriching log data with service metadata using Apache Spark.

**Comparison:**

```
logsDF	metadataDF
value.serviceId	serviceId
otherFields	serviceName
value.serviceId	serviceName
log123	Service X
log456	Service Y
```

**Code:**

```scala
// Scala code example as provided in the implementation section.
```

**Example input / output:**

```text
Input:
- Raw logs: {"serviceId": "log123", "timestamp": ..., ...}
Metadata: {"serviceId": "serviceX", "serviceName": "Service X"}

Output:
- Enriched log: {"serviceId": "log123", "timestamp": ..., "serviceName": "Service X"}
```

**Data:**

```json
{
  "logsDF": {
    "value.serviceId": ["log123", "log456"],
    "otherFields": [/* log details */]
  },
  "metadataDF": {
    "serviceId": ["serviceX", "serviceY"],
    "serviceName": ["Service X", "Service Y"]
  }
}
```

**Flow:**

1. Yes
2. No
3. Read Raw Logs from Kafka
4. Use Structured Streaming?
5. Join with Metadata
6. Output Enriched Logs toKafka/HDFS
7. Process Batch Data Instead

