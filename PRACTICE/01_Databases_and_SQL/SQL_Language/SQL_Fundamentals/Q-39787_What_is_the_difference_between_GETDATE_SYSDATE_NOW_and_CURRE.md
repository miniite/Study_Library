---
type: "practice"
difficulty: "Basic"
---

# What is the difference between GETDATE, SYSDATE, NOW, and CURRENT_TIMESTAMP across databases?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ Timezones and Timestamps](<../../../../01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Timezones_and_Timestamps.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To understand the differences between GETDATE, SYSDATE, NOW, and CURRENT_TIMESTAMP in SQL databases, we need to look at how these functions are implemented in different database systems. These functions are used to get the current date and time, but their behavior can vary significantly depending on the specific database system you're working with.

Let's break down each function:

GETDATE - This is a Microsoft SQL Server-specific function.
SYSDATE - This is an Oracle-specific function.
NOW - This is supported by MySQL and PostgreSQL, but behaves differently in different systems.
CURRENT_TIMESTAMP - This is used across multiple database systems like PostgreSQL, MySQL, SQLite, and others.

#### Triage

The question asks for the differences between these functions. The key points to consider are:

Which databases support each function?
What do they return (date, time, or both)?
Are there any nuances in behavior?

We will reason through each function from first principles to understand their core differences and practical implications.

#### Solution

GETDATE (Microsoft SQL Server)

Behavior: Returns the current date and time.

#### Example
Usage: SELECT GETDATE()
Nuances: This is specific to Microsoft SQL Server and not available in other database systems.

SYSDATE (Oracle Database)

Behavior: Returns the current date and time as a timestamp.

#### Example
Usage: SELECT SYSDATE FROM dual;
Nuances: Oracle's implementation returns a precise timestamp, which includes both date and time down to microseconds.

NOW (MySQL/PostgreSQL)

Behavior:
In MySQL: Returns the current date and time as a string in YYYY-MM-DD HH:MI:SS format.
In PostgreSQL: Returns the current date and time as a timestamp with timezone.

#### Example
Usage:
MySQL: SELECT NOW();
PostgreSQL: SELECT now();
Nuances: The behavior of NOW() can vary between databases, so it's important to check which one you are using.

CURRENT_TIMESTAMP (Multiple Database Systems)

Behavior: Returns the current date and time as a timestamp.

#### Example
Usage:
PostgreSQL: SELECT CURRENT_TIMESTAMP;
MySQL: SHOW VARIABLES LIKE 'system_time_zone'; to see if it returns the correct timezone
Nuances: This is more portable across databases, making it a good choice when you need cross-database compatibility.

#### Closing Summary

In summary, while these functions all aim to return the current date and time, they are specific to different database systems and have distinct behaviors. GETDATE and SYSDATE are tied to Microsoft SQL Server and Oracle respectively, whereas NOW has variations in behavior across MySQL and PostgreSQL, and CURRENT_TIMESTAMP is more portable but behaves similarly across multiple databases.

#### Common Mistakes/Follow-Ups
Mistake: Assuming all these functions behave identically across different database systems.

#### Follow-Up
When writing cross-database code, always consider the specific function used by each system. For example, if you are working with both MySQL and PostgreSQL, using NOW() in MySQL might not give the same result as now() in PostgreSQL.

#### Example
Input/Output
SQL
-- SQL Server
SELECT GETDATE();  -- Output: YYYY-MM-DD HH:MI:SS.SS

-- Oracle
SELECT SYSDATE FROM dual;  -- Output: DD-MON-RR HH24:MI:SS.FF6

-- MySQL
SELECT NOW();  -- Output: YYYY-MM-DD HH:MI:SS

-- PostgreSQL
SELECT now();  -- Output: YYYY-MM-DD HH:MI:SS.SS
Self-Check

This answer is correct and complete.

**Code:**

```sql
-- Example SQL queries for each function
SELECT GETDATE();  -- SQL Server
SELECT SYSDATE FROM dual;  -- Oracle
SELECT NOW();  -- MySQL
SELECT now();  -- PostgreSQL
```

