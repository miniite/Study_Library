---
type: "practice"
difficulty: "Basic"
---

# What is the difference between a subquery and a CTE, and how would a CTE make a multi-step RAG-analytics query more readable?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Recursive CTEs](<../../../../01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Recursive_CTEs.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In SQL, both subqueries and Common Table Expressions (CTEs) are used to break down complex queries into smaller, more manageable parts. However, they serve slightly different purposes and have distinct characteristics that affect their readability and performance.

TL;DR:
Subquery: A subquery is a query nested within another query, often used for filtering or joining data. It can be placed inside the WHERE, FROM, or JOIN clauses.
CTE: CTEs are temporary result sets that you can reference within a SELECT, INSERT, UPDATE, DELETE, or CREATE TABLE statement. They provide an alternative to subqueries and make queries more readable by breaking them into logical steps.

#### Triaging the Question

The question asks for the differences between subqueries and CTEs and how using CTEs can improve multi-step RAG (Retrieve-Analyze-Generate) analytics queries. The interviewer is probing your understanding of SQL query constructs, their syntax, and their impact on code readability and maintainability.

#### Assumptions
We are working with a relational database system.
The context involves complex queries for data analysis tasks.

#### First-Principles Solution
Subqueries

Subqueries are nested within the main query and can be used in various clauses like WHERE, FROM, or JOIN. For example, consider a query to find employees who earn more than the average salary of their department:

SQL
SELECT employee_id, name, salary
FROM employees
WHERE salary > (SELECT AVG(salary) FROM employees WHERE department = employees.department);

Here, the subquery (SELECT AVG(salary) FROM employees WHERE department = employees.department) is used to find the average salary for each department and compare it with an individual's salary.

Common Table Expressions (CTEs)

CTEs are defined using the WITH clause before the main query. They provide a temporary result set that can be referenced multiple times within the same query. Here’s how you might rewrite the previous example using CTEs:

SQL
WITH avg_salaries AS (
    SELECT department, AVG(salary) as average_salary
    FROM employees
    GROUP BY department
)
SELECT e.employee_id, e.name, e.salary
FROM employees e
JOIN avg_salaries a ON e.department = a.department
WHERE e.salary > a.average_salary;

In this example, avg_salaries is the CTE that calculates the average salary for each department. The main query then joins the employees table with avg_salaries to filter employees earning more than their department's average.

Readability and Multi-Step Queries

CTEs significantly enhance readability by breaking down complex queries into simpler, more understandable steps:

Define Intermediate Results: CTEs allow defining intermediate results that can be easily understood.
Logical Flow: The sequence of steps in a query using CTEs is clearer and easier to follow.
Avoid Repetition: CTEs help avoid repeating the same subquery logic multiple times.

#### Example
Multi-Step RAG Analytics Query

Let's consider an example where we want to identify departments that have employees earning above average salaries, generate a report with department names and the total number of such employees, and then calculate the overall percentage:

SQL
WITH avg_salaries AS (
    SELECT department, COUNT(*) as num_employees, AVG(salary) as average_salary
    FROM employees
    GROUP BY department
),
high_salary_departments AS (
    SELECT department
    FROM avg_salaries
    WHERE average_salary > (SELECT AVG(salary) FROM employees)
),
employee_count AS (
    SELECT high_salary_department, COUNT(*) as count
    FROM employees e
    JOIN high_salary_departments hsd ON e.department = hsd.department
    GROUP BY high_salary_department
)
SELECT department, count * 100.0 / (SELECT SUM(count) FROM employee_count) as percentage
FROM employee_count;

This query uses three CTEs:

avg_salaries to calculate the average salary and number of employees per department.
high_salary_departments to filter departments with above-average salaries.
employee_count to count employees in these high-salary departments.

#### Closing Summary
+ Common Mistakes/Follow-Ups

Using CTEs can make complex queries more readable and maintainable. However, it’s important to note:

Performance: Although CTEs are generally easier to read, they might not always perform better than subqueries.
Scoping: Ensure that the scope of CTEs is clear to avoid confusion with outer queries.

For follow-ups, you could be asked about performance implications, trade-offs between CTEs and subqueries, or real-world scenarios where one might be preferred over the other.

**Comparison:**

| Employees Table          |
|--------------------------|
| employee_id (int), name (string), salary (float), department (string) |
| Departments with Avg Salaries |
|--------------------------------|
| department (string), num_employees (int), average_salary (float) |
| High Salary Departments      |
|------------------------------|
| department (string)           |
| Employee Count by Department |
|------------------------------|
| high_salary_department (string), count (int) |

**Code:**

```sql
WITH avg_salaries AS (
    SELECT department, COUNT(*) as num_employees, AVG(salary) as average_salary
    FROM employees
    GROUP BY department
),
high_salary_departments AS (
    SELECT department
    FROM avg_salaries
    WHERE average_salary > (SELECT AVG(salary) FROM employees)
),
employee_count AS (
    SELECT high_salary_department, COUNT(*) as count
    FROM employees e
    JOIN high_salary_departments hsd ON e.department = hsd.department
    GROUP BY high_salary_department
)
SELECT department, count * 100.0 / (SELECT SUM(count) FROM employee_count) as percentage
FROM employee_count;
```

**Example input / output:**

```text
WITH avg_salaries AS (
    SELECT department, COUNT(*) as num_employees, AVG(salary) as average_salary
    FROM employees
    GROUP BY department
),
high_salary_departments AS (
    SELECT department
    FROM avg_salaries
    WHERE average_salary > (SELECT AVG(salary) FROM employees)
),
employee_count AS (
    SELECT high_salary_department, COUNT(*) as count
    FROM employees e
    JOIN high_salary_departments hsd ON e.department = hsd.department
    GROUP BY high_salary_department
)
SELECT department, count * 100.0 / (SELECT SUM(count) FROM employee_count) as percentage
FROM employee_count;

-- Output:
| department | percentage |
|------------|-----------|
| Engineering | 50%       |
```

**Data:**

```json
{
  "employees": [
    {"employee_id": 1, "name": "Alice", "salary": 55000, "department": "Sales"},
    {"employee_id": 2, "name": "Bob", "salary": 60000, "department": "HR"},
    {"employee_id": 3, "name": "Charlie", "salary": 45000, "department": "Marketing"},
    {"employee_id": 4, "name": "David", "salary": 70000, "department": "Sales"},
    {"employee_id": 5, "name": "Eve", "salary": 80000, "department": "Engineering"}
  ]
}
```

**Flow:**

1. Define CTE for AverageSalaries
2. Filter Departments with HighSalaries
3. Count Employees inHigh-Salary Departments
4. Calculate Percentage ofHigh-Salary Departments
5. Return Final Result

