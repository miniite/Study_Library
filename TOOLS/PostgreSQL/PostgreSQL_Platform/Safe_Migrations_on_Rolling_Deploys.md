---
id: "TOOLS/PostgreSQL/PostgreSQL_Platform/Safe_Migrations_on_Rolling_Deploys"
level: 4
type: "concept"
topic: "TOOLS/PostgreSQL/PostgreSQL_Platform"
tools: []
---

# Safe Migrations on Rolling Deploys

**L4 Advanced** · [↑ PostgreSQL Platform](<_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** On a rolling deploy old and new application code run against the same database at the same time, so every schema change must be backward-compatible and lock-light — done as small expand → migrate → contract steps, with `lock_timeout` guarding each DDL statement.

### 1. The big picture
A migration is not just "change the schema, then deploy". In a rolling or blue/green deploy there is a window — seconds to hours — in which version N and version N+1 of the application both talk to one database. If the new schema breaks version N (a dropped column, a renamed table, a new `NOT NULL` without a default), the old pods error out mid-deploy. If the migration takes a strong lock on a hot table, *every* query queues behind it and you have an outage even though the SQL is "correct".

Safe migrations therefore have two independent concerns: **compatibility** (the schema works for both code versions) and **locking** (the DDL doesn't stall production). Migration tools such as Flyway, Liquibase, Alembic or Prisma Migrate give you ordering and bookkeeping — each records which migrations ran in a history table (Flyway's `flyway_schema_history`, Liquibase's `DATABASECHANGELOG`, Alembic's `alembic_version`, Prisma's `_prisma_migrations`) — but none of them makes unsafe SQL safe. Database reliability engineering literature points the same direction: schema changes are a release-management problem, and a lagging replica or lock pile-up is part of the blast radius.

### 2. Core concepts and vocabulary
- **Rolling deploy** — instances are replaced gradually; two code versions coexist.
- **Backward-compatible change** — schema works with old *and* new code.
- **Expand / contract (parallel change)** — add the new structure first (expand), move code and data, remove the old structure last (contract).
- **Lock queue** — a waiting `ACCESS EXCLUSIVE` request blocks all later queries on that table, even simple `SELECT`s.
- **`lock_timeout`** — maximum time a statement waits to *acquire* a lock before failing.
- **`statement_timeout`** — maximum time a statement may *run*.
- **Table rewrite** — DDL that rewrites every row (slow, holds a strong lock).
- **`NOT VALID` constraint** — a constraint enforced for new writes but not yet verified against existing rows.
- **`CREATE INDEX CONCURRENTLY`** — builds an index without blocking writes (cannot run in a transaction block).
- **Backfill** — populating a new column for existing rows, in batches.
- **Prepared statement cache** — plans/column lists cached per connection or driver.

### 3. How it works, step by step
**Why a schema change is dangerous:** most `ALTER TABLE` forms need `ACCESS EXCLUSIVE`. The statement waits for all current users of the table to finish; while waiting it sits in the queue and **blocks every new query**. A single long-running transaction ahead of it turns a 1 ms change into a minutes-long outage.

**The safe pattern, for a new required column:**
1. *Expand:* `ALTER TABLE t ADD COLUMN c …` nullable (or with a constant default — see below). Old code ignores it.
2. *Deploy code* that writes both old and new columns (or writes new and reads with fallback).
3. *Backfill* existing rows in batches (e.g. 10k rows/transaction), throttled for replica lag.
4. *Enforce:* add `CHECK (c IS NOT NULL) NOT VALID`, then `VALIDATE CONSTRAINT`, then `SET NOT NULL` (PG 12+ uses the validated check to skip the scan), then drop the helper check.
5. *Deploy code* that stops using the old structure.
6. *Contract:* drop the old column/table in a later release.

| Change | Safe form | Dangerous form |
|---|---|---|
| Add column | `ADD COLUMN c int` (nullable), or `DEFAULT <constant>` on PG 11+ | `DEFAULT now()`/volatile on old PG — rewrites table; `NOT NULL` without default |
| Add `NOT NULL` | `CHECK … NOT VALID` → `VALIDATE` → `SET NOT NULL` | direct `SET NOT NULL` on big table (full scan under strong lock) |
| Add index | `CREATE INDEX CONCURRENTLY` | plain `CREATE INDEX` (blocks writes) |
| Add FK | `ADD CONSTRAINT … NOT VALID` → `VALIDATE CONSTRAINT` | direct add (scan + locks both tables) |
| Rename column | add new + dual-write + drop old | `RENAME COLUMN` (old code breaks instantly) |
| Change type | new column + backfill + swap | `ALTER COLUMN TYPE` (rewrite, strong lock) |
| Drop column | stop code using it first, then drop | drop while old pods still select it |

### 4. Worked example
Goal: add `orders.currency`, required, default `'USD'`, on a 500M-row table under rolling deploys.

```sql
-- Release 1 migration (fast, safe)
SET lock_timeout = '3s';          -- fail fast instead of piling up the queue
SET statement_timeout = '30s';
ALTER TABLE orders ADD COLUMN currency text;              -- metadata-only
ALTER TABLE orders ALTER COLUMN currency SET DEFAULT 'USD';  -- new rows get it
-- Existing rows stay NULL; old code does not know the column at all.
```
```sql
-- Backfill job (repeat until 0 rows)
UPDATE orders SET currency = 'USD'
WHERE id IN (SELECT id FROM orders WHERE currency IS NULL LIMIT 10000);
-- Commit each batch; sleep if replay_lag grows; then VACUUM (ANALYZE) orders;
```
```sql
-- Release 2 migration
ALTER TABLE orders ADD CONSTRAINT orders_currency_nn
  CHECK (currency IS NOT NULL) NOT VALID;                 -- instant
ALTER TABLE orders VALIDATE CONSTRAINT orders_currency_nn; -- scans, but only SHARE UPDATE EXCLUSIVE
ALTER TABLE orders ALTER COLUMN currency SET NOT NULL;    -- uses the check, no full scan
ALTER TABLE orders DROP CONSTRAINT orders_currency_nn;
```
What to watch: in `pg_stat_activity` look for the DDL in state `active` with `wait_event_type = Lock`; `pg_locks` with `granted = false` shows the queue. If `lock_timeout` fires, the migration fails harmlessly — retry later or find the long transaction (`xact_start` ancient) blocking it.

**Why changing a default is a trap:** `ALTER COLUMN … SET DEFAULT` is metadata-only and safe for *future inserts*, but it does not touch existing rows and old code that omits the column will now get the new default — verify that is acceptable. Likewise, code that lists columns explicitly keeps working when a column is added; code that does `SELECT *` into a positional struct may not.

### 5. Under the hood
- **Lock levels:** `ADD COLUMN` (nullable or constant default, PG 11+) takes `ACCESS EXCLUSIVE` briefly but needs no rewrite — the default is stored in the catalog (`attmissingval`) and applied on read until rows are rewritten. A *volatile* default (e.g. `random()`, `clock_timestamp()`) forces a rewrite. `CREATE INDEX CONCURRENTLY` uses `SHARE UPDATE EXCLUSIVE` and two table scans, waiting for older transactions between phases; failure leaves an `INVALID` index you must drop.
- **`NOT VALID` / `VALIDATE`:** `NOT VALID` skips the existing-row check, taking only a short lock; `VALIDATE CONSTRAINT` scans under `SHARE UPDATE EXCLUSIVE`, which does not block ordinary reads/writes. For foreign keys it also needs a `ROW SHARE` lock on the referenced table.
- **`SET NOT NULL`:** since PG 12 it is skipped-scan if a valid `CHECK (col IS NOT NULL)` already exists.
- **Prepared statements and pools:** a cached plan that was built against the old column list returns the error `cached plan must not change result type` after a column is added/altered (`SELECT *` or wide result descriptors). The server invalidates plans, but a driver that cached the *result shape* may not recover. Fixes: avoid `SELECT *`, rely on the driver's re-prepare, cycle the pool (`RECONNECT`/`DISCARD ALL`, rolling restart) after the migration, and keep pool `server_lifetime` finite.
- **Replication:** DDL is replayed on replicas under the same locks; a long `ACCESS EXCLUSIVE` on the primary stalls replica queries and creates lag. Logical replication does *not* replicate DDL — apply it manually on subscribers first/last depending on direction.
- **Transactional DDL:** most PostgreSQL DDL is transactional, which makes multi-step migrations atomic — but `CONCURRENTLY` operations cannot be inside a transaction, so tools need a "no transaction" mode for those files.

### 6. Performance and design implications
- Always `SET lock_timeout` (2–5 s) in migration sessions, and retry with backoff. A failed migration is cheap; a lock pile-up is an outage.
- Run migrations separately from application startup. If every pod runs migrations on boot, 20 pods race for locks and history rows — use one job, with advisory lock or the tool's lock table.
- Check for long transactions beforehand (`SELECT pid, xact_start FROM pg_stat_activity WHERE xact_start < now() - interval '1 min'`).
- Throttle backfills by replica lag and by autovacuum health — mass updates create bloat.
- Keep migrations small and idempotent (`IF NOT EXISTS`) so reruns after a timeout are safe.
- Test migrations against production-sized data; a 1-second change on a dev DB can be a 1-hour rewrite.
- JSONB: `jsonb_set(col, '{a,b}', '"x"')` returns a new value, but the whole document is still rewritten as a new row version (and TOAST entry) — a "partial" update is not cheaper on disk, only more convenient.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `SET lock_timeout = '3s'` | Abort if lock not acquired | Every migration session |
| `SET statement_timeout` | Abort long statements | Backfills, validations |
| `ADD COLUMN … ` + `SET DEFAULT` | Metadata-only column add | Expand step |
| `ADD CONSTRAINT … NOT VALID` / `VALIDATE CONSTRAINT` | Online constraint adding | CHECK, FK |
| `CREATE INDEX CONCURRENTLY` / `DROP INDEX CONCURRENTLY` | No write blocking | Index changes |
| `pg_stat_activity`, `pg_locks`, `pg_blocking_pids()` | See who blocks whom | Diagnosing stuck DDL |
| `pg_advisory_lock(key)` | Serialize migration runners | Multi-pod deploys |
| `DISCARD ALL` / pool reset | Clear session state and prepared statements | After schema changes |
| Flyway/Liquibase/Alembic history tables | Track applied versions and checksums | Auditing, drift detection |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| Expand/contract over several releases | A hotfix needs one-shot change — then take a maintenance window |
| `NOT VALID` + `VALIDATE` for big tables | Table is tiny — plain DDL is fine and simpler |
| Dual-write + backfill for renames/type changes | You cannot control all writers (batch jobs, other services) — contract only after verifying none use the old column |
| `CONCURRENTLY` builds | Disk or IO is tight — two scans and extra WAL; check for invalid leftovers |
| Online schema-change tools (e.g. `pg_repack`, logical replication swaps) for rewrites | Team lacks runbook; simpler to schedule downtime |

### 9. Common mistakes
- Renaming or dropping a column in the same release as the code change → old pods 500 during rollout → split across releases.
- Adding `NOT NULL` with no default in one step → old code's inserts fail → expand, backfill, then enforce.
- No `lock_timeout` → DDL queues behind a long transaction and blocks all traffic → set it and retry.
- Plain `CREATE INDEX` on a hot table → writes blocked → `CONCURRENTLY`.
- Leaving an `INVALID` index after a failed concurrent build → wasted writes → drop and retry.
- Running migrations in app startup → race/duplicates → a single migration job.
- Not resetting pooled connections → stale prepared statements → rolling restart or `DISCARD ALL`.

### 10. 🎯 Interview kit
**30-second answer:** During a rolling deploy old and new code share the database, so every migration must be backward-compatible and lock-light. I use expand → backfill → enforce → contract, set `lock_timeout`, build indexes `CONCURRENTLY`, add constraints `NOT VALID` then `VALIDATE`, and never rename or drop in the same release as the code that stops using it.

**Structure a longer answer:** state the two risks (compatibility, locking) → show expand/contract timeline → mention specific PostgreSQL features (constant defaults, `NOT VALID`, `CONCURRENTLY`) → operational guardrails (timeouts, long-transaction check, replica lag, pool reset) → example of a column rename.

**Likely follow-ups:**
- *How do migration tools know what has run?* → a history table with version/checksum rows, applied in order.
- *Is adding a column with a default safe?* → yes for constant defaults on PG 11+, no for volatile ones.
- *Why can a trivial `ALTER` cause an outage?* → lock queue behind a long transaction.
- *Cached prepared statement errors after a column add?* → plan/result shape mismatch; reset pool, avoid `SELECT *`.
- *How would you change a column's type?* → new column, dual write, backfill, swap, drop.

**Signals interviewers look for:** you reason about two code versions at once, you know which DDL needs which lock, and you have operational reflexes (timeouts, retries, monitoring).

### 11. Check yourself
- [ ] Explain why an `ALTER TABLE` waiting for a lock can block unrelated `SELECT`s.
- [ ] Lay out expand/contract steps for renaming a column across three releases.
- [ ] Write the `NOT VALID` → `VALIDATE` → `SET NOT NULL` sequence.
- [ ] Say when adding a default requires a table rewrite.
- [ ] Describe how to avoid prepared-statement errors after a schema change.
- [ ] List what to check before running a migration on a busy primary.
<!-- explainer:end -->

## Practice questions

- [Basic] [How do you add a new column to an existing table and set a default value for it?](<../../../PRACTICE/TOOLS/PostgreSQL/PostgreSQL_Platform/Q-36144_How_do_you_add_a_new_column_to_an_existing_table_and_set_a_d.md>)
- [Intermediate] [Your connection pool holds long-lived sessions and a schema change (new column) isn't seen by cached prepared statements, causing errors; explain why and how you'd invalidate or reset the pool safely.](<../../../PRACTICE/TOOLS/PostgreSQL/PostgreSQL_Platform/Q-36271_Your_connection_pool_holds_long-lived_sessions_and_a_schema_.md>)
- [Deep] [An INSERT into a range-partitioned table fails because no partition covers the value; how does a DEFAULT partition catch it, why is a growing default partition dangerous, and how do you automate partition creation?](<../../../PRACTICE/TOOLS/PostgreSQL/PostgreSQL_Platform/Q-36347_An_INSERT_into_a_range-partitioned_table_fails_because_no_pa.md>)
- [Deep] [What is a database migration and how do you handle schema changes safely in production?](<../../../PRACTICE/TOOLS/PostgreSQL/PostgreSQL_Platform/Q-36382_What_is_a_database_migration_and_how_do_you_handle_schema_ch.md>)
- [Deep] [Your Prisma/TypeORM migration succeeded in staging but locked prod; how do you review generated migration SQL for dangerous locks before it ships?](<../../../PRACTICE/TOOLS/PostgreSQL/PostgreSQL_Platform/Q-36006_Your_Prisma_TypeORM_migration_succeeded_in_staging_but_locke.md>)
- [Expert] [Why is adding a column with a volatile default historically expensive, and what changed to make it cheap in modern PostgreSQL?](<../../../PRACTICE/TOOLS/PostgreSQL/PostgreSQL_Platform/Q-35916_Why_is_adding_a_column_with_a_volatile_default_historically_.md>)
- [Expert] [What lock does a plain ALTER TABLE ... ADD COLUMN take, and why does statement_timeout / lock_timeout matter during migrations?](<../../../PRACTICE/TOOLS/PostgreSQL/PostgreSQL_Platform/Q-35920_What_lock_does_a_plain_ALTER_TABLE_ADD_COLUMN_take_and_why_d.md>)
- [Expert] [Why can adding a NOT NULL column with a default lock a large table, and how do you avoid it?](<../../../PRACTICE/TOOLS/PostgreSQL/PostgreSQL_Platform/Q-36383_Why_can_adding_a_NOT_NULL_column_with_a_default_lock_a_large.md>)

## Related concepts

- [Concurrent Index Builds](<../../../01_Databases_and_SQL/Database_Operations/Migrations/Concurrent_Index_Builds/_index.md>) *(Migrations)*
- [Online Schema Changes](<../../../01_Databases_and_SQL/SQL_Language/SQL_Command_Types/Online_Schema_Changes.md>) *(SQL Command Types)*
- [Schema Evolution in Lakehouse Tables](<../../../01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Schema_Evolution_in_Lakehouse_Tables.md>) *(SQL Fundamentals)*

## References

- *Data Contracts: Developing Production-Grade Pipelines at Scale* — Chad Sanderson, Mark Freeman — Implementing Data Contracts › Fix Database Migration Until All Checks Pass
- *Building Data Integration Solutions* — Jay Borthen — Creating a PostgreSQL Database in the Rocky Linux AWS EC2 Instance › Create the postgres user and add to the super users group
- *Delta Lake: Up and Running* — Bennie Haelen, Dan Davis — Basic Operations on Delta Tables › In this Create Table, you do NOT specify a location, so you are\

