---
type: "practice"
difficulty: "Deep"
---

# A query joins five tables and the optimizer's chosen join order produces a giant intermediate — how do you use STRAIGHT_JOIN or a rewrite to guide it, and why is that a last resort?

**Difficulty:** Deep · **Levels:** SDE-1, SDE-1/2 · [↑ MySQL Optimizer Behavior](<../../../../TOOLS/MySQL/MySQL_Platform/MySQL_Optimizer_Behavior.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In this scenario, we're dealing with a complex query involving five tables where the MySQL optimizer has chosen an inferior join order that produces a large intermediate result set. The goal is to guide the query execution plan by using STRAIGHT_JOIN or rewriting the query. However, these are generally considered last-resort options due to their limitations and potential performance impacts.

First, let's understand why the current join order might be problematic. When dealing with many tables, the optimizer has to balance multiple factors like cardinality estimates, indexes, and costs of different operations. Sometimes, its choices may not align perfectly with our specific use case or data distribution.

#### Triage
What's Being Probed

The question is probing your understanding of:

Query Optimization: How MySQL chooses a join order and the role of the optimizer.
Query Rewriting Techniques: Ways to influence query execution without altering the logic too much.
Advanced Join Types: Understanding STRAIGHT_JOIN and when it might be useful.

#### Assumptions
The tables have a relatively large number of rows.
There are no suitable indexes that would significantly improve performance with the current join order.
The query is complex enough that optimizing it solely through indexing or restructuring isn't straightforward.

#### Solution
from First Principles
Option 1: Use STRAIGHT_JOIN

STRAIGHT_JOIN forces MySQL to perform a Cartesian product of the first two tables and then apply subsequent joins in the order specified. This can be useful if we know that one or more early join orders would drastically reduce the size of intermediate results.

SQL
SELECT STRAIGHT_JOIN * FROM table1, table2, table3, table4, table5 WHERE ... ;
Option 2: Query Rewriting

Rewriting the query to explicitly specify join order can sometimes guide MySQL’s optimizer. This approach involves restructuring the FROM clause and possibly using subqueries or temporary tables.

SQL
SELECT * FROM (
    SELECT * FROM table1 JOIN table2 ON ...
) AS t1
JOIN table3 ON ...
JOIN table4 ON ...
JOIN table5 ON ... ;
Why These Are Last Resort Options
Inflexibility: STRAIGHT_JOIN is rigid and can’t adapt to changing data distributions.
Performance Impact: Rewriting queries might introduce additional overhead or suboptimal plans if not done carefully.
Maintenance: Hardcoding join orders increases the complexity of maintaining the query and making future changes.

#### Closing Summary

While STRAIGHT_JOIN and rewriting queries are powerful tools, they should be used sparingly due to their limitations. Instead, consider more flexible approaches like indexing, using EXPLAIN for better understanding, or leveraging materialized views for complex cases.

#### Common Mistakes/Follow-Ups
Forgetting that STRAIGHT_JOIN forces the order without considering other factors.
Not thoroughly testing the rewritten query before deployment.
Overlooking simpler optimizations like adding indexes on join columns.

**Code:**

```sql
-- Example of using STRAIGHT_JOIN
SELECT STRAIGHT_JOIN * FROM table1, table2, table3, table4, table5 WHERE ... ;

-- Example of rewriting the query
SELECT * FROM (
    SELECT * FROM table1 JOIN table2 ON ...
) AS t1
JOIN table3 ON ...
JOIN table4 ON ...
JOIN table5 ON ... ;
```

