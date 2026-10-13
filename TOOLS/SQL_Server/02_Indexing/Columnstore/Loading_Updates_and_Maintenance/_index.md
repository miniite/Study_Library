---
id: "TOOLS/SQL_Server/02_Indexing/Columnstore/Loading_Updates_and_Maintenance"
level: 5
type: "curriculum"
curriculum: "sql_server"
---

# Loading Updates and Maintenance

**L5 Expert** · [↑ Columnstore Indexes](<../_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** A columnstore stays fast only if data arrives in big ordered batches (≥ 102,400 rows, ideally full 1,048,576-row rowgroups), avoids updates, and is reorganized or rebuilt (preferably per partition) when deleted rows, open delta stores or small rowgroups pile up.

### 1. The big picture
The architecture lesson showed that compressed rowgroups are immutable: inserts go to delta stores, and deletes and updates go to the delete bitmap. So the *way you write* to a columnstore decides how healthy it stays. A table loaded with large ordered bulk inserts and rarely updated can run for years with almost no maintenance. The book calls this "the no-maintenance columnstore index". A table fed by trickle inserts, frequent updates and random deletes degrades steadily: small rowgroups, dead rows, big delta stores, and lost ordering.

This lesson covers the practical lifecycle: **loading** (bulk vs trickle, batch sizes, minimal logging), **deletes and updates** (what they cost and how to do them safely), and **maintenance** (when to reorganize vs rebuild, doing it per partition, and the automatic merge from SQL Server 2019 on). It also covers how rowstore fragmentation thinking does *not* transfer directly to columnstore.

### 2. Core concepts and vocabulary
- **Bulk load**: an insert of ≥ **102,400 rows** in one batch, via `INSERT … SELECT`, `BULK INSERT`, bcp or SSIS with a large batch size. The rows are compressed directly into rowgroups, skipping the delta store.
- **Trickle insert**: small inserts (< 102,400 rows) that land in an open delta rowgroup.
- **Minimal logging**: logging only page allocations instead of every row. For a CCI, bulk loads straight into compressed rowgroups are minimally logged. `TABLOCK` is not needed for that (it is needed for parallel insert).
- **Parallel insert**: `INSERT … SELECT` with `WITH (TABLOCK)` can insert into a CCI in parallel (2016+). Each thread builds its own rowgroups, which can leave each thread's last rowgroup small.
- **Columnstore fragmentation**: for columnstore this means deleted rows, undersized rowgroups, rows sitting in delta stores, and disorder on the elimination column. It is *not* page-order fragmentation.
- **REORGANIZE** (online): compresses `CLOSED` delta rowgroups. With `COMPRESS_ALL_ROW_GROUPS = ON` it also compresses `OPEN` ones. It **merges** small compressed rowgroups, and **removes deleted rows** from rowgroups with more than ~10% deleted (2016+).
- **REBUILD**: rewrites the entire index or partition from scratch. It removes all deleted rows, produces full rowgroups, and applies ORDER if defined. Offline for CCI before 2019, online since 2019.
- **Background merge (tuple mover, 2019+)**: merges small compressed rowgroups and purges deleted rows automatically, in the background.
- **Hot / warm / cold data**: recent rows that still change, older rows that are read but stable, and old rows that are rarely read. Maintenance and compression choices differ per tier.

### 3. How it works, step by step
**Loading well:**
1. Stage incoming data (e.g. into a heap or staging table).
2. Insert into the CCI in batches of **1,048,576 rows** (or a multiple of it), **ordered by the main filter column**. Each batch produces full, ordered rowgroups. Batches between 102,400 and 1,048,575 rows still compress directly, but as smaller ("trimmed") rowgroups.
3. Leave the remainder (< 102,400 rows) in the delta store, or compress it at the end with `REORGANIZE WITH (COMPRESS_ALL_ROW_GROUPS = ON)` if no more data is coming soon.
4. With SSIS or bcp, set the *rows per batch / commit size* to 1,048,576 so each batch becomes one full rowgroup. The defaults (like 10,000) send everything into delta stores.
5. For very large one-off loads, consider loading into an empty partition or staging table, building the CCI there, and using `ALTER TABLE … SWITCH` into the target partition.

**Deletes and updates:**
1. A delete of rows in compressed rowgroups only marks them in the delete bitmap. Space is not freed, and scans still read the rows.
2. An update writes the new version to the delta store and marks the old one deleted. Large updates create many dead rows and huge delta stores, and they scramble order.
3. Better patterns: (a) for a whole period, rebuild the affected partition from corrected data, or switch out → fix → switch in; (b) `TRUNCATE TABLE … WITH (PARTITIONS (n))` (2016+) to drop a period instantly; (c) batch large deletes (e.g. 100K–1M rows per transaction) so the log and locks stay small.
4. For tables with a hot, still-changing tail, keep that tail in rowstore (e.g. a separate table or a filtered NCCI design), or use `COMPRESSION_DELAY` so recent rows stay in delta stores until they settle.

**Maintenance decision:**
1. Measure per rowgroup or partition: `deleted_rows / total_rows`, count of rowgroups far below 1,048,576, rows in OPEN/CLOSED delta stores, and ordering overlap on the filter column.
2. If a partition has more than ~10% deleted rows, or many small rowgroups → `REORGANIZE` that partition.
3. If order has decayed, there are lots of deleted rows across all rowgroups, or you need a different ORDER/compression → `REBUILD PARTITION = n` (ideally from sorted data).
4. Only maintain partitions that changed. The old partitions of an append-only fact table never need maintenance.

| Symptom | Fix |
|---|---|
| Many CLOSED delta rowgroups | wait for the tuple mover, or `REORGANIZE` |
| OPEN delta store left after the final load | `REORGANIZE WITH (COMPRESS_ALL_ROW_GROUPS = ON)` |
| Rowgroups with > 10% deleted rows | `REORGANIZE` (purges them) |
| Many small compressed rowgroups | `REORGANIZE` (merges) or `REBUILD` |
| Order lost → `segment skipped` near 0 | `REBUILD PARTITION` from sorted data / ordered CCI |

### 4. Worked example
```sql
-- Health per partition
SELECT p.partition_number,
       COUNT(*)                                        AS rowgroups,
       SUM(CASE WHEN rg.state_desc IN ('OPEN','CLOSED') THEN rg.total_rows ELSE 0 END) AS delta_rows,
       SUM(rg.total_rows)                              AS total_rows,
       SUM(rg.deleted_rows)                            AS deleted_rows,
       CAST(100.0 * SUM(rg.deleted_rows) / NULLIF(SUM(rg.total_rows),0) AS decimal(5,2)) AS pct_deleted,
       SUM(CASE WHEN rg.state_desc = 'COMPRESSED' AND rg.total_rows < 500000 THEN 1 ELSE 0 END) AS small_rowgroups
FROM sys.dm_db_column_store_row_group_physical_stats AS rg
JOIN sys.partitions AS p
  ON p.object_id = rg.object_id AND p.index_id = rg.index_id AND p.partition_number = rg.partition_number
WHERE rg.object_id = OBJECT_ID('Fact.Sale_CCI')
GROUP BY p.partition_number
ORDER BY p.partition_number;

-- Targeted maintenance: only the current, changed partition
ALTER INDEX CCI_fact_Sale_CCI ON Fact.Sale_CCI REORGANIZE PARTITION = 48;

-- Deep fix for a partition whose order has decayed (2019+: ONLINE allowed)
ALTER INDEX CCI_fact_Sale_CCI ON Fact.Sale_CCI REBUILD PARTITION = 47 WITH (ONLINE = ON);

-- Good loading pattern: full rowgroups, ordered
INSERT INTO Fact.Sale_CCI
SELECT * FROM Staging.Sale
ORDER BY [Invoice Date Key];          -- staging holds exactly the new day/week
```
Reading the result: an append-only table shows `pct_deleted` ≈ 0 and no small rowgroups except in the newest partition, so maintain only that partition. If a partition shows 30% deleted and 40 small rowgroups, a reorganize rewrites those rowgroups. Re-run the health query afterwards: the deleted rows disappear and the small rowgroups merge.

### 5. Under the hood
- **Why 102,400?** Below it, compressing a batch would create rowgroups too small to be worth it, so the rows wait in the delta store. At or above it, the insert compresses directly. Within a single statement, rows are split into batches as they stream through the insert. With parallel insert, each thread decides independently, so a 300K-row parallel insert at DOP 4 can end up entirely in delta stores (each thread receives < 102,400 rows).
- **Merge rules.** `REORGANIZE` (and the 2019+ background merge) combines adjacent compressed rowgroups when the combined row count fits in 1,048,576, and rewrites a rowgroup alone when it has more than ~10% deleted rows. Merging can mix date ranges, slightly reducing order, which is acceptable for small tails.
- **Rebuild memory.** Compressing rowgroups needs memory, roughly proportional to columns × rows. Under a small memory grant, rebuilds and loads produce trimmed rowgroups (`MEMORY_LIMITATION`). Wide tables with large strings are the usual victims. Rebuilding one partition at a time keeps the grants reasonable.
- **Online operations.** CCI online rebuild was added in 2019, and online creation of ordered CCI came later (2025). An offline CCI rebuild blocks the table, so partition-level operations are the standard way to keep windows small.
- **Logging.** Bulk loads into compressed rowgroups log page allocations, not rows, even without `TABLOCK`. Inserts into delta stores are fully logged like any B-tree insert.
- **Rowstore fragmentation rules don't apply.** `avg_fragmentation_in_percent` from `sys.dm_db_index_physical_stats` is meaningless for a CCI. Judge health from rowgroup metadata instead.

### 6. Performance and design implications
- Design loads, not just indexes: batch sizes and load order are the biggest levers for columnstore health.
- An append-only, ordered, partitioned fact table needs maintenance only for its newest partition, and often not even that.
- Updates are the enemy. If a business process must update facts (late corrections), redesign: insert correcting rows, or rebuild the period.
- Track over time: rowgroups per partition, average rows per rowgroup, % deleted, and segment-skipped ratios for key queries.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `INSERT … WITH (TABLOCK) SELECT …` | Parallel insert into a CCI | Large loads (watch per-thread batch sizes) |
| bcp `-b 1048576`, SSIS *Rows per batch* / *Maximum insert commit size* | Batches sized to full rowgroups | Every external load |
| `ALTER INDEX … REORGANIZE [PARTITION = n]` | Compress delta, merge, purge deleted rows (online) | Routine, targeted maintenance |
| `… WITH (COMPRESS_ALL_ROW_GROUPS = ON)` | Also compresses OPEN delta rowgroups | After the final load in a window |
| `ALTER INDEX … REBUILD [PARTITION = n] [WITH (ONLINE = ON)]` | Full rewrite | Order decay, heavy deletes |
| `TRUNCATE TABLE t WITH (PARTITIONS (n))` | Instantly empty a partition | Purging old data |
| `ALTER TABLE … SWITCH PARTITION` | Metadata-only data movement | Load or archive a whole period |
| `COMPRESSION_DELAY` | Delay compressing new delta rows | Hot tail that still changes |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| REORGANIZE: small tails, deleted rows, many small rowgroups | Order is badly lost (REORGANIZE doesn't re-sort) |
| REBUILD per partition: deep cleanup, re-ordering | Whole-table rebuilds on billions of rows (time, log, memory) |
| Partition switch for big loads/purges | Non-aligned indexes on the table (they block switching) |
| `COMPRESSION_DELAY` on volatile recent rows | Pure append loads: it only delays compression |

### 9. Common mistakes
- Loading with SSIS/bcp default batch sizes (e.g. 10K) → everything goes to delta stores and then becomes small rowgroups → set the batch size to 1,048,576.
- Running rowstore index maintenance scripts that judge columnstore by `avg_fragmentation_in_percent` → pointless rebuilds → use rowgroup metadata.
- Rebuilding the whole table nightly → long blocking, wasted I/O → maintain only the changed partitions.
- Updating millions of fact rows in place → a delete-bitmap explosion and lost order → rebuild or switch the affected partition.
- Running `COMPRESS_ALL_ROW_GROUPS` after every micro-batch → thousands of tiny rowgroups → let delta stores fill up, and compress at the end of the load window.

### 10. 🎯 Interview kit
**30-second answer:** "Load columnstores in big ordered batches: at least 102,400 rows to bypass the delta store, ideally 1,048,576 so each batch is a full rowgroup. Avoid updates, because they become delete-bitmap entries plus delta inserts. Maintain by partition: REORGANIZE to compress delta stores, merge small rowgroups and purge deleted rows; REBUILD when order or compression has decayed. Judge health from rowgroup metadata, not rowstore fragmentation percentages."
**Structure a longer answer:** write path recap → loading rules (batch size, order, parallel caveat) → delete/update costs and alternatives (switch, truncate partition) → maintenance decision table → monitoring.
**Likely follow-ups:** *REORGANIZE vs REBUILD?* → online merge/purge vs full rewrite that restores order. *Why are my rowgroups 100K rows?* → batch size, parallel threads, memory or dictionary trimming. *What changed in 2019?* → background merge, online rebuild.
**Signals interviewers look for:** concrete thresholds, partition-level thinking, avoiding updates, and correct health metrics.

### 11. Check yourself
- [ ] Explain what happens to a 300,000-row insert at DOP 1 vs DOP 4.
- [ ] Write the health query and choose REORGANIZE or REBUILD for three sample partitions.
- [ ] Describe a safe way to correct one month of fact data.
- [ ] Configure a bcp/SSIS load for full rowgroups.
- [ ] Explain why `avg_fragmentation_in_percent` is the wrong metric for a CCI.
<!-- explainer:end -->

## References

- *Analytics Optimization with Columnstore Indexes in Microsoft SQL Server* — Edward Pollack — Columnstore Index Maintenance
- *Analytics Optimization with Columnstore Indexes in Microsoft SQL Server* — Edward Pollack — Columnstore Index Maintenance › Reorganize to Remove Delta Rowgroups
- *Analytics Optimization with Columnstore Indexes in Microsoft SQL Server* — Edward Pollack — Columnstore Index Maintenance › The No-Maintenance Columnstore Index
- *Analytics Optimization with Columnstore Indexes in Microsoft SQL Server* — Edward Pollack — Delete anD UpDate OperatiOns UP ATE OPERATI › Delete Operations Delete Operations
- *Analytics Optimization with Columnstore Indexes in Microsoft SQL Server* — Edward Pollack — Bulk loading data › Columnstore Reorganize Operations with Each Data Load
- *SQL Server 2025 Query Performance Tuning (7th ed.)* — Grant Fritchey — FULL_BOOK_NEEDS_MANUAL_SPLIT (cont.) › Chapter 12 Dealing With Index Fragmentation
- *Analytics Optimization with Columnstore Indexes in Microsoft SQL Server* — Edward Pollack — Columnstore Index Maintenance › Columnstore Index Maintenance by Partition
- *Analytics Optimization with Columnstore Indexes in Microsoft SQL Server* — Edward Pollack — Bulk loading data › Bulk Loading into Columnstore Indexes
- *Analytics Optimization with Columnstore Indexes in Microsoft SQL Server* — Edward Pollack — BatCh exeCution H XE UTION › CHAPTER 8
- *Analytics Optimization with Columnstore Indexes in Microsoft SQL Server* — Edward Pollack — NoN-Clustered ColumNstore INdexes oN rowstore tables - LUSTERED COLUM STORE! DEXESO OWSTORE › Managing Hot, Warm, and Cold Transactional Data

