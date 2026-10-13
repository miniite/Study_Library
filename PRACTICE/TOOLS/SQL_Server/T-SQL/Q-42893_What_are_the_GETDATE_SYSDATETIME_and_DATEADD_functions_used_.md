---
type: "practice"
difficulty: "Basic"
---

# What are the GETDATE SYSDATETIME and DATEADD functions used for?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ Temp Tables vs Table Variables](<../../../../TOOLS/SQL_Server/T-SQL/Temp_Tables_vs_Table_Variables.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Hello! In a basic database context, GETDATE, SYSDATETIME, and DATEADD are essential T-SQL (Transact-SQL) functions that help you work with dates and times in SQL Server databases. Let's break down each of these functions:

TL;DR
GETDATE(): Returns the current date and time on the server.
SYSDATETIME(): Also returns the current date and time, but it is more precise than GETDATE() as it includes fractional seconds up to 7 digits.
DATEADD(): Adds or subtracts a specified amount of time from a given datetime value.

#### Triaging the Question

The question is asking you to understand the core functionality and uses of these date/time functions in T-SQL. The interviewer may be probing your knowledge of SQL Server's date and time handling capabilities, as well as how different functions can be used depending on specific needs.

#### Assumptions
We are working with a SQL Server database.
You have basic understanding of date and time concepts.

#### First-Principles Solution

GETDATE()

This function returns the current date and time from the server's system clock. It is often used in queries to get the current timestamp when records are inserted or updated.

#### Example
SQL
SELECT GETDATE();
Output: A datetime value representing the current moment on the server.

SYSDATETIME()

This function provides a more precise date and time value, including fractional seconds up to 7 digits. It is particularly useful in scenarios where high precision is required.

#### Example
SQL
SELECT SYSDATETIME();
Output: A datetime2 value with higher precision.

DATEADD()

This function allows you to add or subtract a specified amount of time (in years, months, days, etc.) from a given datetime value.

#### Example
SQL
SELECT DATEADD(day, 5, '2023-10-01');
Output: 2023-10-06 00:00:00.000 (if the exact time is not specified).

#### Closing Summary
+ Common Mistakes/Follow-Ups
Common Mistake: Using GETDATE() and SYSDATETIME() interchangeably when precision matters.

#### Follow-Up
Question: "How would you handle timezone differences in SQL Server?" or "What are some other date/time functions available in T-SQL?"
Triaging the Follow-Up

If asked about handling timezone differences, you can mention that SQL Server supports time zones through the AT TIME ZONE clause. For example:

SQL
SELECT CONVERT(datetimeoffset(7), '2023-10-01 08:00:00', 121) AT TIME ZONE 'Pacific Standard Time';

This shows how to handle datetime values in different time zones, which is crucial for global applications.

