---
id: "TOOLS/SQL_Server/02_Indexing/Columnstore/Columnstore_with_Rowstore"
level: 5
type: "curriculum"
curriculum: "sql_server"
---

# Mixing Columnstore and Rowstore

**L5 Expert** · [↑ Columnstore Indexes](<../_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** Columnstore and rowstore can be mixed on one table: a nonclustered columnstore on an OLTP table gives live analytics, and nonclustered B-trees on a clustered columnstore give fast lookups. Each mix trades write cost for read flexibility.

### 1. The big picture
Workloads are rarely pure. A classic design keeps two systems: an OLTP database tuned for small transactions, and a separate warehouse loaded by ETL for analytics. That separation exists because transactional storage (row-oriented B-trees, normalized tables) is bad at scanning millions of rows, and analytic storage is bad at single-row changes. But ETL adds delay, cost and complexity, and some businesses need "analytics on data as of now".

SQL Server lets you blend the two inside one table:
- **Nonclustered columnstore index (NCCI) on a rowstore table**. The OLTP table keeps its clustered B-tree for transactions, and an updatable columnstore copy of chosen columns serves analytic queries. Microsoft calls this *real-time operational analytics*.
- **Nonclustered rowstore (B-tree) indexes on a clustered columnstore table (CCI)**. The analytic table keeps columnstore storage, and a few B-trees support selective lookups, uniqueness constraints and foreign-key checks that a scan-only structure handles poorly.
- **Batch mode on rowstore** (2019+), a third option. It gives some analytic CPU speed with no extra structure (see *Batch Mode Execution*).

The core question in every case: *are the extra writes worth the faster reads?*

### 2. Core concepts and vocabulary
- **OLTP vs OLAP**: many small reads/writes on few rows vs few large reads over many rows.
- **Real-time operational analytics**: running analytic queries directly against the operational database, with no ETL delay.
- **NCCI**: an updatable (2016+) nonclustered columnstore index on a heap or clustered B-tree. It may include a subset of columns and may be **filtered** (`WHERE …`).
- **Delete buffer**: an NCCI-only B-tree that absorbs deletes quickly. It is later merged into the delete bitmap.
- **COMPRESSION_DELAY**: minutes that delta-store rows must wait before compression. It keeps hot, still-changing rows out of compressed segments.
- **NC B-tree on CCI**: a rowstore index whose leaf rows carry an 8-byte locator (rowgroup id + position) into the columnstore. Supported since 2016.
- **Covering index on a columnstore table**: a B-tree whose key and INCLUDE columns satisfy a query completely, so the columnstore is never touched.
- **Hot / warm / cold data**: recently written (still changing), recent but stable, and old.

### 3. How it works, step by step
**Pattern A: NCCI on an OLTP table**
1. Keep the clustered B-tree (e.g. on `OrderID`) and the existing NC B-trees for OLTP.
2. Create an NCCI on the columns analytics needs, often with a filter that excludes hot rows (e.g. `WHERE OrderStatus = 'Closed'`), or with `COMPRESSION_DELAY` so new rows settle first.
3. Inserts go to the B-tree *and* to the NCCI's delta store. Deletes go to the B-tree and the NCCI delete buffer. Updates touch both.
4. The tuple mover compresses delta rowgroups as usual.
5. Analytic queries choose the NCCI (columnstore scan, batch mode). OLTP queries keep using the B-trees.
6. If the NCCI is filtered, rows outside the filter still sit in the B-tree. The optimizer can combine both: it scans the NCCI for the filtered part and reads the B-tree for the rest, so results stay complete.

**Pattern B: NC B-trees on a CCI**
1. The table is a CCI (warehouse fact or large log).
2. Look at segment metadata for a column that queries filter on but that does not order well, e.g. `Stock Item Key` whose min/max spans almost the full range in every rowgroup. Filters on it get no rowgroup elimination.
3. Add a B-tree keyed on that column and INCLUDE the columns those queries need, e.g. `(Stock Item Key) INCLUDE (Quantity)`. The query becomes a seek, often 100× fewer reads.
4. Or add a unique NC index to enforce a business key (a CCI cannot be unique by itself), or to speed up FK validation and singleton lookups.
5. Every load now also maintains each B-tree. Bulk loads get slower, and the hidden mapping index must track row movement from delta store to compressed rowgroups.

| Mix | Gives you | Costs you |
|---|---|---|
| NCCI on rowstore | Fast analytics on live OLTP data | Extra write work per DML, storage, memory |
| Filtered NCCI | Analytics on cold rows only, hot rows untouched | Queries must tolerate the hybrid plan |
| NC B-tree on CCI | Seeks, uniqueness, FK lookups on a columnstore | Slower loads, more storage, mapping index |
| Batch mode on rowstore | Analytic CPU efficiency, no new structure | No compression or elimination |

### 4. Worked example
```sql
-- Pattern A: OLTP orders table + NCCI for live reporting
CREATE NONCLUSTERED COLUMNSTORE INDEX NCCI_Orders_Analytics
ON Sales.Orders (OrderDate, CustomerID, SalespersonPersonID, OrderStatus, OrderTotal)
WHERE OrderStatus = 'Closed'                -- keep hot rows out
WITH (COMPRESSION_DELAY = 60);              -- 60 minutes before compression

SELECT SalespersonPersonID, SUM(OrderTotal)
FROM Sales.Orders
WHERE OrderDate >= DATEADD(month, -12, SYSDATETIME())
GROUP BY SalespersonPersonID;
-- Plan: Columnstore Index Scan (NCCI) in batch mode (+ B-tree access for non-'Closed' rows if the query needs them)

-- Pattern B: CCI fact table + covering B-tree for a lookup pattern
SET STATISTICS IO ON;
SELECT SUM(Quantity) FROM Fact.Sale_CCI_ORDERED WHERE [Stock Item Key] = 186;
-- Segment reads 26, segment skipped 0  (Stock Item Key is not ordered)

CREATE NONCLUSTERED INDEX IX_Sale_CCI_ORDERED_StockItem
ON Fact.Sale_CCI_ORDERED ([Stock Item Key]) INCLUDE (Quantity);

SELECT SUM(Quantity) FROM Fact.Sale_CCI_ORDERED WHERE [Stock Item Key] = 186;
-- Index Seek on IX_..., a few hundred logical reads instead of thousands of LOB reads
```
Walkthrough: for Pattern B, the book's measurement shows the covering B-tree turns a full columnstore scan into a seek. The index took under a minute to build on a 25M-row table. The price is paid on every subsequent load. Inspect `sys.dm_db_index_usage_stats` later: if the B-tree's `user_seeks` stay low, drop it.

### 5. Under the hood
- **NCCI write path.** Each insert also goes into the NCCI's open delta rowgroup. Deletes go into the **delete buffer** rather than directly to the delete bitmap, because finding a row's position in a compressed rowgroup would require a lookup. A background process later converts buffered deletes into bitmap entries. Scans of the NCCI must anti-join both the delete buffer and the bitmap. You can see this as `delete_buffer_scan_count` in the operational-stats DMV.
- **COMPRESSION_DELAY in practice.** OLTP rows are often updated several times in the minutes after creation (status changes). Without a delay, each update would leave a deleted row in a compressed segment. With a delay, those rows are still in the delta store (a B-tree), where updates are cheap. Choose a delay longer than the typical "hot" lifetime of a row.
- **NC B-tree on CCI locators.** The B-tree stores (rowgroup id, row position) for compressed rows. When the tuple mover compresses a delta rowgroup, those positions change. SQL Server avoids rewriting every B-tree by keeping a mapping index from old to new locators. Lookups may go through it, and loads must maintain it.
- **Key lookups into a CCI** are expensive: fetching one row means decompressing segment data for each needed column. So B-trees on a CCI should *cover* their queries. Plain non-covering B-trees that cause lookups into the columnstore are rarely a good idea except for very few rows.
- **Uniqueness.** A CCI cannot be declared unique. A unique NC B-tree or primary key constraint on a CCI table enforces it (2016+). Every insert must check it, which is another load cost.
- **Partitioning / hot-warm-cold.** An alternative to one hybrid table: keep hot data in a rowstore partition or table and cold data in a columnstore, unified by a view or by partition-level compression choices. The book discusses managing hot, warm and cold data this way. It avoids NCCI overhead on the hottest rows.

### 6. Performance and design implications
- First check whether real-time analytics is required at all. If reports can be minutes or hours behind, a separate warehouse/CCI loaded in bulk is simpler and faster.
- For NCCIs, measure the OLTP impact: transaction latency, log volume, and `row_group_lock_wait_*` on the NCCI, before and after.
- For B-trees on a CCI, keep them few and covering. Measure their load-time cost, and justify each one with usage stats.
- Batch mode on rowstore may cover moderate analytic needs on OLTP tables without any NCCI. Test it first when analytic queries are isolated and modest.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `CREATE NONCLUSTERED COLUMNSTORE INDEX … (cols) [WHERE …]` | Columnstore copy on a rowstore table | Real-time analytics |
| `WITH (COMPRESSION_DELAY = n)` | Keep new rows in delta for n minutes | Hot OLTP rows |
| `CREATE NONCLUSTERED INDEX … ON cci_table (…) INCLUDE (…)` | B-tree on columnstore | Selective lookups |
| `ALTER TABLE … ADD CONSTRAINT PK … NONCLUSTERED` on a CCI | Enforce uniqueness | Business keys on a fact table |
| `sys.dm_db_column_store_row_group_operational_stats` | `delete_buffer_scan_count`, lock waits | NCCI overhead diagnosis |
| `sys.dm_db_index_usage_stats` | Seeks/scans/updates per index | Justify or drop extra indexes |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| NCCI: business needs live analytics and can afford extra write cost | OLTP is latency-critical and analytics can be delayed |
| Filtered NCCI / COMPRESSION_DELAY: rows are hot for a while, then settle | Data is updated randomly across its whole life |
| Covering B-tree on CCI: frequent selective filter on an unordered column | Rare queries: the build and load cost isn't justified |
| Separate warehouse (CCI) via ETL | Real-time is a hard requirement |

### 9. Common mistakes
- Adding an NCCI on all columns of a busy OLTP table → write latency and log growth → include only analytic columns, filter hot rows, set a compression delay.
- Non-covering B-trees on a CCI → key lookups that decompress segments → INCLUDE the needed columns or don't build them.
- Many B-trees on a fact table → nightly loads slow down sharply → keep one or two, justified by usage.
- Assuming the NCCI is always used → the optimizer may still choose a B-tree for small ranges → check plans, and test both paths.

### 10. 🎯 Interview kit
**30-second answer:** "You can mix the two. An updatable nonclustered columnstore on an OLTP table gives real-time analytics. Filtering it, or using COMPRESSION_DELAY, keeps hot rows out of compressed segments. On a clustered columnstore you can add covering nonclustered B-trees for selective lookups or to enforce uniqueness. Both add write cost, so use them only when the reads justify it, and consider batch mode on rowstore as a lighter option."
**Structure a longer answer:** why two systems exist → NCCI pattern (write path, delete buffer, delay, filter) → B-tree-on-CCI pattern (locators, covering, mapping index) → alternatives → how to measure.
**Likely follow-ups:** *What is COMPRESSION_DELAY for?* → avoids deleted rows from updates to hot rows. *Why must B-trees on a CCI cover?* → lookups into compressed segments are expensive. *How is uniqueness enforced on a CCI?* → unique NC B-tree / PK constraint.
**Signals interviewers look for:** framing as a trade-off, knowing the write-path consequences, and asking whether real-time analytics is really required.

### 11. Check yourself
- [ ] Design an NCCI for an orders table where orders change for up to 30 minutes.
- [ ] Explain the delete buffer and why NCCIs have one.
- [ ] Use segment metadata to justify a covering B-tree on a CCI.
- [ ] Compare NCCI, batch on rowstore, and a separate warehouse for one scenario.
- [ ] List the load-time costs of a unique constraint on a CCI.
<!-- explainer:end -->

## References

- *Analytics Optimization with Columnstore Indexes in Microsoft SQL Server* — Edward Pollack — IntroduCtIon to analytIC data In a transaCtIonal database › CHAPTER 2
- *Analytics Optimization with Columnstore Indexes in Microsoft SQL Server* — Edward Pollack — partitioning › Use Rowstore Indexes
- *Analytics Optimization with Columnstore Indexes in Microsoft SQL Server* — Edward Pollack — transaCtional vs. analytiC Workloads RANSA TIONAL VS. NALYTI › The Limits of Transactional Data Storage
- *Analytics Optimization with Columnstore Indexes in Microsoft SQL Server* — Edward Pollack — Ordered Columnstore Indexes › Are Real-Time Operational Analytics Required?
- *Analytics Optimization with Columnstore Indexes in Microsoft SQL Server* — Edward Pollack — Columnstore Index Maintenance › Columnstore Reorder and Rebuild
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 3: SELECT Statements and Filtering Logic › Persisted vs. Non-Persisted Computed Columns
- *Expert Performance Indexing in SQL Server* — Jason Strate, Grant Fritchey — Columnstore Fragmentation
- *Analytics Optimization with Columnstore Indexes in Microsoft SQL Server* — Edward Pollack — transaCtional vs. analytiC Workloads RANSA TIONAL VS. NALYTI › The Need for Two Systems
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 10: Transactions, Concurrency, and Isolation Levels › @@TRANCOUNT
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 12: Azure SQL and Synapse Integration › PolyBase Integration in Synapse Analytics

