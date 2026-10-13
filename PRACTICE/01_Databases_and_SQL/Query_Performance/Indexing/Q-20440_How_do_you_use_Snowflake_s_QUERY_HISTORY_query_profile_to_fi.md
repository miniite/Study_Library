---
type: "practice"
difficulty: "Intermediate"
---

# How do you use Snowflake's QUERY_HISTORY / query profile to find a query with high partitions-scanned vs partitions-total, and what does that ratio tell you?

**Difficulty:** Intermediate · **Levels:** SDE-1/2, SDE-2/3 · [↑ Partitioning and Data Skipping in Lakes](<../../../../01_Databases_and_SQL/Query_Performance/Indexing/Partitioning_and_Data_Skipping_in_Lakes.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the question of using Snowflake’s QUERY_HISTORY and query_profile features to identify queries with high partition scan ratios, let's start by understanding the core concept and the tools at our disposal.

TL;DR

Snowflake provides valuable insights through QUERY_HISTORY and query_profile, which can be used to analyze query performance. Specifically, we're interested in identifying queries that have a high ratio of partitions scanned relative to total partitions processed. This ratio often indicates inefficient data access patterns or suboptimal query execution plans.

#### Triage

When faced with the task of finding queries with high partitions_scanned vs partitions_total ratios, the interviewer is probing your ability to:

Understand Snowflake’s monitoring tools and features.
Interpret performance metrics effectively.
Identify potential inefficiencies in query execution plans.

#### First-Principles Solution
Step 1: Understanding the Metrics
Partitions Total: The total number of partitions accessed by a query.
Partitions Scanned: The number of partitions that were scanned and processed during query execution.

A high partitions_scanned vs partitions_total ratio suggests that many partitions were read but not fully utilized, potentially indicating inefficient data filtering or partitioning strategies.

Step 2: Accessing Query History

To access these metrics, we can use the QUERY_HISTORY view in Snowflake. This view provides a historical log of queries executed along with various performance-related details.

SQL
SELECT
    query_id,
    query_text,
    partitions_scanned,
    partitions_total,
    rows_returned,
    elapsed_time_ms
FROM
    SNOWFLAKE.ACCOUNT_USAGE.QUERY_HISTORY
WHERE
    query_start_time >= 'YYYY-MM-DD HH:MM:SS'
ORDER BY
    partitions_scanned DESC;

This SQL statement retrieves queries sorted by partitions_scanned in descending order, allowing us to focus on those with the highest ratios.

Step 3: Analyzing Query Profiles

For a more detailed analysis of individual query performance, we can use the query_profile view. This view provides an execution plan breakdown, including partition-level details.

SQL
SELECT
    query_id,
    step_id,
    operation_type,
    partitions_scanned,
    partitions_total,
    rows_returned,
    elapsed_time_ms
FROM
    SNOWFLAKE.ACCOUNT_USAGE.QUERY_PROFILE
WHERE
    query_id = 'YOUR_QUERY_ID';

By examining the query_profile, we can pinpoint specific steps where a high number of partitions were scanned but not fully utilized, suggesting inefficiencies.

Step 4: Identifying and Addressing Inefficiencies

High ratios often indicate issues such as:

Poor data partitioning strategy.
Insufficient filters applied in the WHERE clause.
Unoptimized query execution plans.

To address these issues, we can:

Re-evaluate Partitioning Strategy: Ensure that partitions are created based on frequently used columns for filtering.
Optimize Query Filters: Add more specific filters to reduce the number of partitions scanned.
Refine Indexes: Create or modify indexes to support the query’s access patterns.

#### Closing Summary
and Common Mistakes

In summary, using QUERY_HISTORY and query_profile, we can identify queries with high partitions_scanned vs partitions_total ratios by analyzing historical data and execution plans. This analysis helps in pinpointing inefficient query behaviors and provides actionable insights for optimization.

#### Common mistakes
include:

Failing to consider the specific use case of the data.
Ignoring the context of the query volume and frequency.
Overlooking the impact of schema changes on query performance.
Common Follow-Ups
How do you ensure that your queries are optimized over time?
What are some best practices for managing Snowflake’s partitioning strategy?

**Code:**

```

SELECT query_id, partitions_scanned, partitions_total, rows_returned, elapsed_time_ms FROM SNOWFLAKE.ACCOUNT_USAGE.QUERY_HISTORY WHERE query_start_time >= '2023-10-01 00:00:00' ORDER BY partitions_scanned DESC;

CODE
Copy

```sql
SELECT
    query_id,
    step_id,
    operation_type,
    partitions_scanned,
    partitions_total,
    rows_returned,
    elapsed_time_ms
FROM
    SNOWFLAKE.ACCOUNT_USAGE.QUERY_PROFILE
WHERE
    query_id = 'YOUR_QUERY_ID';
```

