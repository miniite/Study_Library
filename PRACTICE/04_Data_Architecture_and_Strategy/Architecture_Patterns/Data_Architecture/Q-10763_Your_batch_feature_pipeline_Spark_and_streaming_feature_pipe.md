---
type: "practice"
difficulty: "Expert"
---

# Your batch feature pipeline (Spark) and streaming feature pipeline (Flink) compute the same feature and diverge, eroding model trust. Architect a batch/stream unification so the two paths reconcile by construction rather than by after-the-fact checks.

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Streaming-first Data Architecture](<../../../../04_Data_Architecture_and_Strategy/Architecture_Patterns/Data_Architecture/Tiered_Lake_Storage/Streaming-first_Data_Architecture.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To architect a batch/stream unification solution in an ML environment where Spark and Flink compute the same feature but diverge, we need to ensure that both pipelines converge on a single source of truth by design. This involves creating a unified data architecture leveraging data lakes and warehouses with a mesh topology for efficient and reliable reconciliation.

#### Triaging the Question

The core issue is reconciling diverging results from batch (Spark) and streaming (Flink) pipelines. The goal is to unify these pipelines such that they produce consistent feature values by construction, rather than relying on post-processing checks. This involves designing a data architecture that ensures both pipelines read from and write to the same authoritative source.

#### Assumptions
Latency Tolerance: We need to balance real-time processing (Flink) with batch processing (Spark), considering acceptable latency for reconciliation.
Data Consistency: Ensuring data consistency across batch and streaming is crucial for model trust.
Scalability: The architecture should handle increasing volumes of data without significant performance degradation.
Step-by-Step Solution
Unified Data Lake:
Use a data lake as the central repository where both Spark and Flink can read/write data.
Store raw, semi-processed, and fully processed data in parquet or ORC formats for efficient access.
Data Mesh Architecture:
Implement a data mesh to manage the flow of data between batch and streaming pipelines.
Use Kafka topics as event streams for Flink to consume real-time data.
Apache Beam Unified Pipeline:
Leverage Apache Beam’s unified model, which supports both batch and stream processing in a single pipeline.
Delta Lake:
Utilize Delta Lake to maintain transactional consistency across read and write operations, ensuring both pipelines can reliably reconcile differences.
Detailed Architecture
Data Ingestion Layer:
Kafka topics for real-time streaming data.
Processing Layers:
Batch Processing (Spark): Reads from the data lake, processes features, writes results back to Delta Lake.
Streaming Processing (Flink): Consumes from Kafka and performs feature computation, writes results to Delta Lake.
Delta Lake:
Acts as a single source of truth for both pipelines.
Reconciliation Mechanism:
Both Spark and Flink read the latest data from Delta Lake.
Use change data capture (CDC) or versioning in Delta Lake to handle incremental updates.

#### Example
Flow
Flink Processing:
Real-time Data
Kafka Topic
Flink Streaming Job
Delta Lake Write Path
Spark Processing:
Batch Data
Data Lake Read Path
Spark Batch Job
Delta Lake Read Path

#### Closing Summary
and Common Mistakes/Follow-Ups

By unifying the batch (Spark) and streaming (Flink) pipelines through a Delta Lake-based architecture, we ensure consistent feature computation by design. This approach eliminates the need for after-the-fact reconciliation checks, improving model trust and reducing operational complexity.

#### Common mistakes
include:

Overlooking data consistency between batch and streaming.
Not leveraging existing unified frameworks like Apache Beam.
Failing to properly utilize CDC mechanisms in Delta Lake for efficient reconciliation.

#### Conclusion

This architecture ensures that both Spark and Flink pipelines produce consistent results by reading from the same authoritative source (Delta Lake), thereby eliminating divergence and improving overall system reliability. The use of a data mesh and unified processing models significantly enhances the robustness and scalability of our ML feature pipeline.

**Comparison:**

| Pipeline | Processing Type | Data Source | Write Path |
|----------|----------------|-------------|------------|
| Flink    | Streaming       | Kafka Topic  | Delta Lake  |
| Spark    | Batch           | Data Lake   |            |

**Code:**

```python
# Example Delta Lake Write Job in Spark
from pyspark.sql import SparkSession

spark = SparkSession.builder.appName("FeatureUnification").getOrCreate()

df = spark.read.format("delta").load("dbfs:/path/to/delta/lake")
df.write.format("delta").mode("append").save("dbfs:/path/to/new/delta/lake")

# Example Flink Streaming Job
from pyflink.dataset import ExecutionEnvironment
from pyflink.table import StreamTableEnvironment, DataTypes
from pyflink.table.descriptors import Schema, OldCsv, FileSystem

env = ExecutionEnvironment.get_execution_environment()
t_env = StreamTableEnvironment.create(env)

t_env.connect(FileSystem().path('kafka/topic')) \
    .with_format(OldCsv()
                 .field("key", "string")
                 .field("value", "string")) \
    .with_schema(Schema()
                 .field("key", DataTypes.STRING())
                 .field("value", DataTypes.STRING())) \
    .register_table_source("mySource")

t_env.connect(FileSystem().path('dbfs:/path/to/delta/lake')) \
    .with_format(DeltaFormat()) \
    .with_schema(Schema() \
        .field("feature1", "string") \
        .field("feature2", "float")) \
    .register_table_sink("mySink")

t_env.from_path("mySource").insert_into("mySink")
```

**Example input / output:**

```text
Input: Real-time data -> Kafka topic, Batch data -> Data lake

Output:
- Flink streaming job writes to Delta Lake as new events arrive.
- Spark batch job reads from the latest version of Delta Lake for feature computation.
- Both pipelines produce consistent results by reading the same authoritative source.
```

**Data:**

```json
{
  "data_lake": {
    "path": "/dbfs/path/to/delta/lake",
    "format": "delta"
  },
  "kafka_topic": {
    "name": "real_time_data",
    "path": "/kafka/topic/path"
  }
}
```

**Flow:**

1. Real-time Data
2. Kafka Topic
3. Flink Streaming Job
4. Delta Lake Write Path
5. Batch Data
6. Data Lake Read Path
7. Spark Batch Job
8. Delta Lake Read Path

