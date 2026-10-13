---
type: "practice"
difficulty: "Deep"
---

# How would you use partition pruning together with a covering index to make a hot dashboard query sub-10ms?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1, SDE-1/2, SDE-2/3 · [↑ Partition Pruning](<../../../../01_Databases_and_SQL/Database_Operations/Partitioning/Partition_Pruning/_index.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To achieve sub-10ms performance for a hot dashboard query using partition pruning and a covering index, we need to carefully design our database schema and indexes based on the data distribution and query patterns. Let's break down the solution step by step.

#### Triaging the Question

The core of this question is about optimizing a specific type of read-heavy query (dashboard queries) for speed by leveraging partition pruning and covering indexes. Partitioning helps in reducing the amount of data that needs to be scanned, while a covering index ensures that all required columns are stored within the index itself, avoiding additional disk I/O.

Key Assumptions
Data Distribution: The dashboard query is run frequently, targeting a specific range or set of values.
Read-Heavy Workload: The primary operation involves reads; writes can be handled less frequently and with more relaxed performance requirements.
Hardware Capabilities: We have modern hardware capable of handling the data volume and read throughput.

#### First Principles Solution

Partitioning Strategy:

Range Partitioning: If the dashboard query often targets a date range, use range partitioning based on dates to divide the table into smaller, more manageable chunks.
Hash Partitioning: For queries that frequently target specific values or sets of values (like user IDs), hash partitioning can be effective by distributing data across partitions.

Covering Index:

Ensure that the index includes all columns required for the dashboard query to minimize additional I/O operations and enhance read performance.

#### Cons
ider adding a composite covering index if necessary, combining multiple columns used in WHERE clauses and ORDER BY clauses.

Query Optimization:

Use SQL hints or query rewrites to leverage partition pruning effectively.
Avoid using non-covering indexes that require a full table scan for missing columns.
Detailed Steps

Partition Design:

SQL
-- Example for range partitioning based on date
CREATE TABLE dashboard_data (
    id INT,
    metric_value FLOAT,
    timestamp TIMESTAMP
) PARTITION BY RANGE (timestamp);

-- Create partitions for a 30-day window
CREATE TABLE dashboard_data_20231001 PARTITION OF dashboard_data FOR VALUES FROM ('2023-10-01') TO ('2023-10-31');
-- Continue creating partitions as needed

Index Design:

SQL
-- Example for a covering index on range partitioned table
CREATE INDEX idx_dashboard_data_covering ON dashboard_data_20231001 (id, metric_value, timestamp);

Query Optimization:

SQL
SELECT id, SUM(metric_value) AS total_metric
FROM dashboard_data_20231001
WHERE timestamp >= '2023-10-15' AND timestamp < '2023-10-30'
GROUP BY id;

#### Closing Summary
and Common Mistakes/Follow-Ups

To ensure sub-10ms performance, it is crucial to carefully design the partitioning strategy based on query patterns and data access. A covering index should be created to include all necessary columns for the queries. Missteps often involve creating non-covering indexes or not properly utilizing partition pruning.

For follow-ups:

How would you handle a sudden spike in read requests?
What if the data distribution changes over time?
Can you discuss potential scaling strategies as the dataset grows?

**Comparison:**

| Partitioned Table Design | Index Design |
|-------------------------|--------------|
| dashboard_data          | idx_dashboard_data_covering |
| Partitions (e.g., 20231001) | (id, metric_value, timestamp) |

**Code:**

```sql
-- Partition creation example
CREATE TABLE dashboard_data (
    id INT,
    metric_value FLOAT,
    timestamp TIMESTAMP
) PARTITION BY RANGE (timestamp);

-- Creating partitions for a 30-day window
CREATE TABLE dashboard_data_20231001 PARTITION OF dashboard_data FOR VALUES FROM ('2023-10-01') TO ('2023-10-31');
```

**Example input / output:**

```text
-- Input query
SELECT id, SUM(metric_value) AS total_metric
FROM dashboard_data_20231001
WHERE timestamp >= '2023-10-15' AND timestamp < '2023-10-30'
GROUP BY id;

-- Output
+----+------------+
| id | total_metric|
+----+------------+
| 1  | 45.8       |
+----+------------+
```

**Data:**

```json
{
  "id": [1, 2, 3],
  "metric_value": [10.5, 20.8, 30.6],
  "timestamp": ["2023-10-15", "2023-10-20", "2023-10-25"]
}
```

**Flow:**

1. Yes
2. Yes
3. No
4. No
5. Dashboard Query
6. Is Data Partitioned?
7. Partition Pruning
8. Read Only Relevant Partitions
9. Is Index Covering RequiredColumns?
10. Covering Index Utilized
11. Query Execution Optimized
12. Full Table Scan Possible
13. Optimize Partition Strategy
14. Sub-10ms PerformanceAchieved
15. Increase Query Time

