---
type: "practice"
difficulty: "Deep"
---

# How do you approach tuning an analytical query that scans a whole fact table when you can't add indexes freely?

**Difficulty:** Deep · **Levels:** SDE-1, SDE-1/2 · [↑ Query Rewrites and Plan Stability](<../../../../01_Databases_and_SQL/Query_Performance/Execution_Plans/Query_Rewrites_and_Plan_Stability.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR

When faced with an analytical query that needs to scan a large fact table without being able to add indexes, the key is to understand and leverage the execution plan. By analyzing the cost-based optimization (CBO) of your SQL queries and refining your data model, you can optimize the performance. This involves identifying bottlenecks, rewriting queries, and utilizing features like materialized views or partitioning.

#### Triaging the Question

The question is asking about optimizing an analytical query that must scan a large fact table but cannot add indexes freely. The interviewer likely wants to assess your understanding of execution plans, cost-based optimization, and strategies for dealing with query performance issues in such scenarios.

Key Assumptions:
You have access to database statistics.
You can modify the SQL query or create additional views/materialized views.
The table is large and contains a lot of data.
Adding indexes is restricted by organizational policies or constraints.
Detailed Solution

Understand the Current Execution Plan:

Use EXPLAIN (or equivalent) to analyze the execution plan of your query.
Identify whether it's doing full table scans, sorts, or other expensive operations.

Analyze Costs and Statistics:

Check if you have up-to-date statistics on the fact table. Outdated statistics can lead to suboptimal plans.
Ensure that database settings (like buffer pool size) are optimized for your workload.

Query Rewriting:

Simplify complex queries by breaking them down into smaller parts.
Use derived tables, common table expressions (CTEs), or temporary views to reduce the load on the fact table.

#### Example
SQL
WITH filtered_data AS (
    SELECT * FROM fact_table WHERE condition1 AND condition2
)
SELECT * FROM filtered_data WHERE additional_conditions;

Partitioning:

If the data is time-series, partition by date or time to reduce the scanned portion of the table.

#### Example
SQL
CREATE TABLE fact_table (
    ...
) PARTITION BY RANGE (YEAR(date_column));

Materialized Views:

Create materialized views that precompute frequently used aggregates or filters.
These can be refreshed periodically to keep the data current.

Index Alternative: Bitmap Indexes:

If adding traditional indexes is not an option, consider bitmap indexing, which can significantly speed up queries with certain conditions.

Hardware Optimization:

Ensure proper hardware setup for your database server.
Optimize settings like buffer cache, shared memory, and I/O subsystems to handle the workload efficiently.

#### Closing Summary

By focusing on execution plan analysis, query rewriting, partitioning, materialized views, and sometimes alternative indexing methods (like bitmap indexes), you can optimize analytical queries that scan large fact tables. The key is understanding how your database manages costs and statistics, and leveraging these insights to craft more efficient queries.

#### Common Mistakes
and Follow-Ups
Mistake 1: Not using EXPLAIN or similar tools to analyze the execution plan.
Mistake 2: Overlooking existing data partitioning that could be leveraged.
Mistake 3: Failing to consider hardware optimization, which can significantly impact performance.

#### Follow-up
questions might include:

How do you handle concurrent access during materialized view refreshes?
What are the trade-offs between using bitmap indexes and traditional B-tree indexes in your scenario?

**Comparison:**

| Partition Strategy        | Explanation                                                                                           |
|---------------------------|-------------------------------------------------------------------------------------------------------|
| Range by Date            | Partitions data based on date ranges, reducing the amount of data scanned for queries.                |
| List by Category          | Partitions data based on a list of values in specific columns, optimizing queries with categorical filters. |

**Code:**

```sql
-- Example of EXPLAIN output
EXPLAIN SELECT * FROM fact_table WHERE condition1 AND condition2;
```

**Example input / output:**

```text
-- Example input and output
Input Query:
EXPLAIN SELECT * FROM fact_table WHERE date_column BETWEEN '2023-01-01' AND '2023-06-30';

Output Explanation:
+----+-------------+------------+-------+-------------------+-----------+---------+------+----------+----------------------------------------------+
| id | select_type | table      | type  | possible_keys     | key       | key_len | ref  | rows    | Extra                                        |
+----+-------------+------------+-------+-------------------+-----------+---------+------+----------+----------------------------------------------+
| 1  | SIMPLE      | fact_table | range | date_column       | date_column | 5       | NULL | 6000000 | Using where; Using temporary; Using filesort |
+----+-------------+------------+-------+-------------------+-----------+---------+------+----------+----------------------------------------------+
```

**Data:**

```yaml
# Schema for fact_table
fact_table:
  - id: integer
  - date_column: timestamp
  - value_column: float
  - category_column: string
```

**Flow:**

1. Analyze Execution Plan
2. Identify Bottlenecks
3. Query Rewriting
4. Partitioning
5. Maintain Materialized Views
6. Index Alternatives
7. Hardware Optimization

