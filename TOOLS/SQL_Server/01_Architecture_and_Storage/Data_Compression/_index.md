---
id: "TOOLS/SQL_Server/01_Architecture_and_Storage/Data_Compression"
level: 4
type: "curriculum"
curriculum: "sql_server"
---

# Data Compression

**L4 Advanced** · [↑ Architecture and Storage](<../_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** Row and page compression change how rows are encoded on 8 KB pages so more rows fit per page — trading extra CPU for fewer pages to read, cache and log.

### 1. The big picture
Because SQL Server reads and caches whole 8 KB pages, the number of pages a table occupies drives I/O, memory use, backup size and even the amount of log a rebuild produces. Data compression attacks that number directly. Many DBAs assume compression is purely a storage saving that costs performance; on an **I/O-bound** server with **spare CPU**, the opposite is often true — queries read fewer pages, more of the table fits in the buffer pool, and scans run faster.

SQL Server offers two rowstore compression levels, configurable per index and per partition: **ROW** (a more compact row format) and **PAGE** (row compression plus prefix and dictionary compression of repeated values on each page). Columnstore indexes have their own, much stronger compression (column segments, dictionaries, run-length encoding; `COLUMNSTORE_ARCHIVE` adds more) — covered in the columnstore lessons. This lesson is about rowstore heaps and B-trees.

### 2. Core concepts and vocabulary
- **Slot** — the physical container of one row on a page; the slot array at the end of the page points to each slot.
- **Row compression** — stores fixed-length types in variable-length form, trims padding, and uses a compact row header.
- **Vardecimal-style numeric storage** — numeric types use only the bytes the actual value needs: a `BIGINT` holding 10 takes 1 byte, 50,000 takes about 4 bytes, `NULL` and 0 take no data bytes.
- **Prefix compression** — per column, finds a common leading value on the page, stores it once in a *compression information (CI)* structure after the page header, and replaces each value with "how many bytes of the prefix to reuse + the remainder".
- **Dictionary compression** — after prefix compression, finds values repeated anywhere on the page (across all columns), stores them once in a page dictionary, and replaces occurrences with a short reference.
- **Page compression** — row + prefix + dictionary compression, in that order.
- **Compression attempt / success** — page compression is only kept if it saves enough space; otherwise the page stays row-compressed. Counted in `sys.dm_db_index_operational_stats`.
- **`sp_estimate_data_compression_savings`** — samples an object and estimates its size under NONE, ROW, PAGE (and columnstore options in newer versions).

### 3. How it works, step by step
**Uncompressed row format (for contrast).** Each row has a header, a fixed-length section where every fixed column takes its full declared size (`CHAR(50)` always 50 bytes, `BIGINT` always 8, even if NULL), a null bitmap, and a variable-length section with an offset array.

![Structure of an uncompressed data page: page header, slots holding rows, and the slot array; each row has a header, fixed-length data, null bitmap and variable-length data](<../../../../_assets/100_SQL_Server_Mistakes_and_How_to_Avoid_-_Peter_A_Carter/CH10_F10_Carter.png>)

**Row compression** rewrites each row:
1. The row header is replaced by a compact *column descriptor* array that stores, per column, a small length code (4 bits) instead of fixed offsets.
2. Integers, decimals, money, date/time types are stored in the minimum bytes needed for the value (0–255 → 1 byte; 256 → 2 bytes, and so on).
3. `CHAR`/`NCHAR` trailing spaces and `BINARY` trailing zeros are not stored; the type's behaviour is unchanged when you read it back.
4. `NCHAR`/`NVARCHAR` (non-MAX) use Unicode compression (SCSU), storing many characters in 1 byte instead of 2.
5. `NULL` and zero values take no data bytes.

![Row-compressed page: the same slots, but each row uses a compact descriptor and variable-length storage for every column](<../../../../_assets/100_SQL_Server_Mistakes_and_How_to_Avoid_-_Peter_A_Carter/CH10_F11_Carter.png>)

**Page compression** runs when a page fills up:
1. Rows are row-compressed first.
2. **Prefix step:** for each column, choose a prefix value (e.g. `abc`) and store it in the CI area. A value `abcd` becomes "reuse 3 bytes + `d`"; a value `abx` becomes "reuse 2 + `x`".
3. **Dictionary step:** scan all columns for repeating values (including already prefix-encoded ones like "3+`d`") and store each once in the page dictionary; rows store a reference.
4. If the result doesn't save enough space, the page is left as row-compressed and the attempt is counted as a failure.
5. Important detail: on a heap or index, new rows go in as row-compressed; page compression is attempted when the page is full. On heaps, regular inserts don't get page compression until a rebuild (or bulk insert with `TABLOCK`).

| Feature | NONE | ROW | PAGE |
|---|---|---|---|
| Variable-length numerics | — | ✔ | ✔ |
| Trim CHAR/BINARY padding | — | ✔ | ✔ |
| Unicode compression | — | ✔ | ✔ |
| Prefix + dictionary | — | — | ✔ |
| Typical saving | 0 | 10–40% | 40–75% on repetitive data |
| CPU cost | none | low | moderate |

### 4. Worked example
```sql
-- 1. Copy a table three times
SELECT * INTO dbo.NoCompression   FROM Sales.SalesOrderDetail;
SELECT * INTO dbo.RowCompression  FROM Sales.SalesOrderDetail;
SELECT * INTO dbo.PageCompression FROM Sales.SalesOrderDetail;

-- 2. Estimate before changing anything
EXEC sp_estimate_data_compression_savings
     @schema_name = 'dbo', @object_name = 'NoCompression',
     @index_id = NULL, @partition_number = NULL, @data_compression = 'PAGE';

-- 3. Apply compression via clustered indexes
CREATE CLUSTERED INDEX CIX ON dbo.NoCompression  (SalesOrderID, SalesOrderDetailID);
CREATE CLUSTERED INDEX CIX ON dbo.RowCompression (SalesOrderID, SalesOrderDetailID)
       WITH (DATA_COMPRESSION = ROW);
CREATE CLUSTERED INDEX CIX ON dbo.PageCompression (SalesOrderID, SalesOrderDetailID)
       WITH (DATA_COMPRESSION = PAGE);

-- 4. Compare page counts
SELECT OBJECT_NAME(ps.object_id) AS tbl, p.data_compression_desc, ps.in_row_used_page_count
FROM sys.dm_db_partition_stats ps
JOIN sys.partitions p ON p.partition_id = ps.partition_id
WHERE ps.object_id IN (OBJECT_ID('dbo.NoCompression'), OBJECT_ID('dbo.RowCompression'), OBJECT_ID('dbo.PageCompression'))
  AND ps.index_id = 1;

-- 5. Compare query cost
SET STATISTICS IO, TIME ON;
SELECT SUM(LineTotal) FROM dbo.NoCompression;
SELECT SUM(LineTotal) FROM dbo.PageCompression;
SET STATISTICS IO, TIME OFF;
```
How to read it:
- Step 2 returns current size and estimated size from a sample; trust it as a guide, not a guarantee.
- Step 4 typically shows row compression cutting pages by a quarter to a third and page compression by half or more on this kind of order data (repeated dates, product IDs, prices).
- Step 5: logical reads drop in proportion to page count; CPU time rises a little for the compressed version. If the uncompressed query had physical reads, elapsed time usually drops.

A cautionary case from the books: a table built from GUIDs and random strings shrank about 28% with ROW (46,982 → 33,687 pages), but PAGE compression saved *nothing more* — random data has no prefixes or repeated values to factor out — while costing more CPU. Running the estimate first would have pointed to ROW.

### 5. Under the hood
- **Where the data stays compressed.** Pages are compressed on disk *and* in the buffer pool. They are decompressed row by row only when a query reads a column value. So compression effectively enlarges your cache — this is a big part of the performance gain.
- **Writes cost CPU.** Inserts and updates must encode values; when a page-compressed page fills, the engine rebuilds its CI structure and retries the insert. Heavy update workloads on page-compressed tables pay this repeatedly.
- **Success counters.** `page_compression_attempt_count` and `page_compression_success_count` in `sys.dm_db_index_operational_stats` show whether page compression is actually working. Images or encrypted data, for instance, fail most attempts (one example: 7 successes vs 46 failures), meaning you pay CPU for nothing.
- **What isn't compressed.** Off-row LOB data (`VARCHAR(MAX)` stored out of row, XML, images), and row-overflow data pages are not row/page compressed. Consider `COMPRESS()`/`DECOMPRESS()` (GZip) for large text you rarely search.
- **Row size limit.** Compression can't be enabled if the maximum possible row size plus compression overhead would exceed 8,060 bytes.
- **Granularity.** Compression is set per index and per partition (`REBUILD PARTITION = n WITH (DATA_COMPRESSION = PAGE)`), so you can page-compress cold historical partitions and leave the hot one uncompressed or ROW.
- **Availability.** Since SQL Server 2016 SP1, compression is available in all editions (previously Enterprise only).

### 6. Performance and design implications
- **Good fit:** large tables, scan-heavy or read-mostly workloads, I/O-bound or memory-bound servers with CPU headroom, data with many repeated values (dates, codes, status flags, foreign keys), wide fixed-length columns that are mostly empty.
- **Poor fit:** CPU-bound servers; tables with heavy random updates; data with no repetition (GUIDs, hashes, encrypted or already-compressed binary).
- **Measure:** page counts (`sys.dm_db_partition_stats`), `STATISTICS IO/TIME`, compression success rates, overall CPU and `PAGEIOLATCH` waits before and after.
- **Side benefits:** smaller backups (on top of backup compression), less log for index rebuilds, faster `DBCC CHECKDB`, smaller memory footprint.
- **Columnstore** often achieves 5–10× compression for analytical tables; reports its reads as *LOB logical reads* in `STATISTICS IO`.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `sp_estimate_data_compression_savings` | Estimates size under a compression setting | Always, before compressing |
| `ALTER TABLE … REBUILD WITH (DATA_COMPRESSION = ROW/PAGE/NONE)` | Compresses a heap or clustered index | Applying/removing compression |
| `ALTER INDEX … REBUILD WITH (DATA_COMPRESSION = …)` | Compresses one index | Nonclustered indexes are separate |
| `… REBUILD PARTITION = n WITH (DATA_COMPRESSION = PAGE)` | Per-partition compression | Archive partitions |
| `sys.partitions.data_compression_desc` | Shows current setting | Audit |
| `sys.dm_db_index_operational_stats` (`page_compression_*`) | Attempts vs successes | Verify page compression pays off |
| `COMPRESS()` / `DECOMPRESS()` | GZip a value | Large LOB text |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| ROW — almost any large table with numerics, CHAR padding, Unicode | CPU already saturated |
| PAGE — read-mostly, repetitive data, archive partitions | Hot OLTP tables with frequent updates; random data (estimate shows little gain over ROW) |
| Columnstore — analytical scans over millions of rows | Singleton lookups and heavy updates |
| Per-partition mix: PAGE for old, ROW/NONE for current | Forgetting that each nonclustered index needs its own setting |

### 9. Common mistakes
- **Never compressing because "compression is slow"** → wasted I/O and memory on I/O-bound servers → estimate and test; often queries get faster.
- **Choosing PAGE without estimating** → extra CPU with no extra savings on random data → run `sp_estimate_data_compression_savings` for both ROW and PAGE.
- **Compressing the clustered index only** → nonclustered indexes stay big → set `DATA_COMPRESSION` on each index.
- **Expecting LOB columns to shrink** → no change for off-row `MAX` data → use `COMPRESS()` or columnstore.
- **Ignoring compression failure counters** → PAGE on images/encrypted data burns CPU → check `page_compression_success_count`.

### 10. 🎯 Interview kit
**30-second answer:** SQL Server rowstore compression comes in ROW and PAGE. ROW stores fixed-length types in variable length — numbers use only the bytes they need, padding is trimmed, Unicode is compacted. PAGE adds prefix and dictionary compression of repeated values per page. Pages stay compressed in memory, so you read and cache fewer pages at the cost of CPU. It pays off on large, I/O- or memory-bound, read-heavy tables; estimate first with `sp_estimate_data_compression_savings`.

**Structure a longer answer:** clarify workload and bottleneck (I/O vs CPU) → explain ROW mechanics → PAGE's prefix/dictionary steps → in-memory benefit → trade-offs and where it fails (random data, LOBs, heavy updates) → scenario: page-compress old partitions of a fact table.

**Likely follow-ups:**
- *Is data decompressed when read into memory?* → no; it's decompressed per column value when a query uses it.
- *Why might PAGE give no gain over ROW?* → no common prefixes/repeated values (GUIDs, random strings).
- *Does compression affect LOBs?* → off-row LOB data isn't compressed.
- *Row vs columnstore compression?* → columnstore compresses per column across a million-row group; much higher ratios, for analytics.

**Signals interviewers look for:** you frame compression as an I/O vs CPU trade, you measure before and after, and you know its limits.

### 11. Check yourself
- [ ] Explain what row compression does to a `BIGINT` holding 10, a `CHAR(50)` holding `'abc'`, and a `NULL`.
- [ ] Walk through prefix then dictionary compression on a page of 'abc', 'abcd', 'abx' values.
- [ ] Run `sp_estimate_data_compression_savings` for ROW and PAGE on a large table and decide which to use.
- [ ] Check page compression success rates for an index.
- [ ] Describe a workload where compression would make things worse.
<!-- explainer:end -->

## References

- *Expert Performance Indexing in SQL Server* — Jason Strate, Grant Fritchey — Row Compression
- *Expert Performance Indexing in SQL Server* — Jason Strate, Grant Fritchey — Page Compression
- *100 SQL Server Mistakes and How to Avoid Them* — Peter A. Carter — Optimization › 10.9 #62 Not compressing large tables
- *Expert Performance Indexing in SQL Server* — Jason Strate, Grant Fritchey — Compression
- *SQL Server 2025 Query Performance Tuning (7th ed.)* — Grant Fritchey — FULL_BOOK_NEEDS_MANUAL_SPLIT (cont.) › Chapter 15 Reduce Query Resource Use
- *Analytics Optimization with Columnstore Indexes in Microsoft SQL Server* — Edward Pollack — Columnstore Compression › Other Compression Algorithms

