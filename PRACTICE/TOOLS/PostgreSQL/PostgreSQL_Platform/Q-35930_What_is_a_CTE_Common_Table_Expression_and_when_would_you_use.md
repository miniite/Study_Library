---
type: "practice"
difficulty: "Intermediate"
---

# What is a CTE (Common Table Expression) and when would you use WITH?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Materialized CTEs](<../../../../TOOLS/PostgreSQL/PostgreSQL_Platform/Materialized_CTEs.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

A Common Table Expression (CTE) in PostgreSQL is essentially a temporary result set that you can reference within a SELECT, INSERT, UPDATE, or DELETE statement. CTEs are particularly useful when you need to break down complex queries into simpler, more manageable parts, improving readability and maintainability of your SQL code.

#### Triage

The question asks for an explanation of what a CTE is and its use cases in PostgreSQL. The interviewer might be probing your understanding of query optimization, readability, and the structure of advanced SQL constructs. Assumptions include knowledge of basic SQL queries and awareness of how PostgreSQL handles different types of queries.

#### First-Principles Solution

Let's break down what a CTE does and when to use it:

Definition: A CTE is defined using the WITH clause in your query, allowing you to give a temporary name to a set of rows returned by a SELECT statement. This result can then be used as a table in the main SQL statement.

Syntax:

SQL
WITH cte_name (column1, column2) AS (
    SELECT ... FROM ...
)
-- Main query using CTE

Use Cases:

Break Down Complex Queries: When a single query becomes too complex and can be split into multiple steps.
Improve Readability: Helps in organizing the logic of your queries, making them easier to understand and maintain.
Avoid Subqueries: CTEs often provide an alternative to subqueries that are harder to read or perform poorly.
Recursive Queries: PostgreSQL supports recursive CTEs for hierarchical data.

#### Example
Suppose you want to find all employees who have been promoted at least twice and their managers. You can break this into two steps: first, identify the promotions, then use that result to find the managers.

SQL
WITH Promotions AS (
    SELECT employee_id, manager_id, promotion_date
    FROM promotions_table
    WHERE promotion_type = 'salary'
),
TwicePromoted AS (
    SELECT employee_id
    FROM Promotions
    GROUP BY employee_id
    HAVING COUNT(*) >= 2
)
SELECT e.employee_name, p.manager_id
FROM TwicePromoted t
JOIN employees_table e ON t.employee_id = e.id;

#### Closing Summary
and Common Mistakes/Follow-Ups

CTEs are a powerful tool in PostgreSQL for writing cleaner and more maintainable SQL queries. They can significantly improve the performance of complex operations by breaking them down into simpler parts. However, it's important to ensure that CTEs do not add unnecessary complexity or reduce query execution efficiency.

#### Common mistakes
include overusing CTEs where simple subqueries would suffice, or using them without considering the potential impact on performance. During follow-ups, the interviewer might ask about specific scenarios where you have used CTEs and how it improved your queries.

**Code:**

```sql
WITH Promotions AS (
    SELECT employee_id, manager_id, promotion_date
    FROM promotions_table
    WHERE promotion_type = 'salary'
),
TwicePromoted AS (
    SELECT employee_id
    FROM Promotions
    GROUP BY employee_id
    HAVING COUNT(*) >= 2
)
SELECT e.employee_name, p.manager_id
FROM TwicePromoted t
JOIN employees_table e ON t.employee_id = e.id;
```

