---
type: "practice"
difficulty: "Deep"
---

# When computing FIRST_VALUE/LAST_VALUE, why can specifying IGNORE NULLS change both correctness and performance, and which engines support it?

**Difficulty:** Deep · **Levels:** SDE-1, SDE-1/2 · [↑ LAG and LEAD](<../../../../01_Databases_and_SQL/SQL_Language/Window_Functions/LAG_and_LEAD.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To understand how IGNORE NULLS impacts FIRST_VALUE and LAST_VALUE in SQL window functions, we need to first grasp their basic functionality and then dive into the nuances introduced by this option. Specifically, we'll explore why it can alter both correctness and performance, and which database engines support it.

TL;DR

Specifying IGNORE NULLS with FIRST_VALUE and LAST_VALUE can change the output of these window functions in ways that affect both correctness and performance. This feature is supported by modern databases like PostgreSQL, SQL Server, and Oracle, but its implementation details vary. Ignoring nulls is useful when dealing with datasets where null values should not influence the results, yet it adds complexity to query optimization.

#### Triage

The question probes your understanding of:

Window Functions: Specifically FIRST_VALUE and LAST_VALUE.
Null Handling in SQL: The implications of treating nulls as special cases.
Query Performance Considerations: How ignoring nulls impacts the efficiency of window function execution.

#### First-Principles Solution
Window Function Basics

FIRST_VALUE and LAST_VALUE are window functions that return the first or last value within a defined partition of rows, respectively. The basic syntax is:

SQL
SELECT ...
FROM table
WINDOW w AS (...);

Within the window specification w, you can define how to partition and order the data.

Ignoring Nulls with IGNORE NULLS

When using IGNORE NULLS, SQL engines handle null values in a special way. For example, if we have:

SQL
SELECT ..., FIRST_VALUE(column) IGNORE NULLS OVER (PARTITION BY partition_col ORDER BY order_col)
FROM table;
Correctness: The function will consider only non-null values when determining the first or last value.
Performance: Depending on the database engine and data distribution, handling nulls can involve additional logic that may impact performance.

#### Performance Considerations

Ignoring nulls involves:

Null Check Overhead: Additional conditional logic to skip over null values.
Row Filtering: Potentially filtering out rows with nulls before applying window functions, which can reduce the amount of data processed but also adds complexity to query optimization.

#### Closing Summary
and Common Mistakes/Follow-Ups

Ignoring nulls is a powerful feature that can improve accuracy in certain scenarios where null values should not influence the result. However, it introduces overhead that can impact performance, especially on large datasets with many nulls. Modern databases like PostgreSQL, SQL Server, and Oracle support this option, but their implementations may differ.

#### Common mistakes
include:

Assuming IGNORE NULLS works uniformly across all engines.
Overlooking the potential for null values to influence query plans and performance.

#### Follow-ups
might involve discussing specific examples or comparing how different engines handle nulls in window functions.

Self-check

This solution covers the core concepts, nuances of handling nulls with window functions, and provides insights into both correctness and performance impacts. It matches a deep understanding level for Database Developers.

