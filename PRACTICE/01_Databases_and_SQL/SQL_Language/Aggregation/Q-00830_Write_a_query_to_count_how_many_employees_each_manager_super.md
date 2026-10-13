---
type: "practice"
difficulty: "Intermediate"
---

# Write a query to count how many employees each manager supervises.

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ Counting with Joins](<../../../../01_Databases_and_SQL/SQL_Language/Aggregation/Counting_with_Joins.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To solve this problem, we need to write a SQL query that counts the number of employees supervised by each manager. The key concept here is using an aggregate function, specifically COUNT, along with a GROUP BY clause to group results by the manager's ID or name.

#### Triaging the Question

The question is asking us to:

Identify and group records based on the manager.
Count the number of employees in each group.

We assume that there are tables containing both managers and their respective employees, with a relationship defined (e.g., manager_id foreign key in the employee table).

#### First-Principles Solution

Let's break it down step-by-step:

Identify the Tables: Assume we have two tables: employees and managers.
Join the Tables: We need to join these tables based on the relationship.
Group by Manager ID: Group records by each manager’s ID.
Use COUNT Function: Apply the COUNT function to count employees for each group.
SQL Query
SQL
SELECT
    m.manager_id,
    m.manager_name,
    COUNT(e.employee_id) AS employee_count
FROM
    managers m
JOIN
    employees e ON m.manager_id = e.manager_id
GROUP BY
    m.manager_id, m.manager_name;

This query will return a result set where each row corresponds to a manager and includes the count of their supervised employees.

#### Closing Summary
+ Common Mistakes/Follow-Ups
Summary: The solution involves joining two tables based on the manager relationship and using COUNT with GROUP BY to aggregate employee counts per manager.

#### Common Mistakes
Forgetting to join the correct tables or using incorrect foreign keys.
Not including the GROUP BY clause, leading to a query that doesn't properly aggregate data.
Using COUNT(*) instead of COUNT(e.employee_id) which might include NULL values in the count.
Code Block
SQL
-- Correct SQL Query
SELECT
    m.manager_id,
    m.manager_name,
    COUNT(e.employee_id) AS employee_count
FROM
    managers m
JOIN
    employees e ON m.manager_id = e.manager_id
GROUP BY
    m.manager_id, m.manager_name;

**Code:**

```sql
-- Correct SQL Query
SELECT
    m.manager_id,
    m.manager_name,
    COUNT(e.employee_id) AS employee_count
FROM
    managers m
JOIN
    employees e ON m.manager_id = e.manager_id
GROUP BY
    m.manager_id, m.manager_name;
```

