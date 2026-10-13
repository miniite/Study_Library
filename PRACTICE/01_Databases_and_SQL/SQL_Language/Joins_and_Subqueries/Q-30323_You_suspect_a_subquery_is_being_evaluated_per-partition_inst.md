---
type: "practice"
difficulty: "Deep"
---

# You suspect a subquery is being evaluated per-partition instead of once — how do you read the plan to confirm and rewrite it?

**Difficulty:** Deep · **Levels:** SDE-1, SDE-1/2 · [↑ Rewriting Subqueries as Joins](<../../../../01_Databases_and_SQL/SQL_Language/Joins_and_Subqueries/Rewriting_Subqueries_as_Joins.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To diagnose and resolve whether a subquery is being evaluated per partition, we need to understand the difference between a correlated subquery (evaluated per row) and a non-correlated subquery (executed once). The goal is to confirm this by analyzing the query execution plan and then rewrite it if necessary. Let's break down the steps from first principles.

Firstly, let’s establish what we are looking for in the execution plan. A correlated subquery that is being evaluated per partition will show multiple executions of the same subquery block within the plan. Conversely, a non-correlated subquery should appear only once in the execution plan, as it can be cached and reused.

#### Triage
What We Are Probing

The question asks us to determine if a subquery is evaluated per row (correlated) or not (non-correlated). The key here is to analyze the query execution plan for signs of correlated subqueries. If we find multiple executions, we need to rewrite the query to ensure it uses a non-correlated approach.

#### Assumptions
We have access to the database’s query execution plan.
The suspected query involves a subquery that might be evaluated per partition.
We understand basic concepts like correlated vs. non-correlated subqueries and their performance implications.
Step-by-Step Solution

Analyze the Execution Plan:

Use EXPLAIN (or equivalent) to get the execution plan of the query in question.
Look for repeated occurrences of the subquery block within the plan. Each time a subquery is evaluated, it will likely be listed as a separate operation.

Identify Correlated Subqueries:

A correlated subquery usually involves a nested loop join or other operations that repeat for each row processed.
Non-correlated subqueries, on the other hand, typically appear once and can leverage caching mechanisms to improve performance.

Rewrite the Query (if necessary):

If we find multiple executions of the same subquery block in the plan, consider rewriting the query to avoid correlated subqueries.
Use common table expressions (CTEs), derived tables, or temporary tables to materialize the subquery results and join with them instead.

#### Example Scenario

#### Cons
ider the following example:

SQL
SELECT t1.id, t2.value
FROM table1 t1
JOIN (
    SELECT id, value FROM table2 WHERE some_condition = t1.some_column
) t2 ON t1.id = t2.id;

Here, the subquery is evaluated for each row in table1, which could be expensive if table2 is large.

Rewritten Query

To avoid this, we can use a CTE or a temporary table:

SQL
WITH subquery_results AS (
    SELECT id, value FROM table2 WHERE some_condition = t1.some_column
)
SELECT t1.id, s.value
FROM table1 t1
JOIN subquery_results s ON t1.id = s.id;

This approach ensures that the subquery is evaluated only once for each unique id in table2.

#### Closing Summary
+ Common Mistakes/Follow-Ups
Common Mistake 1: Overlooking repeated subqueries in the execution plan, leading to inefficient queries.
Common Mistake 2: Failing to recognize the performance benefits of non-correlated subqueries.

#### Follow-Up
Question: "What are some other optimization techniques we can apply beyond just subqueries?"

By carefully analyzing the execution plan and rewriting the query where necessary, we ensure that subqueries are evaluated efficiently, avoiding redundant operations.

**Comparison:**

| Original Query Execution Plan |
| --- |
| 1. Scan on table2 (for each row in table1) |
| 2. Join with table1 |
| Rewritten Query Execution Plan |
| --- |
| 1. CTE execution once for all rows of table2 |
| 2. Join with table1 using the precomputed results |

**Code:**

```sql
-- Example of a correlated subquery
SELECT t1.id, t2.value
FROM table1 t1
JOIN (
    SELECT id, value FROM table2 WHERE some_condition = t1.some_column
) t2 ON t1.id = t2.id;

-- Rewritten query using CTE
WITH subquery_results AS (
    SELECT id, value FROM table2 WHERE some_condition = t1.some_column
)
SELECT t1.id, s.value
FROM table1 t1
JOIN subquery_results s ON t1.id = s.id;
```

**Example input / output:**

```text
-- Original query execution plan (example)
EXPLAIN SELECT t1.id, t2.value
FROM table1 t1
JOIN (
    SELECT id, value FROM table2 WHERE some_condition = t1.some_column
) t2 ON t1.id = t2.id;

-- Rewritten query execution plan (example)
EXPLAIN WITH subquery_results AS (
    SELECT id, value FROM table2 WHERE some_condition = t1.some_column
)
SELECT t1.id, s.value
FROM table1 t1
JOIN subquery_results s ON t1.id = s.id;
```

**Data:**

```sql
CREATE TABLE table1 (id INT PRIMARY KEY, some_column VARCHAR(50));
CREATE TABLE table2 (id INT, value VARCHAR(50), some_condition VARCHAR(50));

INSERT INTO table1 VALUES (1, 'A'), (2, 'B'), (3, 'C');
INSERT INTO table2 VALUES (1, 10, 'A'), (2, 20, 'B'), (4, 30, 'D');
```

**Flow:**

1. Yes
2. No
3. Original Query
4. Has Subquery Per Row?
5. Scan on Table2 (for each rowin Table1)
6. Rewritten Query with CTE
7. Join with Table1
8. CTE Execution Once for AllRows of Table2
9. Join with Table1 usingprecomputed results

