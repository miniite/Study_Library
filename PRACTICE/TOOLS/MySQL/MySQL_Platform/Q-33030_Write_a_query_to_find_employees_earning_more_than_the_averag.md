---
type: "practice"
difficulty: "Intermediate"
---

# Write a query to find employees earning more than the average salary of their own department.

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Basic Query Practice](<../../../../TOOLS/MySQL/MySQL_Platform/Basic_Query_Practice.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To solve this problem, we need to write a SQL query that identifies employees who earn more than the average salary in their respective departments. This involves using subqueries and aggregate functions effectively. Let's break it down step by step.

#### Triaging the Question

The interviewer is probing your understanding of:

Subquery Usage: How to use subqueries to calculate departmental averages.
Aggregate Functions: Utilizing AVG() to find average salaries.
Join Operations: Ensuring that employee records are correctly associated with their departments.

#### First-Principles Solution

First, we need a table structure for our employees and departments. Assume the schema is as follows:

SQL
CREATE TABLE Employees (
    id INT PRIMARY KEY,
    name VARCHAR(100),
    salary DECIMAL(10, 2),
    department_id INT
);

CREATE TABLE Departments (
    id INT PRIMARY KEY,
    dept_name VARCHAR(100)
);

To find employees earning more than the average in their respective departments, we can use a subquery to calculate the average salary for each department and then join this with the employee table.

Step-by-Step Solution
1. Calculate Average Salary per Department

We start by calculating the average salary for each department:

SQL
SELECT AVG(salary) as avg_salary, department_id FROM Employees GROUP BY department_id;

This query returns a list of departments and their respective average salaries.

2. Join with Employee Table

Next, we join this result back to the Employees table on department_id to filter out employees earning more than the average:

SQL
SELECT e.*
FROM Employees e
JOIN (
    SELECT AVG(salary) as avg_salary, department_id FROM Employees GROUP BY department_id
) sub ON e.department_id = sub.department_id
WHERE e.salary > sub.avg_salary;

This query returns a list of employees who earn more than the average salary in their respective departments.

#### Closing Summary
+ Common Mistakes/Follow-Ups

#### Closing Summary
The SQL query above correctly identifies employees earning more than the average salary in their department by first calculating the average salary for each department and then filtering the Employees table based on this information. This ensures that only relevant data is returned, making the query efficient.

#### Common Mistakes/Follow-Ups

Incorrect Join Condition: Ensure you use the correct join condition (e.department_id = sub.department_id) to match employees with their departmental average.
Handling Nulls: If there are null salaries or missing departments in your dataset, consider how to handle these cases.

#### Performance Considerations
For large datasets, ensure that indexes on department_id and salary are used to optimize performance.
Code Blocks
SQL
-- Step 1: Calculate average salary per department
SELECT AVG(salary) as avg_salary, department_id FROM Employees GROUP BY department_id;

-- Step 2: Join with Employee table
SELECT e.*
FROM Employees e
JOIN (
    SELECT AVG(salary) as avg_salary, department_id FROM Employees GROUP BY department_id
) sub ON e.department_id = sub.department_id
WHERE e.salary > sub.avg_salary;
Self-Check

Correctness and completeness confirmed.

