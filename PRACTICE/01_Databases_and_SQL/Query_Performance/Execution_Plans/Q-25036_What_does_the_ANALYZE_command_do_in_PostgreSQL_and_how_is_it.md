---
type: "practice"
difficulty: "Intermediate"
---

# What does the ANALYZE command do in PostgreSQL and how is it different from VACUUM?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ Statistics and Sargability](<../../../../01_Databases_and_SQL/Query_Performance/Execution_Plans/Statistics_and_Sargability.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

The ANALYZE command in PostgreSQL is used to update statistics about table data distribution, which are then used by the query planner (the component that decides the execution plan) to optimize queries. It's different from VACUUM, which is primarily used for reclaiming space and updating tuples marked as deleted or updated. In this response, we'll first explain each command in detail, their differences, and scenarios where they are commonly used.

**Code:**

```sql
-- Example of ANALYZE command
ANALYZE sales_data;

-- Example of VACUUM command with options
VACUUM FULL ANALYZE sales_data;
```

