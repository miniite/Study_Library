---
type: "practice"
difficulty: "Intermediate"
---

# How would you reduce the volume of metrics before they hit long-term storage without losing key signals?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Kafka Retention and Storage](<../../../../03_Big_Data_and_Streaming/Distributed_Processing/Big_Data_Processing/Kafka_Retention_and_Storage.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To reduce the volume of metrics before they hit long-term storage while not losing key signals, we can employ a strategy that combines filtering and aggregation techniques using Apache Kafka and Apache Spark. The core idea is to preprocess the incoming data stream to minimize redundant or less critical information, ensuring only valuable metrics reach our archival systems.

The question here revolves around optimizing data processing pipelines for efficiency and maintaining data integrity. We need to consider both filtering out unnecessary metrics and performing aggregations to reduce volume without compromising on important signals. Key assumptions include that we have a steady stream of metrics from various sources, and long-term storage is expensive or has limited capacity.

Step-by-Step Solution

Data Ingestion with Filtering in Kafka:

Use Kafka Connectors to ingest data from various sources.
Implement filtering logic within the Kafka consumer to discard irrelevant or duplicate records early in the pipeline.

Aggregation and Transformation using Spark Streaming:

Process the filtered stream of metrics using Apache Spark Streaming.
Perform real-time aggregations such as min, max, average, etc., to reduce data volume while retaining critical information.
Apply windowed operations if historical context is needed.

Storage Optimization with Tiered Storage Strategy:

Use a tiered storage strategy where the most critical and recent data is stored in faster, more expensive storage (e.g., SSD-based HDFS).
Archive older or less critical metrics to slower, cheaper storage (e.g., HDD-based cloud storage).

Retained Data for Validation:

Maintain a small subset of unaltered raw data as validation samples.
Use this sample data periodically for comparison with processed metrics to ensure the transformation logic is not losing important signals.

#### Closing Summary
and Common Mistakes

In summary, by combining early filtering in Kafka and real-time aggregation using Spark Streaming, we can significantly reduce the volume of incoming metrics before they reach long-term storage. This approach ensures that only essential information gets stored, optimizing both storage costs and processing efficiency.

#### Common mistakes
include:

Ignoring real-time processing capabilities, leading to delayed data availability.
Failing to validate transformations by not maintaining raw data samples.
Over-aggregating to the point where critical signals are lost.

**Comparison:**

```
Time Window	Metric ID	Mean Value	Max Value
[14:00:00, 14:01)	sensor1	99.0	100
[14:00:00, 14:01)	sensor2	97.0	98
```

**Code:**

```python
from pyspark.sql import SparkSession

spark = SparkSession.builder \
    .appName("Metric Aggregation") \
    .getOrCreate()

# Example input stream
raw_data = [("2023-10-05 14:00:00", "sensor1", 98),
            ("2023-10-05 14:00:00", "sensor2", 96),
            ("2023-10-05 14:01:00", "sensor1", 100),
            ("2023-10-05 14:01:00", "sensor2", 98)]

# Create DataFrame
df = spark.createDataFrame(raw_data, ["timestamp", "metric_id", "value"])

# Define window specifications
window_spec = Window.partitionBy("metric_id").orderBy(F.col("timestamp"))

# Aggregate data with rolling mean and max value over a 1-minute window
aggregated_df = df.withWatermark("timestamp", "5 minutes") \
    .groupBy("metric_id", F.window("timestamp", "1 minute")) \
    .agg(
        F.mean("value").alias("mean_value"),
        F.max("value").alias("max_value")
    )

# Show aggregated data
aggregated_df.show()
```

**Example input / output:**

```text
+-------------------+----------+------------+---------+
|            window  |metric_id | mean_value | max_value|
+-------------------+----------+------------+---------+
|[14:00:00, 14:01)|sensor1   |       99.0 |      100|
|[14:00:00, 14:01)|sensor2   |       97.0 |      98 |
+-------------------+----------+------------+---------+
```

**Data:**

```json
[
  {"timestamp": "2023-10-05 14:00:00", "metric_id": "sensor1", "value": 98},
  {"timestamp": "2023-10-05 14:00:00", "metric_id": "sensor2", "value": 96},
  {"timestamp": "2023-10-05 14:01:00", "metric_id": "sensor1", "value": 100},
  {"timestamp": "2023-10-05 14:01:00", "metric_id": "sensor2", "value": 98}
]
```

**Flow:**

1. Ingest Metrics
2. Filtering in Kafka
3. Send to Spark Streaming
4. Aggreate and Transform Data
5. Store in Tiered Storage
6. Validate Transformed Data

