---
type: "practice"
difficulty: "Basic"
---

# How do you write a SELECT statement to fetch only the top 10 rows in T-SQL?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ T-SQL Basics](<../../../../TOOLS/SQL_Server/T-SQL/T-SQL_Basics.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To retrieve only the top 10 rows from a table using T-SQL, you typically use the TOP keyword followed by a number and an ORDER BY clause to specify the sorting criteria. The SELECT statement is then used to fetch data based on these conditions.

The core idea here is that the TOP keyword limits the result set to the specified number of rows, which in this case is 10. The ORDER BY clause ensures that the rows are returned in a specific order before being limited by TOP.

#### Triage

The question asks for writing a T-SQL statement to fetch only the top 10 rows.
The interviewer might be probing your understanding of SQL basics and familiarity with common clauses like SELECT, ORDER BY, and TOP.
We will assume you have access to a table named Employees with columns such as EmployeeID and Name.

#### First-principles solution

Start with the SELECT statement, specifying which columns you want to retrieve.
Use the FROM clause to indicate the table from which data is fetched.
Incorporate the TOP 10 keyword followed by an ORDER BY clause that defines the sorting criteria.
SQL
-- Example: Selecting top 10 employees sorted by their salary in descending order
SELECT TOP 10 EmployeeID, Name, Salary
FROM Employees
ORDER BY Salary DESC;

#### Closing summary
The provided T-SQL statement fetches the top 10 rows from the Employees table based on a specific column (Salary). It's crucial to understand that without an ORDER BY clause, using just TOP 10 could lead to undefined behavior because the result set would be arbitrary.

#### Common mistakes/follow-ups

Forgetting to include the ORDER BY clause, which can make the query return unpredictable results.
Using TOP followed by a percentage value instead of an integer, though this is more common in other SQL dialects like MySQL and PostgreSQL.
Not understanding that if you need to limit rows based on conditions (e.g., top 10 salaries after filtering), you would use WHERE before ORDER BY, e.g., SELECT TOP 10 * FROM Employees WHERE Department = 'IT' ORDER BY Salary DESC.

**Comparison:**

| Column Name   | Type      |
|--------------|-----------|
| EmployeeID   | int       |
| Name         | varchar(100)  |
| Salary       | decimal(10,2) |

**Code:**

```sql
-- Example T-SQL query to fetch top 10 rows from a table
SELECT TOP 10 EmployeeID, Name, Salary
FROM Employees
ORDER BY Salary DESC;
```

**Example input / output:**

```text
-- Example input to output
Input:
SELECT TOP 10 EmployeeID, Name, Salary
FROM Employees
ORDER BY Salary DESC;

Output:
EmployeeID | Name      | Salary
-----------|-----------|--------
1          | John Doe  | 50000.00
2          | Jane Smith| 48000.00
3          | Alice Wu  | 47500.00
4          | Bob Lee   | 46000.00
5          | Carol Chen| 45000.00
6          | Dave Tan  | 44000.00
7          | Eve Zhang | 43000.00
8          | Frank Li  | 42000.00
9          | Grace Wang| 41000.00
10         | Henry Liu | 40000.00
```

**Data:**

```json
{
  "table": {
    "name": "Employees",
    "columns": [
      { "name": "EmployeeID", "type": "int" },
      { "name": "Name", "type": "varchar(100)" },
      { "name": "Salary", "type": "decimal(10,2)" }
    ]
  }
}
```

**Flow:**

1. Yes
2. No
3. Start
4. Is there a specific column toorder by?
5. SELECT TOP 10 * FROMEmployees ORDER BY[Column]
6. Select arbitrary columns
7. Execute query and retrieve top10 rows
8. Execute query with TOP 10 andarbitrary columns
9. Output top 10 rows
10. End

