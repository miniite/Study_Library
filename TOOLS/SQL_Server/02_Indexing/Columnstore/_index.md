---
id: "TOOLS/SQL_Server/02_Indexing/Columnstore"
level: 4
type: "curriculum"
curriculum: "sql_server"
---

# Columnstore Indexes

**L4 Advanced** · [↑ Indexing](<../_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** A columnstore index stores each column separately, compressed, in chunks of about a million rows, so analytic queries read only the columns and chunks they need, often 10× less I/O than a B-tree.

### 1. The big picture
Every index so far in this section (heaps, clustered and nonclustered B-trees) is a **rowstore**: the columns of a row sit together on an 8 KB page. That suits OLTP work like "fetch order 42" or "update one customer". Analytics looks different: "total quantity by month for the last three years" touches millions of rows but only three or four columns. In a rowstore, every page read brings in *all* columns of each row, so most of the I/O and memory goes to data the query throws away.

A **columnstore index** turns the layout 90°. Rows are cut into large groups. Inside each group, the values of one column are stored and compressed together. A query that needs 4 of 25 columns reads roughly 4/25 of the data, and that data is already heavily compressed. Values within a column are similar (same type, often repeated), so columnstore compression is far better than row or page compression. 5–10× is common, and more is possible on low-cardinality data. Columnstore also pairs with **batch mode execution**, where operators process about 900 rows at a time instead of one. Together these features make big scans and aggregations fast.

The child lessons go deeper: **architecture** (rowgroups, delta store, delete bitmap, encodings), **segment and rowgroup elimination**, **batch mode**, **loading, updates and maintenance**, and **mixing columnstore with rowstore**. This lesson covers the map.

### 2. Core concepts and vocabulary
- **Rowstore / columnstore**: row-oriented storage (B-tree or heap) vs column-oriented storage.
- **Clustered columnstore index (CCI)**: the columnstore *is* the table. There is no other copy of the data. This is the typical choice for fact tables and big analytic tables.
- **Nonclustered columnstore index (NCCI)**: a columnstore copy of selected columns, built on top of a rowstore table (heap or clustered B-tree). It is used for analytics on an OLTP table ("real-time operational analytics").
- **Rowgroup**: a horizontal slice of up to **1,048,576 rows (2²⁰)**. It is the unit of compression and of skipping.
- **Segment**: one column's values within one rowgroup, compressed. It is the basic storage unit of a columnstore. A table with 8 columns and 6 rowgroups has 48 segments.
- **Delta store (delta rowgroup)**: a small ordinary B-tree rowgroup that catches trickle inserts until enough rows accumulate to compress.
- **Delete bitmap / delete buffer**: structures that mark rows as deleted without rewriting a compressed segment, which is read-only.
- **Tuple mover**: a background process that compresses full ("closed") delta rowgroups into columnstore format.
- **Segment / rowgroup elimination**: skipping rowgroups whose min/max metadata proves they cannot match the filter.
- **Batch mode**: an execution mode where operators process vectors of rows. It works best with columnstore, and on rowstore tables too from SQL Server 2019 (compatibility level 150).
- **Archive compression** (`COLUMNSTORE_ARCHIVE`): an extra compression layer for cold data. It saves more space and costs more CPU to read.

### 3. How it works, step by step
1. **Build.** `CREATE CLUSTERED COLUMNSTORE INDEX` reads the table and cuts it into rowgroups of up to 1,048,576 rows. With parallel builds, each thread fills its own rowgroups, so the last group per thread is usually smaller.
2. **Encode and compress each column.** For each rowgroup and column, SQL Server chooses an encoding. **Value encoding** stores numbers as small offsets from a base. **Dictionary encoding** replaces repeated values (mostly strings) with small integer IDs. Then **run-length encoding** collapses runs of equal values, and **bit-packing** stores each value in the minimum number of bits. SQL Server may also reorder rows inside the rowgroup (*VertiPaq optimization*) to make runs longer.
3. **Store.** Each segment is written as a LOB-style chain on ordinary 8 KB pages. The 8 KB page is the only big thing columnstore shares with rowstore. Columnstore uses no B-tree.
4. **Keep metadata.** For each segment SQL Server records row count, size, encoding, and the **minimum and maximum value** (`min_data_id`, `max_data_id`). Unlike statistics, this metadata is always exact.
5. **Query.** The only access operator is a **Columnstore Index Scan**. Columnstore has no seek. The scan reads only the needed columns, skips rowgroups whose min/max cannot match the predicate, decompresses the rest, and passes rows in batches to batch-mode operators. Filters and some aggregates can be **pushed down** into the scan itself.
6. **Write small amounts.** Inserts below about 102,400 rows go into the **delta store** (rowstore). When a delta rowgroup reaches 1,048,576 rows it is *closed*, and the tuple mover compresses it.
7. **Write large amounts.** A bulk insert of **102,400 rows or more** per batch goes straight to compressed rowgroups and skips the delta store. This is much faster and minimally logged.
8. **Delete / update.** A delete in a compressed rowgroup only flags the row in the **delete bitmap**. An update is a delete plus an insert into the delta store. The deleted rows still occupy space and are still read and then filtered out, until maintenance removes them.

| | Rowstore B-tree | Columnstore |
|---|---|---|
| Unit of storage | row on a page | segment (one column × one rowgroup) |
| Access paths | seek, range scan, scan | scan only (with elimination) |
| Best at | singleton lookups, small ranges, frequent updates | scans + aggregates over millions of rows |
| Compression | none / row / page | columnstore (+ archive), typically 5–10×+ |
| Execution | row mode (batch mode from 2019) | batch mode |
| Updates | in place | delete bitmap + delta store |

### 4. Worked example
```sql
-- A fact table with ~25M rows
CREATE TABLE Fact.Sale_CCI (
    SaleKey         bigint NOT NULL,
    [Invoice Date Key] date NOT NULL,
    [Customer Key]  int NOT NULL,
    [Stock Item Key] int NOT NULL,
    Quantity        int NOT NULL,
    [Total Excluding Tax] decimal(18,2) NOT NULL
    -- ... more columns
);
INSERT Fact.Sale_CCI WITH (TABLOCK) SELECT ... FROM Fact.Sale;   -- load

CREATE CLUSTERED COLUMNSTORE INDEX CCI_fact_Sale_CCI ON Fact.Sale_CCI;

-- What did we get?
SELECT rg.row_group_id, rg.state_desc, rg.total_rows, rg.deleted_rows,
       rg.size_in_bytes / 1024 AS size_kb
FROM sys.column_store_row_groups AS rg
WHERE rg.object_id = OBJECT_ID('Fact.Sale_CCI')
ORDER BY rg.row_group_id;

SET STATISTICS IO ON;
SELECT SUM(Quantity)
FROM Fact.Sale_CCI
WHERE [Invoice Date Key] >= '2016-01-01' AND [Invoice Date Key] < '2016-02-01';
```
How to read the results:
- `sys.column_store_row_groups` lists about 24 rowgroups in state `COMPRESSED` with about 1,048,576 rows each. A few may be smaller (the last one per build thread).
- `STATISTICS IO` prints an extra line, `Segment reads N, segment skipped M`. If the data was loaded in random date order, every rowgroup contains every month, so you see `segment skipped 0`. If it was loaded in date order, most rowgroups are skipped. The **elimination** lesson shows how to get that ordering.
- In the plan, the scan's *Actual Execution Mode* is `Batch`, and the Properties pane shows `Actual Number of Locally Aggregated Rows` when aggregate pushdown happened.

### 5. Under the hood
- **Why a million rows per rowgroup?** Each compressed unit is read into memory as a whole. If units are too small, you pay overhead per unit and compression suffers. If they are too large, a query reads too much to answer a narrow question. 2²⁰ balances the two. As a result, tables under a few million rows gain little: a 500K-row table fits in one rowgroup, and every query reads all of it. The book suggests columnstore pays off from roughly 5–10 million rows upward.
- **Rowgroups never span partitions.** A partitioned table has its own set of rowgroups in each partition. This lets partition elimination and rowgroup elimination combine.
- **Metadata is real data.** One billion rows means about 1,000 rowgroups. With 21 columns that is about 21,000 segments of metadata, which must also be read into memory. Ordering and partitioning keep the metadata touched by a query small, too.
- **Versions.** 2012: read-only NCCI. 2014: updatable CCI. 2016: updatable NCCI, NC B-trees on a CCI, string and aggregate pushdown, `REORGANIZE` removes deleted rows. 2017: batch-mode adaptive joins, and LOB columns allowed in CCI. 2019: batch mode on rowstore, and the background merge in the tuple mover. 2022: ordered CCI (`ORDER (...)`), and min/max elimination for strings. 2025: ordered nonclustered columnstore and more online options.

### 6. Performance and design implications
- Good fits: fact tables, logging/telemetry, history tables. They hold tens of millions of rows or more, are mostly appended, and are mostly read through filter-plus-aggregate queries.
- Poor fits: tables under a few million rows; singleton lookups by key (each lookup must decompress segments); heavy `UPDATE` traffic (every update leaves a deleted row plus a delta-store row).
- Measure with `SET STATISTICS IO` (segment reads/skipped, LOB logical reads), actual plans (batch vs row mode), `sys.column_store_row_groups` (rowgroup sizes, deleted rows), `sys.column_store_segments` (min/max per column), and `sys.dm_db_column_store_row_group_physical_stats` (trim reasons).
- Typical numbers: 5–10× compression on fact data, often more for low-cardinality columns. Analytic queries often run 10–100× faster than a rowstore scan, from far fewer reads combined with batch mode.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `CREATE CLUSTERED COLUMNSTORE INDEX cci ON t` | Converts the table to columnstore | Large analytic tables |
| `CREATE NONCLUSTERED COLUMNSTORE INDEX ncci ON t (cols)` | Adds a columnstore copy on a rowstore table | Analytics on an OLTP table |
| `... ORDER (col)` (2022+) | Sorts data when building the columnstore | Enables rowgroup elimination on that column |
| `WITH (DATA_COMPRESSION = COLUMNSTORE_ARCHIVE)` | Extra compression | Cold partitions |
| `WITH (COMPRESSION_DELAY = n)` | Keeps delta rows uncompressed for n minutes | Hot rows that are still being updated |
| `ALTER INDEX ... REORGANIZE` | Compresses closed delta rowgroups, merges small rowgroups, purges deleted rows | Routine maintenance |
| `ALTER INDEX ... REBUILD [PARTITION = n]` | Rewrites rowgroups completely | Heavy fragmentation, re-ordering |
| `sys.column_store_row_groups` | Rowgroup state, rows, deleted rows, size | Health check |
| `sys.column_store_segments` | Per-segment encoding, min/max | Elimination analysis |
| `sys.dm_db_column_store_row_group_physical_stats` | Trim reason, delta store state | Why are rowgroups small? |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| CCI: table is big (≥ ~10M rows), append-mostly, queried by scans + aggregates | Table is small or needs many singleton lookups / updates |
| NCCI: OLTP table also needs live reporting | NCCI adds write overhead on every insert/update/delete |
| NC B-tree on CCI: a few selective lookup queries on a columnstore table | Each extra B-tree slows loads and adds storage |
| `COLUMNSTORE_ARCHIVE` on old partitions | Data is still queried often (extra CPU per read) |
| Ordered CCI / load in date order | Data arrives in random order and nobody maintains it |

### 9. Common mistakes
- Building a columnstore on a 300K-row table → one rowgroup, no elimination, little benefit → keep rowstore, or use batch mode on rowstore.
- Loading in small batches (e.g. 10,000 rows at a time) → everything lands in delta stores and is compressed later, possibly into small trimmed rowgroups → batch inserts to ≥ 102,400 rows (ideally 1,048,576).
- Expecting `WHERE id = 42` to seek → columnstore can only scan → add a nonclustered B-tree for that lookup pattern.
- Ignoring deleted rows → `deleted_rows` grows, scans read and discard dead rows → run `REORGANIZE`, and `REBUILD` partitions when needed.
- Loading data unordered → `segment skipped 0` on every query → order by the main filter column (usually date) or use `ORDER`.

### 10. 🎯 Interview kit
**30-second answer:** "A columnstore index stores data by column instead of by row. Rows are split into rowgroups of about a million rows, and each column within a rowgroup is a compressed segment. Queries read only the columns they need, skip rowgroups using exact min/max metadata, and run in batch mode. That makes large scans and aggregations an order of magnitude faster and the data 5–10× smaller. The costs: no seeks, and updates are handled as delete-bitmap entries plus delta-store inserts."
**Structure a longer answer:** clarify the workload (OLAP vs OLTP, table size) → explain rowgroups, segments, compression and elimination → mention the delta store / delete bitmap write path → trade-offs (no seeks, update cost, maintenance) → close with a scenario: "a 2B-row sales fact table loaded nightly in date order, with a CCI and monthly partitions".
**Likely follow-ups:** *CCI vs NCCI?* → CCI is the table itself, NCCI is an extra copy on a rowstore table. *What is the delta store?* → a B-tree rowgroup for small inserts, compressed by the tuple mover at 1,048,576 rows. *Why can't columnstore seek?* → no sorted key structure; it relies on metadata elimination instead. *When is it a bad idea?* → small tables, point lookups, update-heavy workloads.
**Signals interviewers look for:** knowing the 1,048,576 and 102,400 thresholds, explaining elimination (not "it's indexed by every column"), awareness of the delete/update costs, and mentioning batch mode.

### 11. Check yourself
- [ ] Draw a table with 6 columns and 3 rowgroups and label one segment.
- [ ] Explain why a 500K-row table gains little from a CCI.
- [ ] Name what happens to an insert of 50,000 rows vs 500,000 rows.
- [ ] Read `Segment reads / segment skipped` from `STATISTICS IO` and explain a `skipped 0` result.
- [ ] List three workloads where you would *not* use a columnstore.
<!-- explainer:end -->

## In this section (study in order)

1. [Columnstore Architecture](<Columnstore_Architecture/_index.md>) (L5 Expert)
2. [Segment and Rowgroup Elimination](<Segment_and_Rowgroup_Elimination/_index.md>) (L5 Expert)
3. [Batch Mode Execution](<Batch_Mode_Execution/_index.md>) (L5 Expert)
4. [Loading Updates and Maintenance](<Loading_Updates_and_Maintenance/_index.md>) (L5 Expert)
5. [Mixing Columnstore and Rowstore](<Columnstore_with_Rowstore/_index.md>) (L5 Expert)

## References

- *Analytics Optimization with Columnstore Indexes in Microsoft SQL Server* — Edward Pollack — BatCh exeCution H XE UTION › Batch Mode Execution
- *Analytics Optimization with Columnstore Indexes in Microsoft SQL Server* — Edward Pollack — Columnstore metadata
- *Expert Performance Indexing in SQL Server* — Jason Strate, Grant Fritchey — Columnstore Fragmentation
- *Analytics Optimization with Columnstore Indexes in Microsoft SQL Server* — Edward Pollack — Columnstore Compression › Available Columnstore Metadata
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 8: Query Optimization and Performance Tuning › Columnstore Indexes
- *Analytics Optimization with Columnstore Indexes in Microsoft SQL Server* — Edward Pollack — What are Columnstore Indexes RE COLUMNSTORE INDEXES › Rowgroups and Segments Rowgroups and Segments
- *Analytics Optimization with Columnstore Indexes in Microsoft SQL Server* — Edward Pollack — Segment and rowgroup elimination
- *Expert Performance Indexing in SQL Server* — Jason Strate, Grant Fritchey — Dropping an Index › sys.columnstoresegments
- *Analytics Optimization with Columnstore Indexes in Microsoft SQL Server* — Edward Pollack — Columnstore Index Performance
- *Analytics Optimization with Columnstore Indexes in Microsoft SQL Server* — Edward Pollack — Non-clustered Rowstore Indexes on Columnstore Tables

