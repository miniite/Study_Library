---
type: "practice"
difficulty: "Deep"
---

# If your pipeline ingests Avro from Kafka and needs to serve analytics what format would you convert to and why?

**Difficulty:** Deep · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Row vs Columnar Formats](<../../../../02_Data_Engineering/Storage_and_Lakehouse/File_Formats/Row_vs_Columnar_Formats.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To decide on the optimal columnar file format for serving analytics, we need to consider several factors including data access patterns, storage efficiency, query performance, and compatibility with downstream systems. Given that your pipeline ingests Avro from Kafka and needs to serve analytics, converting to Parquet or ORC would be a logical choice due to their superior performance characteristics.

Avro is primarily designed for schema evolution and is not optimized for analytical queries, which typically require scanning large portions of data. On the other hand, Parquet and ORC are specifically tailored for high-performance analytics, offering features such as columnar storage, dictionary encoding, and support for efficient filter pushdown.

Let's break down why converting to Parquet would be a better choice in this scenario:

Columnar Storage: Both Parquet and ORC store data column-wise, which allows for more efficient scanning of large datasets compared to row-oriented formats like Avro.
Dictionary Encoding: These formats use dictionary encoding, which significantly reduces storage overhead by storing unique values once per column.
Filter Pushdown: Advanced query optimization techniques such as filter pushdown are supported in both Parquet and ORC, enabling the database or processing engine to skip reading data that does not match the filters.

#### Triage

The question is probing your understanding of the trade-offs between different file formats for analytical workloads. You need to justify your choice based on key factors like storage efficiency, query performance, and compatibility with downstream systems.

#### Assumptions
The primary use case involves serving analytics queries.
The pipeline will store data in a way that optimizes read operations for analysis.
Downstream systems are compatible with Parquet/ORC.

#### First Principles Solution
Data Ingestion: The Avro data from Kafka is initially ingested into a raw storage layer, such as HDFS or S3. This step ensures that the data can be processed and transformed before being stored in an optimized format.
Transformation Layer: Use Apache Spark or other big data processing frameworks to transform the Avro data into Parquet/ORC format. During this transformation, apply any necessary schema validation, filtering, or aggregation operations to prepare the data for analytics.
Storage: Store the transformed data in a managed storage layer optimized for analytics workloads, such as HDFS or Amazon S3.
Query Processing: Use tools like Apache Drill or Presto to query the Parquet/ORC files directly from the storage layer.

#### Closing Summary

Converting the Avro data ingested from Kafka into either Parquet or ORC is a strategic choice for serving analytics due to their superior performance characteristics, efficient storage mechanisms, and compatibility with advanced analytical tools. While Avro might be more suitable for raw data ingestion and schema evolution, Parquet/ORC provide the necessary optimizations for high-performance queries.

#### Common Mistakes/Follow-ups
Overlooking Schema Evolution: Ensure that any schema changes in Avro are managed correctly but do not affect the analytics use case.
Ignoring Compatibility: Verify that downstream tools support both Parquet and ORC formats to avoid potential integration issues.
Resource Management: Consider the additional computational resources required for transforming data into these columnar formats, especially if real-time processing is a requirement.

**Comparison:**

```
Avro	Parquet/ORC
Schema evolution friendly	High-performance analytics
Row-oriented storage	Columnar storage
Limited support for filter pushdown	Support for efficient scan operations and filter pushdown
```

**Code:**

```python
# Example PySpark code to convert Avro to Parquet
from pyspark.sql import SparkSession

spark = SparkSession.builder \
    .appName("AvroToParquet") \
    .getOrCreate()

avro_df = spark.read.format("avro").load("/path/to/avro/data")
parquet_df = avro_df.write.parquet("/path/to/output/directory")

# Example PySpark code to read and filter Parquet data
filtered_data = spark.read.parquet("/path/to/output/directory") \
    .filter("column_name > value") \
    .write.format("json").save("/path/to/filtered/data")
```

**Example input / output:**

```text
Input:
- Avro data in Kafka topic
- Avro schema

Output:
- Parquet files stored in HDFS/S3 with optimized columnar structure
- Filtered query results in JSON format
```

**Data:**

```json
{
  "avro_file_path": "/path/to/avro/data",
  "parquet_output_path": "/path/to/output/directory"
}
```

**Flow:**

1. Yes
2. No
3. Avro data in Kafka
4. Ingest into raw storage?
5. Transform to Parquet/ORC
6. Continue using Avro foranalytics
7. Store in optimized analyticsstorage layer
8. Query with tools like ApacheDrill/Presto

