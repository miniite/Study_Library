---
type: "practice"
difficulty: "Expert"
---

# You see a nested loop join over a large outer input causing millions of seeks. How would you push the optimizer toward a hash join, and when is that the wrong fix?

**Difficulty:** Expert · **Levels:** SDE-1, SDE-1/2 · [↑ Clustered Indexes and Indexed Views](<../../../../TOOLS/SQL_Server/SQL_Server_Platform/Clustered_Indexes_and_Indexed_Views.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the issue of a nested loop join over a large outer input in SQL Server, we need to understand what's happening under the hood and how we can influence the query optimizer to choose a different execution plan. The key is recognizing that a hash join could be more efficient for this situation but ensuring it’s the right choice.

#### Triage

The question revolves around understanding why a nested loop join might dominate in certain scenarios and how to steer the SQL Server optimizer toward a hash join, while also considering potential pitfalls of doing so. This involves an analysis of query execution plans, table sizes, indexing, and overall data distribution.

#### First-Principles Solution

Understanding Nested Loop Join vs Hash Join:

A nested loop join is efficient when the inner table is small enough to fit into memory (or at least a few times smaller than the outer table).
A hash join becomes more beneficial when the combined size of both tables exceeds this threshold, as it can handle larger datasets by creating an in-memory hash table.

Identifying the Issue:

The nested loop join is being used because the optimizer thinks that repeatedly accessing the inner table (which could be a large outer input) is more efficient.
This might happen if there are no suitable indexes, or if the statistics on the tables are outdated.

Correcting the Issue Using Query Hints:

Use OPTION (RECOMPILE) to force the query optimizer to reevaluate the plan at runtime based on current data and statistics.
Use WITH (FORCEPLAN) {plan XML} to specify an execution plan, directing SQL Server to use a hash join.

Optimizing Indexes:

Ensure that there are appropriate indexes on columns used in the join conditions and predicates.

#### Cons
ider creating or adjusting statistics to reflect the current state of the data more accurately.

Query Plan Tuning:

Use SET STATISTICS IO ON and SET STATISTICS TIME ON to gather detailed information about the execution plan.
Analyze the query plan using tools like SQL Server Management Studio (SSMS) or third-party tools such as Redgate SQL Monitor.

When Not to Push for Hash Join:

The hash join can be resource-intensive and might not be suitable if it requires too much memory, leading to out-of-memory errors.
If the tables involved are extremely large and frequent recompilations or memory issues arise, consider alternative strategies like partitioning.

#### Closing Summary

By understanding the difference between nested loop joins and hash joins, we can strategically influence SQL Server’s query optimizer. Using query hints and ensuring proper indexing can often guide the optimizer toward a more efficient plan. However, it's crucial to evaluate whether pushing for a hash join is the best approach given the specific characteristics of your workload.

#### Common Mistakes/Follow-Ups
Mistake: Assuming that larger tables always benefit from hash joins without considering memory constraints.

#### Follow-Up
Discussing other optimization techniques such as parallelism, partitioning, or rewriting queries to reduce the size of intermediate results.

**Code:**

```sql
-- Example query with hint
SELECT * FROM large_table lt
JOIN smaller_table st ON lt.key = st.key
OPTION (RECOMPILE);
```

