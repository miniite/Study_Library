---
type: "practice"
difficulty: "Intermediate"
---

# Write a query using a subquery to find employees who earn more than the average salary.

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ Subquery Practice Problems](<../../../../01_Databases_and_SQL/SQL_Language/Joins_and_Subqueries/Subquery_Practice_Problems.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To solve this problem, you need to write a SQL query that finds all employees earning more than the average salary in their department. The key concept here is using a subquery to calculate the average salary and then joining it back to the main table to filter out the desired results.

Let's break down the steps:

#### Triage
Understanding the Problem: We need to find employees whose salary is greater than the average salary in their respective departments.
What the Interviewer is Probing: The interviewer might be evaluating your understanding of subqueries, joins, and aggregate functions like AVG().

#### Assumptions
You have a table named employees with at least two columns: employee_id and salary. Additionally, you need the department ID (dept_id) to link employees to their respective departments.

#### First-Principles Solution
Calculate Average Salary by Department: Use an aggregate function AVG() inside a subquery to compute the average salary for each department.
Join with Main Table: Join this result back to the main employees table on the department ID.
Filter the Result: Select only those employees whose salary is greater than the computed average.
Full Solution
SQL
SELECT e.*
FROM employees e
JOIN (
    SELECT dept_id, AVG(salary) as avg_salary
    FROM employees
    GROUP BY dept_id
) es ON e.dept_id = es.dept_id
WHERE e.salary > es.avg_salary;

#### Closing Summary
+ Common Mistakes/Follow-Ups
Summary: The solution involves using a subquery to calculate the average salary per department and then joining this result back with the main employees table to filter out those employees earning more than the average.

#### Common Mistakes
Forgetting to use an aggregate function like AVG() inside the subquery.
Not properly linking the subquery results back to the main table.
Misunderstanding how to correctly join and filter the data.
Dry-Run

Let's assume we have a sample dataset:

SQL
CREATE TABLE employees (
    employee_id INT,
    name VARCHAR(50),
    salary DECIMAL(10, 2),
    dept_id INT
);

INSERT INTO employees (employee_id, name, salary, dept_id) VALUES
(1, 'Alice', 5000.00, 101),
(2, 'Bob', 6000.00, 101),
(3, 'Charlie', 4800.00, 102),
(4, 'David', 6200.00, 102);

Running the query:

SQL
SELECT e.*
FROM employees e
JOIN (
    SELECT dept_id, AVG(salary) as avg_salary
    FROM employees
    GROUP BY dept_id
) es ON e.dept_id = es.dept_id
WHERE e.salary > es.avg_salary;

Output:

PLAINTEXT
+------------+-------+----------+-----------+
| employee_id | name  | salary   | dept_id   |
+------------+-------+----------+-----------+
| 2           | Bob   | 6000.00  | 101       |
| 4           | David | 6200.00  | 102       |
+------------+-------+----------+-----------+
Confirm Correctness + Completeness

The solution is correct and complete as it addresses the problem statement precisely.

**Comparison:**

| employee_id | name   | salary | dept_id |
|-------------|--------|--------|---------|
| 1           | Alice  | 5000.00 | 101     |
| 2           | Bob    | 6000.00 | 101     |
| 3           | Charlie| 4800.00 | 102     |
| 4           | David  | 6200.00 | 102     |

**Code:**

```sql
SELECT e.*
FROM employees e
JOIN (
    SELECT dept_id, AVG(salary) as avg_salary
    FROM employees
    GROUP BY dept_id
) es ON e.dept_id = es.dept_id
WHERE e.salary > es.avg_salary;
```

**Example input / output:**

```text
Input:
SELECT e.*
FROM employees e
JOIN (
    SELECT dept_id, AVG(salary) as avg_salary
    FROM employees
    GROUP BY dept_id
) es ON e.dept_id = es.dept_id
WHERE e.salary > es.avg_salary;

Output:
+------------+-------+----------+-----------+
| employee_id | name  | salary   | dept_id   |
+------------+-------+----------+-----------+
| 2           | Bob   | 6000.00  | 101       |
| 4           | David | 6200.00  | 102       |
+------------+-------+----------+-----------+
```

**Data:**

```sql
CREATE TABLE employees (
    employee_id INT,
    name VARCHAR(50),
    salary DECIMAL(10, 2),
    dept_id INT
);

INSERT INTO employees (employee_id, name, salary, dept_id) VALUES
(1, 'Alice', 5000.00, 101),
(2, 'Bob', 6000.00, 101),
(3, 'Charlie', 4800.00, 102),
(4, 'David', 6200.00, 102);
```

**Flow:**

1. Employees Table
2. Filter Employees with salary >avg(salary by dept_id)
3. (DeptId, AvgSalary)
4. Join Result
5. Output: Employees earningmore than the average

