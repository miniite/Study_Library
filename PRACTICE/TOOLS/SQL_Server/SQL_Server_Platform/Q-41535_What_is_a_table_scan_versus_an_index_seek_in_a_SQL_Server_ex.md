---
type: "practice"
difficulty: "Deep"
---

# What is a table scan versus an index seek in a SQL Server execution plan?

**Difficulty:** Deep · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ OR Conditions and Index Usage](<../../../../TOOLS/SQL_Server/SQL_Server_Platform/OR_Conditions_and_Index_Usage.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In SQL Server, understanding the difference between a table scan and an index seek is crucial for optimizing query performance. A table scan involves reading every row of a table or index to find matching records, which can be slow if the data volume is large. An index seek, on the other hand, uses an index to locate specific rows efficiently.

To properly understand these concepts, let’s break down what each operation entails and when they are used.

**Code:**

```sql
-- Example table definition for demonstration
CREATE TABLE Employees (
    EmployeeID INT PRIMARY KEY,
    FirstName NVARCHAR(50),
    LastName NVARCHAR(50),
    Department NVARCHAR(100)
);
```

