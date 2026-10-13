---
id: "01_Databases_and_SQL/Database_Operations/Migrations/Schema_Migration_Strategy"
level: 5
type: "deep-dive"
topic: "01_Databases_and_SQL/Database_Operations/Migrations"
tools: []
---

# Schema Migration Strategy

**L5 Expert** · [↑ Migrations](<../_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** A schema migration strategy turns risky, one-shot database changes into small, versioned, tested, reversible steps that the application and the database can survive at every point in between.

### 1. The big picture
A *migration* is any controlled change to a database's structure or content: adding a column, rewriting a stored procedure, moving to a new server or platform. The reason it needs a strategy is that a database, unlike application code, holds state. You cannot "redeploy" a table: the old rows must survive, the old application version may still be running, and a rollback has to deal with data that has already been transformed.

Two scales of migration appear in the books. The first is the **platform migration** (on-premises SQL Server to a newer version or to a cloud service): inventory, phases, test environments, validation. The second is the **everyday schema migration** a team ships weekly: a versioned script, run by a tool, in a pipeline. The same principles apply to both: change in small phases, validate each phase, watch performance, keep the change reversible, and write it down.

This lesson is the umbrella for the rest of the Migrations block: Concurrent Index Builds covers how to run individual risky DDL statements without blocking, and Migrating to Partitioned Tables covers one large structural change end to end.

### 2. Core concepts and vocabulary
- **Migration script** — a file containing the DDL/DML for one change, applied exactly once per environment.
- **Versioned migration** — a script with a unique ordered version (e.g. `V12__add_status_column.sql`) that runs once and is then immutable.
- **Repeatable migration** — a script that is re-applied whenever its content changes; suited to views, procedures and functions that are "redefined" rather than "altered".
- **Schema history table** — a table in the database that records which migrations have run and with what checksum (Flyway and Liquibase both keep one).
- **Checksum mismatch** — the tool detects that an already-applied script was edited; the history no longer matches the files.
- **Baseline** — telling the tool "the database as it exists now is version N" so a legacy database can enter version control without replaying history.
- **Expand / contract (parallel change)** — add the new structure alongside the old, move the application across, then remove the old structure.
- **Backfill** — populating a new column or table from existing data, usually in batches.
- **Shadow database / dry run** — a scratch database with a production-like schema where the migration is rehearsed first.
- **Canary release** — exposing the change to a small share of traffic to limit the blast radius.
- **Rollback** — returning to the previous state; for stateful systems this is far harder than reverting code.

### 3. How it works, step by step
A staged migration, whether it is a platform move or a weekly schema change:

1. **Inventory.** List every object and dependency: tables, procedures, functions, views, triggers, jobs, linked servers, and the applications that call them. You cannot migrate what you have not found.
2. **Choose the target and the shape.** Upgrade in place, move to a cloud service, or change only a table? Decide between an *incremental* path (move piece by piece) and a *lift-and-shift* (move everything, fix later). Incremental costs more planning but gives smaller failures.
3. **Plan in phases, starting with low-risk databases.** Learn the process on something that can afford to break.
4. **Build a staging environment that mirrors production** in schema and, as far as possible, in data volume. Run compatibility tooling (for SQL Server, the Data Migration Assistant) to surface unsupported features and estimate effort.
5. **Rehearse on a shadow database.** Apply the exact scripts that will go to production against a copy with the production schema. Look for errors, long-running statements, and unexpected plan changes.
6. **Validate at each stage.** After each phase check schema (objects, constraints, permissions), logic (procedure outputs on known inputs) and data (row counts, checksums, spot comparisons).
7. **Release with a limited blast radius.** Deploy the schema change first (backward-compatible), then the application, then clean up. Canary the application change.
8. **Monitor performance.** Use Query Store and DMVs (or the platform's equivalent) to compare query duration and plans before and after. Refresh statistics on newly shaped tables so the optimizer is not planning from stale numbers right after cutover.
9. **Document.** Record schema changes and refactored procedures so the next person knows why the database looks the way it does.
10. **Train the people.** Operations and developers need to understand the new platform's features and limits.

### 4. Worked example
Goal: split `customers.full_name` into `first_name` and `last_name` on a table that is read by a live application, using expand/contract.

```sql
-- Release 1 (expand): additive, backward compatible
ALTER TABLE dbo.customers ADD first_name nvarchar(100) NULL,
                              last_name  nvarchar(100) NULL;

-- Backfill in batches (see Migrating to Partitioned Tables for pacing)
UPDATE TOP (5000) dbo.customers
SET first_name = LEFT(full_name, CHARINDEX(' ', full_name + ' ') - 1),
    last_name  = LTRIM(SUBSTRING(full_name, CHARINDEX(' ', full_name + ' '), 200))
WHERE first_name IS NULL;
-- repeat until @@ROWCOUNT = 0
```

Release 2 changes the application to write both old and new columns and to read the new ones. Release 3, after every old app instance is gone, adds `NOT NULL`, then a later release finally drops `full_name`.

Why four steps instead of one `ALTER` plus `UPDATE`? Because at any instant two application versions may be running (rolling deploys, canaries). A single release that drops `full_name` breaks the old version mid-deploy. The migration is *decoupled from the code release*: each step is safe if the next one is delayed or rolled back. If Release 2 misbehaves, you roll back the code; the extra columns are harmless.

### 5. Under the hood   *(optional here)*
How a migration tool keeps order: on start it reads the history table, compares the applied versions and checksums to the script files, and runs only the pending versions in order inside the tool's own bookkeeping (often in a transaction per script where the database supports transactional DDL; SQL Server does, many others do not). A checksum mismatch stops the run, because executing a changed script against a database that already ran the old one would silently diverge environments.

Timestamp-style version numbers (`20261012_1530`) are preferred over sequential integers by many teams because two developers on two branches both pick "the next number" and collide; timestamps make collisions rare, at the price of merge-time order surprises.

Rollback complexity comes from state: a code revert undoes nothing in the data. Mitigations from the books are parallel writes during the transition, and a documented restore from a recent, validated snapshot.

### 6. Performance and design implications
- **Large changesets are fragile.** Bundling a backlog of changes into one release multiplies risk; small, frequent, reviewed changes are safer.
- **Review capacity is the bottleneck.** Build a library of heuristics (which changes are safe, which are dangerous) so developers self-serve safe changes through normal deployment and only ambiguous ones need DBA review. Review recent changes and outcomes periodically as part of post-mortems.
- **Linters help but are not complete.** Static checkers can reject patterns such as non-concurrent index creation or unsafe `NOT NULL` additions; they cannot know table size or traffic.
- **Measure.** Compare Query Store regressions, wait stats, and replication lag before and after each phase.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `flyway migrate / info / validate` | Apply pending, show state, verify checksums | Every deploy; `validate` in CI |
| `flyway baseline` | Mark current DB as a starting version | Bringing a legacy DB under control |
| `flyway repair` | Fix history table (remove failed entries, realign checksums) | After a failed run or a deliberate script fix; back up first |
| Data Migration Assistant | Find compatibility issues, estimate effort | Before a SQL Server platform move |
| Query Store | Per-query regressions over time | Post-migration performance check |
| `UPDATE STATISTICS` / `ANALYZE` | Refresh optimizer statistics | After large structural or data changes |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| Incremental, phased migration | Simple small system where the planning overhead exceeds the risk |
| Lift-and-shift first, optimize later | You have no time to ever do the "later" half |
| Off-the-shelf migration tool (Flyway, Liquibase, Atlas, Bytebase) | A homegrown orchestrator at scale: ongoing maintenance and documentation cost usually outweighs it |
| Down-migrations for every script | Data-destroying changes; prefer forward-fix plus backups, and validate that any down script truly reverses the up |
| Schema and data migrations in one pipeline | Huge backfills that need pacing and separate monitoring |

### 9. Common mistakes
- Editing an applied migration → checksum mismatch, environments diverge → add a new migration; use `repair` only deliberately and after a backup.
- Shipping app change and destructive DDL in one release → old instances break mid-deploy → expand/contract across releases.
- Trusting auto-generated SQL from an ORM → surprising locks, dropped columns, lost edge cases (cascades, unique constraints) → always review the generated script.
- Testing only on a tiny dev database → fine there, hours of locking in production → rehearse with production-sized data.
- An un-rehearsed rollback plan → fails when needed most → treat it as production code and run it regularly.
- Using `repair` instead of `baseline` for initialization → wrong history → baseline legacy databases.

### 10. 🎯 Interview kit
**30-second answer:** I make every change small, versioned and backward compatible: expand, migrate the app, then contract. Scripts run through a tool that records history; I rehearse on a production-shaped copy, canary the release, watch Query Store and replication lag, and keep a rehearsed rollback or forward-fix.

**Structure a longer answer:** clarify (platform move or table change, size, downtime tolerance) → core mechanism (phases, expand/contract, history table) → trade-offs (speed vs risk, build vs buy, rollback vs roll-forward) → close with a real scenario such as the name-split above.

**Likely follow-ups:**
- *How do you roll back after a backfill mutated data?* → Restore from a validated backup/snapshot or keep the old column and parallel-write; plan it before running.
- *Why decouple app and migration releases?* → Mixed versions run during deploys; each step must be safe independently.
- *How do you version procedures and views?* → Repeatable migrations in the same repo and pipeline as table DDL.
- *How do you keep migrations safe in CI?* → Apply all scripts to an ephemeral database, run lint rules, run tests.

**Signals interviewers look for:** staged thinking, awareness that state makes rollback hard, rehearsal and measurement, and a pragmatic build-vs-buy view.

### 11. Check yourself
- [ ] Explain expand/contract and why each step is individually safe.
- [ ] Say what a checksum mismatch means and how you would resolve it without corrupting history.
- [ ] List the validation checks after each migration phase.
- [ ] Describe what makes rolling back a stateful migration harder than rolling back code.
- [ ] Sketch a CI job that rehearses migrations on a shadow database.
<!-- explainer:end -->

## Practice questions

- [Intermediate] [How do you manage stored procedure, function, and view definitions under version control alongside table migrations?](<../../../../PRACTICE/01_Databases_and_SQL/Database_Operations/Migrations/Q-20696_How_do_you_manage_stored_procedure_function_and_view_definit.md>)
- [Deep] [Explain how a shadow-database or dry-run approach validates a migration against a production-like schema before it ships.](<../../../../PRACTICE/01_Databases_and_SQL/Database_Operations/Migrations/Q-20692_Explain_how_a_shadow-database_or_dry-run_approach_validates_.md>)
- [Deep] [How do you communicate and schedule a risky migration with on-call, product, and downstream teams? What's in your runbook?](<../../../../PRACTICE/01_Databases_and_SQL/Database_Operations/Migrations/Q-20746_How_do_you_communicate_and_schedule_a_risky_migration_with_o.md>)
- [Deep] [Explain how you'd integrate a linter like squawk or eugene into CI to statically reject unsafe Postgres migrations, and where static analysis falls short.](<../../../../PRACTICE/01_Databases_and_SQL/Database_Operations/Migrations/Q-20789_Explain_how_you_d_integrate_a_linter_like_squawk_or_eugene_i.md>)
- [Deep] [How do you version and roll out changes to database-side logic (triggers, stored procs, RLS policies) with the same rigor and reversibility as table DDL?](<../../../../PRACTICE/01_Databases_and_SQL/Database_Operations/Migrations/Q-20835_How_do_you_version_and_roll_out_changes_to_database-side_log.md>)
- [Expert] [How do you handle rolling back a migration that already backfilled and mutated data, where the data change isn't trivially reversible?](<../../../../PRACTICE/01_Databases_and_SQL/Database_Operations/Migrations/Q-20693_How_do_you_handle_rolling_back_a_migration_that_already_back.md>)
- [Expert] [Make the build-versus-buy case for a homegrown migration orchestrator versus adopting Atlas or Bytebase across the org. What tips your decision at scale?](<../../../../PRACTICE/01_Databases_and_SQL/Database_Operations/Migrations/Q-20773_Make_the_build-versus-buy_case_for_a_homegrown_migration_orc.md>)
- [Expert] [Design a rollback-by-flag architecture where every schema change ships behind a compatibility toggle so reverting never requires a second risky DDL.](<../../../../PRACTICE/01_Databases_and_SQL/Database_Operations/Migrations/Q-20815_Design_a_rollback-by-flag_architecture_where_every_schema_ch.md>)

## Related concepts

- [Schema Changes Under Long Transactions](<../../../../04_Data_Architecture_and_Strategy/Platform_Strategy/Schema_Evolution/Schema_Changes_Under_Long_Transactions.md>) *(Schema Evolution)*
- [Coordinated Multi-store Schema Changes](<../../../../04_Data_Architecture_and_Strategy/Platform_Strategy/Schema_Evolution/Coordinated_Multi-store_Schema_Changes.md>) *(Schema Evolution)*
- [Migrations with Read Your Writes](<../../../../04_Data_Architecture_and_Strategy/Platform_Strategy/Schema_Evolution/Migrations_with_Read_Your_Writes.md>) *(Schema Evolution)*
- [Multi-tenant Migrations](<../../../../07_Distributed_Systems_and_Design/System_Design/System_Design_Core/Read-through_and_Write-behind_Caching/Multi-tenant_Migrations.md>) *(System Design Core)*
- [Schema Evolution in Lakehouse Tables](<../../../SQL_Language/SQL_Fundamentals/Schema_Evolution_in_Lakehouse_Tables.md>) *(SQL Fundamentals)*
- [Schema Migration Risks](<../../../Database_Internals/DBMS_Concepts/Schema_Migration_Risks.md>) *(DBMS Concepts)*

## References

- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 11: Automation with SSIS, SQL Agent, and PowerShell › Conclusion
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 14: Unit Testing T-SQL with tSQLt › Versioning and Rollback Strategies
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 15: Case Studies and the Future of T-SQL › Structure
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 15: Case Studies and the Future of T-SQL › Migrating Legacy SQL to Modern Platforms
- *Data Engineering with Medallion Architecture* — Miki Eto — Evolution of Data Architecture › Process or steps
- *Data Engineering with Medallion Architecture* — Miki Eto — Cloud Migration and Coexistence Strategies › CHAPTER 9Cloud Migration and Coexistence Strategies
- *Data Engineering with Medallion Architecture* — Miki Eto — Cloud Migration and Coexistence Strategies › Objectives
- *Data Engineering with Medallion Architecture* — Miki Eto — Cloud Migration and Coexistence Strategies › Key considerations
- *Database Reliability Engineering* — Laine Campbell, Charity Majors — Release Management › Database migrations and packaging
- *Database Reliability Engineering* — Laine Campbell, Charity Majors — Release Management › Deployment
- *Database Reliability Engineering* — Laine Campbell, Charity Majors — Release Management › Migrations and Versioning
- *Database Reliability Engineering* — Laine Campbell, Charity Majors — Release Management › Migration Patterns
- *Expert Performance Indexing in SQL Server* — Jason Strate, Grant Fritchey — Impact Analysis › Prepare Deployment and Rollback of Schema
- *Data Contracts: Developing Production-Grade Pipelines at Scale* — Chad Sanderson, Mark Freeman — The Challenges of Scaling Data Infrastructure › The Dangers of Database Migrations
- *Data Contracts: Developing Production-Grade Pipelines at Scale* — Chad Sanderson, Mark Freeman — Implementing Data Contracts › Update Data Migration File for New Schema
- *Data Contracts: Developing Production-Grade Pipelines at Scale* — Chad Sanderson, Mark Freeman — Implementing Data Contracts › Revert Database Migration Changes
- *Data Contracts: Developing Production-Grade Pipelines at Scale* — Chad Sanderson, Mark Freeman — Conclusion: AI and the Future of Data Contracts › Future Implications of Shift Left Data
- *Data Quality Fundamentals* — Barr Moses, Lior Gavish, Molly Vorwerck — Data Quality in the Real World Conversations and Case Studies › You've Recently Migrated to the Cloud

