---
id: "01_Databases_and_SQL/Database_Operations/Migrations/Migrating_to_Partitioned_Tables"
level: 5
type: "deep-dive"
topic: "01_Databases_and_SQL/Database_Operations/Migrations"
tools: []
---

# Migrating to Partitioned Tables

**L5 Expert** · [↑ Migrations](<../_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** To turn a large live table into a partitioned one, you build the new layout beside the old, copy data in paced batches while keeping both in sync, then cut over with a short, rehearsed switch.

### 1. The big picture
Partitioning splits one logical table into physical pieces, usually by date range. It pays off for pruning, maintenance (switch or drop an old partition instead of deleting millions of rows) and data movement. But an existing unpartitioned table cannot simply be "altered into" a partitioned one without rewriting it, and a rewrite of a multi-terabyte table that serves traffic is a migration project.

The books make two points that frame it. First, a partitioned table is *easier to move*: older, unchanging data can live in separate files that can be copied ahead of time, and only the recent partitions need a final catch-up. Second, data migration in general is mostly about schema management and moving *connections*, not only bytes: bulk movement through file or object storage beats row-at-a-time, and tools should be considered before hand-writing a mover.

See Schema Migration Strategy for the surrounding process and Concurrent Index Builds for the lock-avoidance techniques used here.

### 2. Core concepts and vocabulary
- **Partition key** — the column that decides which partition a row lives in; must be chosen for query pruning and maintenance.
- **Partition function / scheme (SQL Server)** — define the boundaries and map partitions to filegroups.
- **Declarative partitioning (PostgreSQL)** — `PARTITION BY RANGE/LIST/HASH` with child tables.
- **Partition switching** — moving a whole partition in or out as a metadata-only operation.
- **Shadow table** — the new, partitioned copy built beside the original.
- **Dual write / trigger sync** — keeping the new table current while the old one is still the source of truth.
- **Cutover** — the moment reads and writes move to the new table.
- **Backfill** — copying historical rows in batches.
- **Replication lag** — how far followers are behind; a main constraint on batch size.

### 3. How it works, step by step
1. **Analyze.** Size and row width, growth rate, read and write patterns, and which queries filter on which columns. This decides the partition key and granularity.
2. **Design.** Choose the key and boundaries. Note the engine rules: in many systems the partition key must be part of unique and primary keys, which can force a key change. Plan how foreign keys that reference the table are handled.
3. **Create the shadow table** with the same columns, the partitioned layout and the needed indexes. Create future partitions ahead of time.
4. **Start capturing changes.** Add triggers, dual writes in the application, or logical replication so inserts, updates and deletes on the old table are applied to the new one.
5. **Backfill in chunks.** Copy by key range, 10k to 100k rows per batch depending on row size, committing each batch and pausing based on lag, I/O and lock waits. Prefer oldest-first; write into partitions that are not yet hot.
6. **Verify.** Row counts per range, checksums or sampled comparisons, constraint checks, and run the real queries to confirm pruning.
7. **Refresh statistics** on the new table so the optimizer is not guessing at cutover.
8. **Cut over.** In a short window: stop writes (or take a brief lock), apply the last changes, rename tables (or switch the view or synonym), re-enable writes, and verify.
9. **Keep the old table** for a rollback period, then drop it after confidence builds.

Alternatives for very large systems: **logical replication cutover** (stream to a new, differently shaped table or server, then redirect), **shadow-table swap** (above), and **blue-green database** (a full second environment). Logical replication minimizes downtime but takes longest to catch up; blue-green is simplest to reason about but costs double the infrastructure and needs extensive testing.

### 4. Worked example
```sql
-- SQL Server: partition an existing orders table by month via a shadow table
CREATE PARTITION FUNCTION pf_month (date)
  AS RANGE RIGHT FOR VALUES ('2026-01-01','2026-02-01','2026-03-01');
CREATE PARTITION SCHEME ps_month AS PARTITION pf_month ALL TO ([PRIMARY]);

CREATE TABLE dbo.orders_new (
    order_id    bigint NOT NULL,
    order_date  date   NOT NULL,
    customer_id int    NOT NULL,
    CONSTRAINT pk_orders_new PRIMARY KEY CLUSTERED (order_date, order_id)
) ON ps_month (order_date);

-- Batched backfill by key range
DECLARE @lo bigint = 0, @step bigint = 50000, @max bigint = (SELECT MAX(order_id) FROM dbo.orders);
WHILE @lo <= @max
BEGIN
    INSERT dbo.orders_new (order_id, order_date, customer_id)
    SELECT order_id, order_date, customer_id
    FROM dbo.orders
    WHERE order_id > @lo AND order_id <= @lo + @step;
    SET @lo += @step;
    WAITFOR DELAY '00:00:01';      -- pacing; tune from lag and I/O
END

-- Cutover (short window)
BEGIN TRAN;
  EXEC sp_rename 'dbo.orders', 'orders_old';
  EXEC sp_rename 'dbo.orders_new', 'orders';
COMMIT;
```
Note the primary key became `(order_date, order_id)`: the partition column had to join the key. Anything that assumed `order_id` was unique alone, such as foreign keys from child tables, must be reworked. After cutover, check the plan for a date-filtered query and confirm that only the relevant partitions are touched.

### 5. Under the hood
- **Why batching:** one 500M-row `UPDATE` or `INSERT` produces one huge log (WAL) volume and one long transaction; followers fall behind, locks last too long and bloat or log growth follows. Batches bound each of these.
- **Partition switch** is a metadata change: no data moves, which is why loading and aging out data via switch is fast. It requires identical structure and an enforced boundary check constraint on the source.
- **Clustered key choice** matters: moving from a sequential bigint to a random UUID changes insert locality, causes page splits and write amplification, and should be mitigated deliberately (sequential GUID variants, larger fill-factor slack).
- **Repartitioning range to hash** (a different layout of an already-partitioned table) is the same pattern again: new table, sync, backfill, swap, and foreign keys have to be recreated against the new parent.
- **Multi-tenant fleets:** very large tenants must be migrated separately with their own pacing so they do not stall the queue while small tenants finish quickly.

### 6. Performance and design implications
- Pruning only happens when queries filter on the partition key in a form the optimizer can use; test it.
- Too many tiny partitions hurt planning time and metadata; too few lose the maintenance benefit.
- Pace the backfill from live signals (replication lag, I/O queue, lock waits) rather than fixed constants.
- Refresh statistics right after load; stale statistics make the first hours after cutover slow.
- Measure: partition row counts, `sys.partitions` (SQL Server), query plans for pruning, replication lag.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `CREATE PARTITION FUNCTION / SCHEME` | Define boundaries and placement (SQL Server) | Building the new layout |
| `PARTITION BY RANGE (...)` | Declarative partitioning (PostgreSQL) | Same, in PostgreSQL |
| `ALTER TABLE ... SWITCH PARTITION` | Metadata-only move of a partition | Loading and aging out |
| `sp_rename` / `ALTER TABLE ... RENAME` | Swap names | Cutover |
| Logical replication / CDC | Stream changes to a new table | Near-zero downtime cutovers |
| `UPDATE STATISTICS` / `ANALYZE` | Refresh statistics | Immediately after backfill |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| Shadow-table swap | You cannot keep the two tables in sync under heavy writes |
| Logical-replication cutover | Long catch-up is unacceptable or the schema difference is large |
| Blue-green database | Budget cannot cover double infrastructure |
| Partition by date | Queries rarely filter by date |
| Hash partitioning | You need range-based aging out |

### 9. Common mistakes
- Not planning for the partition key in unique keys → build fails late → redesign keys first.
- Backfill with one statement → lag spike, huge log → chunk and pace.
- Forgetting foreign keys referencing the old table → broken constraints at cutover → plan the FK strategy beforehand.
- Skipping statistics refresh → bad plans after cutover → run it before switching traffic.
- Dropping the old table immediately → no rollback → keep it for a period.
- Never rehearsing the cutover → surprises during the window → rehearse in staging with real volume.

### 10. 🎯 Interview kit
**30-second answer:** Build a partitioned shadow table, keep it in sync with triggers or replication, backfill in paced batches, verify, refresh stats, then cut over by a short rename or switch, keeping the old table for rollback.

**Structure a longer answer:** clarify size, QPS, downtime tolerance, key and FKs → mechanism (shadow, sync, backfill, cutover) → trade-offs (logical replication vs shadow swap vs blue-green) → scenario: 20 TB at 200k QPS.

**Likely follow-ups:**
- *How do you pace a backfill?* → From replication lag, I/O and lock waits, with a batch size that bounds transaction length.
- *What about foreign keys?* → Recreate against the new parent, validated in phases.
- *What can go wrong at cutover?* → Late writes, plan changes, forgotten consumers; make cutover reversible.
- *How do you handle statistics?* → Refresh before cutover, incrementally if large.

**Signals interviewers look for:** you think in phases, protect replication, rehearse, and keep a rollback.

### 11. Check yourself
- [ ] Explain why the partition key must often join the primary key and what it breaks.
- [ ] Design a batched backfill with pacing from live signals.
- [ ] Compare logical replication, shadow swap and blue-green for a 20 TB table.
- [ ] Describe the cutover window step by step, and its rollback.
- [ ] Explain how partition switching is metadata-only.
<!-- explainer:end -->

## Practice questions

- [Intermediate] [Why do many teams prefer timestamp-based migration ordering over sequential integers? What problem does it solve?](<../../../../PRACTICE/01_Databases_and_SQL/Database_Operations/Migrations/Q-20713_Why_do_many_teams_prefer_timestamp-based_migration_ordering_.md>)
- [Deep] [How do lazy (on-read) versus eager (bulk) schema migrations differ for a NoSQL store, and when do you pick each?](<../../../../PRACTICE/01_Databases_and_SQL/Database_Operations/Migrations/Q-20752_How_do_lazy_on-read_versus_eager_bulk_schema_migrations_diff.md>)
- [Deep] [Some tenants are 10000x larger than the median. How do you shape a fleet-wide migration so whales don't stall the queue and small tenants don't wait days?](<../../../../PRACTICE/01_Databases_and_SQL/Database_Operations/Migrations/Q-20799_Some_tenants_are_10000x_larger_than_the_median_How_do_you_sh.md>)
- [Deep] [How do you plan the ANALYZE / statistics-refresh step of a big migration so the optimizer doesn't chase stale stats on the newly-shaped table right after cutover?](<../../../../PRACTICE/01_Databases_and_SQL/Database_Operations/Migrations/Q-20839_How_do_you_plan_the_ANALYZE_statistics-refresh_step_of_a_big.md>)
- [Expert] [A backfill of 500M rows in one UPDATE statement blew up the WAL and replication lag spiked. How would you have chunked it?](<../../../../PRACTICE/01_Databases_and_SQL/Database_Operations/Migrations/Q-20670_A_backfill_of_500M_rows_in_one_UPDATE_statement_blew_up_the_.md>)
- [Expert] [How would you migrate a large table to a partitioned table with minimal downtime? What's your cutover plan?](<../../../../PRACTICE/01_Databases_and_SQL/Database_Operations/Migrations/Q-20705_How_would_you_migrate_a_large_table_to_a_partitioned_table_w.md>)
- [Expert] [You must migrate a 20TB table to a new physical layout while it serves 200k QPS. Compare a logical-replication cutover, a shadow-table swap, and a blue-green DB approach.](<../../../../PRACTICE/01_Databases_and_SQL/Database_Operations/Migrations/Q-20790_You_must_migrate_a_20TB_table_to_a_new_physical_layout_while.md>)
- [Expert] [A migration must repartition an existing 10TB partitioned table from range to hash partitioning. Design the phased cutover including how existing FKs and queries are handled.](<../../../../PRACTICE/01_Databases_and_SQL/Database_Operations/Migrations/Q-20832_A_migration_must_repartition_an_existing_10TB_partitioned_ta.md>)

## Related concepts

- [Migrations with Read Your Writes](<../../../../04_Data_Architecture_and_Strategy/Platform_Strategy/Schema_Evolution/Migrations_with_Read_Your_Writes.md>) *(Schema Evolution)*
- [Partitioning Strategy](<../../Partitioning/Partition_Pruning/Partitioning_Strategy.md>) *(Partitioning)*
- [Zero Downtime Sharding Migrations](<../../../../07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Distributed_Joins_in_MPP_Systems/Zero_Downtime_Sharding_Migrations.md>) *(Distributed Systems Core)*

## References

- *Analytics Optimization with Columnstore Indexes in Microsoft SQL Server* — Edward Pollack — Segment and rowgroup elimination › Faster Data Movement/Migration
- *Fundamentals of Data Engineering* — Joe Reis, Matt Housley — Ingestion › Data Migration
- *Fundamentals of Data Engineering* — Joe Reis, Matt Housley — Ingestion › Databases and File Export
- *Ultimate AWS Data Engineering* — Rathish Mohan — Partitioning and Sharding Optimizing Data Management › Challenges and Best Practices for Managing Partitioning and Sharding
- *Databricks Certified Associate Developer for Apache Spark* — Saba Shah — Python › Wide transformations

