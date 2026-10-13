---
type: "practice"
difficulty: "Basic"
---

# Write a query to find the average salary of employees in each department.

**Difficulty:** Basic · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ Aggregate Precision and Types](<../../../../01_Databases_and_SQL/SQL_Language/Aggregation/Aggregate_Precision_and_Types.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To solve this problem, we need to write a SQL query that calculates the average salary of employees in each department. The key is to use an aggregate function AVG() along with the GROUP BY clause to group by departments and then calculate the average salary for each group.

#### Triaging the Question

The question asks us to find the average salary per department, which means we are dealing with a grouping operation followed by an aggregation. We will need to ensure that our query correctly identifies distinct departments and computes the average salary for employees within those departments.

#### First-Principles Solution

We assume there is a table named employees with columns like id, name, salary, and department_id. The basic structure of the SQL query would be:

SQL
SELECT department_id, AVG(salary) AS average_salary
FROM employees
GROUP BY department_id;

#### Closing Summary
+ Common Mistakes/Follow-Ups

The above query groups the rows by department_id and calculates the average salary for each group. However, there are a few common pitfalls:

Ensure that the table name and column names match those in your database schema.
Check if the salary values are numeric; non-numeric entries might cause errors or incorrect results.

Common follow-ups could include:

How would you handle departments with no employees?
What if there is a need to also list departments where no average salary can be calculated?

**Code:**

```sql
SELECT department_id, AVG(salary) AS average_salary
FROM employees
GROUP BY department_id;
```

