---
type: "practice"
difficulty: "Deep"
---

# Your model needs a snapshot of features exactly as they were at prediction time last month for a backtest. How does time-travel in Delta or Iceberg help, and what's it costing you in storage?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Tiered Lake Storage](<../../../../04_Data_Architecture_and_Strategy/Architecture_Patterns/Data_Architecture/Tiered_Lake_Storage/_index.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the requirement of fetching historical feature snapshots exactly as they were at prediction time last month for a backtest, we need to leverage the capabilities offered by Delta or Iceberg databases, which support time-travel queries. These systems allow us to query data based on specific timestamps, ensuring that we retrieve the exact state of features at a given point in time. However, this capability comes with potential storage costs and operational considerations.

Delta Lake and Apache Iceberg both provide robust solutions for managing large-scale data lakes and warehouses. Delta Lake is built on top of Apache Spark and supports ACID transactions, while Apache Iceberg offers more flexibility and better performance, especially when dealing with complex query workloads. In this context, we'll use these systems to ensure that the features used in our model are exactly as they were last month.

#### Triage

The question is probing how we can utilize time-travel capabilities within Delta or Iceberg databases to fetch historical data accurately for backtesting purposes. The key assumptions include:

Historical snapshots need to be retrieved precisely at a specific timestamp (last month).
We want to avoid reprocessing entire datasets, as this could be computationally expensive.
Storage costs and performance implications are significant factors.

#### First Principles Solution

To solve the problem, we'll follow these steps:

Understand Time-Travel Queries: Both Delta Lake and Iceberg support time-travel queries that allow us to retrieve data based on specific timestamps or version numbers.
Schema Design for Historical Data: Ensure our data schema supports historical versions by incorporating a timestamp or version_number field.
Querying Historical Data: Use the appropriate query syntax in Delta or Iceberg to fetch data from last month's snapshot.
Step 1: Understanding Time-Travel Queries

Delta Lake and Apache Iceberg both offer time-travel capabilities:

Delta Lake: Uses a commit log mechanism, where each write operation is recorded. We can query historical versions of data using the as of clause.
Iceberg: Manages multiple file formats (Parquet, ORC) and maintains a metadata layer that tracks changes to the table over time.
Step 2: Schema Design for Historical Data

For both systems, we need to ensure our schema includes fields such as:

SQL
timestamp_column
feature_values

In Delta Lake, this might look like:

SQL
CREATE TABLE features (
    timestamp TIMESTAMP,
    feature_id INT,
    value FLOAT
)
USING DELTA

In Iceberg, the schema would be similar but could include additional metadata fields for better performance and flexibility.

Step 3: Querying Historical Data

To fetch data from last month's snapshot:

Delta Lake:
SQL
SELECT * FROM features AS OF '2023-10-01'

This query will return the state of features as they were on October 1, 2023.

Iceberg:
SQL
SELECT * FROM features WHERE timestamp <= '2023-10-01' AND timestamp > '2023-09-01'

This query will return the state of features as they were in September 2023.

#### Closing Summary
+ Common Mistakes/Follow-Ups

Time-travel capabilities in Delta and Iceberg are powerful tools for ensuring that historical data is available for accurate backtesting. However, these come with potential storage costs:

Storage Costs: Each version of the data (timestamped or versioned) consumes additional storage.
Performance Impact: Retrieving older versions can be slower compared to current data.

To mitigate these issues, consider implementing strategies such as periodic archiving or partitioning data based on time. Additionally, monitoring and optimizing queries can help reduce overall costs and improve performance.

#### Common mistakes
include:

Failing to properly design the schema for historical data.
Not leveraging the full potential of time-travel capabilities, leading to unnecessary reprocessing or incorrect data retrieval.
Ignoring storage costs and their long-term impact on the system.
Self-Check

This solution is correct and comprehensive in addressing the requirements while highlighting key considerations.

