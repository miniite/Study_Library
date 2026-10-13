---
type: "practice"
difficulty: "Intermediate"
---

# Explain the difference between GRANT on a schema and GRANT on all tables in a schema, and how default privileges close that gap.

**Difficulty:** Intermediate · **Levels:** Architect/Principal, SDE-1, SDE-1/2, SDE-2/3 · [↑ Column-level Access](<../../../../01_Databases_and_SQL/Database_Operations/Security_and_Access/Column-level_Access/_index.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In database security and access control, GRANT commands are used to manage permissions at various levels of granularity. When discussing GRANT on a schema versus GRANT on all tables within that schema, we're exploring the nuances of privilege management in a PostgreSQL or similar database system. The key difference lies in the scope and inheritance of privileges.

The core idea here is to understand how these different grants affect user permissions, especially in terms of default privileges and their implications for security and administrative convenience.

**Code:**

```sql
-- Example schema grant on all tables
GRANT SELECT, INSERT ON ALL TABLES IN SCHEMA public TO user1;

-- Schema-level grant
GRANT USAGE ON SCHEMA public TO user1;
```

