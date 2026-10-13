---
type: "practice"
difficulty: "Deep"
---

# How would you serve both a real-time current view and a historical SCD Type 2 view of the same dimension in a lakehouse?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ SCD Type 2 in Practice](<../../../../02_Data_Engineering/Modeling_and_Warehousing/Slowly_Changing_Dimensions/SCD_Type_2_in_Practice.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the requirement of serving both a real-time current view and a historical SCD Type 2 view from a single dimension in a lakehouse, we need to consider how to manage changes over time while ensuring that the data remains consistent and accessible for various analytical needs. Let’s break down the solution step-by-step:

#### Triaging the Question

The interviewer is primarily probing your understanding of Slowly Changing Dimensions (SCD), particularly SCD Type 2, and your ability to implement a hybrid architecture that supports both real-time and historical views. They are also likely interested in how you would manage data integrity, scalability, and performance optimizations.

#### Assumptions
The dimension table contains slowly changing attributes.
We need to support both real-time and historical queries.
The lakehouse is implemented using a modern data platform like Delta Lake or Iceberg Table on top of a storage layer such as HDFS or S3.

#### First Principles Solution

SCD Type 2 Implementation: For each dimension record, maintain multiple versions that capture changes over time. Each version has a valid start and end timestamp to indicate the period when it was active.

Real-Time Current View:

Use a materialized view or a continuously updated table that always shows the latest version of the data.
For real-time queries, you can use streaming pipelines (e.g., Flink, KStreams) that write to a delta-logged table, ensuring that the latest data is always available.

Historical SCD Type 2 View:

Maintain an additional table that stores all historical versions of the dimension records.
This table should have a schema similar to the current view but with additional columns for start and end timestamps.

Hybrid Architecture Design:

Use Delta Lake or Iceberg Table for the materialized views, as they support time travel queries which can fetch data from specific points in time.
For real-time ingestion, use Kafka topics to capture changes and write them into delta-logged tables.

Performance Optimization:

Implement partitioning strategies (e.g., by date or geography) to optimize query performance.
Use snapshot isolation techniques to ensure that queries don’t interfere with each other in a multi-stream environment.

Data Consistency and Integrity:

Ensure atomic updates to the dimension tables using transactions.
Use version control mechanisms to track changes and maintain consistency across all versions of records.

#### Closing Summary

In summary, serving both real-time current views and historical SCD Type 2 views in a lakehouse involves implementing a hybrid architecture that leverages Delta Lake or Iceberg Table for efficient data storage and retrieval. By maintaining multiple versions of dimension records with timestamps, we can support both real-time and historical queries effectively.

#### Common Mistakes/Follow-Ups

Mistake: Failing to implement proper versioning mechanisms.

#### Follow-up
How would you ensure that the latest version is always available for real-time queries?

Mistake: Ignoring data consistency and integrity issues.

#### Follow-up
How do you handle conflicts when updating multiple versions of a record simultaneously?

Mistake: Overcomplicating the architecture without considering performance implications.

#### Follow-up
What are some key considerations for optimizing query performance in this setup?

**Comparison:**

```
customer_id	first_name	last_name	email	version	valid_from	valid_to
12345	John	Doe	john.doe@example.com	2	2023-01-01T00:00:00Z	NULL
12345	Johnathan	Doh	john.doh@example.com	1	2022-01-01T00:00:00Z	2023-01-01T00:00:00Z
```

**Code:**

```python
# Example Python code to create a materialized view using Delta Lake
from pyspark.sql import SparkSession

spark = SparkSession.builder.appName("SCDExample").getOrCreate()

# Read current and historical data from Delta Lake
current_view_df = spark.read.format("delta").load("/path/to/current/view")
historical_view_df = spark.read.format("delta").load("/path/to/historical/view")

# Create a materialized view for the real-time current view
materialized_view_df = current_view_df.withColumnRenamed("attribute", "current_attribute")
materialized_view_df.write.format("delta").saveAsTable("real_time_current_view")
```

**Example input / output:**

```text
Input:
- Customer record updates in a streaming pipeline

Output:
- Real-time current view: Latest version of the customer data is always available.
- Historical SCD Type 2 view: All versions of customer data are stored and can be queried by time period.

Request:
- SELECT * FROM real_time_current_view WHERE customer_id = 12345

Response:
- Latest version of customer record with current attributes
```

**Data:**

```json
{
  "dimension_table": {
    "customer_id": int,
    "first_name": str,
    "last_name": str,
    "email": str,
    "version": int,
    "valid_from": datetime,
    "valid_to": datetime
  }
}
```

**Flow:**

1. Real-Time Streaming Pipeline
2. Write to Delta Lake
3. Materialized View forReal-Time Current View
4. Historical SCD Type 2 View
5. Read Historical Data

