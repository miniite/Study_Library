---
type: "practice"
difficulty: "Intermediate"
---

# How do you use pg_stat_statements (or equivalent) to find the queries actually hurting your database health?

**Difficulty:** Intermediate · **Levels:** SDE-1, SDE-1/2 · [↑ Percentiles vs Averages](<../../../../01_Databases_and_SQL/Database_Operations/Monitoring/Percentiles_vs_Averages.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To effectively use pg_stat_statements to identify queries that are detrimental to your PostgreSQL database's performance and health, you need to understand its capabilities and limitations, set up monitoring correctly, analyze the collected data, and then take appropriate actions based on insights gained. Here’s a step-by-step guide tailored for an intermediate-level Database Developer.

First, pg_stat_statements provides detailed statistics about all queries executed by the database, allowing us to identify slow or frequently run queries that could be causing performance issues. It works by intercepting query execution and storing relevant information in a catalog table. However, it has some limitations such as increased CPU usage and memory consumption due to its overhead.

**Code:**

```sql
-- Enable pg_stat_statements
CREATE EXTENSION IF NOT EXISTS pg_stat_statements;
```

