---
id: "TOOLS/MySQL/MySQL_Platform/Online_Schema_Changes"
level: 4
type: "concept"
topic: "TOOLS/MySQL/MySQL_Platform"
tools: ["MySQL"]
---

# Online Schema Changes

**L4 Advanced** · [↑ MySQL Platform](<_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** Changing a live MySQL schema safely means choosing the cheapest DDL algorithm (`INSTANT` → `INPLACE` → online-tool copy), splitting a risky change into small backward-compatible steps (expand → backfill → switch → contract), and watching metadata locks, replica lag and rollback options at every step.

### 1. The big picture
Schemas must change while the application keeps running. The danger is not the SQL syntax but the side effects: a table rebuild that takes hours and doubles disk use, a metadata lock that queues every query behind a stuck transaction, a backfill that floods replicas with lag, or a "drop column" that breaks a service nobody remembered. The goal is a change process that is **online** (no long write blocking), **reversible** (until the last step), and **observable**.

The pack also touches the neighbours of migrations: transactions (`COMMIT`/`ROLLBACK`/`SAVEPOINT`, `SELECT … FOR UPDATE`, optimistic versions), soft deletes, `GRANT`/`REVOKE`, triggers vs application logic, and strict SQL modes — all things you have to reason about when a schema evolves. *Data Pipelines Pocket Reference* contributes the full-vs-incremental extraction view: moving or backfilling a table in chunks keyed by an increasing column is the same idea whether you are extracting to a warehouse or copying to a new column.

### 2. Core concepts and vocabulary
- **DDL algorithm** — how `ALTER TABLE` executes: `INSTANT` (metadata only), `INPLACE` (online rebuild or index build, concurrent DML allowed), `COPY` (create new table, copy, swap; blocks writes).
- **`LOCK` clause** — `NONE` (reads+writes allowed), `SHARED` (reads only), `EXCLUSIVE`.
- **Metadata lock (MDL)** — a lock on the table definition held by every statement/transaction using the table; DDL needs an exclusive MDL briefly at start and end.
- **Online schema change tool** — `gh-ost`, `pt-online-schema-change`, Vitess/`spirit`-style tools that build a shadow table, copy rows in chunks, capture ongoing changes (triggers or binlog), then atomically rename.
- **Expand/contract (parallel change)** — add new structure, dual-write/backfill, move readers, then remove old structure.
- **Backfill** — populating a new column/table from existing data, in restartable chunks.
- **Migration tool** — Flyway/Liquibase/Alembic: versioned scripts applied in order, recorded in a history table, so environments converge.
- **Soft delete** — marking rows deleted (`deleted_at`) instead of removing them.
- **Optimistic concurrency** — update guarded by a `version`/`updated_at` check.
- **Semantic drift** — a column's meaning changing over time.

### 3. How it works, step by step
**Choosing the mechanism**
1. Ask the server: `ALTER TABLE t ADD COLUMN c INT NULL, ALGORITHM=INSTANT;` — if it can't do it instantly, it errors instead of silently rebuilding. Try `INPLACE, LOCK=NONE` next.
2. If only `COPY` is possible (e.g., changing a column's charset or type in many cases), use an external online tool.
3. Run in staging with production-sized data to measure time, disk, and replica lag.

**Expand → backfill → switch → contract**
1. *Expand:* add the new nullable column/table/index (online).
2. *Dual write:* deploy app code that writes both old and new (or add a trigger temporarily).
3. *Backfill:* copy old data in primary-key chunks of ~1,000–10,000 rows with pauses; record the last processed key so the job is restartable.
4. *Verify:* compare counts/checksums; read from new in shadow mode.
5. *Switch:* readers move to the new structure; add `NOT NULL`/constraints once fully populated.
6. *Contract:* stop writing the old structure, wait a full release/rollback window, then drop it.

**Online tool mechanics (gh-ost style)**
1. Create `_t_gho` with the new shape.
2. Copy rows in chunks from `t` while tailing the binary log for ongoing DML and applying it to `_t_gho`.
3. Throttle on replica lag/load; allow pause/postpone cut-over.
4. Cut over by an atomic rename swap (brief exclusive MDL).
5. Keep the old table (`_t_del`) until you're sure.

### 4. Worked example
```sql
-- 1. Which algorithm? Fail fast if it can't be instant.
ALTER TABLE orders
  ADD COLUMN shipped_at DATETIME NULL,
  ALGORITHM=INSTANT;                         -- 8.0.12+

-- 2. Adding NOT NULL with a default: also INSTANT in 8.0.12+ (default stored in metadata)
ALTER TABLE orders
  ADD COLUMN priority TINYINT NOT NULL DEFAULT 0,
  ALGORITHM=INSTANT;
-- On 5.7, or when ALGORITHM=INSTANT isn't supported for the change, this rebuilds the
-- whole table: hours of I/O on a huge table, disk for a full copy, replica lag.

-- 3. Restartable, chunked backfill from a JOIN
SET @last := 0;
-- repeat until 0 rows affected:
UPDATE orders o
JOIN   customer c ON c.customer_id = o.customer_id
SET    o.customer_region = c.region
WHERE  o.order_id > @last AND o.order_id <= @last + 5000
  AND  o.customer_region IS NULL;
-- app/script: record @last += 5000; sleep 100–500 ms; watch replica lag

-- 4. Safe constraint after backfill
ALTER TABLE orders MODIFY customer_region VARCHAR(8) NOT NULL, ALGORITHM=INPLACE, LOCK=NONE;
```
A metadata-lock pile-up looks like this: a forgotten open transaction holds a shared MDL on `orders`; your `ALTER` waits for an exclusive MDL; **every new query on `orders` then queues behind the `ALTER`**. Protect yourself:
```sql
SET SESSION lock_wait_timeout = 5;          -- DDL gives up quickly instead of blocking traffic
ALTER TABLE orders ADD INDEX ix_x (x), ALGORITHM=INPLACE, LOCK=NONE;
-- find blockers
SELECT * FROM performance_schema.metadata_locks WHERE object_name = 'orders';
SELECT * FROM information_schema.innodb_trx;   -- long-running transactions
```

**Dropping a "probably unused" column on 200M rows**
1. Search code and ORMs; check `performance_schema.events_statements_summary_by_digest` for digests containing the column over a full business cycle (month-end jobs!).
2. Make it invisible to catch hidden readers (8.0.23+): `ALTER TABLE t ALTER COLUMN c SET INVISIBLE;` — `SELECT *` and unnamed inserts stop seeing it, explicit references still work.
3. Optionally rename to `c__deprecated` in a staging window to flush out failures.
4. Take a logical backup of the column's data, wait a release cycle, then `ALTER TABLE t DROP COLUMN c, ALGORITHM=INPLACE;` (rebuild; or `INSTANT` where supported in 8.0.29+).

**Transactions refresher**
```sql
START TRANSACTION;
INSERT INTO orders ...;
SAVEPOINT after_order;
INSERT INTO order_items ...;      -- fails
ROLLBACK TO SAVEPOINT after_order;   -- undo only items
COMMIT;                              -- persists the order
```
`autocommit=1` (default) makes each statement its own transaction; DDL always commits implicitly and can't be rolled back (8.0 DDL is atomic per statement, but not transactional with surrounding DML).

**Update-if-unchanged flow**
```sql
-- optimistic
UPDATE doc SET body = ?, version = version + 1 WHERE id = ? AND version = ?;
-- ROW_COUNT() = 0 → someone changed it → return HTTP 409

-- pessimistic (short transaction only!)
START TRANSACTION;
SELECT * FROM doc WHERE id = ? FOR UPDATE;
UPDATE doc SET ... WHERE id = ?;
COMMIT;
```
Over HTTP the user holds the form for minutes — never keep `FOR UPDATE` open across requests; use the version column. For PATCH-style partial updates update only provided fields (`SET a = COALESCE(?, a)` or build the `SET` list dynamically) so concurrent edits to other fields aren't overwritten.

**Enabling `ONLY_FULL_GROUP_BY` on a legacy app:** capture offending queries in staging (`sql_mode` enabled; look for error 1055 in logs), fix with `ANY_VALUE()`/aggregates/explicit grouping, enable per-session for new code first, then globally.

### 5. Under the hood
- **`INSTANT` (8.0.12+)** stores the new column definition and default in the data dictionary and in the table's row metadata; existing rows are not touched — they are interpreted with the default when read. Since 8.0.29 columns can be added anywhere, and a table may undergo up to 64 instant changes before an `OPTIMIZE`/rebuild resets the row version.
- **`INPLACE`** may still rebuild the table internally ("rebuild table in place") but allows concurrent DML via a **row log** of changes applied at the end; it needs temp space and takes brief exclusive MDL at prepare and commit. Index adds are done by sorted build.
- **`COPY`** blocks writes (shared lock) for the duration and generates a full redo/binlog stream (on a replica, a single-threaded apply of a long `ALTER` causes lag).
- **Replication.** An in-place `ALTER` executes on each replica separately after the primary finishes (it appears as one long event), so replicas lag by the duration. Online tools copy via many small transactions, giving fine-grained lag control.
- **Triggers vs binlog tailing.** `pt-osc` installs triggers on the old table (write amplification, FK complications); `gh-ost` reads the binlog (no triggers, pausable, but needs row-based binlog and a replica or primary access).
- **`AUTO_INCREMENT`.** Counter is not decremented by rollback or `DELETE`; gaps are normal. `TRUNCATE` resets it. Since 8.0 it is persisted across restart.
- **`DELETE` vs `TRUNCATE` vs `DROP`.** `DELETE` is row-wise, transactional, fires triggers, leaves counter and space; `TRUNCATE` drops and recreates the table (DDL, implicit commit, no triggers, resets auto-increment); `DROP` removes it entirely.
- **Soft deletes** keep rows and indexes bloating; unique constraints must include a `deleted_at` component (or a generated "active key") to allow re-creation of a deleted value.

### 6. Performance and design implications
- **Disk:** plan for ~2× table size during `COPY`/rebuild plus binlogs.
- **Lag:** throttle backfills by `Seconds_Behind_Source`; chunk size so each transaction runs < 1 s.
- **Locks:** short `lock_wait_timeout` for DDL; run during low traffic; kill long transactions first.
- **Order:** build new indexes before switching queries; drop old indexes after.
- **Position columns for drag-and-drop ordering:** use sparse integers (`1000, 2000, …`), fractional/rank strings (LexoRank-style), or a linked list so a reorder updates one row instead of rewriting thousands; renumber occasionally.
- **Triggers or application code?** Triggers guarantee enforcement for all writers but are invisible, hard to test, execute inside the writing transaction, and are not fired by cascades; business rules that change often belong in versioned application code with tests, while invariant enforcement and audit capture suit the database.
- **Semantic drift:** record an explicit `schema_version`/`rule_version` column or valid-from/valid-to dates, and write views that normalize old and new meanings for readers.
- **Permissions:** `GRANT SELECT ON shop.* TO 'app_ro'@'10.%';` / `REVOKE`; least privilege, separate migration user with DDL rights.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `ALTER TABLE … ALGORITHM=INSTANT/INPLACE/COPY, LOCK=NONE` | Force/forbid DDL strategy | Fail instead of surprising rebuilds |
| `lock_wait_timeout` | DDL/MDL wait limit | Before every online `ALTER` |
| `performance_schema.metadata_locks`, `information_schema.innodb_trx` | Find MDL blockers/long transactions | Stuck DDL |
| `gh-ost`, `pt-online-schema-change` | Online table rebuilds | `COPY`-only changes on big tables |
| Flyway/Liquibase | Versioned migrations | Keep environments in sync |
| `ALTER COLUMN … SET INVISIBLE` | Hide column from `SELECT *` | Pre-drop trial |
| `START TRANSACTION/COMMIT/ROLLBACK/SAVEPOINT` | Transaction control | Multi-step data fixes |
| `SELECT … FOR UPDATE` | Row locks | Short read-modify-write |
| `GRANT/REVOKE`, `CREATE USER` | Privileges | Separate app, migration, reporting users |
| `SELECT VERSION();` / `@@version` | Server version | Gate features (INSTANT, CHECK, invisible) |
| `sql_mode`, `sql_safe_updates` | Strictness | Prevent silent bad data |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| `INSTANT` DDL | Change type not supported (reordering, type change) |
| `INPLACE, LOCK=NONE` | Replicas can't absorb long single-threaded apply |
| `gh-ost` | No row-based binlog or tables with FKs/triggers (pt-osc handles triggers differently) |
| `pt-osc` | Very write-heavy tables (trigger overhead) |
| Expand/contract | You can't deploy app and DB in compatible steps |
| Soft delete | Privacy deletion duties (GDPR) or huge churn tables |
| Optimistic locking | Very hot rows with frequent conflict |
| Triggers | Logic that changes often or must be unit-tested heavily |

### 9. Common mistakes
- **Running a blocking `ALTER` at peak** → queue of waiting queries → algorithm clause, short `lock_wait_timeout`, off-peak.
- **Forgetting an open transaction** → DDL stuck on MDL, then everything stuck → check `innodb_trx` first.
- **One giant `UPDATE` backfill** → long locks, undo bloat, replica lag → chunk with a checkpoint.
- **Adding `NOT NULL` before backfill completes** → failures/long rebuild → nullable first, constrain last.
- **Dropping a column in the same deploy as the code change** → rollback impossible → separate by a full release.
- **No down-path** → can't revert → keep old structure until validated.
- **Relying on `DELETE` resetting `AUTO_INCREMENT`** → it doesn't → `TRUNCATE` or design without it.
- **`FOR UPDATE` across HTTP requests** → hours-long locks → optimistic version.
- **Overwriting the entire object on PATCH** → lost updates → field-level updates with version.

### 10. 🎯 Interview kit
**30-second answer:** I try `ALGORITHM=INSTANT`, then `INPLACE, LOCK=NONE`, and for copy-only changes use gh-ost/pt-osc. I split risky changes into expand → backfill in chunks → switch → contract, set a short `lock_wait_timeout`, watch replica lag, and keep the old structure until verified.

**Structure a longer answer:** change type → DDL algorithm → blocking risks (MDL, disk, lag) → staged rollout → backfill design → verification → rollback → cleanup.

**Likely follow-ups:**
- *Risk of adding `NOT NULL DEFAULT` to a huge table?* → pre-8.0.12 a full rebuild; in 8.0 usually instant, but verify algorithm.
- *How do you safely test a migration?* → restore a prod-size copy, run, measure, run the down script, validate with checksums.
- *Flyway/Liquibase role?* → versioned, ordered, checksummed scripts + history table.
- *`FOR UPDATE` vs optimistic?* → pessimistic for short, contended critical sections; optimistic for user edits.
- *`COMMIT` vs `ROLLBACK` vs `SAVEPOINT`?* → persist, undo all, undo to a marker.
- *Prove a column is unused?* → digests + invisible column + staged rename.

**Signals interviewers look for:** you know MDL pile-ups, you design for replicas and rollback, and you avoid big-bang changes.

### 11. Check yourself
- [ ] Explain `INSTANT` vs `INPLACE` vs `COPY` with examples.
- [ ] Describe how an MDL queue forms behind a blocked `ALTER`.
- [ ] Write a restartable chunked backfill using a primary-key watermark.
- [ ] Outline the gh-ost process and its cut-over.
- [ ] Plan removal of a column from a 200M-row table with a rollback path.
- [ ] Contrast `DELETE`, `TRUNCATE`, and `DROP`.
<!-- explainer:end -->

## Implementations (tools)

- [MySQL](<../_index.md>)

## Practice questions

- [Basic] [How do you check what version of MySQL a server is running from the SQL prompt?](<../../../PRACTICE/TOOLS/MySQL/MySQL_Platform/Q-33411_How_do_you_check_what_version_of_MySQL_a_server_is_running_f.md>)
- [Intermediate] [What is a schema migration and how do tools like Flyway or Liquibase keep environments in sync?](<../../../PRACTICE/TOOLS/MySQL/MySQL_Platform/Q-33066_What_is_a_schema_migration_and_how_do_tools_like_Flyway_or_L.md>)
- [Intermediate] [What is the difference between COMMIT, ROLLBACK, and SAVEPOINT?](<../../../PRACTICE/TOOLS/MySQL/MySQL_Platform/Q-33572_What_is_the_difference_between_COMMIT_ROLLBACK_and_SAVEPOINT.md>)
- [Intermediate] [How would you safely drop a column that you *think* is unused on a 200M-row table, and how do you prove nothing reads it before the destructive migration?](<../../../PRACTICE/TOOLS/MySQL/MySQL_Platform/Q-33240_How_would_you_safely_drop_a_column_that_you_think_is_unused_.md>)
- [Deep] [Explain how you would migrate a wide table's rarely-used columns into a separate 1:1 table (vertical split) without breaking existing queries during rollout.](<../../../PRACTICE/TOOLS/MySQL/MySQL_Platform/Q-33478_Explain_how_you_would_migrate_a_wide_table_s_rarely-used_col.md>)
- [Deep] [How would you write a backward-compatible migration to rename a column without breaking a running application?](<../../../PRACTICE/TOOLS/MySQL/MySQL_Platform/Q-33070_How_would_you_write_a_backward-compatible_migration_to_renam.md>)
- [Deep] [How do you plan and execute a zero-downtime column rename that a running application still reads/writes, using the expand-contract migration pattern?](<../../../PRACTICE/TOOLS/MySQL/MySQL_Platform/Q-33143_How_do_you_plan_and_execute_a_zero-downtime_column_rename_th.md>)
- [Deep] [A migration must add a unique constraint but existing data has duplicates — walk through detecting, deciding a merge/dedup policy, and applying it without downtime.](<../../../PRACTICE/TOOLS/MySQL/MySQL_Platform/Q-33295_A_migration_must_add_a_unique_constraint_but_existing_data_h.md>)

## Related concepts

- [Online Schema Changes](<../../../01_Databases_and_SQL/SQL_Language/SQL_Command_Types/Online_Schema_Changes.md>) *(SQL Command Types)*
- [Schema Evolution in Lakehouse Tables](<../../../01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Schema_Evolution_in_Lakehouse_Tables.md>) *(SQL Fundamentals)*
- [Schema Migration Strategy](<../../../01_Databases_and_SQL/Database_Operations/Migrations/Concurrent_Index_Builds/Schema_Migration_Strategy.md>) *(Migrations)*

## References

- *Data Pipelines Pocket Reference* — James Densmore — Data Ingestion Extracting Data › Full or Incremental MySQL Table Extraction
- *Data Pipelines Pocket Reference* — James Densmore — Data Ingestion Loading Data › create a cursor object and execute the COPY
- *Hadoop: The Definitive Guide* — Tom White — HBase › Tip
- *Learning Spark (2nd ed.)* — Jules S. Damji, Brooke Wenig, Tathagata Das, Denny Lee — Spark SQL and DataFrames Interacting with External Data Sources › MySQL
- *Data Algorithms with Spark* — Mahmoud Parsian — **CHAPTER 7** › CHAPTER 7
- *Azure Data Fundamentals* — Michael John Pena — Azure SQL Services and Open Source Options › Azure Database for MySQL: Powering Web Applications

