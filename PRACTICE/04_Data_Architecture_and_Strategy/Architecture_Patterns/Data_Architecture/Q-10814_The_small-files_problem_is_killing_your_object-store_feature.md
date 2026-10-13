---
type: "practice"
difficulty: "Deep"
---

# The small-files problem is killing your object-store feature lake and compaction now costs more than the training scans. Architect a durable fix across streaming ingestion, table format, and compaction so feature freshness and read cost both stay bounded.

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Small Files and Compaction](<../../../../04_Data_Architecture_and_Strategy/Architecture_Patterns/Data_Architecture/Tiered_Lake_Storage/Small_Files_and_Compaction.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Summary:

The small-files problem in an object-store-based feature lake is causing significant overhead due to frequent compaction operations that are more costly than expected training scans. To architect a durable fix, we need to address the ingestion pipeline, optimize table formats, and implement efficient compaction strategies. The goal is to balance feature freshness with read efficiency while keeping costs under control.

#### Triage

The core of the question involves designing an architecture for a feature lake that can handle high-volume streaming data efficiently. Specifically, it's about:

Ingestion Pipeline: How we manage small files generated during ingestion.
Table Format Optimization: Choosing suitable formats to reduce storage overhead and improve read performance.
Compaction Strategy: Designing an effective compaction mechanism that reduces the number of small files without compromising feature freshness.

#### Assumptions
The feature lake is using a distributed system like Hadoop/HDFS or cloud-based object stores (S3, GCS).
The feature lake supports real-time data streaming.
Compaction operations are resource-intensive and need to be optimized for cost-effectiveness.

#### Solution
Ingestion Pipeline

To address the small-file problem at ingestion, we can adopt a strategy that aggregates small files into larger ones more efficiently:

Batch Ingestion: Design a batch processing system where multiple streaming sources aggregate data before writing it to storage.
File Splitting Control: Implement logic to split data only when necessary, such as during partitioning or sharding.
Table Format Optimization

Optimizing the table format can significantly reduce the number of small files and improve read performance:

Parquet vs. ORC: Use columnar formats like Parquet or ORC that are designed for efficient querying and support advanced compression schemes.
Delta Lake: Implement Delta Lake, which not only provides ACID transactions but also optimizes storage by merging smaller files into larger ones automatically.
Compaction Strategy

Compaction is crucial to reduce the number of small files while ensuring feature freshness:

Incremental Compaction: Design compaction processes that run in parallel and incrementally merge small files into larger ones.
Hybrid Strategies: Combine manual compactions with automated strategies triggered by certain conditions, such as file count or size thresholds.

#### Example
Workflow

Here’s a high-level flow of the solution:

Real-time Streaming Ingestion:

Data streams from various sources are aggregated into batches.
Batches are processed and transformed using ETL pipelines.

Storage Layer:

Aggregated data is written to an object store in Parquet/ORC format for efficient storage and query performance.
Delta Lake can be used to manage transactions and compaction automatically.

Compaction Engine:

An automated compaction engine runs periodically or based on file thresholds, merging small files into larger ones.
Manual compactions are triggered by high storage costs or specific business needs.

#### Closing Summary

By adopting a hybrid approach that includes batch ingestion, optimized table formats, and an efficient compaction strategy, we can ensure feature freshness while keeping read costs under control. This architecture balances performance, cost, and operational efficiency effectively.

#### Common Mistakes/Follow-Ups
Overlooking Incremental Compactions: Failing to implement incremental compactions can lead to excessive manual intervention.
Ignoring Cost Models: Not considering the cost implications of compaction operations can result in unexpectedly high costs.
Inadequate Monitoring: Lack of monitoring and alerting mechanisms for file health and storage usage can lead to hidden issues.

**Comparison:**

```
File ID	Name	Size (MB)	Timestamp
1	small-file-1	5	2023-10-01
2	small-file-2	6	2023-10-02
Compaction Strategy	Frequency	Conditions
Incremental	Periodic/Event	File count/sizes above threshold
```

**Code:**

```python
# Example code snippet for incremental compaction using Delta Lake
from delta import *
import pyspark

spark = pyspark.sql.SparkSession.builder.appName("CompactionExample").getOrCreate()

# Load data into a DataFrame
df = spark.read.format("delta").load("/path/to/data")

# Trigger incremental compaction with specific rules
spark.sql("OPTIMIZE DeltaTable WHERE condition")
```

**Example input / output:**

```text
Input:
Stream of small files being ingested into the feature lake.

Output:
Aggregated batch writes to Parquet/ORC format, with incremental compaction triggered to merge smaller files.
```

**Data:**

```json
{
  "files": [
    {
      "name": "small-file-1",
      "size": 5MB,
      "timestamp": "2023-10-01"
    },
    {
      "name": "small-file-2",
      "size": 6MB,
      "timestamp": "2023-10-02"
    }
  ]
}
```

**Flow:**

1. Yes
2. Alerts/Manual Triggers
3. Stream Ingestion
4. Batch Aggregation?
5. Write to Delta Lake(Parquet/ORC)
6. Incremental Compaction
7. Monitor File Health
8. Trigger Compaction

