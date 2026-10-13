---
type: "practice"
difficulty: "Deep"
---

# Why does adding a column with a volatile default (like now()) rewrite the whole table in older Postgres but a constant default does not?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1, SDE-1/2, SDE-2/3 · [↑ Concurrent Index Builds](<../../../../01_Databases_and_SQL/Database_Operations/Migrations/Concurrent_Index_Builds/_index.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When dealing with database migrations and versioning, understanding how PostgreSQL handles default values is crucial. The key difference between adding columns with volatile defaults (like now()) versus constant defaults lies in the way PostgreSQL processes these operations internally.

To understand why a column with a volatile default like now() rewrites the entire table, while a constant default does not, we need to delve into how PostgreSQL manages constraints and updates during schema changes. Let's break this down from first principles.

**Code:**

```

-- Example of adding a column with now() as default value ALTER TABLE users ADD COLUMN created_at TIMESTAMP DEFAULT NOW();

CODE
Copy

```sql
-- Example of adding a column with a constant default value
ALTER TABLE users ADD COLUMN is_active BOOLEAN DEFAULT FALSE;
```

