---
type: "practice"
difficulty: "Deep"
---

# A stored procedure that ran in 200ms suddenly takes 30 seconds after a data load. Walk me through how you would diagnose whether it is a plan-caching problem versus a statistics problem.

**Difficulty:** Deep · **Levels:** SDE-1, SDE-1/2 · [↑ Stored Procedure Performance and Design](<../../../../01_Databases_and_SQL/Database_Objects/Stored_Procedures/Stored_Procedure_Performance_and_Design.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When troubleshooting a stored procedure that has regressed from 200ms to 30 seconds after a data load, we need to systematically investigate potential causes. The two primary areas of focus are plan caching and statistics. Plan caching can lead to suboptimal execution plans being reused, while outdated or incorrect statistics might cause the query optimizer to choose a poor execution path.

Here’s how I would approach this:

#### Overview

We need to diagnose whether the performance issue is due to plan caching or statistics. Both are common causes for such regressions.
I will start by gathering information using SQL Server's built-in tools and system views, which provide detailed insights into execution plans and query costs.

#### Triage

The interviewer is probing my ability to use diagnostic tools effectively and understand the nuances of execution plans and statistics in SQL Server.

#### Assumptions
include that the data load has not significantly altered the schema or indexes, and we have access to relevant system views such as sys.dm_exec_query_stats, sys.dm_exec_plan_attributes, and sys.dm_exec_sql_text.

#### First-Principles Solution

a. Check Execution Plan Caching: - Use sys.dm_exec_query_stats to identify the most recent execution of the stored procedure. - Examine the plan hash value using sys.dm_exec_plan_attributes to see if there are multiple plans for the same query.

b. Analyze Query Cost and Statistics: - Use sys.dm_exec_sql_text to retrieve the SQL text of the stored procedure. - Use sys.dm_db_index_physical_stats to check statistics on relevant indexes. - Compare current statistics with those before the data load using sys.stats.

c. Review Execution Plan Details: - Look for costly operations in the execution plan, such as full scans or sorts that might have become more expensive due to new data distribution.

#### Closing Summary
and Common Mistakes/Follow-Ups:

By systematically reviewing execution plans and statistics, we can determine if either caching issues or outdated statistics are driving the performance degradation.
I would also suggest implementing regular statistics updates to prevent such issues in the future.

**Comparison:**

                      | idx_column2 (3000 rows, 10 updates) |

**Code:**

```sql
-- Example SQL to check for multiple plans of a stored procedure
SELECT
    qs.execution_count,
    qs.total_worker_time,
    qp.query_plan
FROM
    sys.dm_exec_query_stats AS qs
CROSS APPLY
    sys.dm_exec_sql_text(qs.sql_handle) AS st
CROSS APPLY
    sys.dm_exec_plan_attributes(qs.plan_handle, N'index') as plan_attr
WHERE
    st.text LIKE '%your_stored_procedure_name%'
ORDER BY
    total_worker_time DESC;

-- Example SQL to check statistics for relevant indexes
SELECT
    s.name,
    i.name,
    sp.stats_id,
    stats_row_count,
    stat_updates
FROM
    sys.tables t
JOIN
    sys.indexes i ON t.object_id = i.object_id
JOIN
    sys.stats sp ON i.object_id = sp.object_id AND i.index_id = sp.stats_id
WHERE
    t.name = 'your_table_name'
ORDER BY
    stats_row_count DESC;
```

**Example input / output:**

```text
Input:
- Stored procedure name: sp_example

Output:
- Execution plan analysis for most recent runs of sp_example.
- Statistics report showing stats_row_count and stat_updates for relevant indexes.
```

**Data:**

```json
{
  "table_name": "YourTableName",
  "indexes": [
    {"name": "idx_column1", "stats_row_count": 5000, "stat_updates": 20},
    {"name": "idx_column2", "stats_row_count": 3000, "stat_updates": 10}
  ]
}
```

**Flow:**

1. Yes
2. Yes
3. No
4. No
5. Start
6. Is there multiple plans in theexecution cache?
7. Check for statistics differences
8. Are recent stats different frombaseline?
9. Update statistics and retest
10. Examine execution plan detailsfor costly operations
11. Assess overall query cost andperformance metrics

