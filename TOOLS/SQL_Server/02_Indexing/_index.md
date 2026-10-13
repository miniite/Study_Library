---
id: "TOOLS/SQL_Server/02_Indexing"
level: 3
type: "curriculum"
curriculum: "sql_server"
---

# Indexing

**L3 Practitioner** · [↑ SQL Server](<../_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** An index is an extra, ordered (or column-organized) copy of some of a table's data that lets SQL Server find, sort, join and aggregate rows while touching far fewer pages — paid for with storage, slower writes and maintenance.

### 1. The big picture
Without indexes, every query against a table must read all its pages (a scan). Indexes give the optimizer alternatives: jump straight to the rows it needs (a seek), read rows already in the order a query wants (no sort), or read only the columns an aggregate needs (columnstore). Nearly every performance conversation about SQL Server eventually becomes an indexing conversation, because indexes decide how many pages a query reads — and pages read is the main driver of CPU, memory and I/O cost.

The previous module explained the storage underneath: 8 KB pages, heaps, B-trees, compression. This module builds on it:

| Child lesson | What it covers |
|---|---|
| Clustered indexes | The table itself stored as a B-tree ordered by the clustering key; choosing that key |
| Nonclustered indexes | Separate B-trees pointing back to the rows; key lookups, covering, included columns |
| Index design methodology | How to decide which indexes to build for a workload |
| Index analysis DMVs | Measuring usage, missing indexes, operational stats |
| Index maintenance | Fragmentation, rebuild vs reorganize, statistics, fill factor |
| Columnstore (+ batch mode) | Column-organized compressed storage for analytics |
| Partitioned indexes | Splitting one index into ranges for management and elimination |
| Special indexes | Filtered, indexes on computed columns, indexed views, XML, spatial, full-text |
| Memory-optimized indexes | Hash and range (Bw-tree) indexes for In-Memory OLTP tables |

### 2. Core concepts and vocabulary
- **Rowstore** — data stored row by row on 8 KB pages; heaps and B-tree indexes are rowstore.
- **Heap** — a table with no clustered index; rows in no order, located by RID (file:page:slot).
- **Clustered index** — the table's data pages arranged as a B-tree by the clustering key. One per table.
- **Nonclustered index** — a separate B-tree of key columns (plus optional included columns) with a row locator back to the data: the clustering key, or the RID on a heap. Up to 999 per table.
- **Key columns** — the columns the index is sorted by, used for seeking and ordering (max 16 columns / 1,700 bytes for nonclustered, 900 for clustered).
- **Included columns** — non-key columns stored only at the leaf level to make an index *covering*.
- **Covering index** — an index that contains every column a query needs, so no lookup to the base table is required.
- **Seek / scan** — a seek navigates root → leaf to a starting point and reads a range; a scan reads all leaf pages (or all heap pages).
- **Key lookup / RID lookup** — fetching the rest of a row from the clustered index or heap after finding it in a nonclustered index; one random lookup per row.
- **Selectivity** — the fraction of rows a predicate returns; low fraction = highly selective = good seek candidate.
- **SARGable** — a predicate written so an index seek can use it (`col >= @x`, not `YEAR(col) = 2024`).
- **Columnstore index** — data stored column by column in compressed segments of up to ~1 million rows (rowgroups); built for scans and aggregates.
- **Fill factor** — how full index pages are left after a build or rebuild.
- **Fragmentation** — logical order of pages differing from physical order, or pages half empty.

### 3. How it works, step by step
**How the optimizer decides whether to use an index**
1. The query's predicates, joins, `ORDER BY`, `GROUP BY` and select list are matched against available indexes.
2. Statistics on the index keys estimate how many rows each predicate returns.
3. For each candidate, the optimizer costs the access: seek + range read, plus a lookup per row if the index isn't covering; versus a scan of the clustered index/heap; versus a columnstore scan.
4. It picks the cheapest plan. Because each lookup is a random page read, a non-covering nonclustered index is only chosen when few rows qualify — often well under 1% of the table — the "tipping point". Above that, a scan is cheaper.

**The main structures at a glance**

| Structure | Leaf contains | Best at | Weak at |
|---|---|---|---|
| Heap | Data rows, unordered | Bulk loads, staging | Seeks without a NC index; forwarded records after updates |
| Clustered B-tree | Full data rows in key order | Range queries on the key, ordered retrieval, every lookup | Wide or random keys hurt all indexes |
| Nonclustered B-tree | Key + included cols + row locator | Selective seeks, covering narrow queries | Many rows → lookups; each one adds write cost |
| Clustered columnstore | Compressed column segments | Large scans, aggregates, data warehouse fact tables | Singleton lookups and frequent updates |
| Nonclustered columnstore | Selected columns, optionally filtered, on a rowstore table | Real-time analytics on OLTP tables | Adds write overhead; must be maintained |
| Memory-optimized hash / range | Pointers to in-memory rows | Point lookups (hash) / ranges (range) at extreme concurrency | Needs memory; bucket count sizing |

**Mixing them.** A columnstore table can have nonclustered rowstore B-tree indexes for selective lookups and constraint enforcement; a rowstore table can have a nonclustered columnstore index — optionally filtered to warm/cold rows — so analytics doesn't scan the hot OLTP rows. Ordered columnstore (2022+) sorts data before compressing so segment elimination works better.

### 4. Worked example
```sql
-- A 1M-row order table with only a clustered primary key
CREATE TABLE dbo.Orders
( OrderID    INT IDENTITY PRIMARY KEY CLUSTERED,
  CustomerID INT NOT NULL,
  OrderDate  DATE NOT NULL,
  Status     TINYINT NOT NULL,
  Amount     DECIMAL(10,2) NOT NULL );
-- (load data here)

SET STATISTICS IO ON;

-- Query A: without a supporting index → Clustered Index Scan, ~ all pages read
SELECT OrderID, OrderDate, Amount FROM dbo.Orders WHERE CustomerID = 4711;

-- Add a narrow nonclustered index
CREATE INDEX IX_Orders_Customer ON dbo.Orders (CustomerID);
-- Query A now: Index Seek + Key Lookup (one lookup per matching row)

-- Make it covering
CREATE INDEX IX_Orders_Customer ON dbo.Orders (CustomerID)
INCLUDE (OrderDate, Amount) WITH (DROP_EXISTING = ON);
-- Query A now: Index Seek only, a handful of reads

-- Query B: analytics over the whole table
SELECT Status, SUM(Amount) FROM dbo.Orders GROUP BY Status;
CREATE NONCLUSTERED COLUMNSTORE INDEX NCCI_Orders ON dbo.Orders (Status, Amount);
-- Query B now: Columnstore Index Scan in batch mode, reading two compressed columns
```
**What to look at:** in the Messages tab, `logical reads` drop from thousands (scan) → tens (seek + lookups for a few dozen rows) → single digits (covering seek). For the aggregate, the plan shows `Columnstore Index Scan` with *Actual Execution Mode = Batch*, and `lob logical reads` replaces most page reads, with `segment reads`/`segments skipped` in the output.

### 6. Performance and design implications
- Every index speeds some reads and slows every insert, every delete, and every update that touches its columns; also more log, more memory, more maintenance, larger backups.
- The clustering key is copied into every nonclustered index — a wide clustering key bloats all of them.
- Narrow, selective, frequently used keys win. Column order matters: equality columns first, then range/sort columns.
- Measure with `SET STATISTICS IO, TIME ON`, actual execution plans, Query Store, `sys.dm_db_index_usage_stats` (reads vs writes per index), and `sys.dm_db_missing_index_*` (suggestions, never to be applied blindly).
- Analytics workloads benefit from columnstore: 5–10× compression is typical, plus batch mode execution.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `CREATE [UNIQUE] [CLUSTERED|NONCLUSTERED] INDEX … ON t (keys) INCLUDE (…) WHERE …` | Creates a rowstore index (optionally filtered) | Supporting specific predicates/joins |
| `CREATE [CLUSTERED|NONCLUSTERED] COLUMNSTORE INDEX` | Creates a columnstore | Analytics / large scans |
| `WITH (ONLINE = ON, FILLFACTOR = n, DATA_COMPRESSION = PAGE, DROP_EXISTING = ON)` | Build options | Production builds, redefinitions |
| `ALTER INDEX … REBUILD / REORGANIZE / DISABLE` | Maintenance | Fragmentation, bulk loads |
| `SET STATISTICS IO, TIME ON` | Reads and CPU per query | Comparing index options |
| `sys.indexes`, `sys.index_columns` | Index metadata | Inventory |
| `sys.dm_db_index_usage_stats` | Seeks/scans/lookups/updates since restart | Finding unused indexes |
| `sys.dm_db_missing_index_details` | Optimizer suggestions | Starting point for design |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| Clustered index on a narrow, unique, ever-increasing key for most OLTP tables | Using a random GUID or wide composite as clustering key |
| Nonclustered + INCLUDE for frequent, selective queries | Including half the table's columns in many indexes |
| Columnstore for large fact tables and aggregates | Workloads of singleton lookups and constant row updates |
| Filtered index when queries target a small, well-defined subset | Parameterized queries whose predicate doesn't match the filter |
| Heap for staging tables loaded and truncated in bulk | Long-lived tables with updates (forwarded records) |

### 9. Common mistakes
- Indexing every column separately → many single-column indexes the optimizer rarely uses, heavy write cost → design composite, covering indexes from real queries.
- Applying all missing-index suggestions → duplicate and overlapping indexes → consolidate; check usage stats after a business cycle.
- Non-SARGable predicates (`WHERE CONVERT(varchar, OrderDate) = …`) → index exists but is scanned → rewrite the predicate.
- Ignoring column order in composite keys → index on (OrderDate, CustomerID) can't seek on `CustomerID = @c` alone → leading column must match the predicate.
- Dropping an "unused" index right after a restart → usage stats reset on restart → look over a full business cycle (month-end too).

### 10. 🎯 Interview kit
**30-second answer:** "SQL Server has rowstore indexes — a clustered index, which is the table itself as a B-tree, and nonclustered B-trees that point back to it — and columnstore indexes, which store data column-wise and compressed for analytics. The optimizer uses statistics to choose between seeks, scans and lookups; a nonclustered index is only used for small result sets unless it covers the query. Each index speeds reads but costs writes and maintenance, so I design from the workload and verify with plans, STATISTICS IO and usage DMVs."

**Structure a longer answer:** clarify workload (OLTP vs analytics) → structures (heap, clustered, nonclustered, columnstore) → how the optimizer chooses (selectivity, tipping point, covering) → costs (writes, storage, maintenance) → scenario: slow customer lookup fixed by a covering index, or a report fixed by a nonclustered columnstore.

**Likely follow-ups:**
- *Clustered vs nonclustered?* → clustered = the data in key order, one per table; nonclustered = separate structure with row locator.
- *What is a key lookup and when is it a problem?* → extra random read per row; bad when many rows qualify → cover or let it scan.
- *Why not index everything?* → write amplification, memory, maintenance, plan choice confusion.
- *When columnstore?* → large scans/aggregations, millions of rows; not for point lookups.

**Signals interviewers look for:** thinks in pages read; knows writes pay for indexes; understands covering and column order; mentions measuring, not guessing.

### 11. Check yourself
- [ ] Explain the difference between a seek, a scan and a lookup with the pages each one touches.
- [ ] Turn a seek + key lookup plan into a covering seek and measure the logical reads before and after.
- [ ] State why the clustering key's width affects every nonclustered index.
- [ ] Name a workload for each structure in the table in section 3.
- [ ] Explain the tipping point in your own words.
<!-- explainer:end -->

## In this section (study in order)

1. [Clustered Indexes](<Clustered_Indexes/_index.md>) (L4 Advanced)
2. [Nonclustered Indexes](<Nonclustered_Indexes/_index.md>) (L4 Advanced)
3. [Columnstore Indexes](<Columnstore/_index.md>) (L4 Advanced)
4. [Partitioned Tables and Indexes](<Partitioned_Indexes/_index.md>) (L4 Advanced)
5. [In-Memory OLTP and Hash Indexes](<Memory_Optimized_Indexes/_index.md>) (L4 Advanced)
6. [XML Spatial and Full-Text Indexes](<Special_Indexes/_index.md>) (L4 Advanced)
7. [Index Maintenance](<Index_Maintenance/_index.md>) (L4 Advanced)
8. [Index Analysis with DMVs](<Index_Analysis_DMVs/_index.md>) (L4 Advanced)
9. [Index Design Methodology](<Index_Design_Methodology/_index.md>) (L4 Advanced)

## References

- *Expert Performance Indexing in SQL Server* — Jason Strate, Grant Fritchey — Columnstore Index
- *Analytics Optimization with Columnstore Indexes in Microsoft SQL Server* — Edward Pollack — Columnstore Index arChIteCture R HTE TURE › Non-clustered Columnstore Index Architecture
- *Analytics Optimization with Columnstore Indexes in Microsoft SQL Server* — Edward Pollack — NoN-Clustered ColumNstore INdexes oN rowstore tables - LUSTERED COLUM STORE! DEXESO OWSTORE › Non-clustered Columnstore Indexes
- *Expert Performance Indexing in SQL Server* — Jason Strate, Grant Fritchey — Foreign Keys › Columnstore Index
- *Analytics Optimization with Columnstore Indexes in Microsoft SQL Server* — Edward Pollack — Ordered Columnstore Indexes › Non-clustered Ordered Columnstore Indexes
- *Analytics Optimization with Columnstore Indexes in Microsoft SQL Server* — Edward Pollack — Non-clustered Rowstore Indexes on Columnstore Tables › Non-clustered Rowstore Index Guidance
- *Analytics Optimization with Columnstore Indexes in Microsoft SQL Server* — Edward Pollack — Non-clustered Rowstore Indexes on Columnstore Tables
- *Analytics Optimization with Columnstore Indexes in Microsoft SQL Server* — Edward Pollack — NoN-Clustered ColumNstore INdexes oN rowstore tables - LUSTERED COLUM STORE! DEXESO OWSTORE
- *SQL Server 2025 Query Performance Tuning (7th ed.)* — Grant Fritchey — FULL_BOOK_NEEDS_MANUAL_SPLIT (cont.) › Chapter 9 Index Architecture
- *Expert Performance Indexing in SQL Server* — Jason Strate, Grant Fritchey — Clustered Index › Nonclustered Index

