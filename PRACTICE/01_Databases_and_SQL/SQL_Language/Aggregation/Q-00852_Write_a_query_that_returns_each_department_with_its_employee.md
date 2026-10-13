---
type: "practice"
difficulty: "Intermediate"
---

# Write a query that returns each department with its employee count and average salary, sorted by count descending.

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ Counting with Joins](<../../../../01_Databases_and_SQL/SQL_Language/Aggregation/Counting_with_Joins.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To solve this problem, we need to write a SQL query that aggregates data across departments, calculating both the count of employees in each department and their average salary. The results will be ordered based on the employee count in descending order.

#### Triaging the Question

The question is asking for an SQL query that:

Groups by the department column.
Calculates two aggregate functions:
Count of employees (COUNT(*))
Average salary (AVG(salary))
Orders the result set by the count of employees in descending order.

#### Assumptions
The table name is employees.
The columns are:
employee_id (unique identifier for each employee)
name (name of the employee, not needed for aggregation)
department (name or ID of the department where the employee works)
salary (monthly salary of the employee)

#### First-Principles Solution

We will use SQL to achieve this by:

Selecting the necessary columns.
Using a GROUP BY clause to group results by each unique department.
Applying aggregate functions COUNT(*) and AVG(salary).
Ordering the result set using ORDER BY COUNT(*) DESC.
Full Solution
SQL
SELECT
    department,
    COUNT(*) AS employee_count,
    AVG(salary) AS average_salary
FROM
    employees
GROUP BY
    department
ORDER BY
    employee_count DESC;

#### Closing Summary
and Common Mistakes/Follow-Ups

This query effectively groups the employees table by each department, calculating both the number of employees (employee_count) and their average salary. The results are sorted in descending order based on the count of employees.

#### Common mistakes
to avoid:

Forgetting to use GROUP BY or incorrectly applying it.
Not selecting all required columns for display in the final result set.
Misordering the columns in the SELECT clause, which can lead to incorrect results when using aggregate functions.
Missing the ORDER BY clause or ordering by a different column.

#### Follow-up
questions might include:

How would you handle departments with no employees?
What if there are ties for the same employee count? Would the average salary be meaningful then?

**Code:**

```sql
SELECT
    department,
    COUNT(*) AS employee_count,
    AVG(salary) AS average_salary
FROM
    employees
GROUP BY
    department
ORDER BY
    employee_count DESC;
```

