---
type: "practice"
difficulty: "Deep"
---

# You want at-least-once from Kafka to a warehouse using Spark; where exactly is the offset committed relative to the write, and how do you make the sink idempotent to compensate?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Exactly Once Beyond Kafka](<../../../../TOOLS/Apache_Kafka/Kafka_Core/Exactly_Once_Beyond_Kafka/_index.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To ensure at-least-once semantics in Apache Kafka when writing data to a warehouse using Spark, it's crucial to understand where offsets are committed relative to writes and how to make the sink idempotent. The key steps involve properly configuring your Spark job to commit offsets after successful writes to the warehouse and ensuring that the write operation is idempotent.

#### Triaging the Question

The question asks about the offset committing strategy in Kafka with a Spark-based sink, ensuring at-least-once delivery semantics. It also seeks understanding of how to make the data writing process idempotent. The core concepts here are:

Offset Committing: Where and when offsets are committed.
Idempotency: Ensuring that writes can be performed multiple times without causing duplicates or inconsistencies in the warehouse.

#### First Principles Solution

To achieve at-least-once semantics, Spark must commit Kafka offsets only after successfully writing data to the warehouse. This requires careful handling of offsets and ensuring idempotent writes. Here’s a step-by-step breakdown:

Kafka Consumer Configuration:

Use setAutoOffsetReset to specify how offsets should be handled if they are not found.
Configure maxOffsetsPerTrigger and stopOffsetInclusive to control the number of records fetched in each batch.

Spark Streaming & Kafka Integration:

Utilize Spark’s structured streaming capabilities to process data from Kafka topics.
Ensure that writes to the warehouse are idempotent by using a unique identifier for each message, such as a timestamp or UUID.

Idempotent Write Strategy:

Implement logic to check if records already exist in the warehouse before writing.
Use upsert operations where necessary to update existing records without duplicating them.
Detailed Steps

Configure Kafka Consumer:

SCALA
val kafkaParams = Map(
  "bootstrap.servers" -> "localhost:9092",
  "group.id" -> "test-group",
  "auto.offset.reset" -> "earliest"
)

val topics = List("input-topic")

// Create a Kafka source using structured streaming API
val df = spark.readStream
               .format("kafka")
               .options(kafkaParams)
               .load(topics: _*)

Extract Values and Process Data:

SCALA
import org.apache.spark.sql.functions._

// Extract the value column from the Kafka messages
val processedDF = df.selectExpr("CAST(value AS STRING)")
                    .withColumn("key", col("key"))
                    .withColumn("timestamp", col("offset"))

// Process data as needed, for example, parse JSON or extract fields
val finalDF = processedDF.withColumn("parsedData", from_json($"value", schema))
                         .selectExpr("parsedData.*")

Write to Warehouse with Idempotency:

SCALA
// Ensure the write operation is idempotent by using upsert logic
val sinkDf = finalDF.writeStream
                     .format("delta")  // Assuming Delta Lake for idempotence
                     .option("checkpointLocation", "/path/to/checkpoint")
                     .trigger(ProcessingTime("10 seconds"))
                     .start("/path/to/table")

// Optionally, use a unique key to ensure records are written only once per entity
sinkDf.foreachBatch { (batchDF: DataFrame, batchId: Long) =>
  val uniqueKey = "unique_key"
  val upsertQuery = s"""
    MERGE INTO warehouse.table t
    USING ${batchDF.queryName} s ON t.unique_key = s.unique_key
    WHEN MATCHED THEN UPDATE SET *
    WHEN NOT MATCHED THEN INSERT *
  """
  spark.sql(upsertQuery)
}

#### Closing Summary
and Common Mistakes/Follow-Ups

To summarize, the key is to ensure that offsets are committed after successful writes to the warehouse. This can be achieved by leveraging Spark’s structured streaming capabilities and configuring Kafka consumers appropriately. Additionally, making the sink idempotent ensures that duplicates do not occur.

#### Common Mistakes

Committing offsets before writing data.
Not handling unique keys properly, leading to duplicate records.
Failing to use upsert logic in the write operation.

#### Follow-Ups

How does this solution scale with a large number of topics and partitions?
What are potential failure modes and how can they be mitigated?
How would you integrate error handling into this process?

**Code:**

```scala
val kafkaParams = Map(
  "bootstrap.servers" -> "localhost:9092",
  "group.id" -> "test-group",
  "auto.offset.reset" -> "earliest"
)

val topics = List("input-topic")

// Create a Kafka source using structured streaming API
val df = spark.readStream
               .format("kafka")
               .options(kafkaParams)
               .load(topics: _*)

import org.apache.spark.sql.functions._
df.selectExpr("CAST(value AS STRING)")
  .withColumn("key", col("key"))
  .withColumn("timestamp", col("offset"))
  .writeStream
  .format("delta")  // Assuming Delta Lake for idempotence
  .option("checkpointLocation", "/path/to/checkpoint")
  .trigger(ProcessingTime("10 seconds"))
  .start("/path/to/table")

val uniqueKey = "unique_key"
val upsertQuery = s"""
  MERGE INTO warehouse.table t
  USING ${batchDF.queryName} s ON t.unique_key = s.unique_key
  WHEN MATCHED THEN UPDATE SET *
  WHEN NOT MATCHED THEN INSERT *
"""

spark.sql(upsertQuery)
```

