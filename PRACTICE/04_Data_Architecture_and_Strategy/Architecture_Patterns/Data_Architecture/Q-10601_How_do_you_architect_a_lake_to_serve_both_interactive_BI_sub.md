---
type: "practice"
difficulty: "Deep"
---

# How do you architect a lake to serve both interactive BI (sub-second) and heavy ML training on the same underlying data without either workload degrading the other? What layers/materializations are required?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Central vs Domain Feature Modeling](<../../../../04_Data_Architecture_and_Strategy/Architecture_Patterns/Data_Architecture/Layered_Pipelines_Bronze_Silver_Gold/Central_vs_Domain_Feature_Modeling.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To architect a data lake that serves both sub-second interactive Business Intelligence (BI) queries and heavy Machine Learning (ML) training tasks on the same underlying data without degrading performance, we need to design a layered approach that includes efficient storage, optimized querying mechanisms, and strategic materializations. This architecture ensures that BI queries can run interactively while ML training processes can leverage pre-processed and optimized datasets.

#### Triaging the Question

The core of this question is understanding how to balance interactive query performance (BI) with batch processing requirements (ML). The key points are:

Data Storage: We need a storage system that supports both types of queries efficiently.
Query Optimization: Different query patterns require different optimizations.
Materialization: Pre-processing and caching data can significantly enhance query performance.

#### First Principles Solution
Layered Architecture

Raw Data Layer:

Storage: Use a scalable storage system like Hadoop Distributed File System (HDFS) or Amazon S3 to store raw, unprocessed data.
Purpose: This layer stores the original and most granular data.

Materialized Views Layer:

Intermediate Processing: Implement ETL processes that transform raw data into more structured formats suitable for both BI and ML.
Storage: Use a columnar storage system like Apache Parquet or ORC, which are optimized for both query engines and machine learning algorithms.

Materialized Aggregations Layer:

Pre-Processed Data: Create pre-aggregated datasets that are tailored to common BI queries. These can be used by interactive BI tools.
Purpose: Reduces the computational load on real-time data stores, improving query performance for BI.

Optimized Storage for ML Training:

Distributed File System (DFS): Use DFS like HDFS or cloud storage services to store large datasets required by ML training.
Data Parallelism: Design schemas that support parallel processing, reducing the need for excessive shuffling of data during model training.

Query Engine Layer:

Interactive BI Queries: Use a query engine like Presto or Apache Spark SQL for sub-second responses to complex queries.
ML Training Queries: Employ distributed frameworks like Dask or Spark MLlib for ML training tasks, which can handle large-scale data and parallel processing.

Metadata Layer:

Schema Registry: Maintain a schema registry that stores the metadata of all tables and their transformations.
Purpose: Ensures consistency and traceability across layers.

#### Implementation
Details
Hadoop Ecosystem: Leverage Hadoop for its robustness and scalability, supporting both batch processing (for ML) and interactive queries (BI).
Apache Spark: Use Spark for data transformation, caching, and parallel computing to support both interactive BI and heavy ML training.
Materialization Techniques:
Delta Lake: Utilize Delta Lake as a storage layer that supports ACID transactions and time travel features, making it ideal for materializing views and aggregations.
Apache Iceberg: Another option, offering advanced schema evolution and compaction strategies.

#### Closing Summary

By implementing this layered architecture, we can effectively support both interactive BI queries and heavy ML training tasks on the same underlying data. The key is to leverage optimized storage mechanisms, strategic materializations, and tailored query engines for each workload type.

#### Common Mistakes/Follow-Ups
Ignoring Data Freshness: Ensure that materials are refreshed regularly without affecting performance.
Overcomplicating with Single Stores: Avoid using a single data store (e.g., MySQL) for both interactive queries and batch processing, as it can lead to suboptimal performance.
Not Optimizing Materialized Views: Always optimize materialized views based on the expected query patterns and update frequencies.

**Comparison:**

| Layer                | Purpose                                      | Storage Technology  |
|----------------------|----------------------------------------------|---------------------|
| Raw Data             | Stores unprocessed raw data                  | HDFS, S3            |
| Materialized Views   | Intermediate processed data for BI queries    | Parquet, ORC        |
| Materialized Aggregations | Pre-aggregated datasets for BI               | Delta Lake          |
| Optimized Storage    | Large datasets for ML training              | Distributed File System (DFS) |

**Code:**

```python
# Example of a simple ETL process using Spark
from pyspark.sql import SparkSession

spark = SparkSession.builder.appName("DataLakeETL").getOrCreate()

# Load raw data
raw_data = spark.read.csv("/path/to/raw/data", header=True, inferSchema=True)

# Perform transformations (e.g., aggregations)
transformed_data = raw_data.groupBy('column1', 'column2').agg({'value': 'sum'})

# Write to materialized view layer
transformed_data.write.format("parquet").save("/path/to/materialized/views")
```

**Example input / output:**

```text
Input:
raw_data = spark.read.csv("/path/to/raw/data", header=True, inferSchema=True)

Output:
transformed_data = raw_data.groupBy('column1', 'column2').agg({'value': 'sum'})
transformed_data.write.format("parquet").save("/path/to/materialized/views")
```

**Data:**

```json
{
  "schema": {
    "name": "raw_data",
    "columns": [
      {"name": "column1", "type": "string"},
      {"name": "column2", "type": "int"},
      {"name": "value", "type": "float"}
    ]
  }
}
```

**Flow:**

1. Write to
2. Write to
3. Raw Data Layer
4. Transform & Aggregate
5. Materialized Views Layer
6. Pre-Process for BI Queries
7. Materialized AggregationsLayer
8. Pre-Process for ML Training
9. Optimized Storage for MLTraining
10. Query Engine Layer
11. Interactive BI Queries
12. ML Training Queries
13. Optimized Storage for MLTraining

