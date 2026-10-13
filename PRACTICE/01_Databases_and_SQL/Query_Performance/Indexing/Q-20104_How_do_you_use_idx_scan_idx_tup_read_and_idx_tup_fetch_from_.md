---
type: "practice"
difficulty: "Deep"
---

# How do you use idx_scan, idx_tup_read, and idx_tup_fetch from index statistics to judge whether an index is pulling its weight?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1, SDE-1/2, SDE-2/3 · [↑ Index-only Scans](<../../../../01_Databases_and_SQL/Query_Performance/Indexing/Index-only_Scans.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To determine if an index is performing well using idx_scan, idx_tup_read, and idx_tup_fetch from index statistics, you need to understand these metrics in the context of PostgreSQL's query execution plan. These statistics help identify whether the index is being used efficiently or if it's causing more harm than good.

#### Triage
Understanding the Metrics
idx_scan: The number of times an index was scanned.
idx_tup_read: The number of tuples read from the index during a scan.
idx_tup_fetch: The number of rows fetched after resolving the index to find the actual data in the table.

#### First Principles Solution
Scan Frequency:
A high idx_scan count indicates that the index is being used frequently, which is good if it reduces the number of disk reads.
Tup Read vs Fetch Ratio:
The ratio between idx_tup_read and idx_tup_fetch should be examined. Ideally, this should be low (e.g., < 10:1), indicating that the index is effective in reducing the number of tuples to fetch.
Index Type and Selectivity:
Ensure the index type (B-tree, hash, GiST) and selectivity are appropriate for the query patterns.
Step-by-Step Analysis
Identify Query Patterns: Analyze the queries that use this index.
Calculate Metrics:
SQL
SELECT idx_scan, idx_tup_read, idx_tup_fetch FROM pg_statistic WHERE tablename = 'your_table_name' AND indexname = 'your_index_name';
Evaluate Performance:
If idx_scan is high and the ratio of idx_tup_read to idx_tup_fetch is low, the index is likely performing well.
Conversely, if idx_scan is high but the ratio is very high (e.g., > 50:1), consider whether the index is overused or if it should be rebuilt.

#### Closing Summary

By analyzing these metrics, you can determine whether an index is underperforming. If the index is being scanned frequently but not reducing the number of tuples to fetch significantly, it might be better off removed or modified.

#### Common Mistakes
and Follow-Ups
Overly Broad Indexes: Indexes on columns that are rarely used in queries.
High Maintenance Cost: Overusing an index that leads to frequent scans without significant benefits.
Rebuild vs. Drop: Consider dropping and recreating the index or using a more selective approach.

**Code:**

```sql
-- Example SQL query to get index statistics
SELECT idx_scan, idx_tup_read, idx_tup_fetch
FROM pg_statistic
WHERE tablename = 'your_table_name' AND indexname = 'your_index_name';
```

