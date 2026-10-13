---
type: "practice"
difficulty: "Basic"
---

# What is the purpose of the ANALYZE keyword in EXPLAIN ANALYZE and how does it differ from plain EXPLAIN?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ Reading Execution Plans](<../../../../01_Databases_and_SQL/Query_Performance/Execution_Plans/Reading_Execution_Plans.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

The ANALYZE keyword within the EXPLAIN ANALYZE command is a powerful tool for database developers to gain insights into how their queries are executed and optimized in PostgreSQL. It provides both an execution plan and actual runtime statistics, whereas plain EXPLAIN only gives you the execution plan without any performance metrics.

#### Triage

The question aims to understand the difference between EXPLAIN ANALYZE and EXPLAIN. We need to clarify that ANALYZE is a specific keyword used within EXPLAIN to provide runtime statistics. The key difference lies in what information each command outputs: EXPLAIN shows only how the query will be executed, while EXPLAIN ANALYZE gives both the execution plan and actual performance metrics.

Step-by-Step Solution

Understanding EXPLAIN:

EXPLAIN is a useful command that helps developers understand how a SQL query would be executed by PostgreSQL without actually executing it.
It provides information such as which indexes are being used, the type of join methods (e.g., hash join, nested loop), and the estimated number of rows processed.

Introducing ANALYZE:

The ANALYZE keyword is part of a larger command sequence when combined with EXPLAIN ANALYZE.
When used in this context, ANALYZE tells PostgreSQL to collect statistics on the query's execution and output these results along with the plan.

Execution Plan vs. Runtime Statistics:

EXPLAIN alone provides a high-level overview of how the database plans to execute your query.
EXPLAIN ANALYZE goes one step further by executing the query (or parts of it) and providing runtime statistics such as actual time taken, number of rows processed per operator, disk I/O operations, and more.

Practical Example:

SQL
EXPLAIN SELECT * FROM customers WHERE age > 30;

This command shows the execution plan but does not run the query or provide runtime statistics.

SQL
EXPLAIN ANALYZE SELECT * FROM customers WHERE age > 30;

This command shows both the execution plan and actual runtime statistics, giving you a more comprehensive view of how your query performs in real conditions.

#### Closing Summary
+ Common Mistakes/Follow-Ups

ANALYZE within EXPLAIN ANALYZE is crucial for developers because it provides both the expected behavior (from EXPLAIN) and actual performance metrics. This dual output helps in tuning queries to improve their efficiency, especially when dealing with complex operations or large datasets.

#### Common mistakes
include:

Not using ANALYZE at all, leading to only theoretical plans that may not reflect real-world performance.
Overlooking the runtime statistics provided by EXPLAIN ANALYZE, which can highlight bottlenecks and areas for optimization.

#### Follow-ups
might include discussing how frequently one should run these commands, the impact of data distribution on query performance, and how to interpret the various metrics in the output.

**Code:**

```sql
-- Example EXPLAIN command
EXPLAIN SELECT * FROM customers WHERE age > 30;

-- Example EXPLAIN ANALYZE command
EXPLAIN ANALYZE SELECT * FROM customers WHERE age > 30;
```

