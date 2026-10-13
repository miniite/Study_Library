---
type: "practice"
difficulty: "Intermediate"
---

# What tools or system views help you find the slowest queries running on a database?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ Diagnosing Query Regressions](<../../../../01_Databases_and_SQL/Query_Performance/Execution_Plans/Diagnosing_Query_Regressions.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When dealing with performance issues in databases, identifying and tuning slow queries is crucial. In this scenario, as a fresher to an intermediate experience level Database Developer, you will need to leverage various tools and system views provided by your database management system (DBMS) to pinpoint the culprits. Let's dive into the key steps and tools that can help you.

Firstly, it’s essential to understand what an execution plan is: It’s a detailed breakdown of how the DBMS interprets and executes a query. Execution plans are critical in identifying where bottlenecks may occur. You will use system views and specific tools to visualize these plans and identify slow queries.

#### Triaging the Question

The question asks for tools or system views that help find the slowest queries on a database. The interviewer is probing your understanding of query performance analysis, execution plan visualization, and how to interpret these visualizations to diagnose and resolve issues.

#### Assumptions
You are familiar with common DBMSs such as MySQL, PostgreSQL, SQL Server, or Oracle.
You have basic knowledge of SQL and database architecture.

#### First-Principles Solution
1. Database Management System-Specific Tools
EXPLAIN/EXPLAIN PLAN: Most DBMSs provide an EXPLAIN statement that allows you to see how the query will be executed without actually executing it.
SQL
EXPLAIN SELECT * FROM users WHERE age > 30;
Execution Plan Visualizers: Some systems, like Oracle’s SQL Trace or PostgreSQL’s pg_fair, can provide visual representations of execution plans. These tools help you understand the steps the DBMS takes to execute a query.
2. System Views
MySQL:
SQL
SHOW PROCESSLIST; -- Lists all current threads and their status
SHOW FULL PROCESSLIST; -- Detailed version of SHOW PROCESSLIST
PostgreSQL:
SQL
SELECT * FROM pg_stat_activity WHERE state = 'active'; -- Displays active queries
SQL Server:
SQL
EXEC sp_who2; -- Lists all active sessions and associated processes
Oracle:
SQL
SELECT * FROM V$SESSION_WAIT; -- Wait events can indicate slow queries
3. Performance Metrics
Execution Time: Use EXPLAIN to get an idea of the execution time.
CPU and IO Usage: Monitor system metrics such as CPU usage and I/O operations using OS tools or DBMS-specific monitoring tools.

#### Closing Summary
and Common Mistakes/Follow-Ups

When tuning queries, ensure you:

Understand the query logic and data structures involved.
Use EXPLAIN to visualize execution plans.
Regularly monitor performance with system views.

#### Cons
ider indexing strategies to optimize query execution.

#### Common mistakes
include not using EXPLAIN, ignoring system view outputs, or failing to consider database schema optimization. Always document your findings and implement changes step-by-step for better maintainability.

If the interviewer probes further:

Ask about specific tools: How would you use EXPLAIN in Oracle compared to PostgreSQL?
Discuss indexing strategies: What factors influence index selection?

**Code:**

```sql
-- Example of using EXPLAIN in MySQL
EXPLAIN SELECT * FROM users WHERE age > 30;
```

