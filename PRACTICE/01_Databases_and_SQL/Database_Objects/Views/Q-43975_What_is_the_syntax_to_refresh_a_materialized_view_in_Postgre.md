---
type: "practice"
difficulty: "Deep"
---

# What is the syntax to refresh a materialized view in PostgreSQL, and what does CONCURRENTLY do?

**Difficulty:** Deep · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ Materialized View Refresh Correctness](<../../../../01_Databases_and_SQL/Database_Objects/Views/Materialized_View_Refresh_Correctness.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To refresh a materialized view in PostgreSQL, you use the REFRESH MATERIALIZED VIEW command. The basic syntax is:

SQL
REFRESH MATERIALIZED VIEW [CONcurrently] <view_name>;

The CONCURRENTLY keyword is optional but very important as it allows you to refresh the materialized view without locking the underlying tables, thus allowing concurrent reads and writes.

