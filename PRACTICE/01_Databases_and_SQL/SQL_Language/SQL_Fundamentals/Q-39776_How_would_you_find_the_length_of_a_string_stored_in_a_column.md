---
type: "practice"
difficulty: "Basic"
---

# How would you find the length of a string stored in a column, and which function does your RDBMS use?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ String Functions](<../../../../01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/String_Functions.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To determine the length of a string stored in a database column using SQL, we need to understand how different relational database management systems (RDBMS) handle this task. The primary function used across most RDBMS is LENGTH or LEN, which returns the number of characters in a string.

#### Triaging the Question

The question asks about finding the length of a string stored in a column using SQL and specifies which function an RDBMS uses to accomplish this. This requires knowledge of common SQL functions used for string manipulation, specifically focusing on the LENGTH or LEN function.

#### First-Principles Solution

Understanding the Function:

In MySQL, PostgreSQL, and SQLite, you would use the LENGTH function.
In Microsoft SQL Server (T-SQL), you would use the LEN function.

Basic Usage: To find the length of a string in a column named column_name from a table called table_name, you would write:

SQL
SELECT LENGTH(column_name) FROM table_name;  -- MySQL, PostgreSQL, SQLite

or

SQL
SELECT LEN(column_name) FROM table_name;    -- Microsoft SQL Server (T-SQL)

#### Example
Query: Suppose we have a table employees with a column first_name. To find the length of each first name:

SQL
-- MySQL, PostgreSQL, SQLite
SELECT LENGTH(first_name) FROM employees;

-- Microsoft SQL Server (T-SQL)
SELECT LEN(first_name) FROM employees;

#### Closing Summary
and Common Mistakes/Follow-Ups

The LENGTH or LEN function is widely used across different RDBMS to get the length of a string. However, it’s crucial to be aware that the exact syntax might differ based on the specific database system being used.

#### Common mistakes
include using the wrong function (LENGTH vs. LEN) for a given RDBMS or failing to properly qualify the column name in the query.

Closing

In summary, finding the length of a string stored in a column is straightforward with SQL functions like LENGTH (MySQL, PostgreSQL, SQLite) and LEN (Microsoft SQL Server). Always ensure you use the appropriate function based on your database system.

