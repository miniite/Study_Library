---
type: "practice"
difficulty: "Basic"
---

# How do you use the CASE expression to add conditional logic in a SELECT statement?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ CTEs in SQL Server](<../../../../TOOLS/SQL_Server/SQL_Server_Platform/CTEs_in_SQL_Server.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

The CASE expression in SQL is a powerful tool that allows us to perform conditional logic within our queries, essentially enabling us to create dynamic results based on different conditions. In this context, we will explore how to use the CASE expression in a SELECT statement to add conditional logic.

When you hear "CASE expression," think of it as a way to make decisions inside your SQL query, similar to using if-else statements in programming languages but tailored for SQL. The basic syntax is:

SQL
CASE
    WHEN condition1 THEN result1
    WHEN condition2 THEN result2
    ...
    ELSE default_result
END;

Let's break down the question and explore a practical example.

**Code:**

```sql
-- Example table creation and data insertion
CREATE TABLE Employees (
    EmployeeID INT,
    Department VARCHAR(50),
    Salary INT
);

INSERT INTO Employees (EmployeeID, Department, Salary) VALUES
(1, 'Sales', 30000),
(2, 'Marketing', 40000),
(3, 'HR', 35000),
(4, 'Engineering', 60000);
```

**Example input / output:**

```

Let's say we want to categorize the employees based on their salary into three categories: "Low," "Medium," and "High." We will use a CASE expression in our SELECT statement to achieve this.

SQL
Copy
-- Using CASE to add conditional logic
SELECT
    EmployeeID,
    Department,
    Salary,
    CASE
        WHEN Salary < 40000 THEN 'Low'
        WHEN Salary BETWEEN 40000 AND 50000 THEN 'Medium'
        ELSE 'High'
    END AS SalaryCategory
FROM Employees;

The output would look something like this:

PLAINTEXT
Copy
+-------------+------------+--------+--------------+
| EmployeeID | Department | Salary | SalaryCategory |
+-------------+------------+--------+--------------+
| 1          | Sales      | 30000  | Low           |
| 2          | Marketing  | 40000  | Medium        |
| 3          | HR         | 35000  | Low           |
| 4          | Engineering| 60000  | High          |
+-------------+------------+--------+--------------+
```

