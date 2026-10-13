---
type: "practice"
difficulty: "Basic"
---

# What is the difference between the GETDATE() and SYSDATETIME() functions in SQL Server?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ Date and Time Types](<../../../../TOOLS/SQL_Server/SQL_Server_Platform/Date_and_Time_Types.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In SQL Server, both GETDATE() and SYSDATETIME() are used to retrieve the current date and time; however, they differ significantly in terms of precision and functionality.

TL;DR:
GETDATE() returns a datetime value with lower precision.
SYSDATETIME() returns a datetime2 value with higher precision and is generally recommended for modern applications.

#### Triage

The interviewer is likely probing your understanding of SQL Server's date-time functions, specifically focusing on their differences in terms of data type, precision, and usage. The key assumptions here are that you have basic knowledge of SQL Server and its date-time capabilities.

#### Solution
GETDATE() Function
Return Type: datetime
Precision: 3 fractional parts (0.1 millisecond)
Usage: Suitable for scenarios where higher precision is not required, such as simple logging or reporting.

#### Example
SQL
SELECT GETDATE();
-- Output might be '2023-10-05 14:37:28.123'
SYSDATETIME() Function
Return Type: datetime2
Precision: 7 fractional parts (0.0000001 millisecond)
Usage: Recommended for modern applications and scenarios requiring higher precision, such as financial transactions or time-sensitive operations.

#### Example
SQL
SELECT SYSDATETIME();
-- Output might be '2023-10-05 14:37:28.1234567'

#### Closing Summary

Both GETDATE() and SYSDATETIME() are useful for getting the current date and time in SQL Server, but SYSDATETIME() is generally preferred due to its higher precision and better alignment with modern database practices.

#### Common Mistakes/Follow-Ups
Mistake: Using GETDATE() when high precision is required.

#### Follow-Up
When should I use GETDATE() instead of SYSDATETIME(), and what are the implications of using each function in different contexts?

