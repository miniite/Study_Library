---
type: "practice"
difficulty: "Basic"
---

# Would you use batch or streaming to compute a rolling five-minute error rate for alerting and why?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Latency Budgets in Streaming Pipelines](<../../../../03_Big_Data_and_Streaming/Distributed_Processing/Big_Data_Processing/Latency_Budgets_in_Streaming_Pipelines.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To determine whether to use batch processing with Spark or streaming with Kafka for computing a rolling five-minute error rate, we need to consider several factors such as data volume, latency requirements, and system complexity. The core of this decision involves understanding the nature of big data processing tasks and how different systems handle them.

#### Triage

The question is asking us to decide between batch and streaming approaches based on the specific use case of calculating a rolling five-minute error rate for alerting purposes. This means we need to consider factors such as data ingestion, processing latency, and the frequency at which the computation needs to be performed.

#### Assumptions
The error rate is calculated over every five minutes.
The system should generate alerts based on this calculation.
We are comparing batch (using Spark) versus streaming (using Kafka).
Step-by-Step Solution
Batch Processing with Apache Spark
Data Ingestion: Data can be ingested into a structured format using tools like Kafka, Flume, or directly from sources such as files.
Processing: Using Spark’s batch processing capabilities, we can process the data in batches to calculate the error rate over every five minutes.
Storage: Results are stored temporarily and then processed periodically.

#### Pros
of Batch Processing:

Simplicity: Easier to implement and understand.
Flexibility: Can handle complex transformations and computations.
Cost-effective: Lower resource utilization compared to streaming solutions for less frequent updates.

#### Cons
of Batch Processing:

Latency: The process can take several minutes, which might be too slow for real-time alerts.
Resource Utilization: Spark jobs need to be run periodically, leading to potential inefficiencies if data volumes are high.
Streaming with Apache Kafka
Data Ingestion: Real-time data is pushed into a Kafka topic or direct stream input.
Processing: Using Spark Streaming (or Structured Streaming in later versions), the system continuously processes incoming data to compute the error rate within every five-minute window.
Storage and Synchronization: Results are immediately available for alerting, ensuring real-time responses.

#### Pros
of Streaming Processing:

Real-Time Alerts: The process is continuous, allowing for immediate alerts based on the computed error rate.
Efficient Use of Resources: Can handle high-frequency data streams efficiently without frequent job launches.

#### Cons
of Streaming Processing:

Complexity: More complex to set up and maintain due to the need for real-time processing.
Cost: Higher resource utilization as the system needs to continuously process incoming data.

#### Closing Summary

In summary, if the primary requirement is to compute a rolling five-minute error rate with minimal latency for real-time alerts, using streaming with Apache Kafka would be more suitable. However, if periodic batch processing can meet the alerting requirements without significant delays, and resource utilization is a concern, then batch processing with Spark might be the better choice.

#### Common Mistakes/Follow-Ups
Mistake: Assuming that real-time alerts are not critical just because the error rate calculation occurs every five minutes.

#### Follow-Up
Exploring hybrid approaches where initial computations are done in batches and streamed data is fed into an incremental update mechanism for near-real-time alerts.

**Comparison:**

```
Time	Error Rate
14:00	0.2
14:05	0.3
```

**Code:**

```python
# Example Spark Batch Processing Code
from pyspark.sql import SparkSession

spark = SparkSession.builder.appName("ErrorRateCalculation").getOrCreate()

data_stream = spark.readStream.format("kafka") \
    .option("kafka.bootstrap.servers", "localhost:9092") \
    .option("subscribe", "error_topic") \
    .load()

# Calculate error rate
windowed_error_rate = data_stream.groupBy(window(data_stream.timestamp, "5 minutes")).agg({"value": "mean"})

query = windowed_error_rate.writeStream.outputMode("complete").format("console").start()
query.awaitTermination()
```

**Example input / output:**

```text
Input: Stream of error logs (e.g., {"timestamp": "2023-10-01T14:05:00Z", "value": 1})
Output: Rolling five-minute window error rate calculated every 5 minutes.
```

**Data:**

```json
{
  "data_topic": {
    "name": "error_topic",
    "broker_list": ["localhost:9092"],
    "partitions": 3,
    "replication_factor": 1
  }
}
```

**Flow:**

1. No
2. Yes
3. Data Source
4. Is Real-Time Alert Critical?
5. Batch Processing with Spark
6. Streaming with Kafka & SparkStreaming
7. Compute Error Rate Every 5Minutes
8. Compute Error Rate in RealTime

