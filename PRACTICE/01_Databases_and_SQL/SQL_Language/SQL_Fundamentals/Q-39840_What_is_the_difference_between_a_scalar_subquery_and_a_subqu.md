---
type: "practice"
difficulty: "Intermediate"
---

# What is the difference between a scalar subquery and a subquery that returns multiple rows?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ Scalar Subqueries](<../../../../01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Scalar_Subqueries.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In SQL, both scalar subqueries and multi-row subqueries are used to fetch data from a database, but they operate differently based on the expected result set. A scalar subquery is designed to return a single value, while a subquery that returns multiple rows can be used in various contexts like IN, ANY/SOME, ALL operators.

#### Triaging the Question

The question is essentially asking you to explain the difference between two types of subqueries based on their expected result sets and usage. The interviewer might be probing your understanding of SQL query semantics and how different contexts affect the operations performed by these subqueries.

Key Assumptions:
You understand what a subquery is.
You are familiar with scalar subqueries and multi-row subqueries in SQL.
You can explain the differences in terms of expected result sets and use cases.

#### First-Principles Solution

A scalar subquery returns exactly one row and one column, which means it can return a single value. This is useful when you need to compare or filter data based on a specific condition derived from another query. On the other hand, a multi-row subquery can return multiple rows (and thus multiple columns), making it suitable for scenarios where you need to perform operations like IN, ANY/SOME, or ALL.

Scalar Subquery Example:
SQL
SELECT * FROM employees WHERE salary > (SELECT AVG(salary) FROM employees);

In this example, the scalar subquery (SELECT AVG(salary) FROM employees) returns a single value that is used to filter the employees table. The result of the scalar subquery must be a single column with one or zero rows.

Multi-Row Subquery Example:
SQL
SELECT * FROM orders WHERE order_date IN (SELECT order_date FROM sales WHERE region = 'North');

Here, the multi-row subquery (SELECT order_date FROM sales WHERE region = 'North') returns multiple rows and columns. The result is used in an IN clause to filter the orders table.

#### Closing Summary
+ Common Mistakes/Follow-Ups

#### Common mistakes
include assuming that any subquery can return a single value, which leads to incorrect use cases or misunderstandings about query semantics. Another common issue is not recognizing the implications of using multi-row subqueries in contexts like IN, ANY/SOME, and ALL operators.

#### Follow-ups
might involve discussing performance considerations (e.g., when to prefer one over the other), or exploring more complex scenarios where both types of subqueries are used together.

**Code:**

```sql
-- Scalar Subquery Example
SELECT * FROM employees WHERE salary > (SELECT AVG(salary) FROM employees);

-- Multi-Row Subquery Example
SELECT * FROM orders WHERE order_date IN (SELECT order_date FROM sales WHERE region = 'North');
```

