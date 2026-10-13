---
type: "practice"
difficulty: "Deep"
---

# You need to prove which migration version is actually applied to a production database during an incident when all you have is the schema-history tracking table and the Git log. Walk through correlating the two and what can make them disagree.

**Difficulty:** Deep · **Levels:** SDE-1, SDE-1/2 · [↑ Versioning SQL and Migrations](<../../../../TOOLS/Git_and_GitHub/Git_Core/Versioning_SQL_and_Migrations.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To resolve the discrepancy between the migration version in the Git log and the one applied to the production database, we need to understand how schema-history tracking works and correlate it with the Git commit information. The goal is to identify which migration was actually run during an incident when only a schema-history table and Git logs are available.

#### Triage
Understanding the Question

The question involves correlating two sources of truth—schema-history tracking in a database and Git logs—to determine which migration version was applied correctly or incorrectly. This requires understanding how migrations are recorded, how they impact the schema, and potential discrepancies that can arise during application or rollback processes.

Key Assumptions:
We have a schema_history table that records the schema changes.
We have a git_log with commit messages and timestamps corresponding to migration scripts.
Migrations are consistent in structure: each migration script updates a specific part of the database schema.
Correlating Schema History and Git Logs
Schema History Table:
The schema_history table typically records entries like migration_id, applied_at, and possibly a description of changes.
Git Log Analysis:
Review commit messages for migration scripts, noting the version or name of migrations applied.
Step-by-Step Solution

Identify Migrations in Git Logs:

SH
# Example command to grep relevant commits from git log
git log --grep="migrate"

This will list all commits that mention migration scripts, providing us with the names and possibly versions of migrations applied.

Map Migrations to Schema History:

For each migration identified in Git logs, check if it exists in schema_history.
SQL
SELECT * FROM schema_history WHERE migration_id IN (12345, 67890);
This query will return the actual timestamps and descriptions of these migrations.

Analyze Timestamps:

Compare the timestamps from Git logs with those in schema_history.
Discrepancies may indicate timing issues or misapplied migrations.

Check for Unaccounted Migrations:

Look for any migrations that appear in git_log but are missing in schema_history. These could be potential rollbacks or incomplete applications.
SQL
SELECT migration_id FROM git_log WHERE migration_id NOT IN (SELECT migration_id FROM schema_history);

Verify Migration Version and Content:

For migrations that appear both in Git logs and schema history, verify their content to ensure they match the applied changes.
SH
cat /path/to/migration/12345.sql | diff - <(mysql -h localhost -u root -p test_db -e "SELECT * FROM table_name")

#### Common Mistakes
and Follow-Ups
Ignoring Rollback Logs: Sometimes migrations are rolled back, which may not be reflected in the schema history if a full rollback is not performed.
Manual Interventions: Human errors or manual changes might bypass automated migration processes.
Partial Migrations: Incomplete scripts that only partially update the database can cause discrepancies.

#### Conclusion
and Next Steps

By systematically correlating the schema_history with Git logs, we can pinpoint which migrations were actually applied. If there are mismatches, further investigation into rollback practices or human interventions is necessary to ensure data consistency and reliability.

**Comparison:**

| Migration ID | Applied At |
|--------------|------------|
| 12345        | 2023-10-01 |
| Migration ID |
|--------------|
| 67890        |

**Code:**

```
SH
Copy
# Example command to grep relevant commits from git log
git log --grep="migrate"

# SQL query to fetch schema history entries for specific migrations
SELECT * FROM schema_history WHERE migration_id IN (12345, 67890);

# Check for unaccounted migrations in Git logs
SELECT migration_id FROM git_log WHERE migration_id NOT IN (SELECT migration_id FROM schema_history);
Migration ID	Applied At	Description
12345	2023-10-01	User table create
67890	2023-10-02	Product table update
```

**Data:**

```json
{
  "git_log": [
    { "commit_id": "abc123", "message": "Apply migration: 12345" },
    { "commit_id": "def456", "message": "Rollback migration: 12345" }
  ],
  "schema_history": [
    { "migration_id": 12345, "applied_at": "2023-10-01" },
    { "migration_id": 67890, "applied_at": "2023-10-02" }
  ]
}
```

**Flow:**

1. Identify Migrations in Git Logs
2. Map to Schema History
3. Analyze Timestamps
4. Check UnaccountedMigrations
5. Verify Migration Version andContent
6. Further Investigation

