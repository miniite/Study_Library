---
id: "TOOLS/SQL_Server/06_T-SQL_Programming/Window_Functions/Window_Frames_and_Performance"
level: 5
type: "curriculum"
curriculum: "sql_server"
---

# Window Frames and Performance

**L5 Expert** · [↑ Window Functions](<../_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** The frame (`ROWS` or `RANGE`) decides exactly which rows of the partition an ordered window aggregate sees. The choice changes both the *result* (when order values tie) and the *cost*: the default `RANGE` frame uses an on-disk worktable in tempdb, while `ROWS` frames usually use a fast in-memory one. Batch mode removes most of this cost altogether.

### 1. The big picture
The parent lesson used `SUM(val) OVER (PARTITION BY … ORDER BY …)` for a running total. That query depends on a frame you didn't write: the default `RANGE BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW`. This lesson explains frames precisely, shows the logical difference between `ROWS` and `RANGE`, and then goes inside the execution plan: the *Segment*, *Window Spool* and *Stream Aggregate* operators in row mode, the *Window Aggregate* operator in batch mode, and how to measure what's happening with `STATISTICS IO` and `STATISTICS TIME`.

Window functions are often the fastest way to write analytic queries, especially when they replace self-joins and cursors. But the window *aggregates* that came with SQL Server 2005 (no frame, whole partition) and the default `RANGE` frame can perform worse than older methods. Knowing why lets you get the benefits without the traps.

### 2. Core concepts and vocabulary
- **Frame**: the subset of the partition, relative to the current row, that a framed function uses. Applies to aggregates with `ORDER BY` and to `FIRST_VALUE`/`LAST_VALUE`.
- **ROWS**: frame bounds are counted in *physical rows*: "2 rows before the current row".
- **RANGE**: frame bounds are defined by *values* of the ORDER BY column. `CURRENT ROW` in `RANGE` means "the current row *and all its peers*" (rows with the same ordering value). SQL Server supports only `UNBOUNDED` and `CURRENT ROW` as `RANGE` bounds, not numeric offsets like `RANGE 7 PRECEDING`.
- **Peers / ties**: rows with equal values in the window ORDER BY.
- **Frame bounds**: `UNBOUNDED PRECEDING` (partition start), `n PRECEDING`, `CURRENT ROW`, `n FOLLOWING`, `UNBOUNDED FOLLOWING` (partition end).
- **Default frame**: if a window aggregate has `ORDER BY` but no frame, it is `RANGE BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW`.
- **Segment operator**: marks where each partition (and, for RANGE, each peer group) starts in a sorted stream.
- **Sequence Project**: computes ranking functions in row mode.
- **Window Spool**: stores the rows of the current frame in a worktable so the aggregate can read them. It is either **in memory** (fast, no locks, no I/O) or **on disk** in tempdb.
- **Window Aggregate operator**: the batch-mode operator (SQL 2016+) that computes window functions in one pass, without a spool.
- **Worktable**: an internal temporary table; shows as `Table 'Worktable'` in `STATISTICS IO`.

### 3. How it works, step by step
**Frame syntax:**
```
<function> OVER ( [PARTITION BY …] ORDER BY …
                  { ROWS | RANGE } BETWEEN <start> AND <end> )
-- shortcut: ROWS UNBOUNDED PRECEDING  =  ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
```

**Common frames:**

| Frame | Meaning | Typical use |
|---|---|---|
| `ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW` | Start of partition to here | Running total |
| `ROWS BETWEEN 2 PRECEDING AND CURRENT ROW` | Last 3 rows | 3-period moving average |
| `ROWS BETWEEN 1 PRECEDING AND 1 FOLLOWING` | Neighbors | Smoothing |
| `ROWS BETWEEN CURRENT ROW AND UNBOUNDED FOLLOWING` | Here to end | Remaining balance, `LAST_VALUE` |
| `ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING` | Whole partition | Same as no ORDER BY |

**ROWS vs. RANGE, logically.** They differ only when the ORDER BY has ties. Daily sales with two rows on the same date:

| saledate | amount | ROWS running sum | RANGE running sum |
|---|---|---|---|
| 2026-01-01 | 10 | 10 | 10 |
| 2026-01-02 | 5 | 15 | 35 |
| 2026-01-02 | 20 | 35 | 35 |
| 2026-01-03 | 7 | 42 | 42 |

With `RANGE`, both Jan 2 rows include *all* peers, so both show 35. With `ROWS`, each row stops at itself, and which of the two tied rows comes first is arbitrary. The rule: if the ORDER BY is unique, `ROWS` and `RANGE` give the same numbers, so use `ROWS` for speed. If it is not unique, decide which meaning you want, and with `ROWS` add a tiebreaker so the result is deterministic.

**Row-mode execution of a running total:**
1. Rows arrive sorted by (partition, order) from an index or a Sort.
2. *Segment* flags the first row of each partition (and a second Segment flags peer groups for RANGE).
3. *Window Spool* adds rows to the worktable for the current frame. For a running total it emits, for each row, the frame rows the aggregate needs. The engine optimizes "UNBOUNDED PRECEDING to CURRENT ROW" so it doesn't re-read the whole frame each time: it keeps a fast-track running aggregate.
4. *Stream Aggregate* computes the aggregate per row from the spooled frame.
5. *Compute Scalar* produces the final column.

**Why RANGE is slow:** the in-memory spool is only used when the engine knows the maximum number of rows in the frame in advance. With `ROWS`, that is known. With `RANGE`, the number of peers is unknown, so the spool falls back to an **on-disk worktable in tempdb**, with locking, latching and I/O. In the classic book test on a table of about 31,000 orders, both queries read the base table's 689 pages, but the `RANGE` version also showed about 189,000 logical reads on `Worktable`, while the `ROWS` version showed 0. The plans looked almost identical, with each query at 50% relative cost. Only `STATISTICS IO` revealed the difference.

### 4. Worked example
```sql
SET STATISTICS IO, TIME ON;

-- 1) Default frame (RANGE): on-disk worktable
SELECT CustomerID, SalesOrderID,
       SUM(TotalDue) OVER (PARTITION BY CustomerID ORDER BY SalesOrderID) AS RunningTotal
FROM Sales.SalesOrderHeader;

-- 2) Explicit ROWS: in-memory worktable
SELECT CustomerID, SalesOrderID,
       SUM(TotalDue) OVER (PARTITION BY CustomerID ORDER BY SalesOrderID
                           ROWS UNBOUNDED PRECEDING) AS RunningTotal
FROM Sales.SalesOrderHeader;

-- 3) A POC index removes the Sort
CREATE INDEX IX_SOH_Cust_Order
ON Sales.SalesOrderHeader (CustomerID, SalesOrderID)
INCLUDE (TotalDue);
```
**What to look for** in the Messages tab:
```
-- query 1
Table 'Worktable'. Scan count ~50000, logical reads ~190000 ...
Table 'SalesOrderHeader'. Scan count 1, logical reads 689 ...
-- query 2
Table 'Worktable'. Scan count 0, logical reads 0 ...
Table 'SalesOrderHeader'. Scan count 1, logical reads 689 ...
```
After the index, the *Sort* operator disappears from both plans and CPU time drops further.

**Moving average and comparison with last year:**
```sql
SELECT  YearMonth, Sales,
        AVG(Sales) OVER (ORDER BY YearMonth ROWS BETWEEN 2 PRECEDING AND CURRENT ROW) AS MovingAvg3,
        LAG(Sales, 12) OVER (ORDER BY YearMonth)                                     AS SalesLastYear,
        Sales - LAG(Sales, 12) OVER (ORDER BY YearMonth)                             AS YoYChange
FROM dbo.MonthlySales;
```
`LAG(Sales, 12)` works only if every month is present. Missing months shift the comparison. For gaps, join to a calendar table first.

**Window aggregate without ORDER BY (the 2005 feature) vs. a CTE:**
```sql
-- Window aggregate: subtotal per customer on each row
SELECT CustomerID, SalesOrderID, TotalDue,
       SUM(TotalDue) OVER (PARTITION BY CustomerID) AS CustTotal
FROM Sales.SalesOrderHeader;

-- CTE + join: often fewer worktable reads in row mode
WITH T AS (SELECT CustomerID, SUM(TotalDue) AS CustTotal
           FROM Sales.SalesOrderHeader GROUP BY CustomerID)
SELECT h.CustomerID, h.SalesOrderID, h.TotalDue, T.CustTotal
FROM Sales.SalesOrderHeader AS h
JOIN T ON T.CustomerID = h.CustomerID;
```
In row mode the window version spools each partition into a *Table Spool* and reads it back (the book's test: 689 table reads plus ~139,000 worktable reads, against ~1,378 table reads and no worktable I/O for the CTE). Measure both on your data. In batch mode the window version wins easily.

### 5. Under the hood
- **Two Segments for RANGE.** RANGE needs peer-group boundaries as well as partition boundaries, so the plan has an extra *Segment* operator.
- **In-memory spool limit.** The in-memory worktable is used for ROWS frames whose size is known and up to about 10,000 rows. Larger ROWS frames (`ROWS BETWEEN 20000 PRECEDING AND CURRENT ROW`) also go to disk. Unbounded-preceding running totals are fast-tracked and don't need to hold the whole frame.
- **Extended event `window_spool_ondisk_warning`** fires when a Window Spool spills to disk. Use it to find RANGE frames in a workload.
- **Reusing windows.** Several functions with an identical OVER clause share the Segment/Spool work. Different OVER clauses each add a sort and spool; the book's test showed double the I/O when two aggregates used different windows.
- **Batch mode.** The *Window Aggregate* operator (SQL 2016+ with a columnstore index on the table; SQL 2019+ *batch mode on rowstore* at compatibility level 150) computes frames in memory with no spool, processes ~900 rows per batch, and handles RANGE efficiently. Running totals over millions of rows typically drop from many seconds to well under a second. A common trick on SQL 2016–2017 was a dummy filtered columnstore index (`WHERE id = -1 AND id = -2`) to enable batch mode.
- **Parallelism.** Window functions with `PARTITION BY` can be parallelized by distributing partitions across threads (*Repartition Streams* on the partition columns). Without `PARTITION BY`, the window operator runs serially.
- **Frames with `n FOLLOWING`.** The spool must look ahead, so it buffers more rows before it can emit; cost grows with frame size in row mode.

### 6. Performance and design implications
- Always write the frame explicitly: `ROWS UNBOUNDED PRECEDING` for running totals. This is the single biggest easy win.
- Build POC indexes (partition, order, covering includes) so no Sort is needed.
- Group window functions on identical OVER clauses (or use `WINDOW` in SQL 2022) so they share work.
- Measure with `STATISTICS IO` (worktable reads), `STATISTICS TIME` (run several times and average; warm cache vs. cold cache changes results), and the actual plan (Sort spills, spool type). The estimated relative cost in the plan can hide worktable I/O.
- For large analytic workloads, columnstore and batch mode change the picture completely; test at the target compatibility level.
- Some books measure SSIS/ETL loads where buffer and batch size tuning changed run time a lot. The same principle applies here: measure end-to-end time, not plan cost.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `ROWS UNBOUNDED PRECEDING` | Fast running-total frame | Every running total with a unique order |
| `ROWS BETWEEN n PRECEDING AND CURRENT ROW` | Moving window | Moving averages |
| `RANGE UNBOUNDED PRECEDING` | Includes all peers | Only when tie semantics are wanted |
| `SET STATISTICS IO, TIME ON` | Shows worktable reads and CPU | Comparing frames |
| `window_spool_ondisk_warning` XE | Detects on-disk window spools | Workload audits |
| `ALTER DATABASE … SET COMPATIBILITY_LEVEL = 150` | Enables batch mode on rowstore | SQL 2019+ |
| `OPTION (USE HINT('DISALLOW_BATCH_MODE'))` | Turns batch mode off for a query | Testing / regression check |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| `ROWS` when the order is unique | Order has ties and you need peers summed together → `RANGE` (or group first) |
| `RANGE` for "as of this date, including same-day rows" | Large partitions in row mode → aggregate per date first, then a ROWS running total |
| Window aggregate for subtotals | Row mode, huge partitions → compare with CTE + join |
| Batch mode (columnstore / compat 150) | Small OLTP queries: no benefit, and plans may change |
| `LAG(x, 12)` for year-over-year | Missing periods → calendar table first |

### 9. Common mistakes
- Leaving the frame out → default `RANGE` → tempdb worktable and slow queries → write `ROWS UNBOUNDED PRECEDING`.
- Trusting the plan's relative cost → misses worktable I/O → check `STATISTICS IO`.
- `ROWS` with non-unique order → numbers change between runs → add a tiebreaker.
- Many different OVER clauses in one query → one sort per window → align them or use a POC index per main window.
- Timing a query once → misleading numbers → run several times, compare warm-cache averages.
- `LAST_VALUE` with default frame → returns the current row → `ROWS BETWEEN CURRENT ROW AND UNBOUNDED FOLLOWING`.

### 10. 🎯 Interview kit
**30-second answer:** A frame limits which rows of the partition an ordered window function uses. ROWS counts physical rows; RANGE uses values, so the current row includes all its ties. The default for an ordered aggregate is RANGE UNBOUNDED PRECEDING, which in row mode forces an on-disk worktable in tempdb. Writing ROWS UNBOUNDED PRECEDING gives an in-memory spool and can cut worktable reads from hundreds of thousands to zero. Add a POC index to avoid sorts, and batch mode's Window Aggregate operator removes the spool entirely.

**Structure a longer answer:** frame syntax → ROWS vs. RANGE semantics with a tie example → plan operators (Segment, Window Spool, Stream Aggregate) → why RANGE spills → measurements → batch mode and indexing.

**Likely follow-ups:**
- *When do ROWS and RANGE give different results?* → only with ties in the ORDER BY.
- *How do you see the spool problem?* → `STATISTICS IO` worktable reads; the `window_spool_ondisk_warning` event.
- *How to make it faster still?* → POC index, batch mode (columnstore or compat 150), shared OVER clauses.
- *Does SQL Server support `RANGE INTERVAL '7' DAY PRECEDING`?* → no; use ROWS on a gap-free calendar or a self-join/APPLY.

**Signals interviewers look for:** you know the default frame and its cost; you measure with IO statistics rather than plan cost; you understand ties and determinism.

### 11. Check yourself
- [ ] Build the ROWS vs. RANGE running-total table for data with ties, by hand, then verify it.
- [ ] Run a running total with and without `ROWS` and record worktable logical reads.
- [ ] Create a POC index and confirm the Sort disappears from the plan.
- [ ] Write a 3-month moving average and a year-over-year change with LAG.
- [ ] Enable batch mode (compat 150 or columnstore) and identify the Window Aggregate operator.
<!-- explainer:end -->

## References

- *Expert T-SQL Window Functions in SQL Server* — Kathi Kellenberger, Clayton Groom, Ed Pollack — Summary › Contents
- *Expert T-SQL Window Functions in SQL Server* — Kathi Kellenberger, Clayton Groom, Ed Pollack — Measuring Performance
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 3: SELECT Statements and Filtering Logic › Window Frames
- *Expert T-SQL Window Functions in SQL Server* — Kathi Kellenberger, Clayton Groom, Ed Pollack — Understanding the Performance Implications of Window Aggregates
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 9: Execution Plans and Query Store › Aggregate and Compute Operators
- *Expert T-SQL Window Functions in SQL Server* — Kathi Kellenberger, Clayton Groom, Ed Pollack — Performing Time Comparisons
- *Beginning T-SQL* — Kathi Kellenberger, Lee Everest — Discovering Windowing Functions › The Trouble with Window Aggregates
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 11: Automation with SSIS, SQL Agent, and PowerShell › Performance Considerations
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 7: Dynamic SQL, Triggers, and Table-Valued Functions (TVFs) › Performance Best Practices
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 15: Case Studies and the Future of T-SQL › Performance and Resource Scaling

