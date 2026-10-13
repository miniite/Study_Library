---
type: "practice"
difficulty: "Intermediate"
---

# How does a batch Spark job fit into the ELT idea where you load raw data first and transform it later?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Structured Streaming Micro-batches](<../../../../03_Big_Data_and_Streaming/Distributed_Processing/Big_Data_Processing/Structured_Streaming_Micro-batches.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Summary: A batch Spark job in an ELT pipeline processes raw data stored in Kafka or another source, performing transformations to prepare the data for storage in a more structured format. This involves reading data from Kafka topics, applying necessary transformations and aggregations, and writing the transformed data back to storage systems like HDFS.

#### Triaging the Question
The key aspects of this question revolve around understanding how Spark fits into an ELT (Extract, Load, Transform) workflow. Specifically, it probes your knowledge of Kafka as a source for raw data, Spark’s capabilities in batch processing, and the steps involved in transforming and storing transformed data.

#### Assumptions

We are working with a system where raw data is first extracted from sources such as Kafka.
The processed data needs to be stored in a more structured format (e.g., HDFS) for further analysis.
Spark is used for batch processing due to its efficient handling of large datasets.

Step-by-Step Solution:

Extracting Data: Raw data from Kafka topics are consumed using the KafkaUtils API in Spark. This step involves setting up a Spark Streaming context and defining how often data should be polled (batch interval).

SCALA
// Example in Scala
val kafkaParams = Map("metadata.broker.list" -> "localhost:9092")
val topics = Set("rawDataTopic")
val stream = KafkaUtils.createDirectStream[String, String, StringDecoder, StringDecoder] (
 ssc, kafkaParams, topics)

Processing Data: The extracted data is then processed using Spark’s DataFrame or RDD APIs for transformations such as filtering, mapping, and aggregations.

SCALA
// Example transformation in Scala
val transformedData = stream.map(record => (record.key(), record.value()))
  .filter(_._1.startsWith("prefix"))  // Filter out unwanted data
  .map(_._2.toUpperCase())            // Convert text to uppercase for example

Writing Data: The transformed data is then written back to a storage system like HDFS, often using Spark’s save functions.

SCALA
// Example write operation in Scala
transformedData.saveAsTextFile("hdfs://localhost:9080/processed_data")

#### Closing Summary
and Common Mistakes/Follow-Ups: A batch Spark job effectively fits into an ELT pipeline by leveraging its robust data processing capabilities. Key takeaways include the importance of setting up correct Kafka configurations, understanding Spark’s data structures like DataFrame and RDD for efficient transformations, and ensuring proper error handling during writes to storage systems.

#### Common mistakes
might include improper configuration of Kafka parameters leading to data loss or delays, or not properly handling exceptions in Spark jobs that can lead to partial processing. Follow-up questions could explore the choice between using DataFrames vs. RDDs, dealing with stateful operations, and optimizing performance for large-scale data processing.

**Comparison:**

```
Kafka Parameter	Value
metadata.broker.list	localhost:9092
Transformation Step	Description
Filter	Remove records not starting with "prefix"
Map	Convert all text to uppercase
```

**Code:**

```scala
// Example of reading from Kafka and writing processed data
val kafkaParams = Map("metadata.broker.list" -> "localhost:9092")
val topics = Set("rawDataTopic")

val stream = KafkaUtils.createDirectStream[String, String, StringDecoder, StringDecoder](
 ssc, kafkaParams, topics)

// Processing the data
val transformedData = stream.map(record => (record.key(), record.value()))
  .filter(_._1.startsWith("prefix"))
  .map(_._2.toUpperCase())

// Writing to HDFS
transformedData.saveAsTextFile("hdfs://localhost:9080/processed_data")
```

**Example input / output:**

```text
Input:
Kafka Topic: rawDataTopic

Output:
HDFS Path: hdfs://localhost:9080/processed_data
Contents:
- UPPERCASED_VALUE1
- UPPERCASED_VALUE2
```

**Data:**

```json
{
  "kafkaParams": {
    "metadata.broker.list": "localhost:9092"
  },
  "topics": ["rawDataTopic"],
  "transformedData": [
    { "key": "prefix1", "value": "UPPERCASED_VALUE1" },
    { "key": "prefix2", "value": "UPPERCASED_VALUE2" }
  ]
}
```

**Flow:**

1. Extract from Kafka
2. Filter Data
3. Transform Data
4. Write to HDFS

