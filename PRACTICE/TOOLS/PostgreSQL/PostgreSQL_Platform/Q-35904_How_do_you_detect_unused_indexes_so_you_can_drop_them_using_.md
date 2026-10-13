---
type: "practice"
difficulty: "Deep"
---

# How do you detect unused indexes so you can drop them, using pg_stat_user_indexes?

**Difficulty:** Deep · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Indexing Multi-join Queries](<../../../../TOOLS/PostgreSQL/PostgreSQL_Platform/Indexing_Multi-join_Queries.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To detect unused indexes in PostgreSQL and understand when to drop them, we need to dive into how pg_stat_user_indexes works and what metrics it provides. This tool gives us insights into index usage, helping us identify indexes that are not being utilized effectively.

#### Triage

The question is asking for a method to identify unused indexes using the pg_stat_user_indexes system catalog in PostgreSQL. The interviewer wants to assess your understanding of how database statistics work and how you can leverage them to optimize performance by pruning redundant indexes. Key assumptions include:

You are familiar with PostgreSQL internals.
You understand index usage metrics.

#### First-Principles Solution

First, let's explore pg_stat_user_indexes. This system catalog stores information about the user-defined indexes in a database. It contains various statistics such as the number of rows inserted/updated/deleted, the total and sampled row counts, and other useful metadata.

To detect unused indexes, we can look at metrics like:

idx_scan and idx_tup_fetch: These indicate how many times the index was used to fetch data.
last vacuum or last autoanalyze: These timestamps tell us when the last time the statistics were updated.

We will write a query that filters out indexes with minimal usage. Here’s a step-by-step approach:

Collect Index Usage Data: Use pg_stat_user_indexes.
Filter Out Unused Indexes: Define what constitutes an unused index (e.g., low idx_scan count, long since last used).
Generate Recommendations: Provide a list of indexes that can be considered for removal.
Detailed Solution

Let's start by writing the SQL query to get relevant data from pg_stat_user_indexes.

SQL
-- Query to collect index usage statistics
SELECT
    relname AS table_name,
    idxname AS index_name,
    indisunique, -- true if unique index
    idx_scan,    -- number of times the index was used
    last_analyze,  -- last time statistics were collected
    last_autoanalyze -- last time autoanalyze ran
FROM
    pg_stat_user_indexes
JOIN
    pg_index ON pg_index.indexrelid = pg_stat_user_indexes.indexname::regclass
WHERE
    idx_scan < 10; -- Example threshold for low usage, adjust as needed

-- Optionally, add more filters like unique indexes or long since last used

This query lists indexes that have been scanned fewer than 10 times. You can set a different threshold based on the specific needs and characteristics of your database.

#### Closing Summary

To detect unused indexes in PostgreSQL using pg_stat_user_indexes, we filter out those with minimal usage, typically low idx_scan counts or long gaps since last use. This approach helps identify redundant indexes that can be dropped to improve performance.

#### Common Mistakes/Follow-Ups
Threshold Setting: Be careful setting the threshold; too high and you might miss important indexes, too low may produce a long list.
Index Dependencies: Consider dependencies of the index on other queries before dropping it.
Impact Analysis: Always test the impact of dropping an index in a staging environment to avoid unexpected issues.

Would you like me to walk through more examples or discuss trade-offs further?

**Code:**

```sql
-- Example query for unused indexes
SELECT
    relname AS table_name,
    idxname AS index_name,
    indisunique, -- true if unique index
    idx_scan,    -- number of times the index was used
    last_analyze,  -- last time statistics were collected
    last_autoanalyze -- last time autoanalyze ran
FROM
    pg_stat_user_indexes
JOIN
    pg_index ON pg_index.indexrelid = pg_stat_user_indexes.indexname::regclass
WHERE
    idx_scan < 10; -- Example threshold for low usage, adjust as needed

-- Optionally, add more filters like unique indexes or long since last used
```

