---
id: "TOOLS/SQL_Server/04_Query_Processing/Execution_Plans/Seeks_Scans_and_Lookups"
level: 5
type: "curriculum"
curriculum: "sql_server"
---

# Seeks Scans and Lookups

**L5 Expert** · [↑ Reading Execution Plans](<../_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** A **scan** reads an index from one end to the other, a **seek** navigates the B-tree to a starting key and reads only the matching range, and a **lookup** goes back to the base table for columns the chosen index doesn't have. Whether each is cheap depends on how many rows it touches, not on its name.

### 1. The big picture
Every query plan ends in *data access operators*: the leaves of the plan tree that actually read pages. They decide most of a query's logical reads, so reading them correctly is the first skill of plan analysis.

SQL Server stores tables either as a **heap** (no clustered index; rows in no particular order) or as a **clustered index** (a B-tree ordered by the clustering key, whose leaf level *is* the table). Nonclustered indexes are separate B-trees holding their key columns, any `INCLUDE` columns, and a **row locator** that points back to the base row (the clustering key, or a RID for heaps). Access operators are combinations of "which structure" × "how it is read":

| Structure | Read whole / large part | Navigate to a range | Fetch missing columns per row |
|---|---|---|---|
| Heap | **Table Scan** | (none) | **RID Lookup** |
| Clustered index | **Clustered Index Scan** | **Clustered Index Seek** | (none; it has every column) |
| Nonclustered index | **Index Scan** | **Index Seek** | → Key Lookup / RID Lookup |
| Columnstore | **Columnstore Index Scan** (with segment elimination) | (none; rowgroup elimination instead) | (none) |

The common belief "seek good, scan bad" is wrong in both directions. A seek that returns 90% of the table is worse than a scan, and a scan of a 3-page lookup table is fine. This lesson builds on *Index Architecture* and *Covering Indexes*, and feeds into *Join Operators* (a lookup *is* a nested-loops join).

### 2. Core concepts and vocabulary
- **Scan**: reads all leaf pages of a structure (optionally stopping early, e.g. under `TOP`). Cost ≈ number of pages.
- **Seek**: uses the B-tree's root and intermediate levels to jump to the first qualifying key, then reads forward (a **range scan**) until keys stop qualifying. Cost ≈ tree depth + pages in the range.
- **Seek Predicate**: the part of the WHERE/JOIN condition used to navigate (needs the index's *leading* key column(s) and a SARGable condition).
- **Predicate (residual)**: a condition checked on each row *after* it is read. It filters rows but does not reduce reads.
- **SARGable**: "Search ARGument-able". The predicate has the form `column op expression`, so the optimizer can turn it into a seek range. Wrapping the column in a function or implicit conversion usually breaks this.
- **Key Lookup**: for each row from a nonclustered index, seek into the clustered index by its key to fetch the missing columns. (Older versions called it *Bookmark Lookup*.)
- **RID Lookup**: the same on a heap, using the physical Row ID (file:page:slot).
- **Covering index**: contains every column the query needs (as keys or `INCLUDE`), so no lookup is needed.
- **Ordered property**: *Ordered = True* means the operator must return rows in index order (e.g. for a Merge Join or ORDER BY). *False* allows an allocation-order scan.
- **Tipping point**: the row count at which the optimizer switches from a seek + lookups to a full scan, because the lookups would cost more.

### 3. How it works, step by step
**Index Seek**
1. Read the root page and binary-search it for the seek key.
2. Follow the child pointer down the intermediate levels (a few pages; a B-tree of millions of rows is typically 3–4 levels deep).
3. Arrive at the leaf page holding the first qualifying key.
4. Read forward along the leaf-level linked list while keys satisfy the seek predicate.
5. Apply any residual predicate to each row, then pass qualifying rows up.

A *singleton* seek (`WHERE pk = 42`) touches only depth-many pages, e.g. 3 logical reads. A *range* seek (`WHERE OrderDate >= '2024-01-01'`) touches depth + range pages.

**Index / Table Scan**
1. Read every leaf page, either in logical key order (following the linked list) or in physical allocation order (using IAM pages, when order isn't required and the data can't change under it).
2. Evaluate the predicate on every row.

**Key Lookup** (always appears under a **Nested Loops** join)
1. The outer input is a nonclustered Index Seek/Scan that returns key values plus the clustering key.
2. For **each** row, the lookup does a singleton seek into the clustered index (depth-many reads, e.g. 3) and returns the missing columns (the *Output List*). Sometimes it applies a residual predicate there too.
3. Cost = rows × depth. 10 rows → ~30 reads. 50,000 rows → ~150,000 random reads, often more than scanning the whole table.

That last line is the **tipping point**. If a table has 1,000 pages and each lookup costs 3 reads, the lookup plan stops being cheaper at roughly a few hundred rows, a tiny fraction of the table. The optimizer computes this from *estimated* rows. That's why a bad estimate can produce a plan doing millions of lookups.

### 4. Worked example
```sql
-- AdventureWorks; Ctrl+M for actual plan
SET STATISTICS IO ON;

-- 1) Clustered Index Seek: singleton on the PK
SELECT * FROM Sales.SalesOrderHeader WHERE SalesOrderID = 43659;
--   → Clustered Index Seek, Seek Predicate SalesOrderID = 43659, ~3 logical reads

-- 2) Index Seek + Key Lookup: selective filter, but the index lacks TotalDue
SELECT SalesOrderID, CustomerID, TotalDue
FROM Sales.SalesOrderHeader WHERE CustomerID = 11000;
--   → Index Seek (IX_SalesOrderHeader_CustomerID) → Nested Loops → Key Lookup (Clustered)
--     Key Lookup: Number of Executions = 3, Output List = TotalDue

-- 3) Remove the lookup by covering the query
CREATE INDEX IX_SOH_CustomerID_cover ON Sales.SalesOrderHeader (CustomerID)
INCLUDE (TotalDue);
-- re-run 2) → a single Index Seek, fewer reads

-- 4) Non-SARGable predicate turns a seek into a scan
SELECT SalesOrderID FROM Sales.SalesOrderHeader
WHERE YEAR(OrderDate) = 2013;                         -- function on column → scan
SELECT SalesOrderID FROM Sales.SalesOrderHeader
WHERE OrderDate >= '20130101' AND OrderDate < '20140101';  -- range → seek (if indexed)

-- 5) Seek vs residual predicate on a composite index (a, b)
--   WHERE a = 5 AND b > 10   → Seek Predicate: a = 5 AND b > 10   (both used)
--   WHERE b = 10             → leading column missing → Index Scan with Predicate b = 10
```
How to read them: hover over each access operator and compare **Seek Predicates** with **Predicate**. Compare **Estimated** with **Actual Number of Rows**, check **Number of Executions** (for the lookup), and look at the **Output List**. In `STATISTICS IO`, logical reads tell the real cost. A "seek" with 50,000 reads is a disguised scan.

Clean up: `DROP INDEX IX_SOH_CustomerID_cover ON Sales.SalesOrderHeader;`

### 5. Under the hood
- **Seek ranges.** Each seek is one or more *ranges* `[start, end]` on the key. `IN (1,5,9)` becomes three ranges in one Seek operator. `LIKE 'abc%'` becomes a range `>= 'abc' AND < 'abd'`. `LIKE '%abc'` can't seek.
- **Allocation-order vs index-order scans.** An unordered scan of a heap or clustered index at certain isolation levels (or with `TABLOCK`/`NOLOCK`) can follow the IAM chain. That's faster for fragmented indexes, but under `NOLOCK` it can return rows twice or skip rows during page splits.
- **Read-ahead.** Scans and range seeks issue large asynchronous read-ahead I/Os (up to 512 KB), so they are sequential and efficient. Lookups are random single-page reads. That asymmetry is why the tipping point is low on spinning disks and somewhat higher on SSDs, but it is still real because of CPU and latch cost per page.
- **Lookups and the clustering key.** A nonclustered index row carries the clustering key as its locator. A wide clustering key makes every nonclustered index wider and every lookup more expensive. On a heap, the RID points straight to the page, but **forwarded records** (rows moved after growing) add an extra read per lookup (visible as *forwarded fetches* in `sys.dm_db_index_operational_stats`).
- **Implicit conversions.** Comparing an `nvarchar` parameter with a `varchar` column converts the *column* (because of data-type precedence). Depending on collation, that gives a scan, or a `GetRangeThroughConvert` seek that is less efficient. The plan shows a `CONVERT_IMPLICIT` warning.
- **Scan with `TOP`.** A scan can stop after *n* rows, so "Clustered Index Scan, 1 row read" is possible. Read the actual rows, not just the operator name.
- **Columnstore scans** read only the needed column segments and skip rowgroups whose min/max metadata excludes the predicate (*segment elimination*). Check *Actual Segment Reads / Skips*.

### 6. Performance and design implications
- Judge access by **logical reads and rows read**, not by operator name. Newer versions show *Actual Rows Read* vs *Actual Number of Rows*. A big gap means a residual predicate is reading far more than it returns, and a better index key would help.
- **Key Lookups with many executions** are a top tuning target. Cover with `INCLUDE`, or reconsider which columns the query really needs (`SELECT *` forces lookups).
- **Leading key column matters.** Put equality-searched columns first in composite indexes, then range columns.
- **Keep predicates SARGable.** No functions on columns, matching data types, no leading wildcards.
- **Wide or non-unique clustering keys** inflate every nonclustered index. Prefer narrow, unique, ever-increasing clustering keys.
- **Measure with** `SET STATISTICS IO`, actual plans, `sys.dm_db_index_usage_stats` (`user_seeks`, `user_scans`, `user_lookups`), and `sys.dm_db_index_operational_stats`.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `SET STATISTICS IO ON` | logical/physical/read-ahead reads per table | always, when comparing access paths |
| `CREATE INDEX ... INCLUDE (...)` | adds non-key columns to the leaf to cover queries | eliminating Key Lookups |
| `sys.dm_db_index_usage_stats` | seeks/scans/lookups per index since restart | find indexes driving lookups or unused ones |
| `sys.dm_db_index_operational_stats` | leaf-level operations, forwarded fetches, lock/latch waits | heap and hotspot analysis |
| `WITH (FORCESEEK)` / `WITH (FORCESCAN)` table hints | force access method | testing only |
| `WITH (INDEX(name))` | forces a specific index | testing only |
| Plan properties: *Seek Predicates*, *Predicate*, *Actual Rows Read*, *Ordered*, *Output List* | show how much was really read and why | every plan review |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| **Seek + lookup**: very selective query, few rows, OLTP point reads | estimates may be wrong; row counts vary per parameter (tipping point flips) |
| **Covering index**: frequent, important query with a lookup | covering everything → wide, duplicate indexes slow writes |
| **Scan**: large fraction of rows needed, small tables, analytics | a scan runs thousands of times inside a loop |
| **Heap**: staging tables, bulk load then truncate | updated tables (forwarded records), frequent point lookups |
| **Columnstore scan**: aggregates over millions of rows | singleton lookups, heavy row-by-row updates |

### 9. Common mistakes
- "Seeks are always better than scans" → forcing seeks with hints → millions of random reads → judge by reads, fix the estimate.
- `SELECT *` on a query served by a narrow index → a Key Lookup for every row → select only the needed columns or cover.
- `WHERE CONVERT(date, OrderDate) = @d` or `WHERE LEFT(code, 3) = 'ABC'` → scan → rewrite as a range or a `LIKE 'ABC%'` prefix.
- An `nvarchar` parameter against a `varchar` column → implicit conversion → match the parameter types in application code.
- Composite index with the range column first → only part of the key is seekable → equality columns first.
- Ignoring *Actual Rows Read* → a "seek" that reads 1M rows to return 10 → reorder keys or add a filtered index.

### 10. 🎯 Interview kit
**30-second answer:** A scan reads every leaf page of a heap or index. A seek walks the B-tree from the root to the first qualifying key and reads only the matching range, which needs a SARGable predicate on the leading key columns. When a nonclustered index doesn't contain all the columns the query needs, SQL Server does a Key Lookup (or a RID Lookup on a heap) per row into the base table. It's a nested-loops join of random reads, so it's great for a handful of rows and terrible for thousands. I judge access by logical reads and rows read versus returned. I fix lookups with covering `INCLUDE` columns, and turn scans into seeks by making predicates SARGable and getting key order right.
**Structure a longer answer:** storage structures (heap, clustered, nonclustered) → each operator's mechanics and cost → seek vs residual predicate → lookups and the tipping point → how to verify (STATISTICS IO, Actual Rows Read, executions) → fixes → scenario.
**Likely follow-ups:**
- *Is a Clustered Index Scan the same as a Table Scan?* → logically yes (it reads the whole table), but it scans a B-tree's leaf level rather than a heap.
- *Why did the optimizer choose a scan when an index exists?* → too many rows (tipping point), non-SARGable predicate, wrong leading column, or a stale estimate.
- *What is the difference between Seek Predicate and Predicate?* → navigation (reduces reads) vs row filter after reading (doesn't).
- *How do you remove a Key Lookup?* → `INCLUDE` the output columns, or reduce the select list.
**Signals interviewers look for:** you reject "seek always good," you reason in pages and executions, you know SARGability rules and the cost of a wide clustering key.

### 11. Check yourself
- [ ] Draw a 3-level B-tree and count the reads for a singleton seek, a range seek, and 1,000 key lookups.
- [ ] Produce an Index Seek + Key Lookup plan, then remove the lookup with an `INCLUDE` index.
- [ ] Rewrite `WHERE YEAR(OrderDate) = 2013` so it can seek.
- [ ] Explain the tipping point and why a bad estimate can push past it.
- [ ] In a plan, find a residual predicate and compare *Actual Rows Read* to rows returned.
<!-- explainer:end -->

## References

- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 8: Query Optimization and Performance Tuning › Understanding Scan vs. Seek
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 9: Execution Plans and Query Store › Data Access Operators
- *Expert Performance Indexing in SQL Server* — Jason Strate, Grant Fritchey — Lock Wait Time
- *Beginning T-SQL* — Kathi Kellenberger, Lee Everest — Advanced WHERE Clauses › Using FREETEXTTABLE

