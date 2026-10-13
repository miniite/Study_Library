---
type: "practice"
difficulty: "Basic"
---

# What is the difference between UNION and UNION ALL in SQL Server?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ OR Conditions and Index Usage](<../../../../TOOLS/SQL_Server/SQL_Server_Platform/OR_Conditions_and_Index_Usage.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In SQL Server, both UNION and UNION ALL are used to combine the results of two or more SELECT statements into a single result set. However, they differ fundamentally in how they handle duplicate rows. Let's break down the differences starting from first principles.

#### Triage
Question: What is the difference between UNION and UNION ALL in SQL Server?
What it probes: Understanding of basic SQL operations and the nuances between two similar but distinct set operators.

#### Assumptions
Familiarity with SELECT statements, and understanding of how sets are handled in SQL.

#### Solution

UNION: When you use UNION, it combines the result sets from multiple SELECT statements while eliminating any duplicate rows. This means that every row returned by the combined query must be unique based on all columns involved. If there is a row with the same values across different SELECT statements, SQL Server will consider it a duplicate and omit it.

UNION ALL: On the other hand, UNION ALL simply combines the result sets without removing any duplicates. This means that every row from each individual SELECT statement is included in the final result set, even if they are identical to rows returned by other SELECT statements.

#### Example

Let's consider two tables, Table1 and Table2, with a common column ID.

SQL
CREATE TABLE Table1 (
    ID INT,
    Name VARCHAR(50)
);

INSERT INTO Table1 (ID, Name) VALUES (1, 'Alice');
INSERT INTO Table1 (ID, Name) VALUES (2, 'Bob');

CREATE TABLE Table2 (
    ID INT,
    Name VARCHAR(50)
);

INSERT INTO Table2 (ID, Name) VALUES (3, 'Charlie');
INSERT INTO Table2 (ID, Name) VALUES (1, 'Alice');
UNION Example:
SQL
SELECT * FROM Table1
UNION
SELECT * FROM Table2;

Output:

CODE
| ID |  Name   |
|----|--------|
| 1  | Alice  |
| 2  | Bob    |
| 3  | Charlie|
UNION ALL Example:
SQL
SELECT * FROM Table1
UNION ALL
SELECT * FROM Table2;

Output:

CODE
| ID |  Name   |
|----|--------|
| 1  | Alice  |
| 2  | Bob    |
| 3  | Charlie|
| 1  | Alice  |

#### Closing Summary
+ Common Mistakes/Follow-Ups

Remember, UNION is more useful when you want to ensure that the final result set does not contain duplicate rows. Use it when you are sure about the uniqueness of your data or need a clean result.

In contrast, UNION ALL can be faster because it doesn't perform the overhead of checking for duplicates, making it suitable in scenarios where performance is critical and minor duplication is acceptable.

#### Common Mistakes
Assuming UNION automatically sorts the result set.
Not considering that both queries must have the same number of columns with compatible data types when using either operator.
Overusing UNION ALL for operations where duplicates could significantly impact the final result, which might lead to unnecessary storage and processing.
Self-check

This explanation covers the key differences between UNION and UNION ALL in SQL Server, ensuring completeness and correctness.

