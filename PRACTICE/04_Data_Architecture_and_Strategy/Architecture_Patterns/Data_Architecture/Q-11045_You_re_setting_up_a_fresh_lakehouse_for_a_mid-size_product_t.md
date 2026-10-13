---
type: "practice"
difficulty: "Intermediate"
---

# You're setting up a fresh lakehouse for a mid-size product team. Walk me through how you'd decide the bronze/silver/gold table boundaries for their clickstream and orders data, and what you'd actually put in each layer.

**Difficulty:** Intermediate · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Layered Pipelines Bronze Silver Gold](<../../../../04_Data_Architecture_and_Strategy/Architecture_Patterns/Data_Architecture/Layered_Pipelines_Bronze_Silver_Gold/_index.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When setting up a lakehouse architecture for a mid-size product team dealing with clickstream and order data, the key is to balance data freshness, storage efficiency, processing complexity, and query performance. I’d decide on bronze/silver/gold table boundaries based on these factors.

To start, let's break down the layers:

Bronze Layer: Raw, unprocessed data.
Silver Layer: Cleaned and transformed data that’s ready for analysis but hasn’t been optimized yet.
Gold Layer: Optimized, processed data stored in a form suitable for reporting and analytics.

#### Triaging the Question

The core of this question revolves around understanding the nature of clickstream and order data, deciding on processing priorities, and ensuring the architecture supports both operational and analytical workloads efficiently. We'll assume:

Clickstream Data: High volume, low cardinality, real-time requirements.
Order Data: Lower volume but high cardinality, periodic updates.

#### First Principles Solution

Bronze Layer:

Purpose: Raw data ingestion.
Data: Unprocessed clickstream and order events, directly from source systems (e.g., web servers, e-commerce platforms).
Processing: Minimal, focusing on initial validation and schema enforcement.

Silver Layer:

Purpose: Data cleaning, transformation, and basic aggregation.
Data: Clickstream data cleansed of noise and order data with basic transformations (e.g., converting timestamps to a consistent format).
Processing: Use Spark or Flink for real-time processing if needed, ensuring that the data is cleaned and transformed before moving to analytics.

Gold Layer:

Purpose: Optimized data for reporting and analytics.
Data: Final processed clickstream and order data, optimized for query performance (e.g., using partitioning, indexing).
Processing: Use tools like Delta Lake or Iceberg for ACID transactions and optimizations.
Detailed Breakdown
Bronze Layer
Storage: Raw events should be stored in a cost-effective storage solution. For example, Azure DataLake Storage or Amazon S3.
Ingestion: Use Apache Kafka for real-time ingestion of clickstream data and batch processes for order data.
Validation: Implement simple schema validation to ensure data integrity.
Silver Layer
Storage: Use an intermediate storage like HDFS or a distributed file system compatible with Spark.
Processing: Set up a pipeline using tools like Apache Flink for real-time processing of clickstream data. For orders, use batch processing frameworks like Apache Spark.
Transformation: Cleanse and transform the data to remove inconsistencies and noise.
Gold Layer
Storage: Store processed data in optimized formats such as Delta Lake or Iceberg tables within a data warehouse (e.g., Amazon Redshift, Google BigQuery).
Optimization: Implement indexing and partitioning strategies based on query patterns. For example, clickstream data can be partitioned by date, while order data might benefit from customer ID partitions.
Queries: Ensure that the most common queries are well-tuned for performance.

#### Closing Summary

By setting up a lakehouse architecture with bronze/silver/gold layers, we ensure that raw data is ingested reliably, cleaned and transformed in intermediate steps, and finally optimized for analytical purposes. This approach supports both operational needs (like real-time clickstream analysis) and batch reporting/analytical workloads efficiently.

#### Common Mistakes/Follow-Ups
Mistake 1: Over-processing at the bronze layer, which can lead to higher costs and decreased performance.
Mistake 2: Failing to optimize gold-layer tables for query performance, leading to slower analytics.

#### Follow-Up
Discussing real-world trade-offs in storage and compute choices based on team size and data volume.

