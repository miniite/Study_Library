---
type: "practice"
difficulty: "Deep"
---

# How do you keep an observability lake's write path decoupled from its read path so a heavy forensic query storm never backpressures live ingestion?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Telemetry Lake File Layout](<../../../../04_Data_Architecture_and_Strategy/Architecture_Patterns/Observability_Data_Lake/Telemetry_Lake_File_Layout.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To ensure that an observability data lake’s write path remains decoupled from its read path, we need to design a system that can handle high volumes of ingestions without compromising the performance and reliability of query operations. The key is to implement a robust architecture where ingestion and querying are handled independently but still interact seamlessly. This involves using message queues, batching strategies, and leveraging modern data storage technologies.

#### Triaging the Question

The question revolves around maintaining the decoupling between the write path (ingestion) and read path (querying) in an observability data lake. The core challenge is to prevent high query load from backpressuring the ingestion process. This involves understanding the nature of both paths, identifying potential bottlenecks, and designing a system that can handle them independently while ensuring data integrity and accessibility.

#### Assumptions
Ingestion: Data from various sources (e.g., logs, metrics) is ingested into the data lake.
Querying: Users run complex queries to analyze historical data for debugging, root cause analysis, or compliance purposes.
Performance Requirements: The system must handle high write rates without impacting read performance and vice versa.

#### First-Principles Solution
Message Queues (MQ): Use an MQ like Apache Kafka or Amazon Kinesis to decouple the ingestion and querying paths. Ingested data is first published into a queue, where it can be processed by multiple consumers independently.
Batching: Implement batching strategies to reduce the number of write operations. This can significantly reduce the load on the storage layer.
Provisioning: Ensure adequate compute resources for both ingestion and querying paths. Use auto-scaling and managed services to handle variable workloads.
Data Lake Storage: Utilize scalable, durable storage solutions like Amazon S3 or Azure Data Lake Storage Gen2, which can handle high throughput writes and reads without degrading performance.
Query Optimization: Optimize queries by using indexes, partitioning, and caching to reduce read latency.
Detailed Implementation

Ingestion Pipeline:

Source systems publish data to an MQ topic.

#### Cons
umers from the ingestion path (e.g., Spark Streaming) consume these messages and write them into a distributed file system or cloud storage.
Implement batching strategies where multiple messages are aggregated before being written to the storage layer. This can reduce I/O operations and improve efficiency.

Storage Layer:

Use a scalable, durable storage solution like Amazon S3 or Azure Data Lake Storage Gen2. These solutions provide high throughput for both write and read operations.
Implement lifecycle policies to automatically transition data from hot to cold tiers based on access patterns.

Query Path:

Queries are executed against the stored data using analytics tools (e.g., Athena, Presto). These queries can be optimized with indexing, partitioning, and caching mechanisms.
Use a query optimizer to generate efficient execution plans that leverage indexes and partitions to minimize read latency.

#### Closing Summary

By decoupling the ingestion and querying paths through message queues and batch processing strategies, we ensure that heavy forensic query storms do not impact live data ingestion. This approach provides a scalable, reliable solution for handling high volumes of data while maintaining optimal performance for both writes and reads.

#### Common Mistakes
and Follow-Ups
Not Implementing Batching: Failing to implement batching can lead to excessive I/O operations and degrade write performance.
Insufficient Storage Scalability: Using a storage solution that cannot handle high write rates or read queries can introduce bottlenecks.
Overloading the Query Path: Not optimizing queries can result in slow response times, even with well-designed ingestion paths.

It’s crucial to monitor both paths for performance issues and tune accordingly. Effective monitoring and logging will help identify bottlenecks and ensure that the system remains performant under varying workloads.

**Comparison:**

```
Batch Size	Time Saved (ms)
1	8,000
100	2,000
1,000	500
```

**Code:**

```python
# Example of batching data before writing to storage using PySpark

from pyspark.sql import SparkSession

spark = SparkSession.builder.appName("IngestionBatching").getOrCreate()

def process_messages(messages):
    # Process and aggregate messages
    aggregated_data = []
    for message in messages:
        # Aggregation logic here
        aggregated_data.append(process_message(message))

    return aggregated_data

# Example ingestion function
def ingest_data_into_storage(messages, storage_path):
    batched_data = process_messages(messages)
    df = spark.createDataFrame(batched_data)
    df.write.parquet(storage_path)
```

**Example input / output:**

```text
Input:
- Data messages from various sources published to kafka-topic

Output:
- Aggregated data written to S3://observability-datalake/batch_2023-10-05.parquet
- Queries executed efficiently against the stored data with optimized indexes and partitions.
```

**Data:**

```json
{
  "dataLakeConfig": {
    "storage": "s3://observability-datalake",
    "batchSize": 1000,
    "queue": "kafka-topic"
  },
  "queryOptimization": {
    "indexes": ["timestamp", "logID"],
    "partitions": 10
  }
}
```

**Flow:**

1. Yes
2. No
3. Source
4. Kafka Topic
5. Batch?
6. Process Messages
7. Write to S3
8. Queue Message
9. Spark Streaming Consumer
10. Write Aggregated Data to S3

