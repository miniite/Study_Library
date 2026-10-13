---
id: "02_Data_Engineering/Modeling_and_Warehousing/OLTP_vs_OLAP/Physical_Design_for_OLTP_vs_OLAP"
level: 4
type: "concept"
topic: "02_Data_Engineering/Modeling_and_Warehousing/OLTP_vs_OLAP"
tools: []
---

# Physical Design for OLTP vs OLAP

**L4 Advanced** · [↑ OLTP vs OLAP](<_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** The same logical data needs a different physical design in each world — OLTP optimizes for fast, safe single-row changes (narrow rows, key indexes, small transactions); OLAP optimizes for reading fewer bytes (columnar storage, partitioning, sorted bulk loads, few indexes).

### 1. The big picture
Logical modeling decides *what* tables exist; **physical design** decides *how* they're stored and accessed: key choice, indexes, storage layout, partitioning, compression, how data is loaded, and how the log is used. Because OLTP and OLAP have opposite access patterns (see the workloads lesson), the same entity — say, a sales line — is physically designed differently in each system.

A transactional table is tuned so a request finds one row quickly and changes it without blocking others. An analytic table is tuned so a request can aggregate hundreds of millions of rows while touching as few bytes as possible. Getting this wrong in either direction is expensive: too many indexes slow OLTP writes; "OLTP-style" design in a warehouse produces huge scans, high bills and slow dashboards.

### 2. Core concepts and vocabulary
- **Surrogate key** — a meaningless, system-generated key (identity/sequence). Compact, stable, ideal for OLTP primary keys and for warehouse dimension keys.
- **Clustered index** — table stored in key order (a B-tree); in OLTP usually on the primary key.
- **Covering index** — a nonclustered index containing every column a query needs, avoiding lookups.
- **Partitioning** — splitting a large table into pieces (usually by date) so queries touch only relevant pieces and old data can be switched out cheaply.
- **Clustering / sort key** — the order in which an analytic table's data is physically arranged (or "clustered") so filters can skip chunks.
- **Partition pruning / segment elimination** — the engine skipping partitions or column segments that cannot match.
- **Minimal logging** — bulk operations that log page allocations rather than every row, speeding loads and reducing log growth.
- **Predicate / aggregate pushdown** — pushing filters and aggregations to the storage layer so less data moves up the plan.
- **Sargable** — a predicate form that can use an index/metadata (`col >= @x`, not `YEAR(col) = 2026`).
- **Granularity (grain)** — what one fact row represents; analytic systems often keep multiple grains.

### 3. How it works, step by step
Designing the same "orders" data both ways:

**OLTP design**
1. Normalize to ~3NF: `Customer`, `Order`, `OrderLine`, `Product` — each fact stored once.
2. Use a narrow, ever-increasing surrogate PK (e.g., `BIGINT IDENTITY`): small index entries, append-only inserts at the end of the B-tree, simple foreign keys.
3. Add nonclustered indexes only for known lookup paths (FK columns, `(CustomerId, OrderDate)`); each index speeds reads but taxes every write.
4. Keep transactions short, commit often, keep locks narrow; use the log fully so point-in-time recovery works.
5. Keep hot data small: archive or purge old rows so the working set fits in memory.

**OLAP design**
1. Denormalize to a star: one wide `FactSales` plus conformed dimensions (`DimDate`, `DimCustomer`, `DimProduct`).
2. Store the fact in columnar format, compressed.
3. Partition by date (month/day) and keep data physically ordered by the most common filter (date, sometimes customer).
4. Load in large sorted batches via bulk paths; minimal logging; rerun on failure rather than point-in-time recover within a load.
5. Avoid many secondary indexes; rely on scans + metadata skipping; add summary/aggregate tables or materialized views for repeated heavy queries.
6. Keep history forever (append) and plan retention with partition switching.

| Design choice | OLTP | OLAP |
|---|---|---|
| Normalization | 3NF | Star / wide denormalized |
| Storage | Rowstore | Columnstore / columnar files |
| Keys | Narrow identity PK | Surrogate dim keys + date keys |
| Indexes | Several, targeted | Few; rely on scans & sort order |
| Partitioning | Rare / for maintenance | Routine, by date |
| Load style | Many small commits | Large, sorted, bulk, minimal logging |
| Recovery | Point-in-time critical | Rerun load from before |
| Updates | In place | Append / versioned / SCD |

### 4. Worked example
Same questions, two physical designs.

```sql
-- OLTP: point lookups + frequent small writes
CREATE TABLE dbo.Orders (
    OrderId     BIGINT IDENTITY(1,1) CONSTRAINT PK_Orders PRIMARY KEY CLUSTERED,
    CustomerId  INT       NOT NULL,
    OrderDate   DATETIME2 NOT NULL,
    Status      TINYINT   NOT NULL,
    Total       DECIMAL(12,2) NOT NULL
);
CREATE INDEX IX_Orders_Customer_Date ON dbo.Orders(CustomerId, OrderDate) INCLUDE (Status, Total);

-- OLAP: wide fact, columnar, partitioned by date
CREATE PARTITION FUNCTION pf_month (DATE)
    AS RANGE RIGHT FOR VALUES ('2026-01-01','2026-02-01','2026-03-01');
CREATE PARTITION SCHEME ps_month AS PARTITION pf_month ALL TO ([PRIMARY]);

CREATE TABLE fact.Sales (
    DateKey INT NOT NULL, CustomerKey INT NOT NULL, ProductKey INT NOT NULL,
    Quantity INT NOT NULL, Amount DECIMAL(12,2) NOT NULL, SaleDate DATE NOT NULL
) ON ps_month(SaleDate);
CREATE CLUSTERED COLUMNSTORE INDEX cci_Sales ON fact.Sales;
```

"Last 7 days" on a 500M-row OLAP fact — which tool reduces the scan?

1. **Query rewrite (first!)** — make the filter sargable: `WHERE SaleDate >= DATEADD(day,-7,CAST(GETDATE() AS date))`, not `WHERE CAST(SaleDate AS varchar)...` or a wrapping subquery that blocks pushdown. It costs nothing and often fixes everything.
2. **Partitioning** — with date partitions only 1–2 partitions are read (partition elimination).
3. **Clustering/sort order** — within partitions, ordered data tightens min/max so more segments are skipped.

Verify with the plan: look for "Partitions accessed", "Segments skipped" (columnstore), and `SET STATISTICS IO` reads.

### 5. Under the hood
- **Why surrogate identity keys suit OLTP:** a monotonically increasing key inserts at the tail of the clustered B-tree (few page splits, good fill), is 4–8 bytes (so every nonclustered index entry carrying it stays small), and decouples identity from changing business data. Downsides: hot tail page contention at extreme insert rates, and no natural clustering for range queries.
- **Why OLAP drops most indexes:** the cost model changes. With 30% selectivity, a scan with elimination beats an index lookup storm. Columnstore replaces the B-tree as the primary structure and keeps its own metadata.
- **Pushdown depends on sargability.** A non-sargable expression or a wrapping subquery/view with an aggregate in the middle prevents the optimizer from pushing predicates to the scan, so the whole table is read then filtered. In review, look for functions on filter columns, implicit conversions, and `SELECT *` through layers.
- **Logging economics:** OLAP loads are re-runnable, so minimal logging (`INSERT … SELECT` into a columnstore/heap with `TABLOCK`, `BULK INSERT`, `SELECT INTO`, partition switching) is acceptable. Confusing the two modes is a failure: using minimal logging on a system that needs point-in-time recovery, or using row-by-row commits to load a warehouse.
- **Mixed-workload trap:** 90% short point reads and 10% heavy aggregations share memory and I/O. Aggregations evict hot pages from the buffer pool and consume CPU, inflating p99 for the point reads. Tools: workload isolation (Resource Governor / separate warehouse / replica), query timeouts, scheduling.

### 6. Performance and design implications
- Multi-granularity facts (daily + monthly aggregate tables) trade storage for query time and are normal in analytics.
- Choose the **sort/cluster key by dominant filter** (usually date); add a secondary dimension only if it's frequently filtered and doesn't destroy the primary ordering.
- Watch partition count: too many (thousands) harms metadata and plan compile time; partition by month/day, not hour, unless needed.
- Measure: bytes scanned, partitions/segments eliminated, spill to tempdb, log generated per load.
- Cost spikes in cloud warehouses usually come from unpartitioned scans and always-on compute, not from user count.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `CREATE PARTITION FUNCTION/SCHEME` | Defines date partitioning | Large facts, retention |
| `ALTER TABLE … SWITCH PARTITION` | Instant load/purge of a partition | Load staging → fact; drop old months |
| `INSERT … WITH (TABLOCK)` | Enables minimal logging/bulk path | Warehouse loads |
| `OPTION (MAXDOP n)` / Resource Governor | Limit parallelism/resources | Protect OLTP |
| `CREATE STATISTICS` / update stats | Keep cardinality estimates good | After large loads |
| `CREATE MATERIALIZED/INDEXED VIEW` | Pre-aggregate | Repeated heavy rollups |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| Few targeted indexes (OLTP) | Index count grows "just in case" — write amplification |
| Partition by date (OLAP) | Queries rarely filter on date; too many tiny partitions |
| Summary tables | Source changes often, causing stale aggregates |
| Wide denormalized table | Dimension attributes change and need SCD handling |
| Sort key = date | Real workload filters mainly by customer |

### 9. Common mistakes
- Indexing a warehouse like OLTP (many B-tree indexes) → slow loads, storage bloat, little query benefit → columnar + partition.
- Loading a columnstore with tiny batches → fragmented row groups → batch/bulk.
- Non-sargable filters (`YEAR(d)=2026`, implicit casts) → full scans → rewrite as ranges.
- Over-normalizing the warehouse → many joins on every query.
- Updating dimensions in place without history → can't reproduce past reports.
- Leaving old partitions online forever → cost and slow maintenance → switch out / tier.

### 10. 🎯 Interview kit
**30-second answer:** OLTP physical design favors narrow identity keys, a few covering/lookup indexes, short transactions and fully logged writes. OLAP favors columnar storage, date partitioning, a sort key matching the common filter, few indexes, and large sorted bulk loads with minimal logging. Same data, opposite access patterns.

**Structure a longer answer:** state the workload → logical vs physical → list choices (keys, index, storage, partition, load) → measure → trade-offs → example.

**Likely follow-ups:**
- *Index strategy for a point-lookup table vs star schema?* → OLTP: PK + covering indexes on lookup paths. OLAP: columnstore, partition/sort, few or no secondary indexes.
- *Date-mostly queries but sometimes by customer — sort key?* → Date first (partition + sort); handle customer filter with secondary ordering/clustering within date, a materialized aggregate, or a search-optimization feature; accept it won't be as fast.
- *Why can't you just index like OLTP?* → Selectivity is low; index maintenance slows loads; scans with skipping win.
- *How would you catch a pushdown-defeating query in review?* → Functions on filter columns, casts, wrapping aggregates, plan shows scan with predicate applied late.
- *Mixed workload fix?* → Isolate resources or separate systems.

**Signals interviewers look for:** you tie each physical choice to the access pattern; you know measure-first; you mention partition elimination and sargability.

### 11. Check yourself
- [ ] List four physical differences between an OLTP orders table and its warehouse fact table.
- [ ] Rewrite `WHERE YEAR(SaleDate)=2026` sargably.
- [ ] Explain minimal logging and when it's inappropriate.
- [ ] Decide partition grain for 5 years of 1M rows/day and justify.
- [ ] Explain the mixed-workload trap and two mitigations.
<!-- explainer:end -->

## Practice questions

- [Basic] [What is a 'primary key' and why is it central to how OLTP finds a single record fast?](<../../../PRACTICE/02_Data_Engineering/Modeling_and_Warehousing/OLTP_vs_OLAP/Q-34916_What_is_a_primary_key_and_why_is_it_central_to_how_OLTP_find.md>)
- [Intermediate] [Why do analytics teams often prefer 'wide' tables while app developers prefer 'narrow' related tables?](<../../../PRACTICE/02_Data_Engineering/Modeling_and_Warehousing/OLTP_vs_OLAP/Q-34934_Why_do_analytics_teams_often_prefer_wide_tables_while_app_de.md>)
- [Intermediate] [Explain why an OLTP-tuned buffer cache expects high hit ratio on hot rows while an OLAP scan mostly misses cache — how does that change instance/memory sizing for each tier?](<../../../PRACTICE/02_Data_Engineering/Modeling_and_Warehousing/OLTP_vs_OLAP/Q-35072_Explain_why_an_OLTP-tuned_buffer_cache_expects_high_hit_rati.md>)
- [Deep] [How do you decide the clustering/sort key for an OLAP fact table when queries filter mostly by date but sometimes by customer — and why can't you just index like on OLTP?](<../../../PRACTICE/02_Data_Engineering/Modeling_and_Warehousing/OLTP_vs_OLAP/Q-35017_How_do_you_decide_the_clustering_sort_key_for_an_OLAP_fact_t.md>)
- [Deep] [Why is predicate and aggregate pushdown critical for OLAP query cost, and how does a wrapping subquery or non-sargable expression silently defeat it — how would you catch it in review?](<../../../PRACTICE/02_Data_Engineering/Modeling_and_Warehousing/OLTP_vs_OLAP/Q-35069_Why_is_predicate_and_aggregate_pushdown_critical_for_OLAP_qu.md>)
- [Deep] [Why is the WAL / redo-log design of an OLTP engine tuned for durability of small writes, whereas an OLAP engine may relax durability for bulk loads?](<../../../PRACTICE/02_Data_Engineering/Modeling_and_Warehousing/OLTP_vs_OLAP/Q-34722_Why_is_the_WAL_redo-log_design_of_an_OLTP_engine_tuned_for_d.md>)
- [Deep] [Why do OLAP queries benefit from statistics on many columns while OLTP planners lean heavily on a few high-selectivity indexed columns — how does that shape ANALYZE strategy?](<../../../PRACTICE/02_Data_Engineering/Modeling_and_Warehousing/OLTP_vs_OLAP/Q-34765_Why_do_OLAP_queries_benefit_from_statistics_on_many_columns_.md>)
- [Deep] [A principal-level probe: explain why simply scaling the OLTP database vertically eventually fails to serve analytical demand no matter the hardware, tying it to workload access patterns not just size.](<../../../PRACTICE/02_Data_Engineering/Modeling_and_Warehousing/OLTP_vs_OLAP/Q-34844_A_principal-level_probe_explain_why_simply_scaling_the_OLTP_.md>)

## Related concepts

- [OLTP vs Star Schema Performance](<../Dimensional_Modeling/OLTP_vs_Star_Schema_Performance.md>) *(Dimensional Modeling)*
- [Index Tuning in Practice](<../../../01_Databases_and_SQL/Query_Performance/Indexing/Index_Tuning_in_Practice/_index.md>) *(Indexing)*
- [Rollups vs On-demand Queries](<../../../04_Data_Architecture_and_Strategy/Architecture_Patterns/Data_Architecture/Tiered_Lake_Storage/Rollups_vs_On-demand_Queries.md>) *(Data Architecture)*
- [Choosing a Database for the Workload](<../../../01_Databases_and_SQL/Database_Strategy/Database_Selection/Choosing_a_Database_for_the_Workload.md>) *(Database Selection)*
- [Random Keys and Index Locality](<../../../01_Databases_and_SQL/Query_Performance/Indexing/Random_Keys_and_Index_Locality.md>) *(Indexing)*
- [Primary and Unique Keys](<../../../01_Databases_and_SQL/Database_Design/Constraints_and_Keys/Primary_and_Unique_Keys.md>) *(Constraints and Keys)*

## References

- *Analytics Optimization with Columnstore Indexes in Microsoft SQL Server* — Edward Pollack — IntroduCtIon to analytIC data In a transaCtIonal database › OLAP Data in an Analytic Table in a Transactional Database
- *Analytics Optimization with Columnstore Indexes in Microsoft SQL Server* — Edward Pollack — transaCtional vs. analytiC Workloads RANSA TIONAL VS. NALYTI › Building Better Analytic Data Structures
- *Analytics Optimization with Columnstore Indexes in Microsoft SQL Server* — Edward Pollack — BatCh exeCution H XE UTION › Batch Mode Execution
- *Analytics Optimization with Columnstore Indexes in Microsoft SQL Server* — Edward Pollack — Bulk loading data
- *Data Contracts: Developing Production-Grade Pipelines at Scale* — Chad Sanderson, Mark Freeman — Data Quality Isn't About Pristine Data › OLTP Versus OLAP and Its Implications for Data Quality
- *Data Contracts: Developing Production-Grade Pipelines at Scale* — Chad Sanderson, Mark Freeman — D
- *Azure Data Fundamentals* — Michael John Pena — Understanding Data Workloads › Fewer concurrent users, resource-intensive queries
- *Azure Data Fundamentals* — Michael John Pena — Understanding Data Workloads › Performance Considerations
- *Azure Data Fundamentals* — Michael John Pena — Advanced Azure Data Concepts › O
- *Fundamentals of Data Engineering* — Joe Reis, Matt Housley — Storage › Object stores for data engineering applications
- *Fundamentals of Data Engineering* — Joe Reis, Matt Housley — Storage › Separation of Compute from Storage
- *SQL Server 2025 Query Performance Tuning (7th ed.)* — Grant Fritchey — FULL_BOOK_NEEDS_MANUAL_SPLIT (cont.) › Chapter 19 Memory-Optimized Oltp Tables And Procedures
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 12: Azure SQL and Synapse Integration › Serverless SQL Pools for Ad Hoc Analysis
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 12: Azure SQL and Synapse Integration › Dedicated SQL Pools

