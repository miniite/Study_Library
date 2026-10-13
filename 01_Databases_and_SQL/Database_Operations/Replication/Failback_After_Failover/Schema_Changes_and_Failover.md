---
id: "01_Databases_and_SQL/Database_Operations/Replication/Schema_Changes_and_Failover"
level: 5
type: "deep-dive"
topic: "01_Databases_and_SQL/Database_Operations/Replication"
tools: []
---

# Schema Changes and Failover

**L5 Expert** · [↑ Replication](<../_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** A schema change is just another event in the replication log, so a slow or blocking ALTER stalls every replica behind it; online-schema-change tooling and expand/contract migrations keep each step small, compatible and safe to fail over in the middle of.

### 1. The big picture
Replication ships *everything* the primary logs, including DDL (`ALTER TABLE`, `CREATE INDEX`). Data changes (DML) are replayed concurrently by many primary sessions but, on a replica, are applied through a much narrower pipeline. A DDL that takes 40 minutes on the primary will take roughly 40 minutes on every replica **and block the apply queue behind it**: lag grows by 40 minutes, reads become stale, and if a failover happens during that window the promoted replica is either missing the change or is half-way through it.

The operational problem has three parts: (1) keep the primary online while altering big tables, (2) keep replicas close enough to the primary that they remain usable and promotable, and (3) make sure the application works against *both* the old and the new schema while the change rolls through the topology. This lesson builds on the replication-lag and failback lessons.

### 2. Core concepts and vocabulary
- **DDL vs DML** — data-definition statements change structure and (in MySQL) are logged as statements; data-manipulation statements change rows and can be logged as row images.
- **Blocking DDL** — an operation that holds a metadata lock or rewrites the table, preventing concurrent writes for its duration.
- **Online DDL** — an engine feature that allows reads/writes while the change is made (e.g. InnoDB `ALGORITHM=INPLACE`/`INSTANT`, `LOCK=NONE`; PostgreSQL `CREATE INDEX CONCURRENTLY`).
- **Online schema change tools** — external tools (pt-online-schema-change, gh-ost, GitHub's approach; Spirit and others) that build a shadow table, copy rows in chunks, and swap.
- **Shadow (ghost) table** — a copy of the table with the new schema, filled in the background.
- **Cut-over** — the brief atomic rename that swaps the shadow table for the original.
- **Expand / contract (parallel change)** — add new structure first (expand), migrate code and data, then remove the old structure (contract), with every step backward-compatible.
- **Drift** — rows that differ between primary and replica, detected by checksums.
- **Row-based replication (RBR)** — logs row images; needs a primary key to be efficient when applying.

### 3. How it works, step by step
**What happens with a plain `ALTER TABLE` on a replicated topology**
1. The primary executes the ALTER (table rewrite, metadata lock; possibly blocking writers).
2. It writes the statement to the log when it completes.
3. Each replica's applier reaches that event and executes the same ALTER — for the same duration — while all later transactions queue behind it.
4. Replicas lag by about the ALTER's duration, in addition to existing lag. If you ran it on a table with a long-running reader, the metadata lock wait also stalls the applier.

**How a trigger- or binlog-based online tool avoids that**
1. Create `_t_new` with the target schema (cheap, empty).
2. Copy rows from `t` into `_t_new` in small chunks (e.g. 1,000 rows), pausing when replica lag or primary load exceeds thresholds.
3. Keep `_t_new` in sync with concurrent writes: pt-osc with triggers on `t`; gh-ost by reading the binary log as a pretend replica (no triggers, therefore less load on the primary, and it can be paused or throttled completely).
4. When copy is complete and lag is acceptable, **cut over**: atomically `RENAME TABLE t TO _t_old, _t_new TO t`.
5. Drop `_t_old` later.

Because the row copy is thousands of small, ordinary transactions, replicas apply them with the normal pipeline and remain close to the primary. The tools watch replica lag and slow down automatically.

**Rolling the change through a topology (for blocking engines/changes)**
1. Apply the change on each replica in turn, with replication for that node temporarily stopped or with binary logging off for the session.
2. Promote an altered replica (planned failover).
3. Alter the old primary (now a replica).
4. Demote/promote back if wanted.
This requires the change to be *replication-compatible*: a replica may have an extra column, and the source's row events must still apply.

### 4. Worked example
Adding an index and a nullable column to a 400 GB `orders` table with gh-ost:

```bash
gh-ost \
  --host=replica-1.db --database=shop --table=orders \
  --alter="ADD COLUMN risk_score SMALLINT NULL, ADD INDEX ix_cust_created (customer_id, created_at)" \
  --max-lag-millis=1500 \
  --chunk-size=1000 \
  --throttle-control-replicas="replica-2.db,replica-3.db" \
  --postpone-cut-over-flag-file=/tmp/ghost.postpone \
  --verbose --execute
```

Reading it: gh-ost connects to a replica, tails its binlog to see changes, copies chunks, throttles if any listed replica lags more than 1.5 s, and **waits at the cut-over** until the flag file is removed so you pick the moment. Then the migration finishes.

Safe sequence with an application:
1. **Expand:** add the nullable `risk_score` column (old code ignores it).
2. Deploy code that writes both old and new fields.
3. Backfill existing rows in small batches (keep batch transactions short so the applier can keep up).
4. Deploy code that reads the new column.
5. **Contract:** drop the old column in a later, separate change.

Verification after the change:

```bash
pt-table-checksum --replicate=percona.checksums --chunk-time=0.5 \
  --max-lag=2 --check-interval=1 h=primary,u=chk
# then, on a replica: SELECT * FROM percona.checksums WHERE this_crc <> master_crc;
```

A mismatch means drift. Repair with `pt-table-sync --replicate percona.checksums --sync-to-source --print` first (inspect), then `--execute` for the specific table; re-checksum.

### 5. Under the hood
- **Why the apply queue blocks.** The applier preserves the primary's commit order. A DDL needs an exclusive metadata lock on the table, so even parallel appliers must wait for in-flight transactions on that table and then block later ones. One long statement therefore serializes the table's stream.
- **Instant and in-place operations.** Modern engines handle many changes without a rewrite: MySQL 8 `ALGORITHM=INSTANT` adds a column by changing metadata only; `INPLACE` builds an index while allowing DML via an online log. They are cheap on replicas too — but they still take a brief metadata lock, which queues behind any long transaction.
- **Triggers vs binlog.** Trigger-based tools add write amplification inside every transaction and hold the same metadata locks; binlog-based tools consume the log asynchronously, so a write spike cannot be slowed by the tool, but cut-over correctness depends on event ordering.
- **Replica-side schema may differ.** In row-based replication, columns are mapped by position; extra columns at the end on the replica are tolerated, reordered or missing ones break apply. This is the compatibility rule that makes rolling changes possible.
- **Failover mid-migration.** If the shadow table exists on the old primary only, the promoted replica has an incomplete migration; the tool's state is lost and the half-built table is an orphan. Runbooks must say: *no failover during cut-over*, and after any failover, restart or clean up the migration.

### 6. Performance and design implications
- Budget the copy: rows ÷ chunk throughput. A 400 GB table at throttled speed may take days; plan the cut-over time, not just the start.
- Throttle on **replica lag**, not just primary CPU; otherwise the migration is "safe" for the primary and destructive for read replicas.
- Extra disk is required for the shadow table and for the additional binlog volume — typically at least the table size again.
- Foreign keys are poorly supported by shadow-table tools (renames break constraint names and cascades); check before choosing a tool.
- Every checksum chunk runs as a statement replayed on replicas, so verification itself must be throttled and scheduled off-peak.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `ALTER TABLE … ALGORITHM=INSTANT/INPLACE, LOCK=NONE` | Fails rather than silently blocking | Always specify to avoid surprise table copies |
| `gh-ost` / `pt-online-schema-change` | Shadow-table online migration | Large tables, strict lag budget |
| `--max-lag-millis`, `--chunk-size`, `--critical-load` | Throttle and abort rules | Every run |
| `pt-table-checksum` / `pt-table-sync` | Detect and repair drift | After migrations, failovers, errant writes |
| `CREATE INDEX CONCURRENTLY` (PG) | Build index without long write lock | PostgreSQL index builds |
| `lock_wait_timeout` (session) | Abort DDL if it can't get its lock | Avoid queueing every query behind a stuck ALTER |
| `replica_parallel_workers` | Parallel apply | Faster catch-up after heavy changes |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| Engine online DDL (`INSTANT`/`INPLACE`) | Operation needs a rewrite or the engine falls back to `COPY` |
| gh-ost (binlog-based) | Table has foreign keys; no binlog-reading replica available |
| pt-osc (trigger-based) | Table already has triggers or is hot enough that trigger overhead hurts |
| Rolling replicas then promote | Change isn't backward-compatible with row events |
| Expand/contract with application deploys | Team can't coordinate multi-release changes |

### 9. Common mistakes
- Running a blocking `ALTER` at peak → replicas lag by its duration → use online tooling or off-peak windows with `LOCK=NONE` asserted.
- Treating DDL as instant → surprise outage → estimate from table size, test on a production-size clone.
- Dropping or renaming a column in one release → old code (or lagging replicas) break → expand/contract.
- Leaving a long transaction open while ALTER waits → every later query queues on the metadata lock → set `lock_wait_timeout` and kill blockers.
- Failing over in the middle of a migration → orphaned shadow table and incompatible schemas → freeze failover during cut-over.
- Never verifying after → silent drift → checksum and repair, throttled.

### 10. 🎯 Interview kit
**30-second answer:** DDL replicates like any other event and is applied serially on replicas, so a slow ALTER stalls the apply pipeline and creates lag. I avoid that with online DDL or shadow-table tools that copy in small throttled chunks, and with expand/contract so schema and code are compatible at every step, then verify with checksums.

**Structure a longer answer:** clarify (engine, table size, lag budget, FKs) → mechanism (blocking ALTER vs shadow table + cut-over) → trade-offs (trigger vs binlog tools, rolling vs online) → scenario (migration running when a failover is triggered: pause tool, fail over, clean up, restart).

**Likely follow-ups:**
- *Why can CREATE INDEX stall replicas?* → replayed serially on each replica; later events wait.
- *How do you check replicas without heavy locking?* → chunked checksums with lag-based throttling.
- *How do you prevent migration and failover colliding?* → change-freeze flag, tooling that refuses to start with a failover pending, post-failover reconciliation step.
- *Why require primary keys everywhere?* → row events and chunked copy/checksum all depend on them.

**Signals interviewers look for:** thinking of schema change as a replication event, throttling on replica lag, backward compatibility, explicit failover handling.

### 11. Check yourself
- [ ] Explain why a 30-minute ALTER on the primary causes ~30 minutes of replica lag.
- [ ] Describe a shadow-table migration from creation to cut-over and what each step protects.
- [ ] Contrast trigger-based and binlog-based tools in load and failure behaviour.
- [ ] Write an expand/contract plan for renaming a column with zero downtime.
- [ ] State what you do if the primary fails while a migration is half-complete.
<!-- explainer:end -->

## Practice questions

- [Deep] [How do you keep the master and replicas schema-compatible during an online schema change so replication doesn't break?](<../../../../PRACTICE/01_Databases_and_SQL/Database_Operations/Replication/Q-21193_How_do_you_keep_the_master_and_replicas_schema-compatible_du.md>)
- [Deep] [Why can adding an index on the master with a blocking DDL stall all replicas, and how do tools like pt-online-schema-change help?](<../../../../PRACTICE/01_Databases_and_SQL/Database_Operations/Replication/Q-21194_Why_can_adding_an_index_on_the_master_with_a_blocking_DDL_st.md>)
- [Deep] [Explain how you would verify data consistency between master and slave using pt-table-checksum without heavy locking.](<../../../../PRACTICE/01_Databases_and_SQL/Database_Operations/Replication/Q-21200_Explain_how_you_would_verify_data_consistency_between_master.md>)
- [Deep] [How does replication behave for DDL versus DML, and why can a slow ALTER on the master block the apply pipeline on replicas?](<../../../../PRACTICE/01_Databases_and_SQL/Database_Operations/Replication/Q-21237_How_does_replication_behave_for_DDL_versus_DML_and_why_can_a.md>)
- [Deep] [How do you design continuous consistency verification (checksums / drift detection) across a large replica fleet without the verification itself becoming a load and lag problem?](<../../../../PRACTICE/01_Databases_and_SQL/Database_Operations/Replication/Q-21289_How_do_you_design_continuous_consistency_verification_checks.md>)
- [Deep] [How would you enforce that row-based replication is used everywhere PK-less tables exist, and what's your migration plan to eliminate PK-less tables org-wide?](<../../../../PRACTICE/01_Databases_and_SQL/Database_Operations/Replication/Q-21336_How_would_you_enforce_that_row-based_replication_is_used_eve.md>)
- [Expert] [A checksum mismatch shows drift on one table between master and replica. How do you repair the replica with pt-table-sync safely?](<../../../../PRACTICE/01_Databases_and_SQL/Database_Operations/Replication/Q-21201_A_checksum_mismatch_shows_drift_on_one_table_between_master_.md>)
- [Expert] [How would you architect online schema evolution (gh-ost / pt-osc) across a replicated topology so a DDL on the master never stalls the whole apply pipeline?](<../../../../PRACTICE/01_Databases_and_SQL/Database_Operations/Replication/Q-21298_How_would_you_architect_online_schema_evolution_gh-ost_pt-os.md>)

## Related concepts

- [Zero Downtime Sharding Migrations](<../../../../07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Distributed_Joins_in_MPP_Systems/Zero_Downtime_Sharding_Migrations.md>) *(Distributed Systems Core)*
- [Schema Migration Strategy](<../../Migrations/Concurrent_Index_Builds/Schema_Migration_Strategy.md>) *(Migrations)*
- [Schema Changes Under Long Transactions](<../../../../04_Data_Architecture_and_Strategy/Platform_Strategy/Schema_Evolution/Schema_Changes_Under_Long_Transactions.md>) *(Schema Evolution)*

