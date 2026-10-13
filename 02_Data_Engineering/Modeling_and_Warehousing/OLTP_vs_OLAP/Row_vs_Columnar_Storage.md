---
id: "02_Data_Engineering/Modeling_and_Warehousing/OLTP_vs_OLAP/Row_vs_Columnar_Storage"
level: 4
type: "concept"
topic: "02_Data_Engineering/Modeling_and_Warehousing/OLTP_vs_OLAP"
tools: []
---

# Row vs Columnar Storage

**L4 Advanced** · [↑ OLTP vs OLAP](<_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** Row stores keep each record's fields together so single-row work is cheap; column stores keep each column's values together so analytics reads only what it needs and compresses far better.

### 1. The big picture
A table is a logical grid, but a disk is a one-dimensional sequence of bytes. The engine must decide how to flatten the grid: **row by row** or **column by column**. That single decision shapes performance more than almost any index or hardware choice.

Row-oriented (**rowstore**) layout is what classic transactional databases use: all fields of row 1, then all fields of row 2, and so on. Fetching or updating one whole record touches one contiguous place. Column-oriented (**columnstore**) layout stores all values of column A together, then column B, and so on. A query that sums one column over a billion rows reads one column's bytes and ignores the rest.

This lesson explains both layouts, why each wins in its home workload, and how to decide — and prove — that a workload belongs in a column store. It connects upward to OLTP vs OLAP (the workloads) and downward to file formats like Parquet and engine features like SQL Server columnstore indexes.

### 2. Core concepts and vocabulary
- **Row-oriented storage** — values of one record are stored contiguously (page after page of whole rows).
- **Column-oriented storage** — values of one column are stored contiguously, in separate files or segments per column.
- **Tuple reconstruction** — rebuilding a full row from separate column segments using positions/row IDs; needed for `SELECT *`, joins and multi-column filters.
- **Projection** — choosing which columns a query returns; column stores benefit when it's a small subset.
- **Compression / encoding** — dictionary, run-length (RLE), delta, bit-packing; far more effective when similar values sit together.
- **Row group / segment** — a column store chunks a table into groups of rows (e.g., ~1M rows) and stores each column of the group separately with min/max metadata.
- **Data skipping (segment elimination)** — using min/max metadata to skip whole chunks that cannot match a filter.
- **Delta store** — small row-oriented buffer where recent inserts land before being compressed into column segments.
- **Vectorized (batch) execution** — processing a column in blocks of values (e.g., 900–1,000 at a time) instead of one row at a time.

### 3. How it works, step by step
Take this logical table:

| ID | Symbol | Date | Price |
|---|---|---|---|
| 1 | DOW | 08 Aug | 24,314 |
| 2 | DOW | 09 Aug | 24,136 |
| 3 | S&P | 08 Aug | 2,414 |
| 4 | S&P | 09 Aug | 2,232 |

**Physical row layout:** `[1,DOW,08 Aug,24314][2,DOW,09 Aug,24136][3,S&P,08 Aug,2414][4,S&P,09 Aug,2232]`

**Physical column layout:**
```
ID:     1 2 3 4
Symbol: DOW DOW S&P S&P
Date:   08Aug 09Aug 08Aug 09Aug
Price:  24314 24136 2414 2232
```

Now run two queries:

**Q1 — `SELECT * FROM prices WHERE id = 3` (OLTP-style).**
1. Row store: index seek to the page; read one contiguous row. One I/O.
2. Column store: look up position 3 in *each* of four column segments and stitch the values together. Four separate reads plus reconstruction.
→ Row store wins.

**Q2 — `SELECT AVG(price) FROM prices` (OLAP-style).**
1. Row store: read every page, which includes ID/Symbol/Date bytes it does not need, then discard them.
2. Column store: read only the Price column, decompress, aggregate in vectorized batches.
→ Column store reads a fraction of the bytes (e.g., 1 of 50 columns ≈ 2%), and the Price data is compressed better because it is all one type.

In a real table with 50 columns and 500M rows the gap between those two scans is the gap between minutes and seconds.

### 4. Worked example
SQL Server lets you see both layouts on the same data.

```sql
-- Rowstore (clustered B-tree): good for OLTP
CREATE TABLE dbo.SalesRow (
    SaleId      BIGINT IDENTITY PRIMARY KEY,   -- clustered rowstore
    SaleDate    DATE         NOT NULL,
    CustomerId  INT          NOT NULL,
    ProductId   INT          NOT NULL,
    Quantity    INT          NOT NULL,
    Amount      DECIMAL(12,2) NOT NULL
);

-- Columnstore copy for analytics
CREATE TABLE dbo.SalesCol (
    SaleId BIGINT, SaleDate DATE, CustomerId INT,
    ProductId INT, Quantity INT, Amount DECIMAL(12,2)
);
CREATE CLUSTERED COLUMNSTORE INDEX cci_SalesCol ON dbo.SalesCol;

SET STATISTICS IO, TIME ON;

SELECT SUM(Amount) FROM dbo.SalesRow;   -- reads all pages (whole rows)
SELECT SUM(Amount) FROM dbo.SalesCol;   -- reads only the Amount column segments
```

How to read the result: compare "logical reads" and elapsed time. The columnstore version typically reads an order of magnitude fewer pages (often 5–10× smaller on disk from compression, plus column elimination) and runs in batch mode. Look in the plan for **Columnstore Index Scan** with **Actual Execution Mode: Batch**. Then run `SELECT * FROM dbo.SalesCol WHERE SaleId = 12345` — it will be noticeably worse than the rowstore seek, demonstrating the trade-off.

To *prove* a workload should move: capture the top queries, count columns referenced vs columns in the table, measure bytes read per query, and check the read/write mix. If most queries aggregate a few columns over many rows and writes are bulk, a columnar store will pay off.

### 5. Under the hood
- **Compression is the second win.** Same-typed, similar values compress extremely well. Low-cardinality columns (status, country) use dictionary + RLE; sorted dates delta-encode to tiny deltas. Ratios of 5–10× are common, so even a full-column read touches fewer bytes.
- **Min/max metadata** per segment lets a filter like `WHERE SaleDate >= '2026-01-01'` skip whole row groups. Loading data in sort order (by date) makes segments tightly bounded and elimination effective; random order widens ranges and defeats it.
- **Updates are awkward.** Compressed segments are immutable-ish; an UPDATE becomes delete-bitmap + insert (new version). Hence delta stores and background compaction ("tuple mover"), and why heavy row-level churn hurts column stores.
- **Bulk loads** of ≥ ~100k rows go straight into compressed segments, avoiding the delta store and minimizing log usage — large, sorted batches are the right way to load analytic tables, whereas OLTP wants small committed transactions to keep locks short.
- **Hybrid approaches** exist: PAX layouts (columns within a page), nonclustered columnstore on a rowstore table (operational analytics), and Parquet's row-group/column-chunk structure.

### 6. Performance and design implications
- Column stores: scan speed scales with **columns touched × compressed size**, not with table width.
- Row stores: cost of a lookup is near-constant (index depth ≈ 3–4 pages), independent of table size.
- Wide tables (hundreds of columns) are cheap to query in columnar form and expensive in row form.
- Measure with logical reads / bytes scanned, `sys.dm_db_column_store_row_group_physical_stats` (row group sizes, deleted rows, state), and batch vs row execution mode.
- Maintain: keep row groups near full (~1M rows); too many small groups (from trickle inserts) degrade compression and scans.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `CREATE CLUSTERED COLUMNSTORE INDEX` | Makes the table's primary storage columnar | Large fact tables |
| `CREATE NONCLUSTERED COLUMNSTORE INDEX` | Adds a columnar copy over a rowstore table | Analytics on an OLTP table |
| `ALTER INDEX … REORGANIZE` | Compresses closed delta rowgroups, removes deleted rows | After heavy DML on columnstore |
| `sys.dm_db_column_store_row_group_physical_stats` | Inspect row-group state and fragmentation | Maintenance health check |
| Parquet / ORC | Columnar file formats for lakes | Analytics in object storage |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| **Row store**: point lookups, frequent single-row updates, full-row retrieval | Queries aggregate a few columns across most rows |
| **Column store**: scans, aggregates, wide tables, append/bulk loads | Frequent single-row updates/deletes, `SELECT *` lookups at high QPS |
| **Both** (replica or nonclustered columnstore): mixed needs | You cannot afford the added write cost and maintenance |

### 9. Common mistakes
- Putting an OLTP table with constant single-row updates on a clustered columnstore → delete bitmaps and delta churn → keep it rowstore or add a nonclustered columnstore.
- Trickle-inserting one row at a time into a columnstore → tiny, poorly compressed row groups → batch loads.
- Using `SELECT *` on a column store "just to look" → defeats column elimination → name columns.
- Loading analytic data unsorted → no segment elimination → sort/partition by the filter column.
- Assuming "columnar" always wins → lookups and updates get slower.

### 10. 🎯 Interview kit
**30-second answer:** A row store keeps a record's fields together on disk, so reading or updating one record is a single contiguous access — ideal for OLTP. A column store keeps each column's values together, so analytics read only the needed columns, compress them heavily and skip chunks via min/max metadata — ideal for OLAP. The cost is expensive single-row reconstruction and updates.

**Structure a longer answer:** clarify workload → draw the two layouts for a 4-column table → walk one lookup and one aggregate through each → compression and skipping → trade-offs (updates, loads) → real scenario.

**Likely follow-ups:**
- *Why is a column store faster even for a full scan of one column?* → Less I/O (one column), better compression, and vectorized processing.
- *How do column stores handle inserts?* → Buffer in a row-based delta store, then compress in bulk.
- *How would you prove a workload should move?* → Measure columns referenced per query, rows scanned, read:write mix, and benchmark representative queries.
- *What does sort order do?* → Tightens min/max ranges and improves RLE, enabling skipping.

**Signals interviewers look for:** you can draw the layout, reason about I/O, mention compression and metadata skipping, and name the update/load trade-off.

### 11. Check yourself
- [ ] Draw both physical layouts for a 5-column, 4-row table.
- [ ] Explain why `SELECT *` by key is slower in a column store.
- [ ] Explain why compression is better in columns than in rows.
- [ ] State how you would test whether a table should be columnar.
- [ ] Describe the role of the delta store and why bulk loads bypass it.
<!-- explainer:end -->

## Practice questions

- [Intermediate] [Explain why row-oriented OLTP storage makes single-row reads cheap but wide-column scans expensive, and how you'd prove a workload would benefit from moving to a columnar OLAP store.](<../../../PRACTICE/02_Data_Engineering/Modeling_and_Warehousing/OLTP_vs_OLAP/Q-35085_Explain_why_row-oriented_OLTP_storage_makes_single-row_reads.md>)
- [Deep] [What is a row-oriented storage format and why does it suit OLTP?](<../../../PRACTICE/02_Data_Engineering/Modeling_and_Warehousing/OLTP_vs_OLAP/Q-34642_What_is_a_row-oriented_storage_format_and_why_does_it_suit_O.md>)
- [Deep] [What is a column-oriented storage format and why does it suit OLAP?](<../../../PRACTICE/02_Data_Engineering/Modeling_and_Warehousing/OLTP_vs_OLAP/Q-34643_What_is_a_column-oriented_storage_format_and_why_does_it_sui.md>)

## Related concepts

- [Databases and Schemas](<../../../01_Databases_and_SQL/Database_Internals/DBMS_Concepts/Databases_and_Schemas.md>) *(DBMS Concepts)*
- [Row vs Columnar Formats](<../../Storage_and_Lakehouse/File_Formats/Row_vs_Columnar_Formats.md>) *(File Formats)*
- [Large Values and TOAST](<../../../01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Large_Values_and_TOAST.md>) *(SQL Fundamentals)*

## References

- *Database Internals* — Alex Petrov — Introduction and Overview › Row-Oriented Data Layout
- *Database Internals* — Alex Petrov — Introduction and Overview › Column-Oriented Data Layout
- *Analytics Optimization with Columnstore Indexes in Microsoft SQL Server* — Edward Pollack — transaCtional vs. analytiC Workloads RANSA TIONAL VS. NALYTI › The Limits of Transactional Data Storage

