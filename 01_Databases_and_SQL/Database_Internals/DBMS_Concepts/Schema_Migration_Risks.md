---
id: "01_Databases_and_SQL/Database_Internals/DBMS_Concepts/Schema_Migration_Risks"
level: 4
type: "concept"
topic: "01_Databases_and_SQL/Database_Internals/DBMS_Concepts"
tools: []
---

# Schema Migration Risks

**L4 Advanced** · [↑ DBMS Concepts](<_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** A schema migration is risky because DDL takes locks, may rewrite the table, and must work with old and new application code at the same time — so you ship changes as small, backwards-compatible, reversible steps.

### 1. The big picture
Code can be rolled back by redeploying; a schema and its data cannot. A migration runs against a live database serving traffic, and two things can go wrong at once: the **database** (locks, long rewrites, replication lag, full disks, failed halfway) and the **application** (code expecting the old shape while the new one is deployed, or the reverse).

The discipline that handles both is **expand/contract** (parallel change): make additive, compatible changes first, move data and code, and only then remove the old structure. Data platforms add a further wrinkle — in lakehouses and pipelines, *schema evolution* happens at ingestion, and downstream consumers break if a column is renamed or its type changes silently.

### 2. Core concepts and vocabulary
- **Migration** — a versioned, ordered change script (DDL and sometimes data) applied by a tool (Flyway, Liquibase, EF Migrations, dbt/Alembic).
- **Backwards compatible** — the old application version still works after the change.
- **Expand / contract** — add new structure (expand), migrate, switch, then drop old structure (contract).
- **Online DDL** — schema change that lets reads and writes continue (varies by engine/operation).
- **Blocking DDL** — takes a schema-modification lock for the duration.
- **Table rewrite** — the engine copies every row into a new physical structure (e.g. changing a column type, adding a non-null default in older versions).
- **Metadata-only change** — only the catalog changes; instant (adding a nullable column in most engines).
- **Backfill** — populating new columns for existing rows, in batches.
- **Schema drift** — the real schema diverges from what version control says.
- **Schema-on-write vs schema-on-read** — validate at write or interpret at read; affects where evolution breaks.

### 3. How it works, step by step
A safe "rename column / change meaning" using expand/contract:

1. **Expand:** add the new nullable column (`email_normalized`). Metadata-only; old code unaffected.
2. **Dual-write:** deploy application code (or a trigger) writing both old and new columns.
3. **Backfill:** copy existing data in small batches with throttling; monitor replication lag and locks.
4. **Verify:** compare counts and checksums; add the constraint as `NOT VALID`/`WITH NOCHECK` then validate separately.
5. **Switch reads:** deploy code reading the new column; keep dual-write briefly so rollback is safe.
6. **Contract:** after a full release cycle and no readers remain, drop the old column in a separate migration.

Risk matrix (representative; verify per engine/version):

| Operation | Typical effect |
|---|---|
| Add nullable column | Fast, metadata-only |
| Add column with default | Fast in recent SQL Server/PostgreSQL 11+/MySQL 8 (instant); table rewrite in older versions |
| Add `NOT NULL` without default to populated table | Fails, or full scan/rewrite |
| Change column data type | Usually rewrite + schema lock |
| Add foreign key / check constraint | Full validation scan under lock unless added unvalidated |
| `CREATE INDEX` | Locks writes unless online/`CONCURRENTLY` |
| Drop column | Fast, but breaks old code instantly |
| Rename column/table | Instant, breaks every reader/writer by name |

### 4. Worked example
```sql
-- 1) expand (instant)
ALTER TABLE dbo.Customers ADD phone_e164 VARCHAR(20) NULL;

-- 2) backfill in batches, so locks and log stay small
DECLARE @rows INT = 1;
WHILE @rows > 0
BEGIN
  UPDATE TOP (5000) dbo.Customers
  SET    phone_e164 = dbo.fn_to_e164(phone)
  WHERE  phone_e164 IS NULL AND phone IS NOT NULL;
  SET @rows = @@ROWCOUNT;
  WAITFOR DELAY '00:00:01';         -- let replicas and log backups catch up
END

-- 3) add the constraint without scanning under a long lock
ALTER TABLE dbo.Customers WITH NOCHECK
  ADD CONSTRAINT ck_phone CHECK (phone_e164 LIKE '+%');
ALTER TABLE dbo.Customers WITH CHECK CHECK CONSTRAINT ck_phone;

-- 4) later release: contract
-- ALTER TABLE dbo.Customers DROP COLUMN phone;
```
Why it works: each batch is a short transaction (small lock, small log growth); the `WAITFOR` prevents replication lag piling up; the constraint is validated separately so it does not hold a schema lock while scanning for hours. Postgres equivalents: `CREATE INDEX CONCURRENTLY`, `ADD CONSTRAINT … NOT VALID` then `VALIDATE CONSTRAINT`, and `SET lock_timeout` to fail fast instead of queueing.

### 5. Under the hood
- **Schema locks queue.** An `ALTER TABLE` that must wait for a long-running transaction blocks *everything behind it* (new queries queue behind the pending schema lock). A "quick" DDL can freeze a table. Use `lock_timeout` and retry.
- **Rewrites and logging.** A rewrite writes every row to the log (and to replicas) and may need 2× the table's disk space. Check free space and log size first.
- **Replication.** DDL replays on replicas serially; a long migration causes lag and stale reads. Tools like gh-ost/pt-online-schema-change rebuild a shadow table via triggers or binlog, then swap names atomically.
- **Delta/lakehouse tables.** Schema enforcement rejects non-matching writes by default; schema evolution (`mergeSchema`) adds columns automatically; type changes and renames need explicit operations. Old files are not rewritten; readers see nulls for missing columns.
- **Bronze/Silver patterns.** Ingestion layers often accept schema-on-read to survive upstream change; the schema is enforced when promoting to curated layers, so breakage is caught at a defined boundary.
- **Merge/upsert.** Upserts into file-based tables rewrite whole files (copy-on-write), which makes large corrective backfills costly.

### 6. Performance and design implications
- Estimate before running: row count, table size, free disk, log size, replica lag.
- Run in a lower environment with production-sized data; wall-clock matters more than correctness for DDL.
- Batch by key range, throttle, and make backfills **idempotent and resumable**.
- Add `lock_timeout`/`SET LOCK_TIMEOUT` so a stuck migration fails rather than blocks traffic.
- Keep migrations small, one concern each; avoid mixing DDL and large data changes in one transaction.
- Measure: lock waits, log usage (`sys.dm_db_log_space_usage`), replication lag, error rates after deploy.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `ALTER TABLE … ADD … NULL` | Instant column add | Expand step |
| `CREATE INDEX … WITH (ONLINE = ON)` / `CONCURRENTLY` | Build index without blocking writes | Live tables |
| `WITH NOCHECK` / `NOT VALID` then validate | Defer constraint scan | Large tables |
| `SET LOCK_TIMEOUT` / `lock_timeout` | Bound wait on schema lock | Every production DDL |
| gh-ost / pt-osc / pg_repack | Online table rebuild | Rewrites on MySQL/Postgres |
| Flyway/Liquibase checksums | Detect edited migrations | Preventing drift |
| `mergeSchema` / `ALTER TABLE ADD COLUMNS` (Delta) | Evolve lake schema | Pipelines |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| Expand/contract over several releases | You can afford a maintenance window and the table is small |
| Online DDL | Engine/operation does not support it; Enterprise-edition only features |
| Trigger/shadow-table tooling | Heavy write rate, many triggers/foreign keys |
| Auto schema evolution in bronze | Silver/gold consumers need stable contracts |
| Big-bang migration in a window | Rollback is not rehearsed |

### 9. Common mistakes
- Dropping or renaming a column in the same release as the code change → old instances fail → separate expand and contract.
- Running DDL without a lock timeout → queue pile-up and outage → set timeouts, run off-peak.
- One giant `UPDATE` backfill → long locks, log explosion, lag → batch with throttling.
- Editing an applied migration file → environments diverge → add a new migration.
- Assuming "add column with default" is instant everywhere → rewrite on old versions → check the version docs and test.
- No rollback plan for data changes → stuck → take backups/snapshots; keep the old column until verified.

### 10. 🎯 Interview kit
**30-second answer:** A migration is risky because DDL locks and can rewrite large tables, and the application must work with both schema versions during deployment. I use expand/contract: add compatible structure, dual-write, backfill in throttled batches, switch reads, then drop the old structure in a later release — with lock timeouts and rehearsed rollback.

**Structure a longer answer:** risks (locks, rewrites, replication, compat) → expand/contract steps → tooling/online DDL → verification/rollback → scenario.

**Likely follow-ups:**
- *How do you add a NOT NULL column to a huge table?* → Add nullable, backfill, add constraint unvalidated then validate, then set NOT NULL.
- *Why can a quick ALTER cause an outage?* → It waits behind a long transaction and blocks new queries behind it.
- *How do you roll back?* → Keep old structure until verified; migrations forward-only with compensating changes.
- *How does Delta handle schema changes?* → Enforcement by default; opt-in evolution.

**Signals interviewers look for:** staged releases, awareness of lock queues and replicas, batching, and a rollback story.

### 11. Check yourself
- [ ] List the steps to rename a column with zero downtime.
- [ ] Explain why an ALTER waiting on a lock can block other sessions.
- [ ] Write a resumable batched backfill.
- [ ] Describe how you would validate a new constraint on a 2-billion-row table.
- [ ] Contrast schema enforcement and evolution in a lakehouse table.
<!-- explainer:end -->

## Practice questions

- [Basic] [What is the difference between a full load and an incremental load into a warehouse table?](<../../../PRACTICE/01_Databases_and_SQL/Database_Internals/DBMS_Concepts/Q-22559_What_is_the_difference_between_a_full_load_and_an_incrementa.md>)
- [Basic] [What is a temporary / scratch table in an ETL step and when would you use one instead of a CTE?](<../../../PRACTICE/01_Databases_and_SQL/Database_Internals/DBMS_Concepts/Q-22637_What_is_a_temporary_scratch_table_in_an_ETL_step_and_when_wo.md>)
- [Intermediate] [What is change data capture (CDC) and why is it useful for keeping a warehouse in sync with a source database?](<../../../PRACTICE/01_Databases_and_SQL/Database_Internals/DBMS_Concepts/Q-22561_What_is_change_data_capture_CDC_and_why_is_it_useful_for_kee.md>)
- [Intermediate] [How do migration tools like Flyway, Liquibase, or Prisma Migrate keep schema changes ordered and repeatable?](<../../../PRACTICE/01_Databases_and_SQL/Database_Internals/DBMS_Concepts/Q-22339_How_do_migration_tools_like_Flyway_Liquibase_or_Prisma_Migra.md>)
- [Deep] [How does reading from the transaction log for CDC differ from polling a table with a timestamp column?](<../../../PRACTICE/01_Databases_and_SQL/Database_Internals/DBMS_Concepts/Q-22562_How_does_reading_from_the_transaction_log_for_CDC_differ_fro.md>)
- [Deep] [What is an append-only immutable event table and why do many pipelines prefer never updating existing rows?](<../../../PRACTICE/01_Databases_and_SQL/Database_Internals/DBMS_Concepts/Q-22618_What_is_an_append-only_immutable_event_table_and_why_do_many.md>)
- [Deep] [How would you implement an audit trail table that records every change to a row?](<../../../PRACTICE/01_Databases_and_SQL/Database_Internals/DBMS_Concepts/Q-22284_How_would_you_implement_an_audit_trail_table_that_records_ev.md>)
- [Deep] [How do you roll back a schema migration safely, and why are some migrations (like dropping a column) effectively irreversible in practice?](<../../../PRACTICE/01_Databases_and_SQL/Database_Internals/DBMS_Concepts/Q-22534_How_do_you_roll_back_a_schema_migration_safely_and_why_are_s.md>)

## Related concepts

- [Schema Evolution in Lakehouse Tables](<../../SQL_Language/SQL_Fundamentals/Schema_Evolution_in_Lakehouse_Tables.md>) *(SQL Fundamentals)*
- [Schema Migration Strategy](<../../Database_Operations/Migrations/Concurrent_Index_Builds/Schema_Migration_Strategy.md>) *(Migrations)*
- [Migrations with Read Your Writes](<../../../04_Data_Architecture_and_Strategy/Platform_Strategy/Schema_Evolution/Migrations_with_Read_Your_Writes.md>) *(Schema Evolution)*

## References

- *Databricks Certified Data Engineer Associate Study Guide* — Derar Alhussein — Capturing Data Changes › Disadvantages of APPLY CHANGES INTO
- *Databricks Certified Data Engineer Associate Study Guide* — Derar Alhussein — D
- *Delta Lake: Up and Running* — Bennie Haelen, Dan Davis — Getting Started with Delta Lake › Update scenario
- *Delta Lake: Up and Running* — Bennie Haelen, Dan Davis — Table Deletes, Updates, and Merges › Conclusion
- *Delta Lake: Up and Running* — Bennie Haelen, Dan Davis — Using Time Travel › Chapter 6. Using Time Travel
- *Delta Lake: Up and Running* — Bennie Haelen, Dan Davis — Using Time Travel › RESTORE Considerations and Warnings
- *Fundamentals of Data Engineering* — Joe Reis, Matt Housley — The Data Engineering Lifecycle › Data lineage
- *Fundamentals of Data Engineering* — Joe Reis, Matt Housley — Ingestion › Schema changes
- *Fundamentals of Data Engineering* — Joe Reis, Matt Housley — Modeling Streaming Data › Upsert/merge
- *Building Medallion Architectures* — Piethein Strengholt — Demystifying the Medallion Architecture › Schema Evolution and Management
- *Learning Spark (2nd ed.)* — Jules S. Damji, Brooke Wenig, Tathagata Das, Denny Lee — Building Reliable Data Lakes with Apache Spark › Upserting change data to a table using merge()
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 6: CTEs, MERGE, Pivoting, JSON, and XML › Best Practices for Using MERGE
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 6: CTEs, MERGE, Pivoting, JSON, and XML › Real-Life Use Cases for MERGE
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 15: Case Studies and the Future of T-SQL › System-Versioned Data Mechanics

