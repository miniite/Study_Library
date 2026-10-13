---
type: "practice"
difficulty: "Deep"
---

# How can a subquery in the SELECT list add a computed value like each employee's department average salary?

**Difficulty:** Deep · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ Subquery Practice Problems](<../../../../01_Databases_and_SQL/SQL_Language/Joins_and_Subqueries/Subquery_Practice_Problems.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In this scenario, you need to enhance an SQL query to include a computed column that represents each employee’s department's average salary. This involves using subqueries and joins in a strategic manner. Here's how we can achieve this:

#### Overview
We will use a subquery to calculate the average salary for each department and then join it back with the main employees table to add this value as a new column in the result set.

#### Triage
The question is essentially asking us to compute a derived field (average salary per department) that can be included in the output of an existing query. We need to ensure our solution is both accurate and efficient, considering potential performance impacts.

Let's start by defining the necessary tables:

employees: Contains employee details such as ID, Name, DepartmentID, Salary.
departments: Contains department details such as ID, Name.

#### First-Principles Solution
Identify the Average Salary per Department: We first need to calculate the average salary for each department. This can be done using a subquery or a common table expression (CTE).
SQL
WITH AvgSalaries AS (
    SELECT
        DepartmentID,
        AVG(Salary) AS AvgSalary
    FROM
        employees
    GROUP BY
        DepartmentID
)
Join the Result with Employees Table: Next, we join this result set back to the employees table on the DepartmentID column. This will allow us to include the average salary in the final output.
SQL
SELECT
    e.EmployeeID,
    e.Name,
    e.DepartmentID,
    e.Salary,
    a.AvgSalary
FROM
    employees e
JOIN
    AvgSalaries a ON e.DepartmentID = a.DepartmentID;

#### Closing Summary

This approach ensures that we correctly compute and include the average salary per department for each employee. The use of CTEs can make the query more readable, but subqueries are also valid if preferred.

#### Common Mistakes/Follow-Ups
Mistake: Forgetting to group by DepartmentID in the first subquery.

#### Follow-Up
What would happen if there is no employee in a department? How should null values be handled?

Let's consider an example:

SQL
-- Sample Data
CREATE TABLE employees (
    EmployeeID INT,
    Name VARCHAR(50),
    DepartmentID INT,
    Salary DECIMAL(10,2)
);

INSERT INTO employees (EmployeeID, Name, DepartmentID, Salary) VALUES
(1, 'Alice', 101, 70000.00),
(2, 'Bob', 101, 85000.00),
(3, 'Charlie', 102, 60000.00),
(4, 'David', 102, 90000.00);

CREATE TABLE departments (
    DepartmentID INT,
    Name VARCHAR(50)
);

INSERT INTO departments (DepartmentID, Name) VALUES
(101, 'Engineering'),
(102, 'Sales');

WITH AvgSalaries AS (
    SELECT
        DepartmentID,
        AVG(Salary) AS AvgSalary
    FROM
        employees
    GROUP BY
        DepartmentID
)
SELECT
    e.EmployeeID,
    e.Name,
    e.DepartmentID,
    e.Salary,
    a.AvgSalary
FROM
    employees e
JOIN
    AvgSalaries a ON e.DepartmentID = a.DepartmentID;

This will output the desired result, showing each employee with their department's average salary.

**Comparison:**

```
Employees Table
JOIN
AvgSalaries CTE
Selected Columns
```

**Code:**

```sql
WITH AvgSalaries AS (
    SELECT
        DepartmentID,
        AVG(Salary) AS AvgSalary
    FROM
        employees
    GROUP BY
        DepartmentID
)
SELECT
    e.EmployeeID,
    e.Name,
    e.DepartmentID,
    e.Salary,
    a.AvgSalary
FROM
    employees e
JOIN
    AvgSalaries a ON e.DepartmentID = a.DepartmentID;
```

**Example input / output:**

```text
WITH AvgSalaries AS (
    SELECT
        DepartmentID,
        AVG(Salary) AS AvgSalary
    FROM
        employees
    GROUP BY
        DepartmentID
)
SELECT
    e.EmployeeID,
    e.Name,
    e.DepartmentID,
    e.Salary,
    a.AvgSalary
FROM
    employees e
JOIN
    AvgSalaries a ON e.DepartmentID = a.DepartmentID;

-- Output:
+------------+--------+------------+---------+-------------+
| EmployeeID | Name   | Department | Salary  | AvgSalary   |
+------------+--------+------------+---------+-------------+
|          1 | Alice  |        101 | 70000.0 |      77500.0 |
|          2 | Bob    |        101 | 85000.0 |      77500.0 |
|          3 | Charlie|        102 | 60000.0 |      75000.0 |
|          4 | David  |        102 | 90000.0 |      75000.0 |
+------------+--------+------------+---------+-------------+
```

**Data:**

```sql
CREATE TABLE employees (
    EmployeeID INT,
    Name VARCHAR(50),
    DepartmentID INT,
    Salary DECIMAL(10,2)
);

INSERT INTO employees (EmployeeID, Name, DepartmentID, Salary) VALUES
(1, 'Alice', 101, 70000.00),
(2, 'Bob', 101, 85000.00),
(3, 'Charlie', 102, 60000.00),
(4, 'David', 102, 90000.00);

CREATE TABLE departments (
    DepartmentID INT,
    Name VARCHAR(50)
);

INSERT INTO departments (DepartmentID, Name) VALUES
(101, 'Engineering'),
(102, 'Sales');
```

**Flow:**

1. employees
2. CTE: AvgSalaries
3. JOIN ON DepartmentID
4. result set with Average Salary

