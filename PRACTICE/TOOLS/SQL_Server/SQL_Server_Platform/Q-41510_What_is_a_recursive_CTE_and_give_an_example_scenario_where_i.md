---
type: "practice"
difficulty: "Deep"
---

# What is a recursive CTE and give an example scenario where it is useful?

**Difficulty:** Deep · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ CTEs in SQL Server](<../../../../TOOLS/SQL_Server/SQL_Server_Platform/CTEs_in_SQL_Server.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

A recursive Common Table Expression (CTE) in SQL Server is a powerful feature that allows you to perform operations on sets of data by recursively calling the same CTE within itself. It's particularly useful when dealing with hierarchical or nested data structures, such as organizational charts, file systems, or parent-child relationships.

#### Triage

The question asks for an explanation of recursive CTEs and a scenario where they are beneficial. We need to explain what recursive CTEs are, their syntax in SQL Server, and provide a practical example.

#### First-Principles Solution
What is a Recursive CTE?

A recursive CTE consists of two parts:

Base Case: This is the initial part that defines the starting point or condition for the recursion.
Recursive Case: This part defines how to get from one level in the hierarchy to another.

The WITH clause introduces the recursive CTE, and it can contain both a base query (initial query) and a recursive member (the part that references itself).

#### Example Scenario

Let's consider an organizational chart where each employee has a manager. We want to find all employees who work under a specific supervisor, including direct reports and indirect reports.

Here’s how you would write the SQL:

SQL
WITH EmployeeHierarchy AS (
    -- Base case: Find the top-level manager or the supervisor we are interested in
    SELECT
        EmployeeID,
        ManagerID,
        Name,
        1 AS Level
    FROM Employees
    WHERE SupervisorID = @SupervisorID

    UNION ALL

    -- Recursive case: Get all direct and indirect reports
    SELECT
        e.EmployeeID,
        e.ManagerID,
        e.Name,
        eh.Level + 1 AS Level
    FROM Employees e
    INNER JOIN EmployeeHierarchy eh ON e.ManagerID = eh.EmployeeID
)
SELECT * FROM EmployeeHierarchy;

In this example:

The base case starts by selecting the supervisor and their direct reports.
The recursive part joins the Employees table with the CTE to get all employees reporting to any of the previously selected employees.

#### Closing Summary
+ Common Mistakes/Follow-Ups

Recursive CTEs are incredibly useful for hierarchical data, but they can be resource-intensive if not carefully designed. Key points to remember include:

Performance: Ensure that you use an appropriate index on the ManagerID column to improve performance.
Depth and Limits: Be cautious of infinite recursion by setting a limit on the depth with OPTION (MAXRECURSION n).
Memory Usage: Recursive CTEs can consume significant memory, especially in deeply nested structures.

For follow-ups:

How would you optimize this query for larger datasets?
What are some common pitfalls when using recursive CTEs?

**Comparison:**

| EmployeeID | Name   | ManagerID |
|------------|--------|----------|
| 1          | Alice  |          |
| 2          | Bob    | 1        |
| 3          | Charlie| 2        |
| 4          | David  | 2        |
| 5          | Eve    | 1        |

**Code:**

```sql
WITH EmployeeHierarchy AS (
    SELECT
        EmployeeID,
        ManagerID,
        Name,
        1 AS Level
    FROM Employees
    WHERE SupervisorID = @SupervisorID

    UNION ALL

    SELECT
        e.EmployeeID,
        e.ManagerID,
        e.Name,
        eh.Level + 1 AS Level
    FROM Employees e
    INNER JOIN EmployeeHierarchy eh ON e.ManagerID = eh.EmployeeID
)
SELECT * FROM EmployeeHierarchy;
```

**Example input / output:**

```text
Input:
SupervisorID = 1

Output:
| EmployeeID | Name   | ManagerID |
|------------|--------|----------|
|          1 | Alice  |         |
|          2 | Bob    |        1 |
|          3 | Charlie|        2 |
|          4 | David  |        2 |
|          5 | Eve    |        1 |
```

**Data:**

```json
{
  "Employees": [
    { "EmployeeID": 1, "Name": "Alice", "ManagerID": null },
    { "EmployeeID": 2, "Name": "Bob", "ManagerID": 1 },
    { "EmployeeID": 3, "Name": "Charlie", "ManagerID": 2 },
    { "EmployeeID": 4, "Name": "David", "ManagerID": 2 },
    { "EmployeeID": 5, "Name": "Eve", "ManagerID": 1 }
  ]
}
```

**Flow:**

1. Yes
2. No
3. Start
4. Is Base Case?
5. Select Initial Data
6. Recursive Query
7. Union All
8. End
9. Return Result

