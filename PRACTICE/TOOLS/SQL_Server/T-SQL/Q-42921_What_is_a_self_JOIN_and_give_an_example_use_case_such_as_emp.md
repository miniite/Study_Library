---
type: "practice"
difficulty: "Intermediate"
---

# What is a self JOIN and give an example use case such as employee-manager relationships?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ CROSS APPLY and OUTER APPLY](<../../../../TOOLS/SQL_Server/T-SQL/CROSS_APPLY_and_OUTER_APPLY.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Self JOIN in T-SQL is a type of join where a table is joined with itself, often to establish relationships between records within the same table. A common use case involves scenarios like employee hierarchies or cascading data. For instance, if we have an "employees" table that contains columns for EmployeeID, Name, ManagerID, and Position, a self JOIN can be used to retrieve information about employees and their managers.

**Code:**

```sql
-- Example of a simple T-SQL query using self JOIN
SELECT e1.Name AS EmployeeName, e2.Name AS ManagerName
FROM employees e1 -- alias for the table as 'e1'
JOIN employees e2 -- alias for the table as 'e2'
ON e1.ManagerID = e2.EmployeeID; -- join condition based on ManagerID and EmployeeID
```

**Example input / output:**

```text
-- Input query using self JOIN to find employees and their managers:
SELECT e1.Name AS EmployeeName, e2.Name AS ManagerName
FROM employees e1 -- alias for the table as 'e1'
JOIN employees e2 -- alias for the table as 'e2'
ON e1.ManagerID = e2.EmployeeID;

-- Expected output (based on sample data):
+-------------+--------------+
| EmployeeName | ManagerName  |
+-------------+--------------+
| Bob          | Alice        |
| David        | Bob          |
+-------------+--------------+
```

**Data:**

```json
{
  "employees": [
    { "EmployeeID": 1, "Name": "Alice", "ManagerID": null, "Position": "CEO" },
    { "EmployeeID": 2, "Name": "Bob", "ManagerID": 1, "Position": "CTO" },
    { "EmployeeID": 3, "Name": "Charlie", "ManagerID": 1, "Position": "COO" },
    { "EmployeeID": 4, "Name": "David", "ManagerID": 2, "Position": "DevOps Lead" }
  ]
}
```

