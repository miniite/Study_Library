---
type: "practice"
difficulty: "Deep"
---

# A materialized view got out of sync with base tables after a bulk load bypassed logging; how do you detect the drift and force a correct rebuild?

**Difficulty:** Deep · **Levels:** SDE-1, SDE-1/2 · [↑ Materialized View Refresh Correctness](<../../../../01_Databases_and_SQL/Database_Objects/Views/Materialized_View_Refresh_Correctness.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When dealing with a materialized view (MV) that has become out of sync due to a bulk load that bypassed transaction logs, it's crucial to understand the process from first principles to ensure a comprehensive solution. In this context, we need to detect any discrepancies between the MV and its base tables and then force a correct rebuild of the MV. This involves a series of steps that consider data integrity, performance, and minimal disruption.

Firstly, let's clarify what the question is really asking: How can we detect when an MV has drifted from its base tables? And how do we ensure a proper rebuild without causing further issues or downtime?

Key Assumptions
The bulk load did not use transaction logs for change tracking.
The MV is defined as a materialized view with a query that references the base tables.
We need to maintain data integrity and minimize disruptions.
Step-by-Step Solution
1. Identify Drift

To detect drift, we need to compare the current state of the MV with the expected state based on the latest base table data.

Dry Run Comparison: Compare a sample of data from both the MV and the relevant base tables.
Full Table Scan (if necessary): If the dry run reveals significant discrepancies or if you want to be thorough, perform a full comparison by scanning all rows in the MV and comparing them with their corresponding records in the base tables.
2. Data Comparison

For a detailed comparison:

SQL
SELECT *
FROM (
    SELECT t1.*, 'MV' AS source FROM mv_table t1
    UNION ALL
    SELECT t2.*, 'Base Tables' AS source FROM base_tables t2
) AS combined_data
WHERE (t1.column1 = t2.column1 AND t1.column2 = t2.column2 AND ... AND t1.columnN != t2.columnN)
OR (t1.column1 != t2.column1 OR t1.column2 != t2.column2 OR ... OR t1.columnN != t2.columnN);

This query will return rows where the MV and base tables differ, helping us pinpoint which records are out of sync.

3. Analyze Discrepancies

Review the results to understand why discrepancies exist:

Are there data type mismatches?
Have there been insertions or deletions in the base tables that weren't reflected in the MV?
Is there a timing issue with the last update?
4. Force Rebuild of Materialized View

Once drift is detected, you can force a correct rebuild of the MV by dropping and recreating it:

SQL
-- Drop the materialized view if it exists
DROP MATERIALIZED VIEW mv_table;

-- Recreate the materialized view with proper data integrity checks
CREATE MATERIALIZED VIEW mv_table AS
SELECT column1, column2, ...
FROM base_tables
WHERE condition;
5. Monitor and Validate

After rebuilding, monitor the MV to ensure it accurately reflects the current state of the base tables:

Re-run the comparison query.
Ensure that all expected data rows are present in the MV.

#### Closing Summary
+ Common Mistakes/Follow-Ups

In summary, detecting drift and forcing a correct rebuild involves thorough comparisons, understanding the underlying data discrepancies, and ensuring minimal disruption to the system. Common mistakes include not performing a full scan when necessary, failing to validate after rebuilding, or incorrectly assuming that the bulk load was fully logged.

For follow-ups, consider:

Implementing more robust transaction logging in future loads.
Using change data capture (CDC) technologies for real-time synchronization.
Automating drift detection and rebuild processes using scheduled jobs.

**Code:**

```sql
-- Example SQL to compare MV and base tables
SELECT *
FROM (
    SELECT t1.*, 'MV' AS source FROM mv_table t1
    UNION ALL
    SELECT t2.*, 'Base Tables' AS source FROM base_tables t2
) AS combined_data
WHERE (t1.column1 = t2.column1 AND t1.column2 = t2.column2 AND ... AND t1.columnN != t2.columnN)
OR (t1.column1 != t2.column1 OR t1.column2 != t2.column2 OR ... OR t1.columnN != t2.columnN);
```

