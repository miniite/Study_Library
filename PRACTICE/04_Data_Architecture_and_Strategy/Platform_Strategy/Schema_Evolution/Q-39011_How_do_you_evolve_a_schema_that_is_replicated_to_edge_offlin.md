---
type: "practice"
difficulty: "Expert"
---

# How do you evolve a schema that is replicated to edge/offline-first clients (mobile SQLite, PouchDB) where clients may sync a change made months ago? What conflict and versioning model do you enforce?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Schema Changes Under Long Transactions](<../../../../04_Data_Architecture_and_Strategy/Platform_Strategy/Schema_Evolution/Schema_Changes_Under_Long_Transactions.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When evolving a schema in a distributed environment with offline-first clients like mobile devices using SQLite or PouchDB, it's crucial to maintain consistency across all replicas while managing conflicts efficiently. The key challenge is ensuring that changes made on any client are correctly applied and reconciled when the device eventually syncs back with the server.

To tackle this, we need a robust schema evolution strategy that includes versioning, conflict resolution mechanisms, and incremental migration plans. Let's break down how to approach this problem from first principles.

Firstly, versioning is essential for tracking changes in your database schema. Each major change (like adding or removing tables) should increment the schema version number. This helps clients determine which migrations are needed before syncing with a newer server state.

Secondly, we need to handle conflict resolution effectively. Conflicts can arise when multiple clients modify the same data simultaneously and sync at different times. A common approach is to use optimistic concurrency control (OCC) where each record has a version number that tracks its last update. When conflicts occur, the client can be prompted to resolve them locally or through user intervention.

Now, let's delve into the technical details:

Schema Versioning: Maintain a schema_version field in your database schema and include it in migrations. For example, if you add a new table users_history, increment the version number from 2023 to 2024.

Incremental Migrations: When a client syncs with a server that has a newer schema version, execute incremental migrations. These can be scripts or migration functions that are triggered based on the difference in versions.

SQL
-- Example SQL for an incremental migration
ALTER TABLE users ADD COLUMN last_login TIMESTAMP DEFAULT CURRENT_TIMESTAMP;

Conflict Resolution: Implement OCC by adding a version field to each entity. When a client updates an entity, it includes its current version number. If the server detects that the local version is outdated, it rejects the update and asks for resolution.

Client-Side Logic:

Periodically check if there's a newer schema version.
Execute incremental migrations as needed.
Handle conflicts by either merging changes or resolving them manually through user interaction.

Here’s an example of how you might implement these strategies:

SQL
-- Example SQL for a migration script
ALTER TABLE users ADD COLUMN last_login TIMESTAMP DEFAULT CURRENT_TIMESTAMP;

UPDATE users SET last_login = CURRENT_TIMESTAMP WHERE version < 2024;

By following this approach, we ensure that our schema evolves smoothly while maintaining data integrity and consistency across all clients.

**Comparison:**

| Schema Version | Migration Script | Conflict Resolution |
|----------------|------------------|--------------------|
| 2023           | Add `last_login` column with default value | Increment `version` if outdated; reject update and ask for resolution. |
| 2024           | Update all records to current timestamp | Prompt user to resolve conflicts before syncing. |

**Code:**

```sql
-- Example SQL for adding a new column with a default value
ALTER TABLE users ADD COLUMN last_login TIMESTAMP DEFAULT CURRENT_TIMESTAMP;

-- Example SQL for updating existing records to increment the version number
UPDATE users SET version = 2024 WHERE version < 2024;
```

**Example input / output:**

```text
Input (Client): Update user Alice's last login timestamp to '2023-10-01'.
Output (Server): Rejected update due to outdated version number. Client needs to increment version and retry.

Input (Server): New schema version 2024.
Output (Client): Run migration scripts, update local schema version to 2024.
```

**Data:**

```json
{
  "users": [
    {"id": 1, "name": "Alice", "version": 2023},
    {"id": 2, "name": "Bob", "version": 2023}
  ]
}
```

**Flow:**

1. Yes
2. No
3. Yes
4. No
5. Client Checks Schema Version
6. Version Outdated?
7. Migrate and Sync
8. Sync with Server
9. Update Local Version
10. Server Detects Conflicts
11. Resolve Manually?
12. Notify Client to Resolve
13. Client Re-syncs AfterResolution

