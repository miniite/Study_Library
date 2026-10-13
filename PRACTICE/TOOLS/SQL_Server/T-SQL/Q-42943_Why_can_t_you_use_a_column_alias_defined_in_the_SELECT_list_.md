---
type: "practice"
difficulty: "Deep"
---

# Why can't you use a column alias defined in the SELECT list inside the WHERE clause?

**Difficulty:** Deep · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ T-SQL Basics](<../../../../TOOLS/SQL_Server/T-SQL/T-SQL_Basics.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When working with T-SQL, it's important to understand that the SELECT and WHERE clauses are processed at different stages of query execution. Specifically, columns defined in the SELECT list are calculated after the rows have been filtered by the conditions specified in the WHERE clause. This means that a column alias defined in the SELECT list cannot be used directly within the WHERE clause because the data hasn't been processed at that stage.

#### Triage

This question is probing your understanding of how SQL queries are executed and why certain constructs behave as they do. We need to consider the query execution plan, which processes different clauses in a specific order: FROM, ON, JOIN, WHERE, GROUP BY, HAVING, SELECT, and finally ORDER BY. The key insight here is understanding the dependencies between these clauses.

Step-by-Step Solution
Query Execution Order: SQL queries are processed in a predefined order, which impacts how expressions can be used within different parts of the query.
WHERE Clause Processing: The WHERE clause filters rows based on conditions specified before any column calculations occur.
SELECT List Processing: Columns defined in the SELECT list get calculated after the rows are filtered by the WHERE clause.

#### Example

#### Cons
ider the following T-SQL code:

SQL
SELECT a + b AS sum, sum > 10 FROM table WHERE sum > 5;

In this example, SQL will not recognize sum in the WHERE clause because it hasn't been calculated yet. The execution plan would first filter rows based on the condition table WHERE sum > 5, but by that time, sum is still an uncomputed alias.

Full Solution

To solve such a problem, we can use common table expressions (CTEs) or subqueries to calculate the column alias before using it in the WHERE clause. Here’s how you can do it:

SQL
WITH cte AS (
    SELECT a + b AS sum FROM table
)
SELECT * FROM cte WHERE sum > 5;

Alternatively, you could use a subquery:

SQL
SELECT * FROM (SELECT a + b AS sum FROM table) t WHERE sum > 5;

#### Closing Summary
and Common Mistakes/Follow-Ups

The key takeaway is that the WHERE clause filters rows based on conditions before any columns in the SELECT list are calculated. To use such calculations within the WHERE clause, you need to ensure they are computed first through methods like CTEs or subqueries.

#### Common mistakes
include assuming that column aliases defined in the SELECT list can be used directly in the WHERE clause without proper processing.

Self-Check

This solution covers the core concept and provides a practical example. It aligns with best practices by ensuring correct query execution order.

**Code:**

```sql
-- Example using CTE
WITH cte AS (
    SELECT a + b AS sum FROM table
)
SELECT * FROM cte WHERE sum > 5;

-- Example using subquery
SELECT * FROM (SELECT a + b AS sum FROM table) t WHERE sum > 5;
```

