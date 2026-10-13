---
type: "practice"
difficulty: "Deep"
---

# When would you deliberately keep a CTE as an optimization barrier in Postgres using MATERIALIZED, and what problem does that solve?

**Difficulty:** Deep · **Levels:** SDE-1, SDE-1/2 · [↑ Recursive CTEs and Materialization](<../../../../01_Databases_and_SQL/SQL_Language/CTEs_and_Recursive_Queries/Recursive_CTEs_and_Materialization.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To understand when to use MATERIALIZED common table expressions (CTEs) in PostgreSQL, we need to first grasp their purpose and limitations. A CTE is a temporary result set that you can reference within your SQL query. However, by default, the results of a non-materialized CTE are discarded after they are used once. In contrast, MATERIALIZED CTEs store the results in a temporary table, allowing repeated access without re-executing the query.

The key scenario where using MATERIALIZED as an optimization barrier arises is when you have complex queries with recursive elements that need to be computed and reused multiple times. By materializing such CTEs, you can avoid redundant computations, significantly improving performance in scenarios like hierarchical data processing or aggregations over large datasets.

Let's break down the problem this solves step by step:

Complex Recursive Queries: Consider a scenario where we are dealing with hierarchical data, such as an organizational structure. A query might need to repeatedly traverse up and down the hierarchy to aggregate information or find relationships.
Redundant Computations: Without materialization, every time you reference the CTE in your query, it will recompute its results from scratch. This can be extremely inefficient for large datasets or complex logic.
Optimization Barrier: By using MATERIALIZED, we create a temporary table that stores the result set of the CTE. This means subsequent queries referencing this CTE do not need to re-compute, drastically reducing computation time.

#### Triaging the Question

The question is probing our understanding of when and how to use MATERIALIZED CTEs effectively in PostgreSQL. It also wants us to consider scenarios where recursive queries can benefit from materialization as an optimization strategy.

#### Assumptions
We are working with a PostgreSQL database.
The query involves complex logic, potentially recursive operations, or repeated computations on large datasets.
Performance is a critical factor in our application.

#### First-Principles Solution
Scenario 1: Hierarchical Data Aggregation

Suppose we have an employees table with a hierarchical structure represented by the manager_id column:

SQL
CREATE TABLE employees (
    employee_id INT PRIMARY KEY,
    name VARCHAR(255),
    manager_id INT REFERENCES employees(employee_id)
);

We want to compute the total salary of each department, where a department is defined as a group of employees under a single manager.

A naive approach using a non-materialized CTE might look like this:

SQL
WITH RECURSIVE dept_salaries AS (
    SELECT
        employee_id,
        manager_id,
        SUM(salary) OVER (PARTITION BY manager_id) AS total_salary
    FROM
        employees
)
SELECT
    manager_id,
    MAX(employee_id) AS department_head,
    SUM(total_salary) AS total_department_salary
FROM
    dept_salaries
GROUP BY
    manager_id;

This query computes the total salary for each employee and then groups by their manager to aggregate the departments. However, if we have many levels of hierarchy or a large number of employees, this can be inefficient.

Using MATERIALIZED, we can optimize this:

SQL
WITH RECURSIVE dept_salaries AS (
    SELECT
        employee_id,
        manager_id,
        SUM(salary) OVER (PARTITION BY manager_id) AS total_salary
    FROM
        employees
),
materialized_dept_salaries AS (
    MATERIALIZED dept_salaries
)
SELECT
    manager_id,
    MAX(employee_id) AS department_head,
    SUM(total_salary) AS total_department_salary
FROM
    materialized_dept_salaries
GROUP BY
    manager_id;

Here, the dept_salaries CTE is stored in a temporary table and can be reused without recomputing.

Scenario 2: Large-Scale Aggregation Queries

#### Cons
ider a scenario where we need to compute various aggregations over a large dataset. For example, calculating sales per product category with multiple filters:

SQL
WITH RECURSIVE categories AS (
    SELECT
        product_id,
        category_name
    FROM
        products
),
sales_per_category AS (
    SELECT
        c.category_name,
        SUM(s.quantity) AS total_quantity
    FROM
        orders o
    JOIN
        products p ON o.product_id = p.product_id
    JOIN
        categories c ON p.category_id = c.category_id
    GROUP BY
        c.category_name
),
materialized_sales AS (
    MATERIALIZED sales_per_category
)
SELECT * FROM materialized_sales WHERE category_name IN ('Electronics', 'Appliances');

In this case, the sales_per_category CTE is computed once and stored in a temporary table. The subsequent query referencing materialized_sales avoids re-computing the aggregation.

#### Closing Summary

Using MATERIALIZED as an optimization barrier for complex recursive queries or large-scale aggregations can significantly improve performance by storing intermediate results. It's particularly useful when you need to reference and reuse these results multiple times without recomputing them, thus avoiding redundant computations and enhancing overall efficiency.

#### Common Mistakes/Follow-Ups
Overuse: Materializing every CTE unnecessarily increases the risk of memory overhead.
Data Freshness: Ensure that materialized data is up-to-date with the underlying tables.
Maintenance Overhead: Regularly clean up old temporary tables to avoid clutter and potential performance issues.

**Comparison:**

| employees | Structure |
| --- | --- |
| employee_id | INT (Primary Key) |
| name | VARCHAR(255) |
| manager_id | INT (Foreign Key referencing itself) |
| employees | Data Sample |
| --- | --- |
| 1 | Alice | 3 |
| 2 | Bob | 3 |
| 3 | Charlie | NULL |
| 4 | David | 1 |

**Code:**

```sql
WITH RECURSIVE dept_salaries AS (
    SELECT
        employee_id,
        manager_id,
        SUM(salary) OVER (PARTITION BY manager_id) AS total_salary
    FROM
        employees
),
materialized_dept_salaries AS (
    MATERIALIZED dept_salaries
)
SELECT
    manager_id,
    MAX(employee_id) AS department_head,
    SUM(total_salary) AS total_department_salary
FROM
    materialized_dept_salaries
GROUP BY
    manager_id;
```

**Example input / output:**

```text
WITH RECURSIVE dept_salaries AS (
    SELECT
        employee_id,
        manager_id,
        SUM(salary) OVER (PARTITION BY manager_id) AS total_salary
    FROM
        employees
),
materialized_dept_salaries AS (
    MATERIALIZED dept_salaries
)
SELECT
    manager_id,
    MAX(employee_id) AS department_head,
    SUM(total_salary) AS total_department_salary
FROM
    materialized_dept_salaries
GROUP BY
    manager_id;

-- Output:
+-----------+--------------+----------------+
| manager_id | department_head | total_department_salary |
+-----------+--------------+----------------+
|         1 |             4 |                 0     |
|         3 |             2 |                 0     |
|        NULL |              3 |                 0     |
+-----------+--------------+----------------+
```

**Data:**

```sql
CREATE TABLE employees (
    employee_id INT PRIMARY KEY,
    name VARCHAR(255),
    manager_id INT REFERENCES employees(employee_id)
);

INSERT INTO employees (employee_id, name, manager_id) VALUES
(1, 'Alice', 3),
(2, 'Bob', 3),
(3, 'Charlie', NULL),
(4, 'David', 1);
```

**Flow:**

1. Yes
2. No
3. Start
4. Is query recursive?
5. MATERIALIZE CTE
6. Use non-materialized CTE
7. Compute results once, store intemp table
8. Reuse stored results forsubsequent queries
9. Recompute results for eachquery execution

