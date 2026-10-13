---
type: "practice"
difficulty: "Basic"
---

# What are the common aggregate functions in SQL Server and how does COUNT(*) differ from COUNT(column)?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ CTEs in SQL Server](<../../../../TOOLS/SQL_Server/SQL_Server_Platform/CTEs_in_SQL_Server.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In SQL Server, aggregate functions are used to perform calculations on a set of values and return a single value as the result. Common aggregate functions include SUM, AVG, MAX, MIN, and COUNT. The COUNT function is particularly useful for counting rows or non-null values in a column. However, it's important to understand the subtle differences between COUNT(*) and COUNT(column).

#### Triaging the Question

The question asks about common aggregate functions in SQL Server, specifically focusing on the difference between COUNT(*) and COUNT(column). The interviewer is probing your understanding of basic SQL concepts, particularly the nuances of aggregate functions. We need to clarify that both COUNT(*) and COUNT(column) are used for counting rows, but they differ slightly in their behavior.

#### First-Principles Solution

Let's start by explaining each function:

COUNT(*):

COUNT(*) counts the number of rows in a table or result set, including rows with NULL values.

#### Example
If you have a table named Employees, and there are 50 employees with non-null data, SELECT COUNT(*) FROM Employees; will return 50.

COUNT(column):

COUNT(column) counts the number of rows where the specified column has a non-NULL value.

#### Example
If you have a table named Employees and the Salary column is nullable, using SELECT COUNT(Salary) FROM Employees; will return the number of employees who have a salary recorded (i.e., not NULL).
Detailed Explanation

To understand why these two functions behave differently, let's break it down step by step:

Table Structure:

#### Cons
ider an Employees table with columns: EmployeeID, Name, and Salary.
There are 50 employees in total.

NULL Values:

Some of these employees might have missing salary information, meaning their Salary column is NULL.

Using COUNT(*):

SELECT COUNT(*) FROM Employees; returns the total number of rows, which is 50 (including those with NULL salaries).

Using COUNT(Salary):

SELECT COUNT(Salary) FROM Employees; only counts the rows where the Salary column has a non-NULL value.
If 10 employees have missing salary information, this query will return 40.

#### Closing Summary

In summary, COUNT(*) is used to count all rows, including those with NULL values in any columns. On the other hand, COUNT(column) counts only the rows where the specified column has a non-NULL value. Understanding these differences is crucial when writing SQL queries that require accurate row counting.

#### Common Mistakes
and Follow-Ups
Misunderstanding COUNT(*): Sometimes developers confuse COUNT(*) with COUNT(column), leading to incorrect expectations about the number of rows returned.
Ignoring NULL Values: Developers might overlook how NULL values affect aggregate functions, particularly when dealing with columns that can be nullable.

If asked further, you could explore additional edge cases such as:

What happens if all values in a column are NULL?
How does COUNT behave with different data types?

**Comparison:**

| EmployeeID | Name       | Salary |
|------------|------------|--------|
| 1          | John Doe   | NULL   |
| 2          | Jane Smith | 50000  |
| 3          | Mike Johnson | 60000 |
| 4          | Sarah Lee  | NULL   |

**Code:**

```sql
-- Example for COUNT(*)
SELECT COUNT(*) AS TotalEmployees FROM Employees;

-- Example for COUNT(column)
SELECT COUNT(Salary) AS NonNullSalaries FROM Employees;
```

**Example input / output:**

```text
-- Example input and output for COUNT(*)
Input:
SELECT COUNT(*) AS TotalEmployees FROM Employees;

Output:
TotalEmployees | Value
---------------|------
             50

-- Example input and output for COUNT(column)
Input:
SELECT COUNT(Salary) AS NonNullSalaries FROM Employees;

Output:
NonNullSalaries | Value
----------------|------
              3
```

**Data:**

```yaml
employees:
  - EmployeeID: 1
    Name: John Doe
    Salary: NULL
  - EmployeeID: 2
    Name: Jane Smith
    Salary: 50000
  - EmployeeID: 3
    Name: Mike Johnson
    Salary: 60000
  - EmployeeID: 4
    Name: Sarah Lee
    Salary: NULL
```

**Flow:**

1. Yes
2. Yes
3. No
4. No
5. No
6. COUNT(*)
7. Does column have NULLvalues?
8. COUNT(column)
9. Is the column nullable?
10. Are all values NULL?
11. 0 rows returned
12. Count non-NULL values
13. Total row count

