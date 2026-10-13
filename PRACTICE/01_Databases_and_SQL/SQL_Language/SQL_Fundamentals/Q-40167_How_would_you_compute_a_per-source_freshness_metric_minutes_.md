---
type: "practice"
difficulty: "Intermediate"
---

# How would you compute a per-source freshness metric (minutes since latest loaded row) across dozens of tables in one warehouse query so a single dashboard shows every pipeline's staleness?

**Difficulty:** Intermediate · **Levels:** SDE-1/2, SDE-2/3 · [↑ Replica Staleness](<../../../../01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Replica_Staleness.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To compute a per-source freshness metric, we need to determine the timestamp of the most recent loaded row for each table and then calculate the time difference between that timestamp and the current time. This will give us an idea of how stale the data is in each pipeline. The goal is to achieve this in one query so that the result can be easily visualized on a single dashboard.

#### Triaging the Question

The question asks for a SQL solution that computes the freshness metric across multiple tables and sources. Here, "sources" likely refer to different tables or datasets, possibly from various data pipelines. The freshness is defined as the time elapsed since the most recent insert or update in each table. The challenge is to consolidate this information into one query.

#### First-Principles Solution

To solve this problem, we'll need to:

Identify the latest timestamp for each table.
Calculate the difference between this latest timestamp and the current system time.
Aggregate these differences by source (table).
Step-by-Step Solution
1. Identify Latest Timestamps

First, we need a way to find the most recent insert or update in each table. Typically, this is stored as a created_at or updated_at timestamp in each row.

SQL
WITH latest_timestamps AS (
    SELECT
        table_name,
        MAX(created_at) AS last_inserted_time
    FROM
        your_database.your_schema.*
    GROUP BY
        table_name
)
2. Calculate Freshness

Next, we calculate the freshness for each latest_timestamps entry by subtracting it from the current system time.

SQL
WITH latest_timestamps AS (
    SELECT
        table_name,
        MAX(created_at) AS last_inserted_time
    FROM
        your_database.your_schema.*
    GROUP BY
        table_name
),
freshness AS (
    SELECT
        lt.table_name,
        (CURRENT_TIMESTAMP - lt.last_inserted_time) / 60.0 AS freshness_minutes
    FROM
        latest_timestamps lt
)
3. Final Aggregation and Dashboard-friendly Output

We can then aggregate these results to create a single row for each source, which is what we need for the dashboard.

SQL
WITH latest_timestamps AS (
    SELECT
        table_name,
        MAX(created_at) AS last_inserted_time
    FROM
        your_database.your_schema.*
    GROUP BY
        table_name
),
freshness AS (
    SELECT
        lt.table_name,
        (CURRENT_TIMESTAMP - lt.last_inserted_time) / 60.0 AS freshness_minutes
    FROM
        latest_timestamps lt
)
SELECT
    MIN(table_name) AS source,
    AVG(freshness_minutes) AS avg_freshness_minutes,
    MAX(freshness_minutes) AS max_freshness_minutes,
    COUNT(*) AS num_tables
FROM
    freshness
GROUP BY
    ROLLUP (table_name);

#### Closing Summary
+ Common Mistakes/Follow-Ups

The above query computes the average, maximum, and count of tables per source. This provides a comprehensive view of how stale data is across all sources in one dashboard-friendly result.

#### Common Mistakes

Forgetting to convert time differences into minutes.
Not handling NULL values properly, which could occur if no rows are found.
Overlooking the use of window functions or CTEs for better readability and maintainability.

#### Follow-Ups

How would you handle tables without a created_at timestamp?
What if some sources update more frequently than others?
How would this change if we need to consider both insert and update timestamps?

**Code:**

```sql
WITH latest_timestamps AS (
    SELECT
        table_name,
        MAX(created_at) AS last_inserted_time
    FROM
        your_database.your_schema.*
    GROUP BY
        table_name
),
freshness AS (
    SELECT
        lt.table_name,
        (CURRENT_TIMESTAMP - lt.last_inserted_time) / 60.0 AS freshness_minutes
    FROM
        latest_timestamps lt
)
SELECT
    MIN(table_name) AS source,
    AVG(freshness_minutes) AS avg_freshness_minutes,
    MAX(freshness_minutes) AS max_freshness_minutes,
    COUNT(*) AS num_tables
FROM
    freshness
GROUP BY
    ROLLUP (table_name);
```

