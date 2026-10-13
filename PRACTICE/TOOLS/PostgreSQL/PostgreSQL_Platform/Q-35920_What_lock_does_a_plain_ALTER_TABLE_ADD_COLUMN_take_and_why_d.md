---
type: "practice"
difficulty: "Expert"
---

# What lock does a plain ALTER TABLE ... ADD COLUMN take, and why does statement_timeout / lock_timeout matter during migrations?

**Difficulty:** Expert · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Safe Migrations on Rolling Deploys](<../../../../TOOLS/PostgreSQL/PostgreSQL_Platform/Safe_Migrations_on_Rolling_Deploys.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When adding a new column to an existing table in PostgreSQL using ALTER TABLE ... ADD COLUMN, it typically acquires an ACCESS EXCLUSIVE lock on that table. This is because altering a table structure is considered a major operation, and the database needs to ensure no concurrent modifications can occur during this time.

However, statement timeouts (statement_timeout) and lock timeouts (lock_timeout) are crucial considerations in migration scenarios. These parameters determine how long PostgreSQL will wait for locks to be released before timing out, which directly impacts the overall success of the migration process.

**Code:**

```sql
-- Example query with lock timeout set
ALTER TABLE my_table SET (autovacuum_vacuum_scale_factor = 0);
ALTER TABLE my_table ADD COLUMN new_column datatype WITH OPTIONS;
```

