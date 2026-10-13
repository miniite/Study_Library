---
type: "practice"
difficulty: "Basic"
---

# What are the different types of JOINs available in T-SQL?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ CROSS APPLY and OUTER APPLY](<../../../../TOOLS/SQL_Server/T-SQL/CROSS_APPLY_and_OUTER_APPLY.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In T-SQL, there are several types of JOINs that allow you to retrieve data from multiple tables based on a related column between them. The main types include INNER JOIN, LEFT JOIN (or LEFT OUTER JOIN), RIGHT JOIN (or RIGHT OUTER JOIN), FULL JOIN (or FULL OUTER JOIN), and CROSS JOIN.

To fully understand these JOIN operations, it's important to know the basic structure of T-SQL queries involving JOINs and how they interact with tables. Let's break down each type step by step.

**Comparison:**

| Inner Join | Outer Joins | Full Outer Join (Simulated) | Cross Join |
|------------|-------------|----------------------------|------------|
| Only returns rows where the join condition is met. | Returns all rows from one table and matching rows from another, if any. | Returns all rows from both tables, with NULLs in non-matching columns. | Combines every row of one table with every row of another, no join conditions.

**Code:**

```sql
-- Example tables for demonstration
CREATE TABLE Employees (
    EmployeeID INT PRIMARY KEY,
    FirstName NVARCHAR(50),
    LastName NVARCHAR(50)
);

CREATE TABLE Departments (
    DepartmentID INT PRIMARY KEY,
    DepartmentName NVARCHAR(100)
);
```

**Example input / output:**

```text
-- INNER JOIN example
SELECT E.FirstName, E.LastName, D.DepartmentName
FROM Employees AS E
INNER JOIN Departments AS D ON E.DepartmentID = D.DepartmentID;

-- LEFT JOIN example
SELECT E.FirstName, E.LastName, D.DepartmentName
FROM Employees AS E
LEFT JOIN Departments AS D ON E.DepartmentID = D.DepartmentID;

-- RIGHT JOIN example
SELECT E.FirstName, E.LastName, D.DepartmentName
FROM Employees AS E
RIGHT JOIN Departments AS D ON E.DepartmentID = D.DepartmentID;

-- FULL OUTER JOIN example (T-SQL does not natively support this, but can be simulated)
SELECT E.FirstName, E.LastName, COALESCE(E.DepartmentID, D.DepartmentID) AS DepartmentID,
       COALESCE(D.DepartmentName, 'Not Assigned') AS DepartmentName
FROM Employees AS E
FULL OUTER JOIN Departments AS D ON E.DepartmentID = D.DepartmentID;

-- CROSS JOIN example
SELECT E.FirstName, E.LastName, D.DepartmentName
FROM Employees AS E
CROSS JOIN Departments AS D;
```

**Data:**

```json
{
  "Employees": [
    { "EmployeeID": 1, "FirstName": "John", "LastName": "Doe" },
    { "EmployeeID": 2, "FirstName": "Jane", "LastName": "Smith" }
  ],
  "Departments": [
    { "DepartmentID": 10, "DepartmentName": "HR" },
    { "DepartmentID": 20, "DepartmentName": "IT" }
  ]
}
```

**Flow:**

1. Yes
2. No
3. Left
4. Right
5. No, Full Outer Join?
6. Yes
7. No, Cross Join?
8. Start
9. Is the query about JOIN types?
10. Describe INNER JOIN
11. Example code for INNER JOIN
12. Is it LEFT or RIGHT JOIN?
13. Describe LEFT JOIN
14. Example code for LEFT JOIN
15. Describe RIGHT JOIN
16. Example code for RIGHT JOIN
17. Is it FULL OUTER JOIN?
18. Simulate FULL OUTER JOINusing COALESCE
19. Example code for simulatingFULL OUTER JOIN
20. Describe CROSS JOIN
21. Example code for CROSS JOIN

