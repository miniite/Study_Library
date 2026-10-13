---
type: "practice"
difficulty: "Intermediate"
---

# A migration idempotency question came up: a re-run of the deploy applied a CREATE INDEX twice and errored. How does the migration tool's Git-tracked history plus its DB tracking table prevent double-apply, and what breaks that guarantee if someone edits an applied file?

**Difficulty:** Intermediate · **Levels:** SDE-1, SDE-1/2 · [↑ Versioning SQL and Migrations](<../../../../TOOLS/Git_and_GitHub/Git_Core/Versioning_SQL_and_Migrations.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the issue of a migration tool applying an CREATE INDEX twice due to idempotency concerns, we need to ensure that migrations are handled both at the Git level and in the database itself. This involves tracking applied migrations using a history table within the database, while also maintaining a record in Git.

#### Triaging the Question

The core question is about ensuring migration idempotency by leveraging Git for version control and a database table to track which migrations have already been applied. The key points are:

Git History: Helps track the sequence of migrations.
Database Tracking Table: Manages the state of each migration in the database.

#### Assumptions
Migrations are stored as scripts or SQL files within Git repositories.
A tracking table in the database keeps a record of which migrations have been applied.
The migration tool checks this tracking table before applying any new migrations.
Step-by-Step Solution

Git History:

Each migration is committed to Git with a unique identifier, typically included in the commit message or file name.
The migration tool uses this history to determine which migrations have been applied and need to be reapplied if necessary.

Database Tracking Table:

A table like migrations exists with columns such as migration_id, applied_at, etc.
Each time a migration is run, the corresponding entry in the migrations table is updated or inserted.

Prevention of Double-Apply:

The tool queries the migrations table before running any migrations to check if they have already been applied.
If an CREATE INDEX command is detected and found to be already present, the migration will not proceed to avoid errors.

Handling Edits in Applied Files:

If someone manually edits a file that has already been applied, it can lead to confusion because Git does not track changes within committed files.
The database tracking table remains unchanged, leading to a discrepancy between what Git thinks is the current state and what the database thinks.

#### Closing Summary

To ensure idempotency, both Git and the database need to be in sync. Git tracks which migrations are available for application, while the database keeps track of their actual status. If someone manually edits an applied file, it can break this guarantee as only the database tracking will reflect the correct state.

#### Common Mistakes/Follow-Ups
Failing to update the migrations table after applying a migration.
Manual editing of committed files without updating the tracking mechanism.
Not using Git tags or branches properly for managing different environments (e.g., development, staging).

**Code:**

```sql
-- Example database schema for migrations tracking

CREATE TABLE IF NOT EXISTS migrations (
    id SERIAL PRIMARY KEY,
    migration_file TEXT NOT NULL UNIQUE,
    applied_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

**Example input / output:**

```text
Input:
- Current migrations in Git: [001_create_users_table.sql, 002_add_index_to_users_email.sql]
- Current `migrations` table: { id: 1, migration_file: "001_create_users_table.sql", applied_at: "2023-09-01T14:30:00Z" }, { id: 2, migration_file: "002_add_index_to_users_email.sql", applied_at: "2023-09-01T15:30:00Z" }

Output:
- Migration tool checks `migrations` table before applying any migrations.
- `CREATE INDEX` command detected and found to be already present, so no action is taken.

Error Scenario (Manual Edit):
- Manually edit "002_add_index_to_users_email.sql" in Git but do not update the `migrations` table.
- Migration tool still checks database and finds it applied, causing confusion or errors when trying to reapply migrations.
```

**Data:**

```json
{
  "migrations": [
    { "id": 1, "migration_file": "001_create_users_table.sql", "applied_at": "2023-09-01T14:30:00Z" },
    { "id": 2, "migration_file": "002_add_index_to_users_email.sql", "applied_at": "2023-09-01T15:30:00Z" }
  ]
}
```

