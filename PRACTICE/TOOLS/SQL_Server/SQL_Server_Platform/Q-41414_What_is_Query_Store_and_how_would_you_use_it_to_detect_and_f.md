---
type: "practice"
difficulty: "Deep"
---

# What is Query Store, and how would you use it to detect and force a good plan after a regression?

**Difficulty:** Deep · **Levels:** SDE-1, SDE-1/2 · [↑ Clustered Indexes and Indexed Views](<../../../../TOOLS/SQL_Server/SQL_Server_Platform/Clustered_Indexes_and_Indexed_Views.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

#### Overview
Query Store in SQL Server is a powerful tool that captures query execution information to help diagnose performance issues, especially when dealing with query plans that regress over time. By analyzing historical data, you can identify and force better-performing plans.

#### Triage
The question asks for an explanation of Query Store and how it can be used to detect and force good query plans after a regression. Key points include understanding the purpose of Query Store, identifying when a plan regresses, and implementing techniques to ensure the use of optimal execution plans.

#### First-Principles Solution

Understanding Query Store: SQL Server's Query Store stores metadata about each statement in a database for a configurable period. It captures important information such as query text, plan hash value, execution plan XML, actual execution statistics, and performance metrics like elapsed time and CPU usage.

Identifying Plan Regression:

A regression occurs when the performance of an SQL statement degrades over time.
Use sys.query_store_plan to view historical plans for a specific query or all queries in a database.
Look for increased costs, higher I/O operations, or longer execution times.

Force Good Plan:

Once you identify the problematic plan (typically by its high cost or poor performance metrics), use sp_query_store_force_plan to force SQL Server to recompile and use this specific plan.
Alternatively, manually re-optimize the query using ALTER INDEX REORGANIZE or REBUILD, which might lead to a better execution plan.

Monitor and Maintain:

Continuously monitor the Query Store to ensure that performance issues are caught early.
Periodically review and clean up old plans using sys.query_store_plan and sys.query_store_query_text.

Best Practices:

Enable Query Store for all databases where you have concerns about query performance.
Set appropriate retention periods based on the database size and usage frequency.

#### Common Mistakes/Follow-Ups

Ignoring historical plans can lead to missing valuable insights into past performance issues.
Overuse of sp_query_store_force_plan without understanding the root cause might temporarily improve performance but not address underlying issues.
Not regularly reviewing and cleaning up old, non-performing plans can clutter the Query Store.

#### Closing Summary
Query Store is a vital tool for database administrators and developers to monitor and maintain query performance over time. By leveraging historical execution data, you can identify and force better performing plans, ensuring that your SQL Server environment remains optimized.

Correctness + completeness confirmed.

