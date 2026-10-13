---
id: "01_Databases_and_SQL/Database_Operations/Partitioning/Partitioning_Strategy"
level: 5
type: "deep-dive"
topic: "01_Databases_and_SQL/Database_Operations/Partitioning"
tools: []
---

# Partitioning Strategy

**L5 Expert** · [↑ Partitioning](<../_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** Partitioning splits one large logical table into smaller physical pieces by a key, so queries can skip pieces and maintenance (retention, loading, archiving) can act on a whole piece at once, but only if the key matches how the table is actually queried and managed.

### 1. The big picture
A table that grows without bound eventually hurts in three ways: scans touch too much data, indexes get deep and no longer fit in memory, and housekeeping (deleting old rows, rebuilding indexes, vacuuming, backing up) takes hours and holds locks. **Partitioning** attacks all three by dividing the table into **partitions**, each a separate physical segment with its own storage and (usually) its own local indexes, while the application still sees one table.

Partitioning is a *physical design* decision layered on top of a *logical* schema. It sits between indexing (make lookups cheap) and sharding (spread data across machines). Partitioning normally stays on one server; sharding places pieces on different servers and pushes routing into the application or a proxy. The mental model: partitioning is "one database, many drawers"; sharding is "many databases, one filing scheme".

The strategy question is therefore not "how do I partition?" but "what do I want to be cheap?" The honest answers are: pruning (skip partitions in queries), fast bulk removal (drop a partition instead of a multi-hour DELETE), fast bulk load (build offline, then attach), tiered storage (old partitions on cheap disks), and smaller maintenance units. If none of these is a real need, partitioning adds complexity for no gain. A related warning from the SQL Server literature: if the code in your workload is not compatible with partition elimination, avoid partitioning, because it can hurt performance rather than help.

### 2. Core concepts and vocabulary
- **Partition key** — the column(s) whose value decides which partition a row belongs to.
- **Range partitioning** — each partition holds a contiguous interval of key values (for example one month of `order_date`). The most common choice for time-series data.
- **List partitioning** — each partition holds an explicit set of values (for example `region IN ('EU','UK')` or `status = 'archived'`).
- **Hash partitioning** — a hash of the key modulo N picks the partition; spreads rows evenly but destroys ordering, so range predicates cannot prune.
- **Composite (sub-)partitioning** — two methods combined, e.g. monthly range partitions, each hash-split into 8 sub-partitions.
- **Partition boundary** — the value separating two ranges. Whether it is inclusive or exclusive differs by engine (`VALUES LESS THAN` in MySQL/Oracle, `FROM … TO` upper-exclusive in PostgreSQL, `RANGE LEFT/RIGHT` in SQL Server).
- **Default / catch-all partition** — receives rows that match no defined partition (`MAXVALUE`, `DEFAULT`).
- **Partition pruning / elimination** — the optimizer skipping partitions that cannot contain matching rows (see the Partition Pruning lesson).
- **Aligned (local) index** — an index partitioned the same way as the table; allows partitions to be dropped or switched cheaply. A **global index** spans all partitions.
- **Partition maintenance** — add, split, merge, drop, detach/attach, exchange/switch of partitions.

### 3. How it works, step by step
1. **Define the scheme.** You declare the key and the boundaries. In MySQL-style syntax:
   ```sql
   CREATE TABLE borrowed_books (
     borrow_id   INT,
     member_id   INT,
     book_id     INT,
     borrow_date DATE NOT NULL,
     return_date DATE
   )
   PARTITION BY RANGE (YEAR(borrow_date)) (
     PARTITION p_before_2020 VALUES LESS THAN (2020),
     PARTITION p_2020        VALUES LESS THAN (2021),
     PARTITION p_2021        VALUES LESS THAN (2022),
     PARTITION p_recent      VALUES LESS THAN (MAXVALUE)
   );
   ```
2. **Route on write.** For each inserted row the engine evaluates the partition expression and writes the row to exactly one partition. If no partition matches and there is no default, the insert fails.
3. **Prune on read.** The planner compares the query predicate with partition bounds and drops partitions that cannot match. Only survivors are opened.
4. **Move on update.** If an UPDATE changes the key so the row belongs elsewhere, the engine must delete it from one partition and insert it into another (Oracle requires `ENABLE ROW MOVEMENT`; PostgreSQL does this automatically). That is a heavier operation than an in-place update.
5. **Maintain by partition.** Rolling-window retention becomes: create next period's partition ahead of time, then detach/drop the oldest one. Dropping is a metadata operation, so a 30-day-retention job that would otherwise be `DELETE … WHERE datetime < now() - 30 days` (slow, log-heavy, bloats the table) becomes near-instant.

| Method | Good at | Bad at |
|---|---|---|
| Range | time series, retention, range pruning, archive tiers | hot newest partition takes all writes |
| List | discrete categories, tenant/region isolation, status lifecycle | unbounded or skewed category sets |
| Hash | even spread, point lookups, avoiding write hot-spots | range pruning, easy retention |
| Composite | range for lifecycle + hash for spread | many partitions, more metadata and complexity |

### 4. Worked example
A PostgreSQL `events` table receiving ~50 M rows/day, queried mostly for "last 7 days", with 90-day retention.

```sql
CREATE TABLE events (
  event_id   bigint GENERATED ALWAYS AS IDENTITY,
  tenant_id  int       NOT NULL,
  created_at timestamptz NOT NULL,
  payload    jsonb,
  PRIMARY KEY (event_id, created_at)      -- key must include partition column
) PARTITION BY RANGE (created_at);

CREATE TABLE events_2026_10_12 PARTITION OF events
  FOR VALUES FROM ('2026-10-12') TO ('2026-10-13');
CREATE INDEX ON events (tenant_id, created_at);   -- cascades to every partition

-- daily job
CREATE TABLE events_2026_10_15 PARTITION OF events
  FOR VALUES FROM ('2026-10-15') TO ('2026-10-16');
ALTER TABLE events DETACH PARTITION events_2026_07_14 CONCURRENTLY;
DROP TABLE events_2026_07_14;
```

What to notice: (a) the primary key had to include `created_at`, because a unique constraint on a partitioned table must contain the partition key; (b) the retention step touches no live rows and generates almost no WAL; (c) the partitions must be created *ahead of time*, otherwise inserts for a future date fail, so the creation job is production-critical and needs alerting.

### 5. Under the hood
- **Each partition is a real table.** In PostgreSQL, partitions are ordinary tables with a parent that holds only metadata; in SQL Server, a partition function maps values to partition numbers and a partition scheme maps those to filegroups; in MySQL InnoDB each partition is its own tablespace/file set. This explains why per-partition indexes are small (shallower B-trees, better cache hit rates) and why per-partition operations are cheap.
- **Metadata cost.** Planning time grows with partition count, because the planner must examine each partition's bounds (unless pruning runs at the partition-bound level in a tree/binary search, which modern versions do for range and hash). Thousands of partitions mean larger catalog caches, slower planning and more open file handles; this is the "over-partitioning" failure. Tens to low hundreds is comfortable; many thousands needs justification.
- **Unique keys and the partition column.** An index on a partitioned table is physically a set of per-partition indexes. The engine can only guarantee global uniqueness cheaply if the uniqueness check stays within one partition, hence the rule that unique/primary keys include the partition key. Consequence: you often cannot enforce "email is unique across all time" with a plain constraint.
- **Foreign keys.** Support varies: PostgreSQL allows foreign keys from partitioned tables (since 11) and referencing partitioned tables (since 12); MySQL does not support foreign keys on partitioned InnoDB tables at all; SQL Server allows them but the referenced unique key must include the partition column in practice. Workarounds (triggers, application checks, periodic reconciliation queries) shift integrity enforcement out of the engine.
- **Statistics.** Statistics are kept per partition (or per table and aggregated). With very many hot partitions, keeping them fresh is a cost; stale statistics make pruning-aware plans worse.
- **Partition-wise operations.** When two tables are partitioned compatibly, the engine can join partition-to-partition (partition-wise join) and aggregate each partition independently before combining (partition-wise aggregation), enabling parallelism with less memory per worker. This is often off by default in PostgreSQL (`enable_partitionwise_join`, `enable_partitionwise_aggregate`).
- **Replicas.** Logical or physical replication ships partition DDL like any DDL. On physical replicas it arrives in log order; on logical replication setups, new partitions typically must be created on subscribers too, otherwise replication stalls on the missing target table. Plan DDL rollouts so replicas and primaries stay structurally in sync.

### 6. Performance and design implications
- **Choose the key from the query log, not from the data volume.** The key should appear as an equality or range filter in the majority of heavy queries. A date key is useless if every query filters by `customer_id`.
- **Size rule of thumb.** For engines without strong partition tooling (and lakehouse tables such as Delta), do not partition below roughly 1 TB; compact small files instead. Over-partitioning creates many tiny files or segments and swamps metadata.
- **Partition size target.** Aim for partitions large enough to amortize overhead (hundreds of MB to tens of GB) yet small enough to maintain in a maintenance window.
- **Skew.** Date partitions are naturally skewed (this month is hot, five years ago is cold). Hash sub-partitioning is justified when one partition receives so much write or lookup traffic that a single b-tree or lock becomes the bottleneck; otherwise it is complexity for little gain.
- **Measure.** Compare partitions scanned vs. partitions total in the plan, logical reads before/after, and planning time. Triage order when a big table regresses: first the plan (is pruning happening?), then statistics and indexes, then buffer cache hit ratio and I/O waits; partitioning is the culprit only if plans show unexpected all-partition scans or planning time has exploded.
- **Columnar/lakehouse note.** In columnar storage choose a low-to-medium cardinality column that queries filter on, monitor performance, and revisit the scheme as data characteristics change; an N-day delete is the classic reason to choose daily partitions in a lake.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `PARTITION BY RANGE/LIST/HASH` | declares the scheme | table creation |
| `ALTER TABLE … ATTACH/DETACH PARTITION` (PG) | adds/removes a partition without copying rows | loading, retention |
| `ALTER TABLE … SWITCH PARTITION` (SQL Server) / `EXCHANGE PARTITION` (Oracle/MySQL) | swaps a partition with a staging table | bulk load, archive |
| `ALTER PARTITION FUNCTION … SPLIT/MERGE RANGE` (SQL Server) | adds/removes a boundary | rolling window |
| `DEFAULT` / `MAXVALUE` partition | catch-all | safety net only; keep it empty |
| `enable_partition_pruning`, `enable_partitionwise_*` (PG) | planner switches | diagnose or enable parallel plans |
| `ENABLE ROW MOVEMENT` (Oracle) | lets updates move rows across partitions | before updating key columns |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| Table is very large and queries nearly always filter on one column | Queries rarely filter on the key (all-partition scans plus overhead) |
| You need instant bulk delete/archive/load | Table is small (< a few GB); an index is enough |
| Writes concentrate on one entity or key | You need global unique constraints across partitions |
| Range by time with a rolling window and automation | You cannot automate partition creation and monitor it |
| Hash to spread hot-spot writes | You need range scans across the hash key |

### 9. Common mistakes
- Partitioning by a column queries don't filter on → every query scans all partitions plus pays planning overhead → pick the key from real predicates or skip partitioning.
- Too many partitions (e.g. hourly for ten years) → slow planning, huge catalog, file-handle pressure → coarser granularity, or archive old data out.
- Forgetting to pre-create future partitions → inserts fail at midnight → automate creation days ahead and alert on the "last partition boundary minus today".
- Letting the catch-all partition fill up → it grows unbounded and defeats pruning → monitor row count; split it out regularly.
- Updating the partition key without considering row movement → hidden delete+insert, locks, or errors → treat key columns as immutable.
- Assuming indexes behave globally → unique constraint rejected, or uniqueness silently per-partition → include the key, or enforce elsewhere.
- Running partition DDL on the primary only and ignoring replicas/subscribers → replication errors or divergent schemas → rehearse DDL on a clone and roll through the whole topology.

### 10. 🎯 Interview kit
**30-second answer:** Partitioning divides a large table into physical pieces by range, list or hash on a key. It helps when queries filter on that key (pruning) and when you manage data by lifecycle (drop or attach partitions instead of bulk deletes). It hurts when the key doesn't match access patterns or when there are too many partitions.

**Structure a longer answer:** clarify the goal (performance, retention, load) → pick the method and key from the query patterns → explain pruning and maintenance → call out constraints (unique keys include partition column, FK limits) → trade-offs (planning cost, skew, replicas) → close with a real scenario such as a 90-day rolling event table.

**Likely follow-ups:**
- *Range vs. hash?* → range for time/lifecycle and range pruning; hash for even distribution and point lookups, no range pruning.
- *When is list clearly better?* → discrete categories such as region or tenant tier, where each value has its own lifecycle or storage.
- *Sub-partitioning by hash inside monthly ranges?* → only when one monthly partition is itself a write or lookup hot-spot.
- *Partitioning vs. sharding?* → partitioning is within one server; sharding spreads across servers and needs routing.
- *How do you do rolling retention?* → pre-create future partitions, detach and drop the oldest.
- *Row no longer fits any partition?* → insert fails without a default; updates need row movement.
- *Foreign keys?* → engine-dependent; often limited, so plan alternative integrity checks.
- *What breaks on replicas?* → DDL ordering and missing partitions on logical subscribers.

**Signals interviewers look for:** you start from access patterns and operational goals; you know that unique keys must contain the partition key; you can name cost side-effects (planning, metadata, skew); you mention automation and monitoring of partition creation.

### 11. Check yourself
- [ ] State three concrete reasons to partition and one reason not to.
- [ ] Write a range-partitioned table with a rolling-window job and explain why drop beats delete.
- [ ] Explain why a primary key must include the partition column.
- [ ] Describe what happens to an UPDATE that changes the partition key.
- [ ] Decide between range, list and hash for: logs, multi-tenant orders, user sessions.
- [ ] Explain how you'd tell from a plan that partitioning is, or is not, the bottleneck.
<!-- explainer:end -->

## Practice questions

- [Basic] [Explain the difference between list partitioning and range partitioning with a concrete example where list is clearly the better fit.](<../../../../PRACTICE/01_Databases_and_SQL/Database_Operations/Partitioning/Q-20974_Explain_the_difference_between_list_partitioning_and_range_p.md>)
- [Intermediate] [How does the planner handle a query with no filter on the partition key at all, and what can you do to still get speedup?](<../../../../PRACTICE/01_Databases_and_SQL/Database_Operations/Partitioning/Q-21042_How_does_the_planner_handle_a_query_with_no_filter_on_the_pa.md>)
- [Deep] [How do foreign keys behave on partitioned tables, and what limitations have you had to work around?](<../../../../PRACTICE/01_Databases_and_SQL/Database_Operations/Partitioning/Q-21002_How_do_foreign_keys_behave_on_partitioned_tables_and_what_li.md>)
- [Deep] [When would sub-partitioning by hash inside monthly range partitions be worth the added complexity?](<../../../../PRACTICE/01_Databases_and_SQL/Database_Operations/Partitioning/Q-21041_When_would_sub-partitioning_by_hash_inside_monthly_range_par.md>)
- [Deep] [Explain how partitioning strategy changes when your primary is followed by 10 read replicas — what breaks on replicas that partition maintenance DDL can cause?](<../../../../PRACTICE/01_Databases_and_SQL/Database_Operations/Partitioning/Q-21094_Explain_how_partitioning_strategy_changes_when_your_primary_.md>)
- [Deep] [How would you assess whether your partitioning strategy is the bottleneck versus indexing, buffer pool, or I/O when a large table regresses? What's your triage order?](<../../../../PRACTICE/01_Databases_and_SQL/Database_Operations/Partitioning/Q-21144_How_would_you_assess_whether_your_partitioning_strategy_is_t.md>)
- [Expert] [What is 'partition-wise aggregation' and how does it let the engine parallelize GROUP BY across partitions?](<../../../../PRACTICE/01_Databases_and_SQL/Database_Operations/Partitioning/Q-21059_What_is_partition-wise_aggregation_and_how_does_it_let_the_e.md>)
- [Expert] [How do you design a chaos-testing program that proves partition-drop, split, and exchange operations are safe before they touch production ledgers?](<../../../../PRACTICE/01_Databases_and_SQL/Database_Operations/Partitioning/Q-21113_How_do_you_design_a_chaos-testing_program_that_proves_partit.md>)

## Related concepts

- [Partitioning and Bucketing Design](<../../../../02_Data_Engineering/Storage_and_Lakehouse/Partitioning_and_Bucketing/Partitioning_and_Bucketing_Design.md>) *(Partitioning and Bucketing)*
- [Partitioning and Partition Pruning](<../../../Database_Internals/DBMS_Concepts/Partitioning_and_Partition_Pruning.md>) *(DBMS Concepts)*
- [Audit and Compliance Platforms](<../../../../07_Distributed_Systems_and_Design/System_Design/High_Level_Design/Audit_and_Compliance_Platforms.md>) *(High Level Design)*
- [Concurrent Partition Overwrites](<../../../../02_Data_Engineering/Storage_and_Lakehouse/Partitioning_and_Bucketing/Concurrent_Partition_Overwrites.md>) *(Partitioning and Bucketing)*

## References

- *Designing Data-Intensive Applications* — Martin Kleppmann — Partitioning › Chapter 6. Partitioning
- *Designing Data-Intensive Applications* — Martin Kleppmann — Partitioning › Partitioning by Key Range
- *Designing Data-Intensive Applications* — Martin Kleppmann — Partitioning › Partitioning Secondary Indexes by Term
- *Designing Data-Intensive Applications* — Martin Kleppmann — Partitioning › Strategies for Rebalancing
- *Delta Lake: The Definitive Guide* — Denny Lee et al. — Maintaining Your Delta Lake › Table partitioning rules
- *Delta Lake: The Definitive Guide* — Denny Lee et al. — Performance Tuning Optimizing Your Data Pipelines with Delta Lake › Bash\
- *Delta Lake: The Definitive Guide* — Denny Lee et al. — Performance Tuning Optimizing Your Data Pipelines with Delta Lake › Lead by example
- *Ultimate Data Engineering Design Patterns* — Bragadeesh Sundararajan — Data Engineering Fundamentals › 2. Data Engineering Fundamentals
- *Data Engineering Fundamentals* — Zhaolong Liu — CHAPTER 3Data Storage and Management › CHAPTER 3Data Storage and Management(#contents.xhtmlch03a)
- *100 SQL Server Mistakes and How to Avoid Them* — Peter A. Carter — Optimization › 10.7 #60 Not partitioning large tables
- *100 SQL Server Mistakes and How to Avoid Them* — Peter A. Carter — Optimization › 10.8 #61 Not understanding the limitations of partition elimination
- *Database Internals* — Alex Petrov — Distributed Transactions › Database Partitioning

