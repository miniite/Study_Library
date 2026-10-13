---
type: "practice"
difficulty: "Intermediate"
---

# What is the difference between a scalar subquery and a table subquery, with an example of each?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ PostgreSQL Query Basics](<../../../../TOOLS/PostgreSQL/PostgreSQL_Platform/PostgreSQL_Query_Basics.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In PostgreSQL, both scalar subqueries and table subqueries serve as powerful tools to incorporate data from one query into another, but they operate in fundamentally different ways. A scalar subquery returns a single value (like an integer or string), while a table subquery can return multiple rows and columns, effectively acting like a temporary result set.

Firstly, let's look at the syntax for each type of subquery:

Scalar Subquery: Returns one column with one row.
Table Subquery: Can return multiple rows and columns.

Now, we'll dive into examples to illustrate these differences.

**Code:**

```sql
-- Example 1: Scalar Subquery

CREATE TABLE employees (
    id SERIAL PRIMARY KEY,
    name TEXT NOT NULL,
    department_id INTEGER REFERENCES departments(id)
);

CREATE TABLE departments (
    id SERIAL PRIMARY KEY,
    dept_name TEXT NOT NULL
);

In this example, we have two tables: employees and departments. The goal is to find the number of employees in each department.

To use a scalar subquery:

SQL
Copy
SELECT
    dept_id,
    (SELECT COUNT(*) FROM employees WHERE department_id = d.id) AS num_employees
FROM departments d;

This query counts the number of employees for each department by using a scalar subquery that returns a single count value.

SQL
Copy
-- Example 2: Table Subquery

CREATE TABLE sales (
    id SERIAL PRIMARY KEY,
    sale_date DATE NOT NULL,
    amount INTEGER NOT NULL,
    employee_id INTEGER REFERENCES employees(id)
);

Here, we have an additional table sales that records sales transactions. The goal is to find the total sales for each employee.

To use a table subquery:

SQL
Copy
SELECT
    e.id,
    e.name,
    (SELECT SUM(s.amount) FROM sales s WHERE s.employee_id = e.id) AS total_sales
FROM employees e;

This query sums up the total sales for each employee by using a table subquery that returns multiple rows and columns.
```

**Example input / output:**

```text
-- Input 1 (Scalar Subquery)
SELECT
    dept_id,
    (SELECT COUNT(*) FROM employees WHERE department_id = d.id) AS num_employees
FROM departments d;

-- Output 1:
+---------+----------------+
| dept_id | num_employees  |
+---------+----------------+
|       1 |               2 |
|       2 |               3 |
+---------+----------------+

-- Input 2 (Table Subquery)
SELECT
    e.id,
    e.name,
    (SELECT SUM(s.amount) FROM sales s WHERE s.employee_id = e.id) AS total_sales
FROM employees e;

-- Output 2:
+----+-------+------------+
| id | name  | total_sales|
+----+-------+------------+
|  1 | John  |          50 |
|  2 | Jane  |          75 |
|  3 | Mike  |         100 |
+----+-------+------------+
```

**Flow:**

1. Yes
2. No
3. Yes
4. No
5. Scalar Subquery
6. Returns one row with onecolumn?
7. COUNT(*)
8. Table Subquery
9. Returns multiple rows andcolumns?
10. SUM(amount)
11. G

