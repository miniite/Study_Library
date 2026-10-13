---
type: "practice"
difficulty: "Deep"
---

# How do you interpret and act on a high signal from sys.dm_db_index_usage_stats when deciding to drop an index?

**Difficulty:** Deep · **Levels:** SDE-1, SDE-1/2 · [↑ Filtered Indexes and Plan Cache](<../../../../TOOLS/SQL_Server/T-SQL/Filtered_Indexes_and_Plan_Cache.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When faced with a decision about whether to drop an index based on the sys.dm_db_index_usage_stats dynamic management view (DMV), it's crucial to interpret its outputs accurately and act thoughtfully, considering both performance implications and potential future needs. TL;DR: Use the DMV data to identify unused or rarely used indexes, but don’t rush to drop them without validating their necessity across different query scenarios.

#### Triage
What is Being Probed

The question asks for an interpretation of a high signal from sys.dm_db_index_usage_stats and how it should inform your decision-making process regarding dropping an index. The key points are:

Understanding the meaning of "high signal" in this context.
Evaluating whether the DMV data alone is sufficient to justify dropping an index.

#### Cons
idering other factors that might influence such a decision.

#### Assumptions
You have access to sys.dm_db_index_usage_stats, which provides statistics on how SQL Server indexes are used by queries.
The "high signal" refers to a significant value in one or more columns from the DMV, indicating low usage of an index.

#### First-Principles Solution

Identify Low Usage Indexes: Review sys.dm_db_index_usage_stats for indexes with very low or no recent usage (e.g., zero reads, writes, etc.). This query can help identify such indexes:

SQL
SELECT
    OBJECT_NAME(i.object_id) AS TableName,
    i.name AS IndexName,
    i.type_desc AS IndexType,
    us.user_seeks,
    us.user_scans,
    us.user_lookups,
    us.user_updates
FROM
    sys.dm_db_index_usage_stats us
INNER JOIN
    sys.indexes i ON us.object_id = i.object_id AND us.index_id = i.index_id
WHERE
    OBJECTPROPERTY(us.object_id, 'IsUserTable') = 1
    AND (us.user_seeks + us.user_scans + us.user_lookups + us.user_updates) = 0;

Evaluate Index Necessity: Even if an index is unused, it might be necessary for maintaining performance in certain query scenarios. Check:

If the index supports a frequently used or complex query.
If dropping the index could impact the execution plan of existing queries.

#### Cons
ider Future Needs: Sometimes, indexes are created to support future query requirements that haven’t been identified yet.

Monitor and Validate: Before actually dropping an index, monitor its usage for some time (e.g., a week or month) using sys.dm_db_index_usage_stats again. This helps ensure that the decision is well-informed.

Impact Analysis: Use SQL Server’s execution plan feature to analyze how queries might perform without the index.

#### Closing Summary

In summary, while high signals in sys.dm_db_index_usage_stats can indicate that an index may not be necessary for current workload, it's important to conduct a thorough analysis before dropping it. Factors such as future query requirements and potential impact on existing queries must be considered. Always validate the decision by monitoring usage over time.

#### Common Mistakes/Follow-Ups
Overzealous Index Removal: Dropping an index based solely on low usage without considering its necessity.
Inadequate Monitoring: Not monitoring the index’s usage post-deletion to verify if it was indeed unnecessary.
Ignoring Future Requirements: Failing to consider potential future queries that might require the dropped index.

**Code:**

```sql
SELECT
    OBJECT_NAME(i.object_id) AS TableName,
    i.name AS IndexName,
    i.type_desc AS IndexType,
    us.user_seeks,
    us.user_scans,
    us.user_lookups,
    us.user_updates
FROM
    sys.dm_db_index_usage_stats us
INNER JOIN
    sys.indexes i ON us.object_id = i.object_id AND us.index_id = i.index_id
WHERE
    OBJECTPROPERTY(us.object_id, 'IsUserTable') = 1
    AND (us.user_seeks + us.user_scans + us.user_lookups + us.user_updates) = 0;
```

