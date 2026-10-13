---
id: "01_Databases_and_SQL/Database_Operations/Migrations/Concurrent_Index_Builds"
level: 4
type: "concept"
topic: "01_Databases_and_SQL/Database_Operations/Migrations"
tools: ["PostgreSQL"]
---

# Concurrent Index Builds

**L4 Advanced** · [↑ Migrations](<../_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** Online and concurrent DDL lets you build an index or add a constraint on a busy table without blocking writes, by doing the slow work in phases and holding strong locks only for brief moments.

### 1. The big picture
A plain `CREATE INDEX` must read the whole table, sort the keys, and write the index. While that happens, writers are either blocked or the index would miss their changes. On a table with hundreds of millions of rows this means minutes to hours of blocked writes, which in production means an outage.

Every major engine therefore offers a way to build online: SQL Server's `ONLINE = ON`, PostgreSQL's `CREATE INDEX CONCURRENTLY`. The same idea (do it in phases, keep strong locks short, validate later) also applies to constraints and column additions. This lesson explains the pattern and the engine-specific caveats. It sits inside the Migrations block with Schema Migration Strategy (the process around changes) and Migrating to Partitioned Tables (a larger change that uses these techniques).

### 2. Core concepts and vocabulary
- **Online DDL** — a schema operation that lets concurrent reads and writes continue.
- **Lock queue** — locks are granted in order; one waiting strong lock makes everything behind it wait too.
- **`ACCESS EXCLUSIVE` (PostgreSQL) / `Sch-M` (SQL Server)** — the strongest table lock; blocks all other access.
- **`CREATE INDEX CONCURRENTLY`** — PostgreSQL's non-blocking build; takes a weaker lock and scans the table more than once.
- **Invalid index** — an index left behind by a failed concurrent build; it is maintained by writes but not used by queries.
- **`NOT VALID` constraint** — a constraint enforced for new writes immediately but not yet proven for existing rows.
- **`VALIDATE CONSTRAINT`** — checks existing rows later using a weaker lock.
- **Table rewrite** — the engine physically rebuilds every row; very expensive on large tables.
- **Backfill** — filling a new column for old rows, in batches.

### 3. How it works, step by step
**Online index build (general model):**
1. Take a short, strong schema lock; register the new index so writers start maintaining it.
2. Scan the base table and build the index from a snapshot while normal traffic continues.
3. Apply the changes that writers made during the scan (SQL Server uses a row-version / side structure; PostgreSQL does a second scan and waits for older transactions).
4. Take another short lock to mark the index ready and swap it in.

**PostgreSQL specifics:** it waits for all transactions that could see the old state, so a long-running open transaction stalls the build. Because it must be able to commit between phases, it cannot run inside a transaction block.

**Constraints without a long lock (PostgreSQL):**
1. `ALTER TABLE ... ADD CONSTRAINT ... CHECK (...) NOT VALID;` — instant; new and updated rows are checked.
2. Clean up any existing violating rows.
3. `ALTER TABLE ... VALIDATE CONSTRAINT ...;` — scans the table but only takes a lock that still permits reads and writes.

The same two-step works for foreign keys (`ADD FOREIGN KEY ... NOT VALID`, then `VALIDATE`).

**`NOT NULL` on an existing column with NULLs:** add `CHECK (col IS NOT NULL) NOT VALID`, backfill the NULLs, validate, then set `NOT NULL` (recent PostgreSQL versions use the validated check to skip the full scan), then drop the check.

### 4. Worked example
```sql
-- SQL Server: build without blocking writers (Enterprise features/edition rules apply)
CREATE NONCLUSTERED INDEX ix_orders_customer_date
    ON dbo.orders (customer_id, order_date)
    WITH (ONLINE = ON, MAXDOP = 4, SORT_IN_TEMPDB = ON);

-- PostgreSQL: concurrent build, run OUTSIDE a transaction
CREATE INDEX CONCURRENTLY ix_orders_customer_date
    ON orders (customer_id, order_date);

-- Find an invalid index after a failure (PostgreSQL)
SELECT c.relname
FROM pg_index i JOIN pg_class c ON c.oid = i.indexrelid
WHERE NOT i.indisvalid;

DROP INDEX CONCURRENTLY ix_orders_customer_date;   -- then retry
```
What to look at: while the build runs, writes on `orders` keep succeeding; check `pg_stat_progress_create_index` (PostgreSQL) or blocked sessions in `sys.dm_exec_requests` (SQL Server). If the PostgreSQL command is cancelled or hits a deadlock/unique violation, an *invalid* index remains and still slows writes, so drop it and rebuild.

### 5. Under the hood
- **Why not in a transaction block (PostgreSQL):** the build runs as several transactions with waits for other transactions in between. A surrounding transaction would keep one snapshot and one set of locks for the whole time, making the waits impossible. Migration tools therefore need a way to mark a script "no transaction", and such a script should contain only that one statement so a failure cannot leave a half-applied multi-statement migration.
- **Cost:** concurrent builds take roughly two table scans and more total work than a plain build, and they use more I/O and CPU, so they are slower but non-blocking.
- **Volatile vs constant defaults:** in older PostgreSQL, adding a column with *any* default rewrote the table; from version 11, a constant default is stored as metadata and no rewrite occurs, while a volatile default such as `now()` or `random()` still forces a rewrite. For a large table, add the column nullable, backfill in batches, then add the constraint.
- **Lock queue effect:** an `ALTER TABLE` that waits behind a long transaction blocks all later queries, even plain `SELECT`, because they queue behind it. Set `lock_timeout` so a migration fails fast instead of stalling the table.

### 6. Performance and design implications
- Online builds trade duration for availability; schedule them off-peak anyway, because they compete for I/O and for log/WAL.
- Watch log growth and replication lag while the build runs.
- Keep long-running transactions out of the way; they delay the final phases.
- Backfills must be chunked (thousands of rows per batch with a pause) to avoid long transactions, bloat and lag.
- Measure: progress views, blocked-process queries, lock waits, replication delay.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `WITH (ONLINE = ON)` | SQL Server online index build | Large tables with live writes |
| `RESUMABLE = ON` | SQL Server: pause/resume a build | Long builds that may be interrupted |
| `CREATE INDEX CONCURRENTLY` | PostgreSQL non-blocking build | Production tables |
| `DROP INDEX CONCURRENTLY` | Drop without blocking | Removing old or invalid indexes |
| `NOT VALID` / `VALIDATE CONSTRAINT` | Two-phase constraint add | Checks and foreign keys on big tables |
| `lock_timeout` | Abort if a lock is not granted in time | Every production DDL |
| `pg_stat_progress_create_index` | Build progress | While building |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| Online / concurrent build | Table is small or idle; plain build is faster and simpler |
| `NOT VALID` then `VALIDATE` | You need the guarantee immediately for existing rows |
| Add column nullable, then backfill | Tiny tables where a single `ALTER` is fine |
| `lock_timeout` plus retry | Unbounded waiting is acceptable (rare) |

### 9. Common mistakes
- Running a concurrent build inside the migration tool's transaction → error → configure the script as non-transactional.
- Ignoring a failed build → invalid index keeps costing write time → detect with `indisvalid` and drop it.
- Starting DDL while a long transaction is open → everything queues behind it → check for long transactions and set `lock_timeout`.
- One giant `UPDATE` backfill → WAL/log explosion and replication lag → batch and pace.
- Assuming a default value is free → table rewrite on old versions or volatile defaults → check the version and the default's volatility.

### 10. 🎯 Interview kit
**30-second answer:** Use the engine's online path: `ONLINE = ON` in SQL Server or `CREATE INDEX CONCURRENTLY` in PostgreSQL. It builds in phases with only brief strong locks, at the cost of more time and resources. For constraints, add them `NOT VALID` and validate separately.

**Structure a longer answer:** clarify engine, size and traffic → mechanism (phases, locks, catching up with changes) → trade-offs (slower, needs cleanup if it fails, no transaction block) → scenario: adding a foreign key to a 500M-row table.

**Likely follow-ups:**
- *Why not in a transaction?* → Multiple transactions and waits are required.
- *Build failed, now what?* → Find invalid indexes, drop concurrently, retry.
- *Why does one blocked ALTER stall reads?* → Lock queue.
- *Add NOT NULL without a long lock?* → Check constraint `NOT VALID`, backfill, validate, then set.

**Signals interviewers look for:** lock awareness, failure handling, knowledge of version differences, and chunked backfills.

### 11. Check yourself
- [ ] Draw the phases of an online index build and where locks are taken.
- [ ] Explain why `CREATE INDEX CONCURRENTLY` refuses to run in a transaction block.
- [ ] Write the sequence to add a validated foreign key with minimal locking.
- [ ] Detect and remove an invalid index.
- [ ] Explain the lock-queue effect and the role of `lock_timeout`.
<!-- explainer:end -->

## Go deeper

- [Schema Migration Strategy](<Schema_Migration_Strategy.md>)
- [Migrating to Partitioned Tables](<Migrating_to_Partitioned_Tables.md>)

## Implementations (tools)

- [PostgreSQL](<../../../../TOOLS/PostgreSQL/_index.md>)

## Practice questions

- [Deep] [How would you create an index on a huge production table without blocking writes, and what are the caveats of doing so?](<../../../../PRACTICE/01_Databases_and_SQL/Database_Operations/Migrations/Q-20669_How_would_you_create_an_index_on_a_huge_production_table_wit.md>)
- [Deep] [Describe how you'd add a foreign key constraint to an existing large table without a long validation lock.](<../../../../PRACTICE/01_Databases_and_SQL/Database_Operations/Migrations/Q-20678_Describe_how_you_d_add_a_foreign_key_constraint_to_an_existi.md>)
- [Deep] [In Postgres, how do NOT VALID plus a later VALIDATE CONSTRAINT let you add a check constraint with minimal locking?](<../../../../PRACTICE/01_Databases_and_SQL/Database_Operations/Migrations/Q-20679_In_Postgres_how_do_NOT_VALID_plus_a_later_VALIDATE_CONSTRAIN.md>)
- [Deep] [How do you add a NOT NULL constraint to an existing column that already has some NULLs, without a full-table exclusive lock?](<../../../../PRACTICE/01_Databases_and_SQL/Database_Operations/Migrations/Q-20700_How_do_you_add_a_NOT_NULL_constraint_to_an_existing_column_t.md>)
- [Deep] [Walk me through how you'd add a NOT NULL column with a default to a 200M-row table in Postgres without locking writes for minutes.](<../../../../PRACTICE/01_Databases_and_SQL/Database_Operations/Migrations/Q-20716_Walk_me_through_how_you_d_add_a_NOT_NULL_column_with_a_defau.md>)
- [Deep] [Why does adding a column with a volatile default (like now()) rewrite the whole table in older Postgres but a constant default does not?](<../../../../PRACTICE/01_Databases_and_SQL/Database_Operations/Migrations/Q-20717_Why_does_adding_a_column_with_a_volatile_default_like_now_re.md>)
- [Deep] [Why is CREATE INDEX CONCURRENTLY not allowed inside a transaction block, and how does that constrain your migration tool's design?](<../../../../PRACTICE/01_Databases_and_SQL/Database_Operations/Migrations/Q-20739_Why_is_CREATE_INDEX_CONCURRENTLY_not_allowed_inside_a_transa.md>)
- [Expert] [CREATE INDEX CONCURRENTLY failed and left an invalid index. Walk me through detecting it and recovering cleanly.](<../../../../PRACTICE/01_Databases_and_SQL/Database_Operations/Migrations/Q-20680_CREATE_INDEX_CONCURRENTLY_failed_and_left_an_invalid_index_W.md>)

## Related concepts

- [Safe Migrations on Rolling Deploys](<../../../../TOOLS/PostgreSQL/PostgreSQL_Platform/Safe_Migrations_on_Rolling_Deploys.md>) *(PostgreSQL Platform)*
- [Online Schema Changes](<../../../SQL_Language/SQL_Command_Types/Online_Schema_Changes.md>) *(SQL Command Types)*
- [Lock Granularity](<../../../Database_Internals/DBMS_Concepts/Lock_Granularity.md>) *(DBMS Concepts)*

