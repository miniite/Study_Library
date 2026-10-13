---
type: "practice"
difficulty: "Basic"
---

# How do you view the execution plan for a query in PostgreSQL using EXPLAIN?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ Reading Execution Plans](<../../../../01_Databases_and_SQL/Query_Performance/Execution_Plans/Reading_Execution_Plans.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To view the execution plan of a query in PostgreSQL, we use the EXPLAIN command. This tool gives us insight into how PostgreSQL plans to execute our queries, which is crucial for optimizing performance and understanding the underlying operations.

The EXPLAIN command provides a text-based representation of the execution plan, detailing each step involved in processing the query. This includes information on the join methods, indexes used, sorts performed, and more.

