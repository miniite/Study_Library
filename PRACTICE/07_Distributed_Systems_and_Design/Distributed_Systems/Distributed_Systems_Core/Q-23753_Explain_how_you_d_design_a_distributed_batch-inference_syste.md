---
type: "practice"
difficulty: "Deep"
---

# Explain how you'd design a distributed batch-inference system to score 5 billion rows nightly within a fixed window, handling partition skew, making it idempotent so a mid-job failure resumes safely, and co-scheduling it so it doesn't starve real-time serving of GPUs.

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Workload Isolation on Shared Clusters](<../../../../07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Mixed_Version_Fleets/Workload_Isolation_on_Shared_Clusters.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To design a distributed batch-inference system that scores 5 billion rows nightly within a fixed window, while handling partition skew, ensuring idempotency in case of mid-job failures, and co-scheduling it to not starve real-time serving of GPUs, we need to consider multiple layers of architecture, from data ingestion and processing to fault tolerance and resource management. The core idea is to break down the problem into manageable components and ensure each component can handle large-scale distributed systems effectively.

Firstly, let's restate what the question is really asking: We need a scalable system that efficiently processes 5 billion rows per night without overloading real-time serving resources and handles failures gracefully. The interviewer might be probing our understanding of distributed architectures, fault tolerance strategies, and co-scheduling techniques.

We will assume:

Data arrives in batches.
A nightly window is defined for processing.
Real-time inference requests can arrive at any time during the day but must not starve real-time serving resources.
The system should handle partition skew effectively.
Step-by-Step Design
1. Data Ingestion

We need a robust data ingestion layer that can handle high throughput and ensure data consistency. We will use Apache Kafka for its distributed architecture, fault tolerance, and support for streaming data. Each row of data is ingested into a topic, and consumers (worker nodes) pull data from this topic.

2. Data Processing

For the actual batch processing, we will leverage Apache Spark, which is well-suited for large-scale data processing with built-in fault tolerance mechanisms. We'll partition the data based on some key that ensures even distribution across worker nodes to handle partition skew effectively.

3. Fault Tolerance and Idempotency

To ensure idempotence in case of mid-job failures, we will implement a commit mechanism using Apache Kafka’s exactly-once semantics. Each processed row will be written back into Kafka with a unique identifier (transaction ID). The worker nodes will check the transaction log before processing to avoid reprocessing rows.

4. Co-Scheduling Real-Time Serving

To ensure that real-time serving is not starved, we will use a co-scheduling strategy where resource allocation between batch and real-time inference is managed dynamically. We can achieve this by configuring Spark’s Dynamic Resource Allocation (DRA) feature to allocate more resources to real-time serving if necessary.

5. Monitoring and Logging

We need comprehensive monitoring and logging to track the system's health, performance, and any issues that arise during processing. Tools like Prometheus for metrics and Grafana for dashboards can be used here.

Architecture Diagram
Data Ingestion (Kafka)
Worker Node 1
Worker Node 2
...
Transformed Data, KafkaCommit Log
Transformed Data, KafkaCommit Log
Distributed Real-time Serving(TensorFlow Serving)

#### Implementation
Details
Code Example for Spark Worker Node
PYTHON
from pyspark.sql import SparkSession

# Initialize Spark session with appropriate configurations
spark = SparkSession.builder \
                    .appName("Batch Inference") \
                    .getOrCreate()

# Read data from Kafka topic
df = spark.readStream.format('kafka') \
                     .option('kafka.bootstrap.servers', 'localhost:9092') \
                     .option('subscribe', 'data-topic') \
                     .load()

# Transform the data using Spark DataFrame operations
transformed_df = df.selectExpr("CAST(value AS STRING)").select(from_json("value", schema).alias("data")) \
                   .select("data.*")

# Process each row and write back to Kafka with unique transaction ID
result = transformed_df.writeStream.format('kafka') \
                                 .option('kafka.bootstrap.servers', 'localhost:9092') \
                                 .option('topic', 'processed-topic') \
                                 .outputMode('append') \
                                 .trigger(processingTime='1 minute') \
                                 .start()

# Wait for the stream to finish
result.awaitTermination()

#### Closing Summary
and Common Mistakes

This architecture ensures high throughput, fault tolerance, idempotency, and efficient resource management. However, common mistakes include underestimating partition skew, failing to implement exactly-once semantics properly, or not effectively managing real-time serving resources.

In summary, designing a distributed batch-inference system requires a robust data ingestion layer, fault-tolerant processing with Apache Spark, idempotency via Kafka commit logs, and dynamic resource allocation for co-scheduling real-time serving.

**Comparison:**

```
Partition Key	Example Value
userId	user123
timestamp	1672548000
Feature Name	Feature Value
feature1	0.5
feature2	0.3
```

**Example input / output:**

```text
Input:
- Kafka topic 'data-topic' with 5 billion rows of data.

Output:
- Processed and committed to Kafka topic 'processed-topic'.
- Real-time inference serving remains unaffected.
```

**Data:**

```json
{
  "schema": {
    "value": {
      "type": "record",
      "fields": [
        {"name": "userId", "type": "string"},
        {"name": "timestamp", "type": "long"},
        {"name": "data", "type": "map"}
      ]
    }
  },
  "exampleRecords": [
    {
      "value": {
        "userId": "user123",
        "timestamp": 1672548000,
        "data": {"feature1": 0.5, "feature2": 0.3}
      }
    },
    {
      "value": {
        "userId": "user456",
        "timestamp": 1672549000,
        "data": {"feature1": 0.8, "feature2": 0.2}
      }
    }
  ]
}
```

**Flow:**

1. Data Ingestion
2. Kafka Topic 'data-topic'
3. Worker Node 1
4. Worker Node 2
5. Transformed Data, KafkaCommit Log
6. Transformed Data, KafkaCommit Log
7. Distributed Real-time Serving(TensorFlow Serving)

