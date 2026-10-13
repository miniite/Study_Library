---
type: "practice"
difficulty: "Intermediate"
---

# How does NULL behave in comparisons and how do you test for it in T-SQL?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ CROSS APPLY and OUTER APPLY](<../../../../TOOLS/SQL_Server/T-SQL/CROSS_APPLY_and_OUTER_APPLY.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

NULL values in SQL can be tricky, especially when dealing with comparisons. In this answer, we'll explore how NULL behaves in comparisons within T-SQL and learn effective ways to handle them.

Firstly, let's understand what NULL means in the context of a database. A NULL value represents an unknown or undefined value, which is different from a zero (0) or an empty string (''). When dealing with NULL values, it’s essential to recognize that comparing a column against a specific value (including NULL itself) typically results in an indeterminate result.

Now, let's dive into the specifics of how NULL behaves in comparisons and how we can test for them effectively.

**Code:**

```sql
-- Example table creation
CREATE TABLE Employees (
    EmployeeID INT,
    FirstName VARCHAR(50),
    LastName VARCHAR(50),
    DateOfBirth DATE
);

-- Inserting some sample data, including NULL values
INSERT INTO Employees (EmployeeID, FirstName, LastName, DateOfBirth)
VALUES
    (1, 'John', 'Doe', '1985-03-24'),
    (2, 'Jane', 'Smith', NULL),
    (3, 'Alice', 'Johnson', '1976-06-15');
```

**Example input / output:**

```text
-- Testing for NULL values using IS NULL and IS NOT NULL
SELECT *
FROM Employees
WHERE DateOfBirth IS NULL; -- This will return rows where DateOfBirth is NULL

SELECT *
FROM Employees
WHERE DateOfBirth IS NOT NULL; -- This will return rows where DateOfBirth is not NULL
```

**Flow:**

1. Yes
2. No
3. Yes
4. No
5. Is the value null?
6. DateOfBirth IS NULL
7. Select row
8. Skip this row
9. E
10. Select row
11. Skip this row

