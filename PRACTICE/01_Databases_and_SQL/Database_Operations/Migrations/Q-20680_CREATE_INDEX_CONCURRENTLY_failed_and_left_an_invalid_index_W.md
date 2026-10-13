---
type: "practice"
difficulty: "Expert"
---

# CREATE INDEX CONCURRENTLY failed and left an invalid index. Walk me through detecting it and recovering cleanly.

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-1, SDE-1/2, SDE-2/3 · [↑ Concurrent Index Builds](<../../../../01_Databases_and_SQL/Database_Operations/Migrations/Concurrent_Index_Builds/_index.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When a CREATE INDEX CONCURRENTLY operation fails, it can leave behind a partially created or corrupted index. As a database developer with expertise in migrations and versioning, you must understand how to detect such issues and recover from them cleanly. This involves understanding the lifecycle of concurrent index creation, identifying failure points, and implementing recovery strategies.

#### Triaging the Question

The question is probing your ability to diagnose failures during concurrent index creation, particularly when CREATE INDEX CONCURRENTLY encounters an issue but does not clean up properly. Key assumptions include:

The database is a PostgreSQL instance that supports CREATE INDEX CONCURRENTLY.
You have access to relevant system catalogs and logs.

#### First Principles Solution

Identify the Failure Point:

Check the pg_locks catalog for any locks held by the failing transaction.
Examine the pg_stat_activity view to identify which session is stuck or failed.
Review the PostgreSQL logs for error messages related to the index creation.

Detect Invalid Index:

Use SQL queries to check if the index was partially created:
SQL
SELECT pg_class.relname AS index_name, pg_index.indisready
FROM pg_class
JOIN pg_index ON pg_class.oid = pg_index.indrelid
WHERE index_name LIKE '%your_index_name%';
If indisready is f, the index creation was not successful.

Recover from Failure:

If the index is partially created, you can attempt to finish it or drop and recreate:
SQL
-- Attempt to finish a failing index (if it’s still in progress)
SELECT pg_cancel_backend(pid) FROM pg_stat_activity WHERE query LIKE 'CREATE INDEX CONCURRENTLY%';
If the index creation is stuck, consider dropping and recreating:
SQL
DROP INDEX IF EXISTS your_index_name;
CREATE INDEX CONCURRENTLY your_index_name ON your_table (your_column);

Maintain Version Control:

Ensure versioning in your migration scripts to prevent such issues by testing migrations thoroughly.
Use a rollback plan if the index creation fails.

#### Closing Summary
and Common Mistakes

To recover from a failed CREATE INDEX CONCURRENTLY, you need to identify the issue, detect the invalid index, and then either attempt to finish it or drop and recreate. It's crucial to have robust logging and monitoring in place to quickly diagnose such issues. Common mistakes include not properly checking for partial indexes or failing to clean up after a failed migration.

#### Common Mistakes/Follow-Ups
Mistake: Not understanding the lifecycle of concurrent index creation.

#### Follow-Up
Discuss best practices for versioning migrations and handling failures.
Mistake: Attempting to continue using an invalid index without proper validation.

#### Follow-Up
Explain how to validate the integrity of indexes after recovery.

