---
type: "practice"
difficulty: "Intermediate"
---

# How do you tune a query whose cost blows up because of an unexpected cross product from a missing join condition?

**Difficulty:** Intermediate · **Levels:** SDE-1, SDE-1/2 · [↑ Query Rewrites and Plan Stability](<../../../../01_Databases_and_SQL/Query_Performance/Execution_Plans/Query_Rewrites_and_Plan_Stability.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When tuning a query that exhibits high costs due to an unexpected cross product, it's essential to first understand why this happens and then apply appropriate strategies to optimize the query plan. Typically, a cross product (or Cartesian join) occurs when there is no proper join condition between two tables in a JOIN clause, leading to every row from one table being matched with every row of the other table—resulting in an explosion of rows processed by the database engine.

Triaging the Problem

The question is essentially asking for steps and strategies to diagnose and resolve performance issues caused by missing join conditions. This involves understanding how SQL query execution plans are generated, identifying the root cause, and applying corrective measures like adding or modifying join conditions, indexing, or re-writing queries.

#### Assumptions
The database management system (DBMS) in use is PostgreSQL.
The performance issue manifests as high cost or long execution time of a specific query.
We have access to the EXPLAIN output and possibly the SQL query itself.

#### First-Principles Solution

Let's break down the process step-by-step:

Review the Query: Examine the SQL query to identify any potential missing join conditions. Look for JOIN clauses without explicit join criteria (e.g., ON, USING).

Analyze the Execution Plan: Use the DBMS’s EXPLAIN feature to inspect the execution plan of the query. This will help us understand how the database engine is interpreting and executing the query.

Identify the Cross Product: If a cross product is identified, it typically appears as a CROSS JOIN in the execution plan with a high cost. Identify which tables are involved and why the join condition might have been omitted.

Add or Modify Join Conditions: Based on domain knowledge and schema understanding, add appropriate join conditions to ensure each row from one table is matched only with relevant rows from another table. This will drastically reduce the number of rows processed and improve performance.

Indexing: Ensure that indexes are available on columns used in the join conditions. Indexes can significantly speed up join operations by reducing the amount of data that needs to be scanned.

Query Optimization Techniques: If adding join conditions isn’t sufficient, consider other optimizations such as rewriting subqueries, using common table expressions (CTEs), or partitioning large tables.

Testing and Validation: After making changes, run the query again with EXPLAIN and verify the new execution plan. Ensure that the performance has improved without introducing any logical errors.

#### Closing Summary

By systematically identifying and addressing missing join conditions, we can prevent cross products from occurring, leading to more efficient and faster query execution. It’s crucial to have a good understanding of SQL and your database schema to ensure that optimizations are both effective and maintainable.

#### Common Mistakes/Follow-Ups
Mistake 1: Overlooking the importance of join conditions in query performance.
Mistake 2: Adding unnecessary indexes, which can actually degrade performance if not carefully managed.

#### Follow-Up
Discussing alternative optimization strategies like rewriting queries or using more advanced indexing techniques for specific use cases.

**Code:**

```sql
-- Example SQL Query with Potential Missing Join Condition
SELECT *
FROM customers c
JOIN orders o ON c.customer_id = o.customer_id  -- Ensure this join condition is present
WHERE order_date > '2023-01-01';
```

**Example input / output:**

```text
-- Example EXPLAIN Output with Missing Join Condition
EXPLAIN SELECT *
FROM customers c
JOIN orders o ON true  -- This is a Cartesian join, leading to cross product.
WHERE order_date > '2023-01-01';

-- Example New EXPLAIN Output after Adding Join Condition
EXPLAIN SELECT *
FROM customers c
JOIN orders o ON c.customer_id = o.customer_id  -- Corrected join condition
WHERE order_date > '2023-01-01';
```

**Flow:**

1. Review Query
2. Identify Missing JoinConditions
3. Explain Query Plan
4. Identify Cross Product
5. Add/Modify Join Conditions
6. Indexing Optimization
7. Query Rewriting
8. Test and Validate

