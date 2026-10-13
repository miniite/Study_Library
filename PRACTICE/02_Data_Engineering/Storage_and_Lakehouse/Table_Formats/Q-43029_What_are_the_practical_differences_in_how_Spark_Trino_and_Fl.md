---
type: "practice"
difficulty: "Deep"
---

# What are the practical differences in how Spark, Trino, and Flink integrate with Iceberg (write support, maintenance, catalog config)? Where do teams get surprised?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Table Format Internals](<../../../../02_Data_Engineering/Storage_and_Lakehouse/Table_Formats/Table_Format_Internals.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In this deep-dive into how Apache Spark, Trino, and Apache Flink integrate with Iceberg table formats—specifically focusing on write support, maintenance, and catalog configuration—we'll explore practical differences and common surprises faced by teams deploying these systems.

#### Triaging the Question

The question is probing into the nuances of integrating Iceberg tables with different big data processing frameworks. Key aspects include:

Write Support: How each framework supports writing to an Iceberg table.
Maintenance: Operational considerations such as compaction, garbage collection, and optimization.
Catalog Configuration: The setup required in the respective systems to connect to Iceberg catalogs.

#### Assumptions
and Context
We are comparing Apache Spark 3.x, Trino 387+ (as of writing), and Flink 1.14+.
The focus is on Iceberg tables using Delta Lake or Hudi as storage formats.
Step-by-Step Solution
Spark Integration with Iceberg

Apache Spark offers robust support for Iceberg through the org.apache.spark.sql.iceberg package. Here’s how it works:

Write Support:

Append Mode: Spark can append new data to an existing Iceberg table using DataFrameWriter.
Batch Inserts: Spark batches writes into segments, optimizing write performance.

Maintenance:

Compaction: Spark triggers compactions automatically based on the configured policies in the Iceberg table.
Garbage Collection: Spark manages metadata changes and garbage collection for Delta Lake storage format.

Catalog Configuration:

Hadoop Catalog: Requires setting up Hadoop configurations like fs.defaultFS to point to the storage location.
Iceberg Support in Spark: Enables via spark.sql.extensions=org.apache.spark.sql.iceberg.IcebergSparkSessionExtension.
Trino Integration with Iceberg

Trino provides seamless integration for querying Iceberg tables:

Write Support:

Limited Direct Writes: Trino is primarily a read-only engine, making direct writes via Trino complex and not recommended.

Maintenance:

Compaction and Garbage Collection: These are managed by the underlying storage system (e.g., Delta Lake or Hudi).
Catalog Configuration: Requires setting up the Iceberg catalog in Trino’s configuration.

Catalog Configuration:

Iceberg Catalog Support: Trino requires a separate Iceberg catalog service to manage schemas and metadata.
Configuration: In trino.properties, set catalogs=iceberg and configure paths to tables and storage locations.
Flink Integration with Iceberg

Apache Flink offers incremental support for Iceberg via the Iceberg connector:

Write Support:

Append Mode: Flink can write data incrementally to an Iceberg table using its connectors.

Maintenance:

Compaction and Garbage Collection: These operations are handled by the underlying storage system (e.g., Delta Lake or Hudi).
Checkpointing: Flink’s checkpointing mechanism ensures durability, but requires coordination with the Iceberg compaction policies.

Catalog Configuration:

Iceberg Catalog Support: Similar to Trino, an Iceberg catalog service is required.
Configuration: Set up the Iceberg catalog in Flink’s configuration and ensure compatibility with the storage format.
Common Surprises
Trino Write Limitations: Teams often expect full CRUD capabilities but are surprised by Trino's read-only nature for direct writes to Iceberg tables.
Catalog Management: Ensuring that all systems (Spark, Trino, Flink) use the same Iceberg catalog can be a challenge. Misconfigurations here lead to data inconsistencies and performance issues.
Compaction Overheads: Teams might underestimate the overheads of compactions in Delta Lake or Hudi storage formats, leading to potential delays in query performance.

#### Closing Summary

In summary, while Spark provides rich support for writing to Iceberg tables, Trino and Flink have more limitations but still offer significant value through their respective strengths. Catalog configuration is a critical step that requires careful management across systems.

#### Common Mistakes/Follow-Ups
Misconfiguring the catalog service.
Overlooking compaction policies which can impact performance.
Expecting full CRUD capabilities from Trino when it’s primarily designed for read operations on Iceberg tables.

**Comparison:**

| Framework | Write Support          | Maintenance           | Catalog Configuration |
|-----------|-----------------------|----------------------|----------------------|
| Spark     | Append Mode, Batching | Compaction, GC        | Hadoop Catalog       |
| Trino     | Limited Direct Writes  | Managed by Storage    | Separate Iceberg Cat  |
| Flink     | Incremental Writes     | Compaction, Checkpoint| Iceberg Catalog      |

**Code:**

```scala
// Example Spark code snippet
import org.apache.spark.sql.SparkSession

val spark = SparkSession.builder()
  .appName("Iceberg Integration")
  .enableHiveSupport() // For Hadoop catalog support
  .getOrCreate()

spark.read.format("iceberg")
  .load("/path/to/table")
  .show()
```

**Example input / output:**

```text
Input (Spark write to Iceberg):
df.write.format("iceberg").option("table", "my_table").mode("append").save("/path/to/table")

Output (Trino query on Iceberg table):
SELECT * FROM iceberg.my_table LIMIT 10;
```

**Data:**

```yaml
# Example configuration for Trino Iceberg Catalog
cATALOGS:
  iceberg:
    type: iceberg
    connector:
      class: org.apache.iceberg.trino.IcebergCatalog
      options:
        iceberg.catalog.uri: "hdfs://path/to/catalog"
```

**Flow:**

1. Spark
2. Write to Iceberg
3. Append Mode
4. Compaction
5. Catalog Config
6. Hadoop
7. Trino
8. Read from Iceberg
9. Limited Writes
10. Catalog Config
11. Iceberg Service
12. Flink
13. Write to Iceberg
14. Incremental Mode
15. Catalog Config
16. Iceberg Service

