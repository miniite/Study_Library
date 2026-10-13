---
type: "practice"
difficulty: "Basic"
---

# How do you get the current date and time using SQL functions like NOW, GETDATE, or SYSDATE?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ Date Functions](<../../../../01_Databases_and_SQL/SQL_Language/SQL_Functions/Date_Functions.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To retrieve the current date and time in SQL, we use specific functions depending on the database management system (DBMS) you're working with. For a fresh Database Developer, it's essential to understand how these functions work across different DBMSes.

The question asks about getting the current date and time using SQL functions like NOW, GETDATE, or SYSDATE. Let's break down each of these:

NOW(): This function is commonly used in MySQL, PostgreSQL, and some other databases to return the current date and time.
GETDATE(): This function is specific to Microsoft SQL Server. It returns the current date and time as a datetime value.
SYSDATE: This function is used in Oracle databases to get the current date and time.

Each of these functions serves the same purpose but with different syntax depending on the DBMS.

#### Triaging the Question

The interviewer might be probing for an understanding of:

The specific function(s) available in different SQL environments.
Basic syntax differences between these functions.
How to handle potential variations or errors in query execution.

#### First-Principles Solution

Here’s a step-by-step explanation:

MySQL
SQL
SELECT NOW();

This will return the current date and time in a format like YYYY-MM-DD HH:MI:SS.

Microsoft SQL Server
SQL
SELECT GETDATE();

This returns the current date and time as a datetime value, typically formatted like 2023-10-04 15:30:45.678.

Oracle
SQL
SELECT SYSDATE FROM DUAL;

DUAL is a one-row table that exists in every Oracle database; it’s used when you need to return a single value or perform a query operation.

#### Closing Summary
+ Common Mistakes/Follow-Ups

In summary, to get the current date and time in SQL, use NOW() for MySQL and PostgreSQL, GETDATE() for Microsoft SQL Server, and SYSDATE with the DUAL table for Oracle. Always check the DBMS documentation to ensure you are using the correct function.

A common mistake is assuming that one function will work across all databases without checking the specific syntax requirements of each system. Additionally, forgetting about the DUAL table in Oracle can lead to errors if not used correctly.

For follow-ups, an interviewer might ask:

How would you handle time zone differences?
Can these functions be combined with other SQL queries?

**Code:**

```sql
-- MySQL
SELECT NOW();

-- Microsoft SQL Server
SELECT GETDATE();

-- Oracle
SELECT SYSDATE FROM DUAL;
```

