---
type: "practice"
difficulty: "Deep"
---

# How do you roll back a schema migration safely, and why are some migrations (like dropping a column) effectively irreversible in practice?

**Difficulty:** Deep · **Levels:** SDE-1, SDE-1/2 · [↑ Schema Migration Risks](<../../../../01_Databases_and_SQL/Database_Internals/DBMS_Concepts/Schema_Migration_Risks.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Rolling back a schema migration is a critical aspect of database development, especially given the potential impact on data integrity and application behavior. In this context, we will discuss how to roll back a schema migration safely and delve into why certain migrations, such as dropping a column, are effectively irreversible.

To begin with, let's consider what a schema migration entails: it involves modifying the structure of a database schema (tables, columns, constraints) to introduce new features or correct issues. Safely rolling back a migration requires careful planning and understanding of both the current state of the database and potential impacts on applications using the data.

The key idea here is to maintain a versioned approach to migrations and have the necessary safeguards in place.

**Code:**

```sql
-- Example of a versioned schema migration table
CREATE TABLE IF NOT EXISTS schema_migrations (
    id INT AUTO_INCREMENT PRIMARY KEY,
    migration_name VARCHAR(255) NOT NULL,
    applied TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

**Example input / output:**

```text
// Example of applying a new migration
INSERT INTO schema_migrations (migration_name) VALUES ('AddColumnForCustomerAddress');

// Example of rolling back the last migration
DELETE FROM schema_migrations WHERE migration_name = 'AddColumnForCustomerAddress';
```

**Flow:**

1. Yes
2. No
3. Apply Migration
4. Is Rollback Required?
5. Rollback Migration
6. Mark Migration as Applied
7. Revert Changes Manually orUsing a Script
8. Update Schema MigrationsTable

