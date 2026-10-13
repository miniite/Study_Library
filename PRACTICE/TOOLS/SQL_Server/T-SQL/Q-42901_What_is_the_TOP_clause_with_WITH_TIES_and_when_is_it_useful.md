---
type: "practice"
difficulty: "Intermediate"
---

# What is the TOP clause with WITH TIES and when is it useful?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ T-SQL Basics](<../../../../TOOLS/SQL_Server/T-SQL/T-SQL_Basics.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Let's dive into the TOP clause in SQL, specifically focusing on its use with the WITH TIES option. The TOP clause is a fundamental part of querying data and can be very useful when you want to retrieve a certain number of rows from a result set that meet your criteria.

The TOP clause is used to limit the number of returned rows, but it gets even more interesting when combined with WITH TIES. This option is particularly useful in scenarios where you need to return all rows that have the highest (or lowest) value for a specific column, without arbitrarily limiting the result set. In other words, it ensures that if there are ties on the top value, they are included in the result.

#### Triaging the Question

The core of this question is about understanding when and why to use the TOP clause with WITH TIES. The key idea here is recognizing situations where you need to include all rows that share the same highest (or lowest) value for a given column. This can be crucial in scenarios like identifying employees with the highest salary or finding multiple users with the most points.

#### First-Principles Solution

First, let's look at how TOP works without WITH TIES. Consider a simple table of employee salaries:

SQL
CREATE TABLE Employees (
    EmployeeID INT,
    FirstName VARCHAR(50),
    LastName VARCHAR(50),
    Salary DECIMAL(18, 2)
);

INSERT INTO Employees (EmployeeID, FirstName, LastName, Salary) VALUES
(1, 'Alice', 'Johnson', 75000.00),
(2, 'Bob', 'Smith', 80000.00),
(3, 'Charlie', 'Davis', 80000.00),
(4, 'Diana', 'Wilson', 85000.00);

To get the top 1 salary:

SQL
SELECT TOP 1 Salary
FROM Employees;

This will return only one row with the highest salary.

However, if you use TOP with WITH TIES, it will return all rows that have this highest salary value:

SQL
SELECT TOP (1) WITH TIES Salary
FROM Employees
ORDER BY Salary DESC;

In this case, both Bob and Charlie's salaries are included in the result set because they share the top salary.

#### Closing Summary
+ Common Mistakes/Follow-Ups

The TOP clause with WITH TIES is particularly useful when you want to retrieve all rows that have a certain top value. For example, this can be used in scenarios like identifying all employees with the highest performance score or finding multiple users with the most points.

A common mistake is assuming that TOP without WITH TIES will return multiple rows for ties; it does not. Similarly, using TOP 10 WITH TIES might not always get you exactly 10 rows if there are ties beyond the first few records.

When asked about this in an interview, make sure to explain these nuances clearly and provide a real-world example of when such a query would be beneficial.

**Comparison:**

| EmployeeID | FirstName | LastName | Salary   |
|------------|----------|---------|----------|
| 1          | Alice    | Johnson | 75000.00|
| 2          | Bob      | Smith   | 80000.00|
| 3          | Charlie  | Davis   | 80000.00|
| 4          | Diana    | Wilson  | 85000.00|

**Code:**

```sql
-- Example table creation and data insertion
CREATE TABLE Employees (
    EmployeeID INT,
    FirstName VARCHAR(50),
    LastName VARCHAR(50),
    Salary DECIMAL(18, 2)
);

INSERT INTO Employees (EmployeeID, FirstName, LastName, Salary) VALUES
(1, 'Alice', 'Johnson', 75000.00),
(2, 'Bob', 'Smith', 80000.00),
(3, 'Charlie', 'Davis', 80000.00),
(4, 'Diana', 'Wilson', 85000.00);

-- Query to get top salary with ties
SELECT TOP (1) WITH TIES Salary
FROM Employees
ORDER BY Salary DESC;
```

**Example input / output:**

```text
-- Input: None

-- Output:
-- +--------+
-- | Salary |
-- +--------+
-- | 80000.00|
-- | 80000.00|
-- +--------+
```

**Data:**

```sql
CREATE TABLE Employees (
    EmployeeID INT,
    FirstName VARCHAR(50),
    LastName VARCHAR(50),
    Salary DECIMAL(18, 2)
);

INSERT INTO Employees (EmployeeID, FirstName, LastName, Salary) VALUES
(1, 'Alice', 'Johnson', 75000.00),
(2, 'Bob', 'Smith', 80000.00),
(3, 'Charlie', 'Davis', 80000.00),
(4, 'Diana', 'Wilson', 85000.00);
```

**Flow:**

1. Create Employees Table
2. Insert Data
3. Query Top Salary with Ties
4. Return Multiple Rows if Tied
5. Understand Use Case for Ties

