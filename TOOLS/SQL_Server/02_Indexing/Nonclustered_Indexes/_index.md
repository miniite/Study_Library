---
id: "TOOLS/SQL_Server/02_Indexing/Nonclustered_Indexes"
level: 4
type: "curriculum"
curriculum: "sql_server"
---

# Nonclustered Indexes

**L4 Advanced** · [↑ Indexing](<../_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** A nonclustered index is a separate B-tree sorted by its own key columns, whose leaf rows carry a row locator back to the table — great for selective seeks, perfect when it *covers* the query, and expensive when it forces one lookup per row.

### 1. The big picture
The clustered index stores the table in one order. Real workloads search by many other columns: customer, status, email, order date. A nonclustered (NC) index is a second, smaller copy of selected columns, sorted differently — like a library card catalog that lists books by author and tells you the shelf number, without moving the books.

A table can have up to 999 NC indexes, but each one is maintained on every insert and delete, and on updates of its columns, so the art is having *few, well-shaped* NC indexes. This lesson explains their structure, the row locator and lookups, covering and included columns, and how the optimizer decides whether to use one. Choosing *which* NC indexes to create is the next lessons' topic (design methodology, DMVs).

### 2. Core concepts and vocabulary
- **Index key** — the ordered column list, up to 16 columns / 1,700 bytes. Defines sort order and what the index can seek on.
- **Leading column** — the first key column; a seek needs a predicate on it (or on a left prefix of the key).
- **Included columns** — non-key columns stored only in leaf rows (`INCLUDE (...)`); not sorted, not usable for seeking, but available to cover the query. Up to 1,023 columns, LOB types allowed.
- **Row locator** — how a leaf row points to the full row: the **clustering key** if the table has a clustered index; the 8-byte **RID** (file:page:slot) if it's a heap; on a clustered columnstore table, an 8-byte rowgroup ID + row offset.
- **Key lookup** — fetching the rest of a row by seeking the clustered index with the locator (shown as *Key Lookup (Clustered)* in plans; called a *bookmark lookup* in old versions).
- **RID lookup** — the heap equivalent: direct jump to file:page:slot (plus a hop if the row was forwarded).
- **Covering index** — contains every column the query references (keys + includes + the clustering key, which is always there for free).
- **Index intersection / index join** — the optimizer combines two NC indexes on the same table by joining their row locators.
- **Selectivity** — fraction of rows returned; NC indexes with lookups only pay off for highly selective predicates.
- **Unique NC index** — enforces uniqueness on its key (unique constraints are implemented this way).

### 3. How it works, step by step
**Structure.** An NC index is a B-tree with the same shape as a clustered index: root, intermediate levels and a leaf level linked in key order. Differences:
- Leaf rows contain *key columns + included columns + row locator*, not the full row.
- In a non-unique NC index the row locator is also added to the key at the upper levels, to make each entry unique; in a unique NC index it appears only at the leaf.
- Because rows are narrow, far more rows fit per page than in the clustered index — scanning an NC index is cheaper than scanning the table.

**Using an NC index to answer a query**
1. *Seek* through the index levels to the first matching key (needs a predicate on the leading column(s)).
2. *Range read* along the leaf until the predicate stops matching.
3. For each qualifying row, if columns are missing, perform a *lookup*:
   - clustered table → seek the clustered index with the clustering key (~2–4 logical reads each);
   - heap → RID lookup (1 read, 2 if forwarded).
4. Lookups are executed by a Nested Loops join, one per row, random I/O.

**Why the optimizer may ignore your index.** Cost of seek + lookups grows linearly with rows: 10 rows × 3 reads = 30 reads is trivial; 50,000 rows × 3 = 150,000 reads can exceed scanning the whole clustered index (say 10,000 pages). The crossover — the *tipping point* — usually lies at a small percentage of the table (often well below 5%, depending on row size). Beyond it, the plan switches to a clustered index scan, and the NC index is unused.

**How to remove lookups**
1. **Cover with INCLUDE** — add the columns the query outputs as included columns. Best when they're only needed for output.
2. **Add to the key** — when they are also filtered, joined or sorted on.
3. **Change the clustered index** — if this access path dominates, cluster on it.
4. **Index intersection** — sometimes two narrow NC indexes joined on the locator cover the query (seen as a Merge/Hash join between two index seeks).
5. **Accept the lookup** — for few rows, a lookup is fine; covering everything bloats indexes.

**Row locator behavior on changes**
- Clustered table: when the clustered index page splits, rows move physically but their clustering key doesn't change, so NC indexes are *not* touched. This is a big reason to prefer clustered tables.
- Heap: a row that grows and no longer fits moves to another page; a *forwarding stub* stays behind so NC indexes needn't be updated, but every lookup now costs an extra hop.
- Build order: create the clustered index *before* the NC indexes. Building it afterwards changes every locator from RID to clustering key, so all NC indexes are rebuilt.

**Column order in a composite key.** An index on (LastName, FirstName) is sorted by LastName, then FirstName within each LastName:
- `WHERE LastName = 'Ng'` → seek. `WHERE LastName = 'Ng' AND FirstName = 'Ann'` → seek on both.
- `WHERE FirstName = 'Ann'` → can't seek (no leading column); at best scans the whole index.
- Rule of thumb: equality columns first, then the range or ORDER BY column; output-only columns go in INCLUDE.

### 4. Worked example
```sql
USE AdventureWorks2022;
SET STATISTICS IO ON;

-- Existing index: IX_SalesOrderHeader_CustomerID (CustomerID)
SELECT SalesOrderID, OrderDate, TotalDue
FROM Sales.SalesOrderHeader
WHERE CustomerID = 29825;
-- Plan: Index Seek (IX_..._CustomerID) → Nested Loops → Key Lookup (PK_SalesOrderHeader)
-- STATISTICS IO: ~ 2 reads for the seek + ~3 per row for the lookups (12 rows → ~40 reads)

-- Widen the range so many rows qualify
SELECT SalesOrderID, OrderDate, TotalDue
FROM Sales.SalesOrderHeader
WHERE CustomerID BETWEEN 11000 AND 20000;
-- Plan flips to Clustered Index Scan (~690 reads): lookups would have cost far more

-- Cover it
CREATE NONCLUSTERED INDEX IX_SOH_Customer_Covering
ON Sales.SalesOrderHeader (CustomerID)
INCLUDE (OrderDate, TotalDue);

-- Re-run both: Index Seek only, no lookup. Narrow query: ~2–3 reads;
-- wide range: a fraction of the clustered scan, since NC leaf rows are small.
-- SalesOrderID (clustering key) is in the index automatically.

-- Inspect the definition
SELECT i.name, c.name AS col, ic.key_ordinal, ic.is_included_column
FROM sys.indexes i
JOIN sys.index_columns ic ON ic.object_id = i.object_id AND ic.index_id = i.index_id
JOIN sys.columns c ON c.object_id = ic.object_id AND c.column_id = ic.column_id
WHERE i.name = 'IX_SOH_Customer_Covering';
```
**Reading the plan.** Hover over the Key Lookup: its *Output List* shows exactly which columns were missing (OrderDate, TotalDue) — the list to INCLUDE. *Number of Executions* equals the rows from the seek. In the covered version, the seek's *Output List* contains all the columns, and the lookup operator disappears. Note the important cost lesson from the books too: adding an index can make *other* operations a bit more expensive — e.g. a query that previously read 10 pages may read 19 once an extra index must be maintained or chosen differently — so always measure both before and after.

### 5. Under the hood
- **Leaf row format.** An NC index leaf row is a regular row structure (status bits, fixed columns, null bitmap, variable columns) holding keys, included columns and the locator. On a heap the locator is the 8-byte RID; on a clustered table, the full clustering key (plus the uniquifier if any). Columns that belong to both the NC key and the clustering key are stored only once.
- **Upper levels.** Non-leaf rows contain the key (plus locator when the index is non-unique) and a child page pointer. Included columns never appear above the leaf — that's why INCLUDE makes covering cheap compared with widening the key.
- **Filtered NC indexes** (`WHERE` clause) contain only qualifying rows — smaller, cheaper to maintain; covered in *Special Indexes*.
- **Write path.** Inserting a row inserts one entry into every NC index (each with its own possible page split). Updating a column that's in an NC index's key = delete + insert in that index; updating an included column updates in place. Deletes leave ghost records in each index.
- **Per-index locks and latches.** Each index has its own pages, so concurrent inserts can contend on several hot spots; deadlocks between a reader using an NC index + lookup and a writer updating the clustered index then the NC index are a classic pattern.

### 6. Performance and design implications
- Lookups are the hidden tax: watch *Key Lookup* operators with large Number of Executions in plans and Query Store.
- Each extra NC index adds roughly one extra page write per insert/delete; heavy OLTP tables usually do well with a handful of NC indexes, while read-mostly tables can afford more.
- A wide clustering key makes every NC index wider (fewer rows per page, more reads).
- Include only what the important queries need; "include everything" duplicates the table.
- Measure with `SET STATISTICS IO`, the actual plan (lookups, estimated vs actual rows), `sys.dm_db_index_usage_stats` (`user_seeks`, `user_scans`, `user_lookups`, `user_updates`), and `sys.dm_db_index_operational_stats` (lock/latch waits per index).

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `CREATE NONCLUSTERED INDEX ix ON t (k1, k2) INCLUDE (c1, c2)` | NC index with covering columns | Removing lookups |
| `CREATE UNIQUE NONCLUSTERED INDEX …` / `UNIQUE` constraint | Enforces uniqueness | Alternate keys (email, code) |
| `… WHERE Status = 1` | Filtered NC index | Small hot subsets |
| `WITH (DROP_EXISTING = ON, ONLINE = ON)` | Redefine in place | Adding includes to an existing index |
| `ALTER INDEX ix ON t DISABLE` / `REBUILD` | Disable keeps metadata, drops data; rebuild re-enables | Testing impact of removing an index; bulk loads |
| `sys.index_columns.is_included_column`, `key_ordinal` | Index definition | Auditing indexes |
| `sys.dm_db_index_usage_stats.user_lookups` | Lookups served by the clustered index | Finding lookup-heavy tables |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| Key columns = predicates and joins, INCLUDE = output columns | Putting output-only columns into the key (wider upper levels, more maintenance) |
| Covering index for a frequent, important query | Many near-duplicate covering indexes for every report |
| Accepting lookups for a few rows | Lookups executing thousands of times per call |
| Composite index with equality columns first | Leading column rarely used in predicates |
| Consolidating overlapping indexes ((A) and (A,B) → (A,B)) | Dropping an index whose uniqueness or FK support is needed |

### 9. Common mistakes
- One single-column index per column → the optimizer can use at most one or two, lookups everywhere → design composite, query-shaped indexes.
- Wrong key order (`(OrderDate, CustomerID)` for `WHERE CustomerID = @c`) → index scan instead of seek → lead with the equality column.
- Assuming "the index exists so it's used" → tipping point makes the optimizer scan → cover the query or reduce rows.
- Including LOB/huge columns casually → index nearly as big as the table → include only needed columns.
- Building NC indexes on a heap and then adding a clustered index → all NC indexes rebuilt → create the clustered index first.
- Forgetting the clustering key is already in every NC index → adding it again to INCLUDE does nothing (it's harmless but noise).

### 10. 🎯 Interview kit
**30-second answer:** "A nonclustered index is a separate B-tree on chosen key columns; its leaf rows hold the keys, any included columns and a row locator — the clustering key, or a RID on a heap. If the query needs columns not in the index, SQL Server does a key lookup per row, which is only cheap for few rows; past the tipping point the optimizer scans instead. To fix that I make the index covering with INCLUDE, keeping equality columns first in the key. Each NC index costs write and maintenance overhead, so I keep them few and query-shaped."

**Structure a longer answer:** structure → row locator → lookups and tipping point → covering with INCLUDE vs key → column order → cost on writes → scenario: a report with 40k key lookups fixed by INCLUDE, reads from 120k to 400.

**Likely follow-ups:**
- *Key column vs included column?* → keys are sorted and seekable and stored at all levels; includes only at the leaf, not seekable.
- *What's the row locator on a heap vs a clustered table?* → RID vs clustering key; why clustered tables don't update NC indexes on page splits.
- *Why might adding an index slow the system?* → write overhead, bigger log, locks, plan changes.
- *Max columns/size?* → 16 key columns, 1,700 bytes (NC), 1,023 included columns.

**Signals interviewers look for:** explains lookups in terms of reads, knows INCLUDE vs key, understands column order and leading-column rule, balances read gains against write costs.

### 11. Check yourself
- [ ] Draw an NC index leaf row for a clustered table and for a heap.
- [ ] Find a key lookup in a plan, read its Output List and build the covering index.
- [ ] Demonstrate the tipping point by widening a range until the plan flips to a scan.
- [ ] Explain why a clustered-index page split doesn't update NC indexes, but a heap row move needs a forwarding pointer.
- [ ] Given `WHERE A = @a AND B > @b ORDER BY B`, choose key order and includes.
<!-- explainer:end -->

## In this section (study in order)

1. [Covering Indexes and INCLUDE](<Covering_and_Included_Columns/_index.md>) (L5 Expert)
2. [Filtered Indexes](<Filtered_Indexes/_index.md>) (L5 Expert)

## References

- *SQL Server 2025 Query Performance Tuning (7th ed.)* — Grant Fritchey — FULL_BOOK_NEEDS_MANUAL_SPLIT (cont.) › Chapter 9 Index Architecture
- *Expert Performance Indexing in SQL Server* — Jason Strate, Grant Fritchey — Clustered Indexes › Nonclustered Indexes
- *SQL Server 2025 Query Performance Tuning (7th ed.)* — Grant Fritchey — FULL_BOOK_NEEDS_MANUAL_SPLIT (cont.) › Chapter 11 Key Lookups And Solutions
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 8: Query Optimization and Performance Tuning › Bookmark Lookups vs. RID Lookups
- *Beginning T-SQL* — Kathi Kellenberger, Lee Everest — Writing Simple SELECT Queries › Using BETWEEN with NOT
- *Beginning T-SQL* — Kathi Kellenberger, Lee Everest — Advanced WHERE Clauses › Using NOT with Parentheses
- *Beginning T-SQL* — Kathi Kellenberger, Lee Everest — 13. Implementing Logic in the Database › Adding a Primary Key to a Table

