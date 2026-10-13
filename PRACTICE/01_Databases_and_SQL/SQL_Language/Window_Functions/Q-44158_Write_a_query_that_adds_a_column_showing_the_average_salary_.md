---
type: "practice"
difficulty: "Basic"
---

# Write a query that adds a column showing the average salary across the whole table next to every employee row.

**Difficulty:** Basic · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Top-N per Group](<../../../../01_Databases_and_SQL/SQL_Language/Window_Functions/Top-N_per_Group.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To solve this problem, we need to add an additional column to our result set that shows the average salary of all employees in the database. This can be achieved using a window function in SQL, which allows us to perform calculations across a set of rows related to the current row.

#### Triage

The question is asking for a query to calculate and display the average salary for each employee while keeping their original data intact. The interviewer might be testing your understanding of window functions, particularly AVG(), and how to use them in combination with other SQL clauses.

We assume that we have an employees table with columns such as employee_id (a unique identifier), name, and salary. Our goal is to modify the query to include a new column showing the average salary across all rows.

#### First-Principles Solution

To achieve this, we will use the window function AVG() in combination with the OVER clause. The OVER clause allows us to define the window of rows over which the AVG() function operates. Here's how you can write the query:

SQL
SELECT
    employee_id,
    name,
    salary,
    AVG(salary) OVER () AS avg_salary
FROM
    employees;

#### Closing Summary
and Common Mistakes/Follow-Ups

This query will return a result set where each row includes the original employee_id, name, and salary columns, plus an additional column named avg_salary which shows the average salary across all rows in the table.

#### Common Mistakes

Forgetting to include the AVG(salary) OVER () AS avg_salary part.
Using a window function incorrectly or not understanding its syntax.

#### Follow-Ups

Discuss scenarios where this approach might not be efficient if you have a very large dataset.
Explore other window functions that could be used in similar contexts, such as RANK() for ranking employees by salary.

**Code:**

```sql
SELECT
    employee_id,
    name,
    salary,
    AVG(salary) OVER () AS avg_salary
FROM
    employees;
```

