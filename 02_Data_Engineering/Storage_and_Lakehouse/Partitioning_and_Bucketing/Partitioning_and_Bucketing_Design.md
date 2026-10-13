---
id: "02_Data_Engineering/Storage_and_Lakehouse/Partitioning_and_Bucketing/Partitioning_and_Bucketing_Design"
level: 4
type: "concept"
topic: "02_Data_Engineering/Storage_and_Lakehouse/Partitioning_and_Bucketing"
tools: []
---

# Partitioning and Bucketing Design

**L4 Advanced** · [↑ Partitioning and Bucketing](<_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** Partitioning splits a table by a low-cardinality column so queries read only the relevant slices, and bucketing hashes rows on a key into a fixed number of groups so joins and sampling avoid shuffles — both must be designed backward from the queries and the file sizes they produce.

### 1. The big picture
A table of hundreds of gigabytes to many terabytes cannot be scanned for every query. **Partitioning** divides it into smaller segments; a query that filters on the partition column reads only matching segments, which reduces both latency and, on pay-per-scan services such as Athena, cost. In data lakes a partition is typically a directory (`order_date=2026-10-01/`); in databases it is a physical sub-table (PostgreSQL declarative partitioning, SQL Server partition functions, with partition switching for archiving and loading); in columnstore tables it combines with rowgroup elimination, and since older data is rarely modified and newer data is read most, date partitioning is a natural fit.

The guiding rule from the AWS and medallion books: work backward from your queries, pick fields that are frequently used to filter, and keep **cardinality low** so you do not drown in metadata and tiny files. Beware **skew**: if most data or most queries concentrate on one value, partitioning helps little.

**Bucketing** (clustering into hash buckets) is a second, finer technique: `hash(key) mod N` decides the bucket, so rows with the same key always land together. Joins between two tables bucketed identically on the join key can avoid a shuffle, and bucket pruning speeds point lookups.

### 2. Core concepts and vocabulary
- **Partition column / key** — column whose values define partitions (date, country, tenant).
- **Partition pruning** — planner skips partitions that cannot match the filter.
- **Cardinality** — number of distinct values; high cardinality means too many partitions.
- **Skew** — uneven distribution of rows or queries across partitions.
- **Over-partitioning** — so many partitions that metadata and tiny files dominate.
- **Bucket** — hash-based group of rows; number of buckets fixed at design time.
- **Bucket join / shuffle-free join** — join where both sides are bucketed (and sorted) on the same key and count.
- **Range / list / hash partitioning** — by value ranges, explicit lists, or hash modulo.
- **Partition switching (SQL Server)** — metadata operation moving a whole partition in or out of a table.
- **Z-order / clustering** — sorting within or across files to improve data skipping on additional columns.

### 3. How it works, step by step
**Choosing a partition design:**
1. List the 5–10 most important query patterns and their filter columns.
2. Pick the column that appears in most selective filters and has low cardinality (usually a date or date-derived value).
3. Estimate rows and bytes per partition; target partition sizes in the hundreds of MB to several GB, with files of ~128 MB–1 GB.
4. Check skew: top partitions vs median.
5. If the filter column has high cardinality (customer_id), do not partition on it; consider bucketing or clustering.
6. Decide granularity: day vs hour vs month. Finer granularity helps freshness and targeted reprocessing but multiplies partitions.
7. Implement and verify pruning in the query plan (partition count read).

**Choosing bucketing:**
1. Identify large-table-to-large-table joins on a key.
2. Choose bucket count so each bucket is a reasonable file size (and a multiple of expected parallelism).
3. Write both tables with the same bucketing on the join key; sort within buckets if the engine supports sort-merge without sort.
4. Join; confirm no exchange (shuffle) appears in the plan.

### 4. Worked example
```sql
-- Delta/Spark: partition by date, cluster by a frequently filtered high-cardinality column
CREATE TABLE sales.events (
  event_id BIGINT, customer_id BIGINT, event_ts TIMESTAMP, event_date DATE, amount DECIMAL(12,2)
) USING DELTA
PARTITIONED BY (event_date);

OPTIMIZE sales.events ZORDER BY (customer_id);   -- Z-order works inside partitions;
                                                 -- never use the partition column itself

-- Hive/Spark bucketing for a join
CREATE TABLE sales.orders_b
USING PARQUET
CLUSTERED BY (customer_id) SORTED BY (customer_id) INTO 64 BUCKETS
AS SELECT * FROM sales.orders;

-- PostgreSQL declarative partitioning
CREATE TABLE measurements (id bigint, taken_at date, v numeric) PARTITION BY RANGE (taken_at);
CREATE TABLE measurements_2026_10 PARTITION OF measurements
  FOR VALUES FROM ('2026-10-01') TO ('2026-11-01');

-- SQL Server partition function
CREATE PARTITION FUNCTION pf_TransactionsDateRange (DATE)
AS RANGE LEFT FOR VALUES ('2024-01-01','2024-06-30','2024-12-31');
```
Verify: `EXPLAIN` should show only matching partitions scanned (e.g., "PartitionFilters: [event_date = 2026-10-01]"). A before/after metric for stakeholders: number of files and bytes read, query time, and (on Athena) dollars per query.

**Tiered layouts:** hourly partitions for freshness and monthly for finance cost reporting can be reconciled by partitioning the base table by day (or hour) and exposing a monthly aggregate/view; finance queries the aggregate, so no data is duplicated at the detailed level.

### 5. Under the hood
- **Directory partitions** store the value in the path, not in the file (in Hive-style layouts), so changing the column type from DATE to STRING changes the path semantics and predicate comparisons: a filter such as `dt >= current_date - 7` stops matching lexicographically or forces implicit casts, and pruning silently fails, scanning 10x more. Guard it with CI tests that assert partitions scanned on a canary query.
- **Pruning needs a sargable predicate:** the filter must compare the partition column directly (no function wrapping), and types must match.
- **Small files:** partitions × writers × time = files. A job with 200 shuffle partitions writing to 365 date partitions can create 73,000 files.
- **Partitioning in PostgreSQL** is implemented as separate tables under a parent; the planner prunes at plan time and, in newer versions, at execution time for parameters.
- **Columnstore:** partition boundaries align with rowgroup elimination; partitioning also lets you rebuild or compress old partitions independently and switch them out.
- **Bucketing caveats:** bucket count is fixed; changing it requires a rewrite; skewed keys create hot buckets; table formats like Iceberg offer bucket *transforms* inside partition specs which are easier to evolve.
- **Z-order** applies within a partition and cannot reuse the partition column.

### 6. Performance and design implications
- Too few partitions → large scans; too many → planning overhead, listing cost, small files; the sweet spot depends on engine, typically thousands, not millions.
- Partition by date even for multi-tenant SaaS and cluster by tenant, unless tenant-level isolation, deletion or quotas dominate; partitioning by tenant makes sense when tenants are few and large, or when per-tenant erasure is required.
- For many-to-many junction tables with no natural range, hash partition (or bucket) on the most common lookup key.
- Late data and backfills are cheapest when the partition equals the processing unit.
- Monitor: partitions per table, files per partition, p50/p95 partition size, skew ratio, bytes scanned per query.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `PARTITIONED BY` / `PARTITION BY RANGE` | Defines partitioning | Table creation |
| `CLUSTERED BY ... INTO n BUCKETS` | Hash bucketing | Large joins on same key |
| `OPTIMIZE ... ZORDER BY` | Co-locates values within files | Secondary filter columns |
| `spark.sql.shuffle.partitions` / `repartition()` | Controls write parallelism | Fix output file counts |
| `EXPLAIN` / plan UI | Shows partition filters | Verify pruning |
| Partition projection / `MSCK REPAIR` | Discover partitions | Athena/Hive catalogs |
| `ALTER TABLE ... SWITCH PARTITION` | Metadata-only move | Archive/load in SQL Server |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| Partition by date | Queries rarely filter on date, or table is small (<~1 GB) |
| Partition by tenant | Thousands of tenants (explosion) |
| Hourly partitions | Volume per hour is tiny — files too small |
| Bucketing | Join keys change often or are skewed |
| Z-order/clustering | Table small or filters vary wildly |

### 9. Common mistakes
- Partitioning on high-cardinality columns (user_id) → millions of partitions and tiny files → partition on date, cluster on id.
- Adding hour as sub-partition "just in case" → 3x slower writes → measure and drop.
- Changing a partition column's type → silent loss of pruning → test plans in CI.
- Ignoring skew → one straggler task dominates job → salting or different key.
- Assuming partitioning replaces indexing/statistics → full scans inside partitions → add clustering/statistics.
- Forgetting to register new partitions in the catalog → data invisible → automate discovery.

### 10. 🎯 Interview kit
**30-second answer:** Partitioning divides a large table by a low-cardinality column that queries filter on, so the engine reads only relevant partitions and scans (and pays for) less. I choose keys by working backward from queries, check skew and file sizes, avoid high-cardinality keys, and use bucketing or clustering for join and secondary-filter performance.

**Structure a longer answer:** clarify → core mechanism → trade-offs → close with a real scenario.

**Likely follow-ups:**
- *Date vs tenant_id in multi-tenant SaaS?* → date for time-bounded workloads and lifecycle; tenant for isolation/erasure; often date partition + tenant clustering.
- *Why did over-partitioning slow a small table?* → file listing/open overhead; show files scanned and time before/after.
- *Hourly vs monthly needs?* → partition at finest needed, expose rollups.
- *How does pruning break on type change?* → predicate no longer comparable; check plan in CI.
- *Junction table?* → hash partition on lookup key.

**Signals interviewers look for:** query-driven design, awareness of small files and skew, concrete metrics.

### 11. Check yourself
- [ ] Choose partition and clustering columns for an events table given three query patterns.
- [ ] Calculate expected files per day for a job given partition counts and shuffle parallelism.
- [ ] Explain when bucketing eliminates a shuffle.
- [ ] Describe a CI check that detects lost partition pruning.
<!-- explainer:end -->

## Practice questions

- [Basic] [In simple terms, what is partitioning in a data table and why do we do it?](<../../../PRACTICE/02_Data_Engineering/Storage_and_Lakehouse/Partitioning_and_Bucketing/Q-35398_In_simple_terms_what_is_partitioning_in_a_data_table_and_why.md>)
- [Basic] [How does partitioning in PostgreSQL declarative partitioning work at a high level for a fresher?](<../../../PRACTICE/02_Data_Engineering/Storage_and_Lakehouse/Partitioning_and_Bucketing/Q-35483_How_does_partitioning_in_PostgreSQL_declarative_partitioning.md>)
- [Intermediate] [What is the benefit of partitioning for cloud query costs when you pay per data scanned (like Athena)?](<../../../PRACTICE/02_Data_Engineering/Storage_and_Lakehouse/Partitioning_and_Bucketing/Q-35440_What_is_the_benefit_of_partitioning_for_cloud_query_costs_wh.md>)
- [Intermediate] [How would you explain to a stakeholder why over-partitioning made their 'small and fast' table slower, using a concrete before/after metric you'd show?](<../../../PRACTICE/02_Data_Engineering/Storage_and_Lakehouse/Partitioning_and_Bucketing/Q-35592_How_would_you_explain_to_a_stakeholder_why_over-partitioning.md>)
- [Deep] [What are the trade-offs of partitioning by date vs by tenant_id in a multi-tenant SaaS database?](<../../../PRACTICE/02_Data_Engineering/Storage_and_Lakehouse/Partitioning_and_Bucketing/Q-35345_What_are_the_trade-offs_of_partitioning_by_date_vs_by_tenant.md>)
- [Deep] [Describe how you'd approach partitioning a graph-like or many-to-many junction table where no natural range exists.](<../../../PRACTICE/02_Data_Engineering/Storage_and_Lakehouse/Partitioning_and_Bucketing/Q-35388_Describe_how_you_d_approach_partitioning_a_graph-like_or_man.md>)
- [Deep] [Your team wants hourly partitions for freshness but finance wants monthly for cost — how do you design a tiered partition layout that satisfies both without duplicating data?](<../../../PRACTICE/02_Data_Engineering/Storage_and_Lakehouse/Partitioning_and_Bucketing/Q-35526_Your_team_wants_hourly_partitions_for_freshness_but_finance_.md>)
- [Deep] [A recruiter-facing report suddenly scans 10x more data after a partition column type changed from DATE to STRING — how does that break pruning and how do you catch it in CI?](<../../../PRACTICE/02_Data_Engineering/Storage_and_Lakehouse/Partitioning_and_Bucketing/Q-35581_A_recruiter-facing_report_suddenly_scans_10x_more_data_after.md>)

## Related concepts

- [Ordering and Partition Keys](<../../../07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Ordering_and_Partition_Keys.md>) *(Distributed Systems Core)*
- [Partitioning and Data Skipping in Lakes](<../../../01_Databases_and_SQL/Query_Performance/Indexing/Partitioning_and_Data_Skipping_in_Lakes.md>) *(Indexing)*
- [Lakehouse Fundamentals](<../Lakehouse/Lakehouse_Fundamentals.md>) *(Lakehouse)*
- [Partitioning Strategy](<../../../01_Databases_and_SQL/Database_Operations/Partitioning/Partition_Pruning/Partitioning_Strategy.md>) *(Partitioning)*
- [Partitioning and Partition Pruning](<../../../01_Databases_and_SQL/Database_Internals/DBMS_Concepts/Partitioning_and_Partition_Pruning.md>) *(DBMS Concepts)*
- [Bucketed Joins in Spark](<../../Pipelines_and_Integration/Pipeline_Orchestration/Bucketed_Joins_in_Spark.md>) *(Pipeline Orchestration)*

## References

- *AWS Certified Data Engineer Associate Study Guide* — Sakti Mishra et al. — Data Modeling Strategies for Data Lakes › Partition your data
- *AWS Certified Data Engineer Associate Study Guide* — Sakti Mishra et al. — Data Modeling Strategies for Data Lakes › Bucket your data
- *AWS Certified Data Engineer Associate Study Guide* — Sakti Mishra et al. — Capacity Reservations › Best Practices When Using Amazon Athena
- *AWS Certified Data Engineer Associate Study Guide* — Sakti Mishra et al. — Cost Optimization for Data Pipelines › Practice Questions
- *Analytics Optimization with Columnstore Indexes in Microsoft SQL Server* — Edward Pollack — Segment and rowgroup elimination › CHAPTER 11
- *Analytics Optimization with Columnstore Indexes in Microsoft SQL Server* — Edward Pollack — Segment and rowgroup elimination › Partition Elimination Partition Elimination
- *Analytics Optimization with Columnstore Indexes in Microsoft SQL Server* — Edward Pollack — partitioning › Database Maintenance Database Maintenance
- *Analytics Optimization with Columnstore Indexes in Microsoft SQL Server* — Edward Pollack — partitioning › Partition and Rowgroup Sizing
- *Building Medallion Architectures* — Piethein Strengholt — Laying the Groundwork › Table Partitioning
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 8: Query Optimization and Performance Tuning › Partitioned Indexes
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 15: Case Studies and the Future of T-SQL › Partitioning Large Tables
- *Azure Data Fundamentals* — Michael John Pena — Azure Storage Solutions › Partitioning Strategy: The Key to Performance
- *Delta Lake: Up and Running* — Bennie Haelen, Dan Davis — Basic Operations on Delta Tables › Partitions
- *Delta Lake: Up and Running* — Bennie Haelen, Dan Davis — Basic Operations on Delta Tables › Partitioning by a single column
- *Delta Lake: Up and Running* — Bennie Haelen, Dan Davis — Basic Operations on Delta Tables › Partitioning by multiple columns
- *Delta Lake: Up and Running* — Bennie Haelen, Dan Davis — Performance Tuning › Partitioning

