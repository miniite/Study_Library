---
type: "practice"
difficulty: "Intermediate"
---

# A query joins a table on a VARCHAR column against another's INT and it's slow with implicit casting — how do you spot the type-coercion index kill and fix it?

**Difficulty:** Intermediate · **Levels:** SDE-1/2, SDE-2/3 · [↑ MySQL Optimizer Behavior](<../../../../TOOLS/MySQL/MySQL_Platform/MySQL_Optimizer_Behavior.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In this scenario, we're dealing with a performance issue in MySQL where an inefficient query is causing slow execution times due to implicit type coercion when joining tables on columns of different data types. Specifically, there's a VARCHAR column being joined against an INT column without proper indexing or explicit casting. Let’s break down the steps required to diagnose and resolve this problem.

Firstly, we need to understand that MySQL performs implicit type conversion in certain situations, such as when comparing values from columns with different data types. This can lead to inefficient execution plans because it may prevent the use of indexes. The core of our task is to identify if a type coercion index kill has occurred and then correct the issue.

#### Triage
What's Being Probed

The interviewer is probing your understanding of:

Indexing strategies in MySQL.
Type coercion and its impact on query performance.
Optimizing queries using explicit casting or other techniques.
Step-by-Step Solution
1. Analyze the Query Execution Plan

First, we need to inspect the query execution plan to see if MySQL is indeed performing type coercion. We can do this by enabling detailed EXPLAIN output:

SQL
EXPLAIN SELECT * FROM table1 t1 JOIN table2 t2 ON t1.varchar_column = t2.int_column;

Look for signs of type coercion in the type and possible_keys columns. For example, if MySQL is using a ALL access method on one of the tables, it might indicate that no index was used due to implicit casting.

2. Identify the Type Coercion

If the execution plan shows that an index could have been used but wasn't, it suggests type coercion. We can further confirm this by checking the query’s actual execution plan:

SQL
EXPLAIN FORMAT=JSON SELECT * FROM table1 t1 JOIN table2 t2 ON t1.varchar_column = t2.int_column;

The JSON output will give us more detailed information about how MySQL is interpreting and executing the query. Specifically, look for clues like "Impossible WHERE" or "Using filesort" which often indicate type coercion issues.

3. Explicit Casting

To fix the issue, we need to ensure that both columns are of compatible data types during the join condition. We can achieve this by explicitly casting the VARCHAR column to INT:

SQL
SELECT * FROM table1 t1 JOIN table2 t2 ON CAST(t1.varchar_column AS UNSIGNED) = t2.int_column;

Alternatively, we could also create a virtual computed column in table1 that stores the integer value of the VARCHAR column and then use it for the join:

SQL
ALTER TABLE table1 ADD COLUMN int_varchar_column INT AS (CAST(varchar_column AS UNSIGNED));
SELECT * FROM table1 t1 JOIN table2 t2 ON t1.int_varchar_column = t2.int_column;
4. Re-indexing and Optimization

After making the necessary changes, it’s a good practice to re-analyze and optimize the indexes involved:

SQL
ANALYZE TABLE table1, table2;
OPTIMIZE TABLE table1, table2;

This ensures that MySQL has up-to-date statistics for better query optimization.

#### Closing Summary
+ Common Mistakes/Follow-Ups

In summary, slow joins due to implicit casting can be effectively managed by explicitly casting columns or creating computed columns. It’s crucial to regularly review and optimize your indexes and queries as part of a good database maintenance strategy.

#### Common mistakes
include not recognizing the impact of type coercion, relying solely on implicit conversions, and failing to update statistics and re-analyze tables after making schema changes.

For follow-up questions, you might be asked about alternative solutions like using an intermediate table or partitioning strategies for large datasets. Always consider the broader context of your database design and performance goals.

**Code:**

```sql
-- Example SQL code for explicit casting
SELECT * FROM table1 t1 JOIN table2 t2 ON CAST(t1.varchar_column AS UNSIGNED) = t2.int_column;

-- Adding a computed column example
ALTER TABLE table1 ADD COLUMN int_varchar_column INT AS (CAST(varchar_column AS UNSIGNED));
SELECT * FROM table1 t1 JOIN table2 t2 ON t1.int_varchar_column = t2.int_column;
```

