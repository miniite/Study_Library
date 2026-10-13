---
type: "practice"
difficulty: "Deep"
---

# You need to partition a Kafka-fed data-lake table by event date but Kafka delivers events with up to a day of lateness; how do you avoid rewriting old partitions on every batch?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Partitions and Ordering](<../../../../TOOLS/Apache_Kafka/Kafka_Core/Partitions_and_Ordering.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To handle late-arriving data in a Kafka-fed data lake, partitioning by event date is crucial but challenging due to potential latency. The key idea here is to maintain the integrity of existing partitions while processing new events that might arrive with delays. We'll leverage Kafka's offset management and partitioning strategies to ensure we only process new or updated data without overwriting historical partitions.

The question is essentially asking for a robust strategy to handle late-arriving messages in a streaming pipeline. The interviewer is probing your understanding of Kafka, partitioning techniques, and how to manage state in a distributed system. Assumptions include that the event date is available within the message metadata or payload, and we are using a batch processing approach with Apache Spark, Flink, or similar.

To solve this problem, let's start from first principles:

Understand Kafka Partitioning: Each message in Kafka is assigned to a specific partition based on its key (if provided). This allows for parallel processing of data.
Event Date as Key: We will use the event date as part of the key or directly as the partition column if we're using Kafka's built-in partitioning strategy.
Handling Late Events: Late events can arrive after the time they were supposed to, which means they should be processed and stored in their correct partitions without affecting existing data.
Step-by-Step Solution
Define Partition Key: Use the event date as part of the key or column for partitioning.
Batch Processing Framework: Utilize a batch processing framework like Apache Spark with Kafka connectors to read from Kafka topics.
State Management: Maintain state between batches using the OffsetManager to ensure that only new and updated data is processed.

Here's an outline of how you can implement this in Spark:

SCALA
// Define the schema for the input and output data
case class EventRecord(id: String, eventDate: LocalDate, value: Double)

// Create a Kafka consumer configuration
val kafkaConfig = Map(
  "bootstrap.servers" -> "localhost:9092",
  "group.id" -> "data-lake-group",
  "enable.auto.commit" -> false,
  "auto.offset.reset" -> "earliest"
)

// Define the Spark session and read from Kafka topic
import spark.implicits._
val df = spark.readStream
  .format("kafka")
  .options(kafkaConfig)
  .load()
  .selectExpr("CAST(value AS STRING)")
  .as[String]
  .map(_.parseJson.toString())
  .map(json => json.as[EventRecord])
  .withWatermark("eventDate", "1 day")

// Define the processing logic
val processedDF = df.groupBy($"eventDate".year, $"eventDate".month, $"eventDate".day)
  .agg(avg("value").alias("averageValue"))

// Write to a data lake partitioned by event date
processedDF.writeStream
  .format("parquet")
  .partitionColumn("eventDate")
  .outputMode(OutputMode.Append())
  .option("path", "data-lake-path")
  .start()

#### Closing Summary

In summary, the solution involves using the event date as a key or partition column to ensure that late events are processed correctly without affecting existing partitions. This approach leverages Kafka's built-in mechanisms for managing offsets and Spark’s stateful processing capabilities.

#### Common Mistakes
and Follow-Ups
Overwriting Old Partitions: A common mistake is forgetting to use the outputMode set to Append() in Spark, which ensures that only new data is written.
Offset Management: Failing to manage offsets properly can lead to duplicate or missed events. Using Kafka's offset management features and stateful processing are crucial.

#### Performance Considerations
For large-scale systems, consider the performance implications of handling late events, especially if they significantly increase the number of partitions.

**Comparison:**

| year | month | day | averageValue |
|------|-------|-----|-------------|
| 2023 | 10    | 1   | 100.5       |

**Code:**

```scala
case class EventRecord(id: String, eventDate: LocalDate, value: Double)
import spark.implicits._
val kafkaConfig = Map(
  "bootstrap.servers" -> "localhost:9092",
  "group.id" -> "data-lake-group",
  "enable.auto.commit" -> false,
  "auto.offset.reset" -> "earliest"
)

val df = spark.readStream
  .format("kafka")
  .options(kafkaConfig)
  .load()
  .selectExpr("CAST(value AS STRING)")
  .as[String]
  .map(_.parseJson.toString())
  .map(json => json.as[EventRecord])
  .withWatermark("eventDate", "1 day")

val processedDF = df.groupBy($"eventDate".year, $"eventDate".month, $"eventDate".day)
  .agg(avg("value").alias("averageValue"))

processedDF.writeStream
  .format("parquet")
  .partitionColumn("eventDate")
  .outputMode(OutputMode.Append())
  .option("path", "data-lake-path")
  .start()
```

**Example input / output:**

```

Input:

CODE
Copy
{ "id": "12345", "eventDate": "2023-10-01", "value": 100.5 }

Output:

CODE
Copy
{
  "year": 2023,
  "month": 10,
  "day": 1,
  "averageValue": 100.5
}
```

**Data:**

```json
{
  "id": "12345",
  "eventDate": "2023-10-01",
  "value": 100.5
}
```

**Flow:**

1. Yes
2. No
3. Read from Kafka
4. Is event date late?
5. Apply watermark and processnew events
6. Process existing data
7. Write to Data Lake (partitionedby event date)
8. Write to Data Lake (partitionedby event date)

