---
type: "practice"
difficulty: "Intermediate"
---

# Write a query that flags each row as 'above average' or 'below average' compared to the overall average salary.

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Top-N per Group](<../../../../01_Databases_and_SQL/SQL_Language/Window_Functions/Top-N_per_Group.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address this problem, we need to determine whether each employee's salary is above or below the average salary across all employees. We'll use SQL window functions to calculate the average salary and then compare each row against it.

#### Triaging the Question

The core of the question involves calculating the overall average salary and using a comparison with that value to classify each row as either 'above average' or 'below average'. The key is understanding how window functions work in SQL. Specifically, we'll use the AVG() function combined with OVER() clause to achieve this.

#### Assumptions
We assume the data is stored in a table named employees, which has columns employee_id and salary.
The goal is to add a new column called salary_status that will contain either 'above average' or 'below average'.

#### First Principles Solution
Calculate Overall Average Salary: We need to find the average salary across all rows.
Use Window Functions: Use SQL window functions to compare each row's salary with the overall average.
SQL
SELECT
    employee_id,
    salary,
    CASE
        WHEN salary > AVG(salary) OVER () THEN 'above average'
        ELSE 'below average'
    END AS salary_status
FROM employees;

#### Closing Summary
and Common Mistakes/Follow-Ups:

This query effectively uses the AVG() window function without partitioning to calculate the overall average salary. The CASE statement then classifies each row based on its salary relative to this average.

#### Common Mistakes

Forgetting to use the OVER () clause, which is necessary for calculating a windowed aggregate.
Incorrectly using a subquery or temporary table instead of a window function, which would complicate the query and performance.
Misunderstanding that AVG() over an unpartitioned result set calculates the overall average.

#### Follow-Ups

How might this change if we wanted to compare each row against its department's average salary?
What happens if there are multiple rows with exactly the same average salary value?

**Code:**

```sql
SELECT
    employee_id,
    salary,
    CASE
        WHEN salary > AVG(salary) OVER () THEN 'above average'
        ELSE 'below average'
    END AS salary_status
FROM employees;
```

**Example input / output:**

```text
+-------------+--------+----------------+
| employee_id | salary | salary_status  |
+-------------+--------+----------------+
| 1           | 5000   | below average  |
| 2           | 6000   | above average  |
| 3           | 4500   | below average  |
| 4           | 7000   | above average  |
+-------------+--------+----------------+
```

