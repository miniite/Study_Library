---
type: "practice"
difficulty: "Intermediate"
---

# You're building a streaming pipeline that ingests metrics from 50k hosts to feed an anomaly detector. Walk me through your ingestion topology: how do you partition, batch, and window the telemetry so the detector sees consistent per-host time series?

**Difficulty:** Intermediate · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Provisional and Final Results](<../../../../03_Big_Data_and_Streaming/Streaming_and_Messaging/Streaming_Architecture/Provisional_and_Final_Results/_index.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

#### Overview
In this scenario, we need to design a robust real-time streaming pipeline that efficiently processes 50k hosts' metrics to feed an anomaly detector. The key aspects are partitioning, batching, and windowing telemetry data to ensure consistent per-host time series for accurate anomaly detection.

#### Triaging the Question
The question is asking how to organize and process high volumes of metric data in real-time so that each host's data stream can be analyzed independently while maintaining temporal consistency. The interviewer is probing your understanding of streaming architectures, partitioning techniques, batch processing strategies, and windowing concepts.

#### Assumptions

Each host generates metrics at a consistent rate.
The anomaly detector requires up-to-date and complete time series for each host to function correctly.
We aim for an efficient design that can handle the scale without overwhelming resources or causing significant latency.

#### Solution
from First Principles:

Partitioning by Host ID:

To ensure parallel processing, we partition the stream based on the host ID. This approach allows us to distribute the workload across multiple nodes in a cluster, ensuring that each node processes data for a subset of hosts.
Partition Key: host_id (e.g., h1, h2, ..., h50k)

#### Example
Kafka Topic Configuration:
YAML
topic: host_metrics
partitions: 100  # To handle up to 50k hosts, we can distribute the load across more partitions.
replication-factor: 3  # Ensures fault tolerance and high availability.

Batch Processing for Data Consistency:

Metrics from a single host need to be processed in batches to ensure that the anomaly detector receives complete time series data before making decisions.
Batch Size: We can use fixed-size batches (e.g., 100 or 500 metrics) or time-based windows. Time-based windows are more flexible and allow us to adjust batch sizes dynamically based on the incoming rate of metrics.

#### Example
Spark Streaming Configuration:
PYTHON
spark.streaming.batchSize = 500  # Process 500 metrics per batch.

Sliding or Tumbling Windows:

Using a sliding window can help us capture recent events and update the state of each host's time series in real-time. A tumbling window ensures that we process data in fixed-size, non-overlapping batches.
Window Size: We might use a 1-minute window to ensure that we have enough data points for effective anomaly detection while maintaining low latency.

Aggregation and State Management:

Each node processes its assigned hosts' metrics within the defined batch or window. Aggregations like mean, standard deviation, min, max, etc., are computed per host.

#### Example
Aggregation Logic:
PYTHON
def process_metrics(metrics):
    aggregated_metrics = {
        'mean': sum(metrics) / len(metrics),
        'std_dev': statistics.stdev(metrics)
    }
    return aggregated_metrics

Publishing Processed Data:

Once the metrics are processed, they are published to a downstream system or stored in a time-series database for further analysis.

#### Example
Kafka Topic for Aggregated Metrics:
YAML
topic: host_aggregates
partitions: 100  # Match the number of partition keys used earlier.
replication-factor: 3

Monitoring and Failure Handling:

Implement robust monitoring to ensure that nodes process data within expected timeframes.
Use retries, fallback strategies, and load balancing to handle failures gracefully.

#### Closing Summary
The design involves partitioning the stream by host ID, processing metrics in batches to maintain consistency, using sliding windows for timely updates, aggregating the results, and publishing them for further analysis. This approach ensures that each host's data is processed independently while providing consistent time series data required by the anomaly detector.

#### Common Mistakes/Follow-Ups

Not properly partitioning the stream can lead to uneven load distribution.
Insufficient batch processing or incorrect window size can result in incomplete or outdated data being passed to the anomaly detector.
Failure to implement robust state management and monitoring mechanisms can cause data loss or processing delays.

**Comparison:**

```
Host ID	Time Window	Aggregated Metrics
h1	2023-10-01T10:00:00Z - 2023-10-01T10:01:00Z	Mean = 51.2, Std Dev = 1.5
h2	2023-10-01T10:00:00Z - 2023-10-01T10:01:00Z	Mean = 49.8, Std Dev = 2.0
```

**Code:**

```python
# Example Spark Streaming Code for Processing Metrics
from pyspark.streaming import StreamInput
from pyspark.sql.functions import col, mean, std_dev

def process_metrics(stream_input: StreamInput):
    windowed_stream = stream_input.window(60)  # 1-minute window
    aggregated_metrics = windowed_stream.groupBy("host_id").agg(mean(col("metric_value")).alias("mean"),
                                                                 std_dev(col("metric_value")).alias("std_dev"))
    return aggregated_metrics

# Example Kafka Consumer Code for Processing Aggregated Metrics
from kafka import KafkaConsumer

consumer = KafkaConsumer('host_aggregates', bootstrap_servers=['localhost:9092'])
for message in consumer:
    print(f"Received metric from host_id: {message.key} - mean: {message.value['mean']}, std_dev: {message.value['std_dev']}")
```

**Example input / output:**

```text
Input:
[
  {"host_id": "h1", "timestamp": "2023-10-01T10:00:01Z", "value": 50.2},
  {"host_id": "h1", "timestamp": "2023-10-01T10:00:02Z", "value": 52.3},
  ...
]

Output:
[
  {"host_id": "h1", "mean": 51.2, "std_dev": 1.5},
  {"host_id": "h2", "mean": 49.8, "std_dev": 2.0}
]
```

**Data:**

```json
{
  "metrics": [
    {"host_id": "h1", "metric_value": 50.2},
    {"host_id": "h1", "metric_value": 52.3},
    ...
    {"host_id": "h50k", "metric_value": 49.8}
  ]
}
```

**Flow:**

1. Ingest Metrics
2. Partition by Host ID
3. Process in Batches
4. Window Metrics
5. Compute Aggregations
6. Publish to Aggregated Topic

