---
type: "practice"
difficulty: "Deep"
---

# How do you use ATAM to evaluate a proposal to adopt a lakehouse (Delta/Iceberg) as the single ML data-and-feature substrate versus keeping separate warehouse and streaming stores? Which scenarios expose the freshness, point-in-time-correctness, and operational-complexity tradeoffs?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ ATAM for Platform Decisions](<../../../../07_Distributed_Systems_and_Design/Software_Architecture/Trade_off_Analysis/ATAM_for_Platform_Decisions.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When evaluating a proposal to adopt a lakehouse (Delta/Iceberg) as the single ML data-and-feature substrate versus keeping separate warehouse and streaming stores, we need to conduct an Architectural Trade-off Analysis (ATAM). ATAM is a structured approach for systematically identifying and analyzing trade-offs between competing requirements in system design. In this context, our primary goals are to ensure freshness of data, point-in-time correctness, and operational simplicity.

Firstly, let's break down the core requirements:

Freshness: Ensuring that the ML models have access to the latest available data.
Point-in-Time Correctness: Maintaining a consistent view of historical data across multiple points in time.
Operational Simplicity: Reducing complexity and operational overhead.

We will now walk through an ATAM step-by-step, starting from first principles:

#### Triage
What is the Proposal Asking?

The proposal suggests using a lakehouse (Delta/Iceberg) as a unified data substrate to replace separate warehouse and streaming stores. The key aspects we need to consider include:

Data Ingestion: How will data be ingested into the lakehouse, and what are the implications for real-time processing?
Data Storage: What storage formats and schemas will be used in the lakehouse, and how do they affect querying performance?
Data Processing: Will there be additional ETL processes needed to maintain consistency between the warehouse and streaming stores? How will these impact system complexity?

#### First Principles Solution

Ingestion:

Batch Ingestion: For historical data, batch ingestion from warehouses can be supported using tools like Spark or Airflow.
Streaming Ingestion: Real-time data streams can be ingested directly into the lakehouse using tools like Kafka Connect and Delta Streams.

Storage:

Delta Lake: This is a great choice for its ACID properties, transactional capabilities, and support for complex operations such as window functions.
Iceberg Table: Offers more flexible schema evolution compared to Delta Lake but might not have the same level of transactional guarantees.

Processing:

Real-time Queries: The lakehouse can handle both batch and real-time queries efficiently due to its support for incremental processing.
ETL Processes: If a separate warehouse is maintained, ETL processes will be required to keep the lakehouse synchronized with historical data stored in the warehouse.

Operational Simplicity:

Reducing the number of storage systems can simplify operations and maintenance.
However, this may introduce additional complexity if both batch and real-time workloads need to coexist efficiently.

#### Closing Summary
+ Common Mistakes/Follow-ups

By adopting a lakehouse as the single substrate, we can streamline data management and reduce operational overhead. However, careful consideration is needed for data freshness, point-in-time correctness, and system complexity. The primary tradeoffs include:

Increased Complexity: Managing both batch and real-time workloads within a unified system.

#### Performance Considerations
Ensuring that the lakehouse can handle the throughput required by both historical and real-time queries.

#### Common mistakes
in this analysis might include:

Overlooking the need for separate ETL processes if real-time data needs to be integrated with historical data stored in the warehouse.
Ignoring the potential impact on query performance when dealing with large volumes of real-time data.

#### Follow-ups
could involve detailed discussions around specific use cases, such as:

How will you handle incremental changes in the schema?
What are the backup and recovery strategies for both batch and streaming stores?

**Comparison:**

```
2023-10-01
2023-10-03
2023-10-05
2023-10-07
2023-10-09
2023-10-11
2023-10-13
2023-10-15
2023-10-17
2023-10-19
2023-10-21
BatchIngestFromWarehouse
ETLProcess
StreamIngestDirectly
IncrementalProcessing
Historical Data Ingestion
Real-time Data Ingestion
Data Ingestion and Processing for Lakehouse vs Separate Stores
```

**Code:**

```python
# Example of ingesting data into Delta Lake using PySpark
from pyspark.sql import SparkSession

spark = SparkSession.builder.appName("LakehouseExample").getOrCreate()

# Ingesting historical data from warehouse
warehouse_data = spark.read.parquet("/path/to/warehouse/data")
warehouse_data.write.format("delta").save("/path/to/lakehouse/historical")

# Ingesting real-time data using Delta Streams
streaming_data = spark.readStream.format("kafka").option("kafka.bootstrap.servers", "localhost:9092").load()
streaming_data.writeStream.format("delta").outputMode("append").start("/path/to/lakehouse/streaming")
```

**Example input / output:**

| Scenario        | Historical Data Ingestion   | Real-time Data Ingestion | Query Performance |
|-----------------|----------------------------|-------------------------|------------------|
| Warehouse & Lakehouse | Batch ingestion from warehouse<br>Synchronized using ETL | Stream ingestion directly into lakehouse<br>Incremental processing support | Efficient handling of both batch and real-time queries |
| Separate Stores  | Direct batch & stream ingestion | High complexity in querying across multiple stores | Potentially lower performance due to data duplication |

**Data:**

```json
{
  "warehouseData": {
    "location": "/path/to/warehouse/data",
    "format": "parquet"
  },
  "lakehouseHistorical": {
    "location": "/path/to/lakehouse/historical",
    "format": "delta"
  },
  "lakehouseStreaming": {
    "location": "/path/to/lakehouse/streaming",
    "format": "delta"
  }
}
```

**Flow:**

1. Yes
2. No
3. Historical
4. Real-time
5. Data Ingestion
6. Is the data historical?
7. Batch Ingest from Warehouse
8. Stream Ingest into Lakehouse
9. ETL Process forSynchronization
10. Incremental ProcessingSupport
11. Querying
12. Data Source: Historical orReal-time?
13. Lakehouse Historical Data
14. Lakehouse Streaming Data

