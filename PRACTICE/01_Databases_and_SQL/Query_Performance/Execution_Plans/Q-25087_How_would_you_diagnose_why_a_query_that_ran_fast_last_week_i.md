---
type: "practice"
difficulty: "Intermediate"
---

# How would you diagnose why a query that ran fast last week is now slow?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ Diagnosing Query Regressions](<../../../../01_Databases_and_SQL/Query_Performance/Execution_Plans/Diagnosing_Query_Regressions.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To diagnose why a previously fast-running query is now slow, we'll follow a systematic approach focusing on execution plans and query tuning. We need to identify if changes in data distribution or schema, new indexes, or other factors have affected the query's performance.

First, let’s restate what this question is probing: it’s about diagnosing a performance regression in a database query by examining its execution plan and understanding potential causes of slowdowns. The key assumptions are that we have access to relevant logs and monitoring tools, and the database is relatively stable with no recent major schema changes.

Step-by-Step Solution

Reproduce the Query Execution:

First, ensure that you can run the query again in a controlled environment to capture its current execution plan.

Compare Execution Plans:

Use tools like EXPLAIN or similar commands available in your database (e.g., EXPLAIN ANALYZE in PostgreSQL) to get the current and historical execution plans of the query.
Compare these plans side-by-side to identify any significant differences.

Analyze Key Differences:

Look for changes in the plan such as:
Different choice of execution path (e.g., from hash join to nested loop).
Changes in access methods (e.g., from index scan to sequential scan).
Variations in estimated vs actual row counts and costs.

Check for Data Distribution:

Evaluate if the data has changed significantly, affecting the query’s performance. This could be due to data skew or an increase in row count that impacts the choice of execution plan.

Examine Index Usage:

Ensure that indexes are being used efficiently as they were before.
Check for missing indexes or outdated statistics that might have led to suboptimal query plans.

Monitor Resource Utilization:

Review system resource utilization (CPU, I/O) during the execution of the slow query compared to the fast one.

Tune Query and Indexes:

Based on your findings, consider optimizing the query or adding/removing indexes if necessary.

Validate Changes:

After making any adjustments, monitor the performance again to ensure that the changes have resolved the issue.

#### Closing Summary

In summary, diagnosing a slow-running query involves comparing its execution plans with historical data and analyzing various factors like data distribution, index usage, and resource utilization. The key is to methodically identify and address potential issues step-by-step to restore optimal performance.

#### Common Mistakes
and Follow-Ups
Failing to Compare Plans: Not having a baseline for comparison can lead to missing critical insights.
Ignoring Resource Utilization: Overlooking system resources can hide underlying bottlenecks.
Over-Tuning Without Data: Jumping straight into tuning without understanding the root cause can be ineffective.

**Code:**

```sql
-- Example of running EXPLAIN ANALYZE on a query to capture current execution plan
EXPLAIN ANALYZE SELECT * FROM sales WHERE date > '2023-10-01';
```

**Flow:**

1. Reproduce the Query
2. Compare Execution Plans
3. Analyze Key Differences
4. Check Data Distribution
5. Examine Index Usage
6. Monitor Resource Utilization
7. Tune Query and Indexes
8. Validate Changes

