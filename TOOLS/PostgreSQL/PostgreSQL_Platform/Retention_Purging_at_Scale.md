---
id: "TOOLS/PostgreSQL/PostgreSQL_Platform/Retention_Purging_at_Scale"
level: 4
type: "concept"
topic: "TOOLS/PostgreSQL/PostgreSQL_Platform"
tools: []
---

# Retention Purging at Scale

**L4 Advanced** · [↑ PostgreSQL Platform](<_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** Deleting old rows from a huge PostgreSQL table with plain `DELETE` is slow, WAL-heavy and creates bloat that doesn't shrink the file — so for time-based retention you partition by time and `DETACH`/`DROP` whole partitions, falling back to batched deletes plus vacuum (and `pg_repack` for space) only when partitioning isn't possible.

### 1. The big picture
Every append-only dataset — events, logs, audit trails, metrics — needs a retention policy: keep 90 days, drop the rest. The naive implementation, `DELETE FROM events WHERE created_at < now() - interval '90 days'`, looks harmless but is one of the most common production incidents: the statement runs for hours, generates gigabytes of WAL (which replicas must replay, causing lag), holds a snapshot that blocks vacuum, and at the end the table is *not smaller* on disk.

The reason is MVCC. PostgreSQL never removes a row in place: `DELETE` only marks the tuple dead. Space is reclaimed later by vacuum and, even then, is reusable *inside* the table rather than returned to the operating system. The cure is to make retention a **metadata operation** on a whole partition — dropping a partition is a file removal, not a row-by-row operation. This lesson ties together partitioning, vacuum/bloat, WAL/replication and backup strategy.

### 2. Core concepts and vocabulary
- **Retention policy** — rule for how long data is kept (e.g. 90 days hot, 1 year archive).
- **Dead tuple** — a row version no transaction can see anymore, awaiting vacuum.
- **Bloat** — table/index space occupied by dead tuples or free space not returned to the OS.
- **Declarative partitioning** — a parent table routed into child tables by `RANGE`, `LIST` or `HASH` on a partition key.
- **Partition pruning** — the planner/executor skipping partitions that cannot match the `WHERE` clause.
- **`DETACH PARTITION [CONCURRENTLY]`** — removes a partition from the parent, leaving it as a standalone table.
- **`DROP TABLE` (on a partition)** — deletes the underlying files immediately.
- **`TRUNCATE`** — empties a table by swapping in new storage; cheap, but takes `ACCESS EXCLUSIVE`.
- **TOAST** — out-of-line storage for large values (big `jsonb`/`text`); has its own table to vacuum.
- **`pg_repack`** — extension that rebuilds a table and indexes online and returns space to the OS.
- **`VACUUM FULL`** — rewrites the table under an exclusive lock; reclaims space but blocks everything.

### 3. How it works, step by step
**Partition-based retention (preferred):**
1. Create the parent partitioned by the time column (often daily or monthly).
2. A scheduler (`pg_partman`, cron, or your own job) pre-creates future partitions so inserts never fail.
3. Inserts are routed by the partition key to the matching child; queries on recent time ranges prune old partitions.
4. When a partition falls out of the window: `ALTER TABLE events DETACH PARTITION events_2026_06 CONCURRENTLY;`
5. Optionally archive (`COPY` to object storage, or `pg_dump -t`), then `DROP TABLE events_2026_06;`.
6. The files disappear immediately — no dead tuples, no vacuum, negligible WAL (only the catalog change).

**Batched-delete retention (when you cannot partition):**
1. Delete in small slices — e.g. 5–10k rows per transaction — ordered by an indexed column so each slice is a short index range scan.
2. Pause between batches (`pg_sleep`) or watch replica lag; keep each transaction short so vacuum can keep up.
3. Let autovacuum clean the dead tuples; run `VACUUM (ANALYZE)` after large purges.
4. Only if you must shrink the file, run `pg_repack` (or `VACUUM FULL` in a maintenance window).

| Method | Speed | WAL | Reclaims disk to OS | Locks | Needs design up front |
|---|---|---|---|---|---|
| Single big `DELETE` | very slow | huge | no | row locks, long snapshot | no |
| Batched `DELETE` + vacuum | slow, steady | high | no (reusable only) | brief row locks | no |
| `DETACH` + `DROP` partition | instant | tiny | **yes** | short, `CONCURRENTLY` available | **yes** — partition key |
| `TRUNCATE` | instant | tiny | yes | `ACCESS EXCLUSIVE` | whole table only |
| `pg_repack` | slow (rewrite) | high | yes | brief at start/end | extension, 2× space |

### 4. Worked example
```sql
-- Parent table, partitioned by month
CREATE TABLE events (
  id         bigint GENERATED ALWAYS AS IDENTITY,
  tenant_id  int          NOT NULL,
  payload    jsonb        NOT NULL,
  created_at timestamptz  NOT NULL,
  PRIMARY KEY (id, created_at)          -- partition key must be in the PK
) PARTITION BY RANGE (created_at);

CREATE TABLE events_2026_09 PARTITION OF events
  FOR VALUES FROM ('2026-09-01') TO ('2026-10-01');
CREATE TABLE events_2026_10 PARTITION OF events
  FOR VALUES FROM ('2026-10-01') TO ('2026-11-01');

-- Pruning check: only events_2026_10 should appear in the plan
EXPLAIN SELECT count(*) FROM events
WHERE created_at >= '2026-10-05' AND created_at < '2026-10-06';

-- Retention: remove the month that is out of the window
ALTER TABLE events DETACH PARTITION events_2026_09 CONCURRENTLY;
-- (optional) COPY events_2026_09 TO PROGRAM '...';  -- archive first
DROP TABLE events_2026_09;
```
Reading the plan: you should see an `Append` node with a single child scan (older ones are removed at plan time for constants, or shown as `(never executed)` for runtime pruning with parameters). If every partition appears, your predicate does not constrain the partition key directly (for example a function wrapped around `created_at`).

Batched fallback for a non-partitioned table:
```sql
DO $$
DECLARE n int;
BEGIN
  LOOP
    DELETE FROM events
    WHERE ctid IN (
      SELECT ctid FROM events
      WHERE created_at < now() - interval '90 days'
      LIMIT 5000
    );
    GET DIAGNOSTICS n = ROW_COUNT;
    EXIT WHEN n = 0;
    COMMIT;                         -- allowed in a DO block called outside a transaction (PG 11+)
    PERFORM pg_sleep(0.2);
  END LOOP;
END $$;
```
Note this needs an index on `created_at` or each batch will rescan the table.

### 5. Under the hood
- **Why DELETE is expensive:** each deleted row sets `xmax` on the tuple, writes a WAL record, updates every index entry's visibility lazily, and later vacuum must scan the heap and indexes to remove it. Index entries are removed only by vacuum, so indexes bloat too.
- **Why the file does not shrink:** vacuum marks space free in the free-space map; only trailing empty pages can be truncated back to the OS. Free space in the middle stays. That is why `VACUUM FULL`/`pg_repack` exist.
- **Dropping a partition:** removes the catalog entries and unlinks the relation files — O(number of files), not O(rows). `DETACH … CONCURRENTLY` (PG 14+) uses a two-transaction protocol that avoids a long `ACCESS EXCLUSIVE` lock on the parent, but it cannot run inside a transaction block and cannot be used if a default partition exists.
- **Foreign keys:** a foreign key *referencing* a partitioned table is supported since PG 12; foreign keys *from* the partition to other tables are just dropped with it. Dropping a partition that others reference by FK fails — you either drop the referencing rows first or model without FK.
- **Unique constraints and indexes** on partitioned tables must include the partition key, so global uniqueness across time needs a different design (an `id` plus `created_at` composite, or a UUID with no global unique index).
- **TOAST:** large `jsonb` values live in a separate TOAST table that is vacuumed separately. A wide, rarely-read `jsonb` column can make the heap look small while TOAST is huge; dead TOASTed rows still count toward vacuum work and WAL. Dropping the partition removes heap, TOAST and indexes together.

### 6. Performance and design implications
- Choose partition size so a retention unit is **one partition**: daily for high-volume, monthly for moderate. Aim for tens to low hundreds of partitions, not tens of thousands (planning time grows with partition count).
- Always include the partition key in queries; add a retention-window predicate to application queries so pruning works even if old partitions linger.
- Physical backup strategy changes too: `pg_dump -j N -Fd` parallelizes *per table* — partitioned tables dump in parallel by partition, and keeping old partitions out of the database also shortens dumps. For fast recovery, prefer `pg_basebackup` plus continuous WAL archiving (PITR): restore is file copy plus replay rather than re-loading and re-indexing a logical dump, at the cost of storing a full physical copy and archived WAL.
- A single huge `DELETE` also holds a long transaction, which pins the xmin horizon: every table in the database accumulates bloat until it ends. Monitor `pg_stat_activity` for old `xact_start`.
- Replica impact: big deletes mean WAL volume → replay lag; partition drops almost don't.
- Monitor with `pg_total_relation_size()`, `pg_stat_user_tables.n_dead_tup`, `pgstattuple` or `pg_stat_progress_vacuum`.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `PARTITION BY RANGE (col)` | Create time-partitioned parent | New append-only tables |
| `ALTER TABLE … DETACH PARTITION … CONCURRENTLY` | Remove partition with minimal locking | Online retention |
| `DROP TABLE partition` | Free disk instantly | After archive |
| `pg_partman` | Automates partition creation and retention | Production partitioning |
| `TRUNCATE` | Instantly empty a table | Staging/temp tables |
| `VACUUM (ANALYZE, VERBOSE)` | Reclaim dead tuples, refresh stats | After batched purges |
| `pg_repack` | Online rewrite to return space | Bloated non-partitioned table |
| `autovacuum_vacuum_scale_factor` (per table) | Trigger vacuum earlier on big tables | Large tables with heavy deletes |
| `pg_dump -Fd -j N` | Parallel logical dump by table | Backups of partitioned data |
| `pg_basebackup` + WAL archive | Physical base backup + PITR | Fast restore, warm standby |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| Partition + drop for time-series with one clear retention rule | Retention depends on per-row attributes (tenant, status) — partitions can't express it |
| Batched delete for modest tables or per-tenant erasure | Table is multi-TB — the purge never finishes before new data arrives |
| `pg_repack` to compact a bloated table once | Disk is >50% full (needs ~2× space), or no primary key (required) |
| `TRUNCATE` on staging/ephemeral data | Table is referenced by FKs or needs concurrent reads |
| Archive to object storage before drop | Legal "right to erasure" requires deletion from archives too |

### 9. Common mistakes
- Running one giant `DELETE` → hours of runtime, replica lag, bloat, no space back → partition or batch.
- Forgetting to pre-create future partitions → inserts fail at midnight on month boundary → automate creation and add a default-partition alarm.
- Partition key missing from the primary key → `CREATE TABLE` error or duplicate ids across partitions → include it and rethink global uniqueness.
- `DETACH` without `CONCURRENTLY` on a busy parent → blocks queries → use it, outside a transaction block.
- Trusting `VACUUM` to shrink files → disk still full → use `pg_repack` or drop partitions.
- Purging at peak hour → latency spike → schedule low-traffic window and throttle.

### 10. 🎯 Interview kit
**30-second answer:** For time-based retention I partition by time and drop whole partitions: it's a metadata/file operation with almost no WAL and instantly returns disk space. A plain `DELETE` leaves dead tuples, generates huge WAL, and doesn't shrink files. For non-partitionable tables I delete in small batches, let vacuum catch up, and use `pg_repack` if I truly need space back.

**Structure a longer answer:** clarify volume, query pattern and retention rule → explain MVCC cost of `DELETE` → partition design (key, size, pre-creation, pruning) → drop/detach/archive flow with locking → fallbacks and monitoring.

**Likely follow-ups:**
- *Why doesn't DELETE free disk space?* → MVCC marks dead; vacuum frees internally; only `FULL`/repack/drop returns to OS.
- *How does pruning work?* → planner (and executor for parameters) excludes partitions whose bounds contradict `WHERE`.
- *What complicates dropping partitions?* → FKs referencing it, unique indexes needing the key, default partitions with `CONCURRENTLY`.
- *How do you speed a 300 GB dump?* → directory format with `-j`, which parallelizes across tables; or switch to physical backup + WAL archive.
- *What is TOAST's role?* → large values stored out of line; separate vacuum and bloat.

**Signals interviewers look for:** you reason from MVCC and WAL, not folklore; you plan operations (pre-creation, locks, archival); you know the trade-offs of each purge method.

### 11. Check yourself
- [ ] Explain why a 90-day `DELETE` on a 2 TB table doesn't reduce table size.
- [ ] Design a monthly range-partitioned table, including pruning and retention steps.
- [ ] Say what `DETACH PARTITION CONCURRENTLY` restricts.
- [ ] Write a safe batched-delete loop and name what to monitor.
- [ ] Compare `VACUUM FULL`, `pg_repack` and partition drop for space reclaim.
- [ ] Describe how `pg_dump -j` and `pg_basebackup` + WAL archiving differ for recovery speed.
<!-- explainer:end -->

## Practice questions

- [Intermediate] [A pg_dump of a 300GB database takes many hours and pins a snapshot that blocks vacuum; how would you speed backups with parallel directory-format dumps and what does the -j flag actually parallelize?](<../../../PRACTICE/TOOLS/PostgreSQL/PostgreSQL_Platform/Q-36243_A_pg_dump_of_a_300GB_database_takes_many_hours_and_pins_a_sn.md>)
- [Intermediate] [How would you take a backup of a PostgreSQL database? Explain pg_dump vs a physical backup.](<../../../PRACTICE/TOOLS/PostgreSQL/PostgreSQL_Platform/Q-36381_How_would_you_take_a_backup_of_a_PostgreSQL_database_Explain.md>)
- [Deep] [Your queries against a partitioned table are slow because the planner locks and inspects every partition at plan time; how does runtime partition pruning differ from plan-time pruning and when does each kick in?](<../../../PRACTICE/TOOLS/PostgreSQL/PostgreSQL_Platform/Q-36241_Your_queries_against_a_partitioned_table_are_slow_because_th.md>)
- [Deep] [You need to reclaim space from a bloated table without an exclusive lock that VACUUM FULL takes; explain how pg_repack works online and what edge cases (triggers, exclusion constraints) make it risky.](<../../../PRACTICE/TOOLS/PostgreSQL/PostgreSQL_Platform/Q-36284_You_need_to_reclaim_space_from_a_bloated_table_without_an_ex.md>)
- [Deep] [You need to store 5MB documents; would you use bytea in Postgres, large objects (lo), or S3 with a URL, and how do TOAST limits, backup size, and replication traffic drive that decision?](<../../../PRACTICE/TOOLS/PostgreSQL/PostgreSQL_Platform/Q-36305_You_need_to_store_5MB_documents_would_you_use_bytea_in_Postg.md>)
- [Deep] [You retire old time-series data monthly; explain why DETACH PARTITION plus DROP is dramatically better than a huge DELETE, and what FK and index edge cases complicate detaching a partition.](<../../../PRACTICE/TOOLS/PostgreSQL/PostgreSQL_Platform/Q-36349_You_retire_old_time-series_data_monthly_explain_why_DETACH_P.md>)
- [Deep] [How does declarative partitioning route rows, and what determines which partition an INSERT lands in?](<../../../PRACTICE/TOOLS/PostgreSQL/PostgreSQL_Platform/Q-35910_How_does_declarative_partitioning_route_rows_and_what_determ.md>)
- [Deep] [A large DELETE on a partitioned table is slow and generates massive WAL; why is detaching and dropping the whole partition dramatically better for time-based retention, and what edge cases (FKs, indexes) complicate it?](<../../../PRACTICE/TOOLS/PostgreSQL/PostgreSQL_Platform/Q-35810_A_large_DELETE_on_a_partitioned_table_is_slow_and_generates_.md>)

## Related concepts

- [Partitioning and Bucketing Design](<../../../02_Data_Engineering/Storage_and_Lakehouse/Partitioning_and_Bucketing/Partitioning_and_Bucketing_Design.md>) *(Partitioning and Bucketing)*
- [Partitioning Strategy](<../../../01_Databases_and_SQL/Database_Operations/Partitioning/Partition_Pruning/Partitioning_Strategy.md>) *(Partitioning)*
- [Large Values and TOAST](<../../../01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Large_Values_and_TOAST.md>) *(SQL Fundamentals)*

