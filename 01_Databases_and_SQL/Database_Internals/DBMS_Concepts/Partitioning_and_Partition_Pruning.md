---
id: "01_Databases_and_SQL/Database_Internals/DBMS_Concepts/Partitioning_and_Partition_Pruning"
level: 4
type: "concept"
topic: "01_Databases_and_SQL/Database_Internals/DBMS_Concepts"
tools: []
---

# Partitioning and Partition Pruning

**L4 Advanced** · [↑ DBMS Concepts](<_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** Partitioning splits one logical table into smaller physical pieces by a key so the engine can skip the pieces a query cannot need (partition pruning) and manage old data cheaply.

### 1. The big picture
Large tables hurt in three ways: scans read too much, maintenance (index rebuilds, statistics, backups) takes too long, and removing old data via `DELETE` is slow and log-heavy. Partitioning addresses all three by dividing the table into **partitions** that share one logical name. Queries with a predicate on the **partition key** touch only the relevant partitions; a whole partition can be dropped, archived or swapped out in a metadata operation.

Partitioning is not the same as **sharding**. Partitioning divides data *within one database/engine*; sharding spreads data across *separate nodes* with a routing layer (hash, range or directory lookup). Distributed stores like Cassandra hash a partition key to place data on nodes, and lakehouse tables partition by folder, but the pruning idea is the same everywhere: use the key in the predicate to skip data.

### 2. Core concepts and vocabulary
- **Partition key** — the column(s) whose value decides which partition a row belongs to.
- **Range partitioning** — contiguous ranges (months, ID ranges); the usual choice for time.
- **List partitioning** — explicit value sets (region = 'EU').
- **Hash partitioning** — hash of the key modulo N; spreads load evenly, poor for range queries.
- **Partition function / scheme (SQL Server)** — the function defines boundaries; the scheme maps partitions to filegroups.
- **Partition pruning (elimination)** — the optimiser excludes partitions that cannot satisfy the predicate.
- **Aligned index** — an index partitioned the same way as the table; required for fast switching.
- **Partition switching** — moving a whole partition in or out as a metadata-only operation.
- **Hot / warm / cold data** — access-frequency tiers; partitions let you place each on different storage.
- **Skew** — uneven partition sizes or traffic.

### 3. How it works, step by step
1. **Design:** pick a key that appears in most filters and has sensible cardinality (order date, tenant id).
2. **Define boundaries:** e.g. monthly ranges; decide which side of a boundary a value belongs to (`RANGE LEFT` vs `RANGE RIGHT` in SQL Server).
3. **Map to storage:** each partition to a filegroup/tablespace (optionally different disks or tiers).
4. **Insert:** a row's partition-key value is passed through the function to select its partition.
5. **Query:** the optimiser compares the predicate with the boundaries at compile time (static pruning) or at run time with parameters (dynamic pruning), then scans only matching partitions.
6. **Maintain:** add a new partition ahead of time (`SPLIT`), drop/merge or switch out old ones (`SWITCH`), rebuild indexes per partition.

Pruning only happens when the predicate is **sargable on the raw partition key**:

| Predicate | Prunes? |
|---|---|
| `order_date >= '2026-03-01' AND order_date < '2026-04-01'` | Yes |
| `order_date = '2026-03-15'` | Yes |
| `YEAR(order_date) = 2026` | Usually no (function on the column) |
| `CAST(order_date AS varchar) LIKE '2026-03%'` | No |
| Filter on a non-key column only | No — scans all partitions |

### 4. Worked example
```sql
-- SQL Server monthly range partitioning
CREATE PARTITION FUNCTION pf_month (date)
AS RANGE RIGHT FOR VALUES ('2026-01-01','2026-02-01','2026-03-01');

CREATE PARTITION SCHEME ps_month
AS PARTITION pf_month ALL TO ([PRIMARY]);

CREATE TABLE dbo.Sales (
  sale_id   BIGINT NOT NULL,
  sale_date DATE   NOT NULL,
  amount    DECIMAL(12,2),
  CONSTRAINT pk_sales PRIMARY KEY CLUSTERED (sale_date, sale_id)
) ON ps_month (sale_date);

-- which partition does each row live in?
SELECT $PARTITION.pf_month(sale_date) AS part_no, COUNT(*) AS rows
FROM   dbo.Sales GROUP BY $PARTITION.pf_month(sale_date);

-- pruning check: look at the plan's "Actual Partition Count"
SELECT SUM(amount) FROM dbo.Sales
WHERE  sale_date >= '2026-02-01' AND sale_date < '2026-03-01';

-- retire the oldest month instantly
ALTER TABLE dbo.Sales SWITCH PARTITION 1 TO dbo.Sales_Archive PARTITION 1;
```
In the execution plan, the scan operator's properties show *Actual Partition Count = 1* when pruning worked. Note the primary key includes the partition key — unique indexes on a partitioned table must contain it so uniqueness can be checked within a partition. `SWITCH` requires identical structure, aligned indexes and a matching check constraint on the target; it takes seconds regardless of row count.

### 5. Under the hood
- **Metadata, not magic.** A partitioned table is a set of rowsets (B-trees/heaps), one per partition, plus the function metadata. Each partition has its own statistics/histograms only in some engines; SQL Server historically keeps table-level statistics, which can mislead estimates for single-partition queries (incremental statistics help).
- **Elimination in the plan.** Static pruning is resolved when the plan is compiled (constants); parameterised queries use run-time elimination, visible as a "Seek Predicates" over `PtnId`.
- **Switching.** The partition's allocation units are re-pointed to another table's metadata; no data is copied. This is how sliding-window loads work: build into a staging table, then switch in.
- **Distributed tables.** In Cassandra-style systems the partition key is hashed (consistent hashing) to a token; the node owning that token range stores the data. Choosing a high-cardinality key avoids hot nodes. In directory-based sharding a lookup service resolves key→node, adding a hop and a single point of failure.
- **Lakehouse/Parquet.** Directory-style partitions (`dt=2026-03-01/`) let engines skip files by path; too many small partitions create small-file overhead.

### 6. Performance and design implications
- Pruning reduces I/O proportional to partitions skipped; it does not by itself make a *seek* faster — a good index still matters.
- Partitioning can *hurt* queries that do not filter by the key: more rowsets to open, parallelism quirks, sometimes worse plans.
- Too many partitions (thousands) increases metadata, compile time and memory; typical guidance is dozens to a few hundred.
- Skew: monthly partitions with a heavily loaded current month concentrate writes; combine with good clustering.
- Manageability wins: per-partition index rebuilds, statistics, backup of read-only filegroups, tiered storage for cold data.
- Measure: partition counts in plans, `sys.partitions`, row counts per partition, page counts, IO statistics before/after.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `CREATE PARTITION FUNCTION / SCHEME` | Defines boundaries and storage mapping | Table setup |
| `$PARTITION.fn(col)` | Returns the partition number of a value | Verification and skew checks |
| `ALTER PARTITION FUNCTION … SPLIT/MERGE RANGE` | Adds/removes boundaries | Rolling window |
| `ALTER TABLE … SWITCH PARTITION` | Metadata-only move | Load/archive |
| `ALTER INDEX … REBUILD PARTITION = n` | Rebuild one partition | Maintenance |
| `PARTITION BY RANGE/LIST/HASH` (Postgres, MySQL) | Declarative partitioning | Non-SQL-Server engines |
| `EXPLAIN` / actual plan | Confirm pruning | Every design change |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| Large time-series/fact tables with a date filter | Table is small enough to index normally |
| You need fast archive/purge | Queries rarely filter on the key |
| Different storage tiers per age | Key choice causes data skew |
| Hash partitioning for even write distribution | Range queries dominate |
| Unique constraints can include the partition key | Global uniqueness on another column is required |

### 9. Common mistakes
- Expecting partitioning to speed up every query → no gain/worse performance → verify pruning in plans and test.
- Wrapping the key in a function (`YEAR(col)`) → no pruning → rewrite as a range.
- Forgetting to create future partitions → rows land in the last partition → automate boundary creation.
- Non-aligned indexes → `SWITCH` fails → align all indexes.
- Choosing a skewed key (country with 90% in one value) → hot partition → choose a better key or composite.
- Confusing partitioning with sharding → wrong scalability expectations → remember one node vs many.

### 10. 🎯 Interview kit
**30-second answer:** Partitioning splits a table by a key into smaller physical units. The optimiser prunes partitions that cannot satisfy the predicate, so scans read less, and old data can be switched or dropped in metadata. It only helps when queries filter on the partition key in a sargable way.

**Structure a longer answer:** why (size, maintenance, retention) → range/list/hash → pruning conditions → maintenance (switch, split) → trade-offs and a sharding contrast.

**Likely follow-ups:**
- *Partitioning vs sharding?* → Same node vs across nodes with routing.
- *Why did my query not prune?* → Function on the column, no predicate on key, implicit conversion, parameter sniffing issues.
- *How do you delete a year of data fast?* → Switch the partition out (or `DROP PARTITION`/truncate), not `DELETE`.
- *Hash vs range?* → Hash for even distribution of point lookups; range for ordered scans.

**Signals interviewers look for:** you verify pruning rather than assume it, you know it is a manageability feature first, and you separate partitioning from sharding.

### 11. Check yourself
- [ ] Write predicates that prune and ones that do not.
- [ ] Explain why a unique index must include the partition key.
- [ ] Describe a sliding-window load using switch.
- [ ] Compare range, list and hash with a use case each.
- [ ] State how to confirm pruning in an execution plan.
<!-- explainer:end -->

## Practice questions

- [Intermediate] [What is data partitioning in a data lake (e.g. partitioning by date) and how does it speed up queries?](<../../../PRACTICE/01_Databases_and_SQL/Database_Internals/DBMS_Concepts/Q-22569_What_is_data_partitioning_in_a_data_lake_e_g_partitioning_by.md>)
- [Intermediate] [How would you estimate the cost of a query in a pay-per-scan warehouse and reduce bytes scanned with partitioning and column pruning?](<../../../PRACTICE/01_Databases_and_SQL/Database_Internals/DBMS_Concepts/Q-22626_How_would_you_estimate_the_cost_of_a_query_in_a_pay-per-scan.md>)
- [Intermediate] [How would you find the top 10 slowest queries hitting your production database right now?](<../../../PRACTICE/01_Databases_and_SQL/Database_Internals/DBMS_Concepts/Q-22264_How_would_you_find_the_top_10_slowest_queries_hitting_your_p.md>)
- [Deep] [What is the small files problem in a data lake and why does it hurt query performance?](<../../../PRACTICE/01_Databases_and_SQL/Database_Internals/DBMS_Concepts/Q-22570_What_is_the_small_files_problem_in_a_data_lake_and_why_does_.md>)
- [Deep] [What is a broadcast join in a distributed engine and when is it preferred over a shuffle (hash) join?](<../../../PRACTICE/01_Databases_and_SQL/Database_Internals/DBMS_Concepts/Q-22622_What_is_a_broadcast_join_in_a_distributed_engine_and_when_is.md>)
- [Deep] [What is clustering / distribution key in a cloud warehouse like Redshift or Snowflake and how does it affect join performance?](<../../../PRACTICE/01_Databases_and_SQL/Database_Internals/DBMS_Concepts/Q-22624_What_is_clustering_distribution_key_in_a_cloud_warehouse_lik.md>)
- [Deep] [How would you debug a query that is fast in staging but slow in production with the same schema?](<../../../PRACTICE/01_Databases_and_SQL/Database_Internals/DBMS_Concepts/Q-22258_How_would_you_debug_a_query_that_is_fast_in_staging_but_slow.md>)
- [Deep] [Explain the difference between a nested loop join, a hash join, and a merge join in the query planner.](<../../../PRACTICE/01_Databases_and_SQL/Database_Internals/DBMS_Concepts/Q-22260_Explain_the_difference_between_a_nested_loop_join_a_hash_joi.md>)

## Related concepts

- [Advanced Aggregation](<../../SQL_Language/Aggregation/Advanced_Aggregation.md>) *(Aggregation)*
- [Data Warehouse Fundamentals](<../../../02_Data_Engineering/Modeling_and_Warehousing/Data_Warehousing/Data_Warehouse_Fundamentals.md>) *(Data Warehousing)*
- [Distributed Joins in MPP Systems](<../../../07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Distributed_Joins_in_MPP_Systems/_index.md>) *(Distributed Systems Core)*

## References

- *Cassandra: The Definitive Guide* — Jeff Carpenter, Eben Hewitt — has emerged as the key technology for orchestrating the deployment and maintenance of containers
- *Designing Data-Intensive Applications* — Martin Kleppmann — Data Models and Query Languages › MapReduce Querying
- *Designing Data-Intensive Applications* — Martin Kleppmann — Partitioning › Parallel Query Execution
- *Designing Data-Intensive Applications* — Martin Kleppmann — Comparing Hadoop to Distributed Databases › Diversity of processing models
- *Expert Performance Indexing in SQL Server* — Jason Strate, Grant Fritchey — Shrink Operations
- *Fundamentals of Data Engineering* — Joe Reis, Matt Housley — Storage › Hot data
- *Building Data Integration Solutions* — Jay Borthen — Key Concepts in Data Integration › Batches

