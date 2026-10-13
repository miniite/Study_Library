---
id: "TOOLS/SQL_Server/02_Indexing/Nonclustered_Indexes/Covering_and_Included_Columns"
level: 5
type: "curriculum"
curriculum: "sql_server"
---

# Covering Indexes and INCLUDE

**L5 Expert** · [↑ Nonclustered Indexes](<../_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** An index *covers* a query when every column the query needs is in the index (key or INCLUDE), so SQL Server never goes back to the table. INCLUDE columns make covering cheap, and overlapping, over-wide indexes are the price of overdoing it.

### 1. The big picture
The nonclustered-index lesson showed that an NC index's leaf rows hold the index key plus a row locator. When a query needs columns that aren't there, SQL Server performs a **key lookup** (or **RID lookup** on a heap) for *every qualifying row*: a seek into the clustered index, which costs 3–4 logical reads each. For 10 rows that's nothing. For 10,000 rows it's 30,000–40,000 random reads. The optimizer knows this, so above a surprisingly small number of rows (often a fraction of a percent of the table) it gives up on the NC index and scans the whole table instead. The NC index you built becomes useless for that query.

**Covering** fixes this at the root. Put the missing columns into the index, and the leaf contains everything, so there are no lookups and the index stays usable for large ranges. **Included columns** (`INCLUDE (...)`) are the efficient way to do it. They sit only at the leaf, are not sorted, don't count toward the 16-column / 1,700-byte key limit, and don't bloat the upper levels of the B-tree. The design skill is choosing *which* columns go in the key (to seek and sort) and which in INCLUDE (only to cover), while avoiding a sprawl of near-duplicate indexes.

### 2. Core concepts and vocabulary
- **Covering index**: an index containing all columns referenced by a query (SELECT list, WHERE, JOIN, GROUP BY, ORDER BY) for one table.
- **Key lookup**: per-row seek into the clustered index to fetch missing columns. In a plan it is a *Key Lookup (Clustered)* under a *Nested Loops* join.
- **RID lookup**: the same thing on a heap, using the 8-byte file:page:slot locator.
- **Key columns**: the ordered columns of an index (max 16 columns / 1,700 bytes for NC). They are stored at *all* levels and determine seekability and order.
- **Included columns**: non-key columns stored only in leaf rows. Max 1,023, they can be LOB types (`varchar(max)`, `xml`, but not legacy `text/ntext/image`), and they are not ordered.
- **Tipping point**: the row count at which the optimizer prefers a scan over seek+lookups.
- **Overlapping indexes**: two or more indexes whose leading key columns are the same, so one could usually serve both.
- **Index intersection**: the optimizer seeks two different NC indexes and joins their row locators. This is a rare alternative to a single covering index.
- **Seek predicate vs residual predicate**: a predicate used to navigate the B-tree vs one evaluated on each row after it is read.

### 3. How it works, step by step
**Why lookups get expensive:**
1. The query `SELECT OrderDate, Total FROM Orders WHERE CustomerID = @c` seeks `IX_Customer(CustomerID)`.
2. For each matching leaf row, it takes the clustering key and seeks the clustered index (root → intermediate → leaf): about 3 logical reads.
3. With 2,000 matching rows, that is about 6,000 reads of random pages. A clustered scan of a 5,000-page table might be cheaper.
4. The optimizer compares estimated costs and may choose the scan. If the row estimate is wrong (e.g. parameter sniffing), it may choose seek+lookup for a value with 200,000 rows, which is a classic performance disaster.

**Fixing it with covering:**
1. List every column the query touches on that table: `CustomerID` (filter), `OrderDate`, `Total` (output).
2. Columns used for **seeking or ordering** go into the key, in the right order: equality columns first, then range/sort columns.
3. Columns only **returned** (or used in residual filters you don't need to seek on) go into `INCLUDE`.
4. Result: `CREATE INDEX IX_Orders_Customer ON Orders(CustomerID, OrderDate) INCLUDE (Total)`. This seek returns everything, and `ORDER BY OrderDate` comes for free.
5. Recheck the plan: no Key Lookup, a single *Index Seek*, and far fewer logical reads.

**Key vs INCLUDE decision:**

| Column role in query | Put it in |
|---|---|
| Equality predicate (`=`), needed to seek | Key, leading |
| Range predicate (`>`, `BETWEEN`) or ORDER BY / GROUP BY | Key, after the equality columns |
| Only in SELECT list | INCLUDE |
| Large (e.g. `nvarchar(2000)`, LOB) and only returned | INCLUDE (or leave it out and accept a lookup for few rows) |
| Needs uniqueness enforcement | Key of a UNIQUE index (INCLUDE columns don't count) |

**Consolidating overlapping indexes:**
1. Find indexes on the same table whose key columns share a leading prefix, e.g. `IX_A(CustomerID)`, `IX_B(CustomerID, OrderDate)`, `IX_C(CustomerID) INCLUDE (Total)`.
2. One index `(CustomerID, OrderDate) INCLUDE (Total)` serves all three access patterns, because a seek on `CustomerID` alone uses the left prefix.
3. Drop the redundant ones after checking usage stats, constraint dependencies and hints that reference them by name.
4. This reduces write cost, storage, memory, and maintenance time.

### 4. Worked example
```sql
-- Problem query
SET STATISTICS IO ON;
SELECT soh.SalesOrderID, soh.OrderDate, soh.TotalDue
FROM Sales.SalesOrderHeader AS soh
WHERE soh.CustomerID = 29825;
-- Plan: Index Seek (IX_SalesOrderHeader_CustomerID) → Nested Loops → Key Lookup (Clustered)
-- Logical reads: ~ 2 + 3 per row

-- Covering version
CREATE NONCLUSTERED INDEX IX_SOH_Customer_Covering
ON Sales.SalesOrderHeader (CustomerID, OrderDate)
INCLUDE (TotalDue);
-- SalesOrderID is the clustering key → already present in every NC leaf row

SELECT soh.SalesOrderID, soh.OrderDate, soh.TotalDue
FROM Sales.SalesOrderHeader AS soh
WHERE soh.CustomerID = 29825
ORDER BY soh.OrderDate;
-- Plan: Index Seek only; no Sort (key order matches ORDER BY); logical reads ≈ 2–3

-- Find overlapping indexes on a table (same first key column)
SELECT i.name, STRING_AGG(CASE WHEN ic.is_included_column = 0 THEN c.name END, ', ')
         WITHIN GROUP (ORDER BY ic.key_ordinal) AS keys,
       STRING_AGG(CASE WHEN ic.is_included_column = 1 THEN c.name END, ', ') AS includes
FROM sys.indexes i
JOIN sys.index_columns ic ON ic.object_id = i.object_id AND ic.index_id = i.index_id
JOIN sys.columns c ON c.object_id = ic.object_id AND c.column_id = ic.column_id
WHERE i.object_id = OBJECT_ID('Sales.SalesOrderHeader')
GROUP BY i.name;
```
Walkthrough: note that `SalesOrderID`, the clustering key, didn't need to be included. Every NC index on a clustered table already carries the clustering key in its leaf rows (and in its upper levels too, if the NC index is non-unique). Also note the Key Lookup's *Output List* in the original plan. It tells you exactly which columns to INCLUDE. If the lookup also has a *Predicate*, a filter column is missing from the index too.

### 5. Under the hood
- **Where included columns live.** Only in leaf-level rows. Intermediate pages hold key columns (plus the locator for non-unique indexes) only. So INCLUDE columns don't increase tree depth much, but they do make leaf rows wider, giving fewer rows per page, more pages to scan, and more buffer pool space.
- **Covering as a "mini clustered index".** A covering NC index is effectively a narrower copy of the table, sorted differently. Scanning it is cheaper than scanning the clustered index because its rows are narrower. That's why the optimizer sometimes chooses a covering NC *scan*.
- **The cost side.** Every included column must be maintained when it changes. An `UPDATE` of `TotalDue` now updates the clustered index *and* every NC index that includes `TotalDue`. Wide includes also enlarge the log and backups.
- **Lookups and blocking.** Seek+lookup plans take locks on two structures in sequence. A writer updating the clustered row and an NC index in the opposite order can deadlock with them. Covering the reader removes the lookup and often removes that deadlock.
- **Other ways to remove lookups** (from the performance-tuning book): (1) **include** the columns; (2) **convert** the NC index to the clustered index, when the query pattern is dominant and the key is suitable; (3) **index intersection**, where two NC indexes are joined on the locator (rarely better than one covering index); (4) **reduce the columns** the query asks for (stop `SELECT *`).
- **Key size and uniqueness.** Making a column a key column rather than an INCLUDE affects uniqueness: a `UNIQUE` index enforces uniqueness on key columns only. Moving columns into the key "for safety" can make a unique index incorrect or exceed 1,700 bytes.
- **Filters on included columns.** A predicate on an included column can't be a seek predicate. It becomes a residual predicate evaluated per leaf row. This is still cheaper than a lookup, but if the predicate is selective, the column probably belongs in the key.

### 6. Performance and design implications
- Measure with logical reads (`STATISTICS IO`) and plan shape: Key Lookups with high *Actual Executions* are the target.
- Use the missing-index DMVs and the Query Store's top queries to find lookup-heavy queries, but merge their suggestions by hand, because missing-index suggestions never consolidate and often over-include.
- Index count and width matter for writes: for OLTP tables, keep NC indexes few (often ≤ 5–10) and narrow. For read-mostly reporting tables, wider covering indexes are fine.
- Check whether `SELECT *` is really needed. Covering a `SELECT *` query means duplicating the table.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `CREATE INDEX … (keys) INCLUDE (cols)` | Covering index | Remove lookups |
| `CREATE INDEX … WITH (DROP_EXISTING = ON)` | Replace an index definition in one step | Widening/consolidating |
| Plan: Key Lookup *Output List* / *Predicate* | Shows missing columns | Choosing INCLUDE columns |
| `SET STATISTICS IO ON` | Logical reads before/after | Proving the gain |
| `sys.dm_db_missing_index_details` (`included_columns`) | Optimizer's suggestion | Starting point, not final design |
| `sys.dm_db_index_usage_stats` (`user_lookups`) | Lookups per clustered index | Find tables suffering lookups |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| A frequent query does many lookups | The query runs rarely: write cost outweighs the gain |
| INCLUDE for output-only columns | The included column changes very frequently |
| Consolidate overlapping indexes into one | Constraints, hints or plan guides reference an index by name |
| Change the clustered index to the dominant access path | The clustering key would become wide or volatile |

### 9. Common mistakes
- Creating every missing-index suggestion → dozens of overlapping indexes → merge by leading columns, and drop the duplicates.
- Putting output-only columns in the key → wider upper levels, key-size errors, wrong uniqueness → move them to INCLUDE.
- Including big LOB columns "just in case" → huge index, slow writes → include only what the hot query returns.
- Covering a `SELECT *` → a full second copy of the table → change the query, or accept lookups for small row counts.
- Forgetting that the clustering key is already in every NC index → redundant includes → leave it out.

### 10. 🎯 Interview kit
**30-second answer:** "A covering index contains every column a query needs, so there are no key lookups. Each lookup is a separate clustered seek per row, and beyond a small tipping point the optimizer switches to a scan. Seek and sort columns go in the key; output-only columns go in INCLUDE, which lives only at the leaf and avoids key-size limits. The cost is write and storage overhead, so I consolidate overlapping indexes instead of piling them up."
**Structure a longer answer:** lookup mechanics and cost → tipping point → key vs INCLUDE rules → example plan before/after → consolidation and write costs.
**Likely follow-ups:** *Key column vs included column?* → seekable/ordered at all levels vs leaf-only, unordered. *How do you know which columns to include?* → the Key Lookup's Output List and Predicate. *Why might the optimizer ignore your NC index?* → lookups past the tipping point.
**Signals interviewers look for:** quantifying lookup cost, correct key ordering (equality → range), awareness of write costs, and consolidation discipline.

### 11. Check yourself
- [ ] Reproduce a Key Lookup plan and remove it with INCLUDE.
- [ ] Order key columns for `WHERE a = @a AND b > @b ORDER BY b` and explain why.
- [ ] Find and consolidate overlapping indexes on one table.
- [ ] Explain why the clustering key never needs to be included.
- [ ] Estimate the logical reads of 5,000 key lookups vs a 3,000-page scan.
<!-- explainer:end -->

## References

- *Expert Performance Indexing in SQL Server* — Jason Strate, Grant Fritchey — Included Columns
- *SQL Server 2025 Query Performance Tuning (7th ed.)* — Grant Fritchey — FULL_BOOK_NEEDS_MANUAL_SPLIT (cont.) › Chapter 11 Key Lookups And Solutions
- *Expert Performance Indexing in SQL Server* — Jason Strate, Grant Fritchey — Overlapping Indexes
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 8: Query Optimization and Performance Tuning › Covering Indexes
- *Expert Performance Indexing in SQL Server* — Jason Strate, Grant Fritchey — Covering Indexes
- *SQL Server 2025 Query Performance Tuning (7th ed.)* — Grant Fritchey — FULL_BOOK_NEEDS_MANUAL_SPLIT (cont.) › Chapter 10 Index Behaviors
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 8: Query Optimization and Performance Tuning › Understanding Key Lookups
- *Expert Performance Indexing in SQL Server* — Jason Strate, Grant Fritchey — Hash and Range Indexes › Included Columns
- *Beginning T-SQL* — Kathi Kellenberger, Lee Everest — Advanced WHERE Clauses › Using CONTAINS
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 8: Query Optimization and Performance Tuning › Identifying Expensive Operators

