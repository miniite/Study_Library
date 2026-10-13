---
type: "practice"
difficulty: "Deep"
---

# How would you test that a cursor-based procedure and its set-based rewrite produce identical results?

**Difficulty:** Deep · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ Cursors vs Set-based SQL](<../../../../01_Databases_and_SQL/Database_Objects/Cursors/Cursors_vs_Set-based_SQL.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To ensure that a cursor-based procedure and its set-based version produce identical results, we need to meticulously compare their outputs under various scenarios. Cursors operate row-by-row, while set-based operations process multiple rows at once. We must test both approaches thoroughly to validate they yield the same outcomes.

First, let's establish what the question is probing:

Understanding of Cursor and Set-Based Operations: The interviewer wants to assess your understanding of how these two methods work.
Comprehensive Testing Strategy: They are looking for a systematic approach to testing that both versions produce the same results under all possible conditions.

Now, let's break down the steps in detail:

Identify the Scope and Input Data:

Define the input data schema: Identify the tables involved and their relationships.
Determine the edge cases: Think about null values, unique constraints, or large datasets.

Set-Based Version:

Write a set-based SQL query that mimics the logic of the cursor-based procedure.
Test with different data points (including edge cases) to ensure both versions produce identical results.

Cursor-Based Procedure:

Implement the cursor-based version in your database system.
Run this implementation and collect its output.

Comparison:

Compare the outputs of both approaches row-by-row if necessary, or use aggregate functions to check overall correctness (e.g., sums, counts).

Validation:

Document any discrepancies found and investigate their causes.
Refactor the code if needed to ensure consistency.

Performance Testing:

Assess the performance of both versions, especially for large datasets, to determine which approach is more efficient.

**Comparison:**

| EmployeeID | Name    | Salary  |
|------------|---------|---------|
| 1          | Alice   | 5000.00 |
| 2          | Bob     | 6000.00 |
| 3          | Charlie | 7000.00 |

**Code:**

```sql
-- Example input data schema
CREATE TABLE Employees (
    EmployeeID INT PRIMARY KEY,
    Name VARCHAR(100),
    Salary DECIMAL(10, 2)
);

-- Set-based version of the procedure
WITH EmpSalaries AS (
    SELECT EmployeeID, Name, Salary
    FROM Employees
)
SELECT * INTO #TempSetBasedOutput
FROM EmpSalaries;

-- Cursor-based version of the procedure
DECLARE @EmployeeID INT;
DECLARE @Name VARCHAR(100);
DECLARE @Salary DECIMAL(10, 2);

DECLARE EmpCursor CURSOR FOR
    SELECT EmployeeID, Name, Salary FROM Employees;

OPEN EmpCursor;

FETCH NEXT FROM EmpCursor INTO @EmployeeID, @Name, @Salary;
WHILE @@FETCH_STATUS = 0
BEGIN
    -- Process each row (e.g., insert into a temporary table)
    INSERT INTO #TempCursorOutput(EmployeeID, Name, Salary) VALUES (@EmployeeID, @Name, @Salary);

    FETCH NEXT FROM EmpCursor INTO @EmployeeID, @Name, @Salary;
END;

CLOSE EmpCursor;
DEALLOCATE EmpCursor;
```

**Example input / output:**

```text
-- Run the set-based version
WITH EmpSalaries AS (
    SELECT EmployeeID, Name, Salary
    FROM Employees
)
SELECT * INTO #TempSetBasedOutput
FROM EmpSalaries;

-- Run the cursor-based version and insert output into a temporary table
DECLARE @EmployeeID INT;
DECLARE @Name VARCHAR(100);
DECLARE @Salary DECIMAL(10, 2);

DECLARE EmpCursor CURSOR FOR
    SELECT EmployeeID, Name, Salary FROM Employees;

OPEN EmpCursor;

FETCH NEXT FROM EmpCursor INTO @EmployeeID, @Name, @Salary;
WHILE @@FETCH_STATUS = 0
BEGIN
    INSERT INTO #TempCursorOutput(EmployeeID, Name, Salary) VALUES (@EmployeeID, @Name, @Salary);

    FETCH NEXT FROM EmpCursor INTO @EmployeeID, @Name, @Salary;
END;

CLOSE EmpCursor;
DEALLOCATE EmpCursor;

-- Compare the outputs
SELECT * FROM #TempSetBasedOutput;
SELECT * FROM #TempCursorOutput;
```

**Data:**

```json
{
  "employees": [
    { "employeeId": 1, "name": "Alice", "salary": 5000 },
    { "employeeId": 2, "name": "Bob", "salary": 6000 },
    { "employeeId": 3, "name": "Charlie", "salary": 7000 }
  ]
}
```

**Flow:**

1. Yes
2. No
3. No
4. No
5. Yes
6. Yes
7. Yes
8. No
9. Set-Based Query
10. Is it a SELECT?
11. Does it use aggregatefunctions?
12. Store result in temp table
13. Is it anINSERT/UPDATE/DELETE?
14. Use temporary tables forcomparison
15. Directly modify target table
16. Is it a Cursor?
17. Does it process one row at atime?
18. Process each row and store intemp table
19. Directly modify target table
20. Compare with set-basedversion
21. Compare with set-basedversion
22. Compare with set-basedversion
23. Compare with set-basedversion

