---
id: "TOOLS/SQL_Server/02_Indexing/Columnstore/Batch_Mode_Execution"
level: 5
type: "curriculum"
curriculum: "sql_server"
---

# Batch Mode Execution

**L5 Expert** · [↑ Columnstore Indexes](<../_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** Batch mode makes each plan operator process a vector of up to about 900 rows per call, using tight CPU-friendly loops, instead of one row per call. It is the CPU half of columnstore speed, and since SQL Server 2019 it is available on rowstore tables too.

### 1. The big picture
A query plan is a tree of operators (scan, filter, hash join, aggregate…). In classic **row mode** (the *iterator* or *Volcano* model), each operator asks its child for *one row* at a time by calling `GetNext()`. For OLTP queries touching a few rows this is fine. For analytics over 100 million rows, the per-row overhead dominates: function calls, branching, copying row data, checking the row's format, cache misses. The CPU spends more time *moving* rows than *computing* on them.

**Batch mode** changes the contract: operators exchange **batches**, typically up to about 900 rows each, stored column by column in memory. An operator runs one loop over a column vector (e.g. "add these 900 Quantity values"), which modern CPUs execute very efficiently (cache-friendly, few branches, sometimes SIMD instructions). Columnstore reduces *I/O*. Batch mode reduces *CPU*. Together they produce the 10–100× improvements columnstore is known for. Batch mode was introduced with columnstore (2012) and greatly extended in 2014–2017. From 2019 (compatibility level 150) the optimizer can also use it on plain rowstore tables, a feature called **batch mode on rowstore**.

### 2. Core concepts and vocabulary
- **Row mode**: the one-row-per-call iterator execution model.
- **Batch**: an in-memory unit of up to ~900 rows (the size is chosen so a batch fits the CPU cache), stored as column vectors plus a *qualifying rows* bitmap.
- **Batch mode operator**: an operator implementation that consumes and produces batches. Examples: Columnstore Index Scan, Hash Match (join and aggregate), Filter, Compute Scalar, Sort, Window Aggregate (2016+), Adaptive Join (2017+).
- **Mode transition**: a point in the plan where batch-mode output is converted to rows, or vice versa. Transitions cost CPU, and a row-mode operator in the middle of a plan "breaks" the batch pipeline.
- **Aggregate pushdown**: computing `SUM/COUNT/MIN/MAX` inside the columnstore scan, sometimes directly on compressed/encoded data, so rows never leave the scan.
- **String predicate pushdown**: evaluating string filters against the dictionary once, rather than on every row.
- **Batch mode on rowstore**: (2019+, compatibility level 150+) batch execution on B-tree/heap tables when the optimizer's heuristics judge it worthwhile (large tables, expensive aggregates/joins).
- **Adaptive join**: a batch-mode join that chooses between a hash join and a nested loops join at runtime, based on the actual row count.
- **Memory grant feedback**: adjusts a query's memory grant on later executions, based on the previous actual use. It started in batch mode in 2017 and was extended to row mode in 2019.

### 3. How it works, step by step
1. **Plan choice.** During optimization, the optimizer considers batch-mode operator variants when a columnstore index is involved, or (on 2019+) when its batch-on-rowstore heuristics fire. The plan shows `Estimated Execution Mode = Batch` per operator.
2. **Scan produces batches.** The columnstore scan decompresses segments directly into column vectors of about 900 values. It applies pushed-down predicates by marking a *qualifying rows* bitmap instead of copying survivors.
3. **Operators loop over vectors.** A Filter evaluates `Quantity > 10` over the whole vector in one tight loop and updates the bitmap. A Compute Scalar fills an output vector. No per-row function calls happen.
4. **Hash operators work in batches.** A batch-mode hash join builds its hash table and probes it with whole batches. A hash aggregate updates group accumulators for a whole batch. These operators also spill to tempdb in batches when memory runs out.
5. **Parallelism.** Batch-mode operators share work across threads differently than row mode. Most batch plans have fewer Exchange (Parallelism) operators, because threads cooperate on shared hash tables.
6. **Transition out.** At the top, or before a row-only operator, batches are converted to rows. You see an operator in Row mode above Batch operators.
7. **Runtime adaptivity (2017+).** An adaptive join reads its build input. If the row count is below the adaptive threshold, it switches to nested loops. Otherwise it continues as a hash join.

| Aspect | Row mode | Batch mode |
|---|---|---|
| Rows per operator call | 1 | up to ~900 |
| Data layout in flight | row structs | column vectors + bitmap |
| CPU efficiency | high overhead per row | amortized, cache- and SIMD-friendly |
| Best for | OLTP, small result sets, seeks | scans, joins, aggregates over large data |
| Requirements | none | columnstore, or 2019+ batch on rowstore (compat 150) |

### 4. Worked example
```sql
-- Same aggregate in row mode and batch mode
SET STATISTICS TIME, IO ON;

-- Force row mode for comparison (2016+ hint)
SELECT [Customer Key], SUM(Quantity) AS qty
FROM Fact.Sale_CCI
GROUP BY [Customer Key]
OPTION (USE HINT('DISALLOW_BATCH_MODE'));

-- Default: batch mode
SELECT [Customer Key], SUM(Quantity) AS qty
FROM Fact.Sale_CCI
GROUP BY [Customer Key];

-- Batch mode on rowstore (2019+, compat 150+): table has only a B-tree
ALTER DATABASE CURRENT SET COMPATIBILITY_LEVEL = 160;
SELECT [Customer Key], SUM(Quantity) AS qty
FROM Fact.Sale            -- rowstore
GROUP BY [Customer Key];
```
What to look at in the actual plans:
- Each operator's properties: **Actual Execution Mode** = `Batch` or `Row`, and **Actual Number of Batches**.
- The CPU time in `STATISTICS TIME`: the batch version usually uses a fraction of the row version's CPU for the same reads.
- On the columnstore scan: **Actual Number of Locally Aggregated Rows**. If this is large, aggregate pushdown handled those rows inside the scan.
- In the rowstore query: a regular Clustered Index Scan, but with execution mode `Batch` and a Hash Match in batch mode. That is batch mode on rowstore. It gets no compression or elimination, only the CPU benefit.

### 5. Under the hood
- **Why ~900 rows?** A batch's column vectors should fit in the CPU's L2 cache, so a loop over them doesn't stall on main memory. The exact count depends on column widths.
- **Qualifying-rows bitmap.** Filters don't compact batches. They flip bits, and later operators skip unqualified positions. This avoids copying.
- **Operating on encoded data.** Some predicates and aggregates run on the encoded values (dictionary IDs, value-encoded integers) without full decompression. A string filter like `= 'Red'` is resolved once against the dictionary to an ID, and then integers are compared.
- **What blocks batch mode.** Historically: some data types (LOBs, XML, sql_variant, CLR, older types), cursors, certain functions, `OUTPUT` clauses, scalar UDTs, row-mode-only operators (e.g. some forms of Merge Join, Stream Aggregate, Nested Loops in older versions). Each release shrinks this list. When one operator can't run in batch mode, everything above it usually runs in row mode.
- **Batch mode on rowstore heuristics.** The optimizer considers it only if the query touches large tables (roughly 128K+ rows), uses operators that benefit (joins, aggregates, window functions), and the estimated cost is high. Disable it with `ALTER DATABASE SCOPED CONFIGURATION SET BATCH_MODE_ON_ROWSTORE = OFF`, or the hint `DISALLOW_BATCH_MODE`. Force it for testing with `ALLOW_BATCH_MODE`.
- **Compatibility level.** Many batch features are gated by compatibility level: adaptive joins and batch memory grant feedback at 140, batch mode on rowstore at 150. A slow analytic query on a new server sometimes just needs the right level. The book advises checking compatibility level first when columnstore performance is unexpected.
- **Pre-2016 trick.** Older versions only used batch mode in plans that touched a columnstore, so people created an empty filtered NCCI (e.g. `WHERE id = -1 AND id = -2`) to "unlock" batch mode. On 2019+ this hack is unnecessary.

### 6. Performance and design implications
- Batch mode helps when there are many rows per operator. It does not help single-row lookups.
- Watch for **mode transitions** in the middle of plans: one row-mode operator above the scan can throw away most of the benefit. Rewrite the query so the expensive part (join and aggregate) stays in batch mode.
- **Batch on rowstore vs NCCI**: for a mixed OLTP/analytic table, batch mode on rowstore may give "good enough" analytic speed without the write cost of a nonclustered columnstore. But it lacks columnstore compression, elimination and metadata. Test both.
- Batch-mode hash operators need memory grants. Watch spills (warnings on the operator) and grant feedback.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| Plan property *Actual Execution Mode* | Shows Batch/Row per operator | Every analytic plan review |
| `OPTION (USE HINT('DISALLOW_BATCH_MODE'))` | Forces row mode | A/B testing, regressions |
| `OPTION (USE HINT('ALLOW_BATCH_MODE'))` | Allows batch mode where heuristics would skip it | Testing batch on rowstore |
| `ALTER DATABASE SCOPED CONFIGURATION SET BATCH_MODE_ON_ROWSTORE = OFF` | Disables batch on rowstore DB-wide | Regressions after upgrade |
| `BATCH_MODE_ADAPTIVE_JOINS = OFF` / `BATCH_MODE_MEMORY_GRANT_FEEDBACK = OFF` | Disable those features | Troubleshooting |
| `COMPATIBILITY_LEVEL = 150/160/170` | Enables newer batch features | After upgrades |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| Large scans/joins/aggregates: let batch mode run | OLTP point queries: batch overhead is wasted |
| Batch on rowstore for occasional analytics on OLTP tables | Constant heavy analytics: a columnstore gives far more |
| Raise compatibility level to get adaptive features | Without plan regression testing (use Query Store) |

### 9. Common mistakes
- Assuming the columnstore scan alone gives the speed → a row-mode aggregate above it wastes CPU → check each operator's execution mode.
- Forgetting compatibility level after migrating a database → old level, no batch on rowstore → set the level and test with Query Store.
- Using a scalar UDF or unsupported type in an analytic query → the plan falls back to row mode → inline the logic (or rely on 2019+ UDF inlining) and remove the unsupported type.
- Comparing elapsed time only → batch plans are often parallel → compare CPU time and reads as well.

### 10. 🎯 Interview kit
**30-second answer:** "Row mode passes one row per operator call. Batch mode passes about 900 rows at a time as column vectors, so operators run tight loops with far less overhead per row. It's what makes columnstore queries CPU-efficient. It supports aggregate and predicate pushdown and adaptive joins, and since 2019 it can run on rowstore tables too, at compatibility level 150."
**Structure a longer answer:** iterator model and its overhead → batches and column vectors → operators and pushdown → where it applies (columnstore, batch on rowstore) → what blocks it → how to verify in the plan.
**Likely follow-ups:** *Does batch mode need columnstore?* → not since 2019 (compat 150). *What is an adaptive join?* → it picks hash vs nested loops at runtime from a row threshold. *How do you see it?* → Actual Execution Mode and Actual Number of Batches.
**Signals interviewers look for:** understanding the CPU (not I/O) nature of the gain, mode transitions, compatibility-level gating, and knowing that batch on rowstore is not the same as columnstore.

### 11. Check yourself
- [ ] Explain why per-row overhead matters at 100M rows but not at 10 rows.
- [ ] Find the execution mode of every operator in an actual plan.
- [ ] Demonstrate aggregate pushdown (locally aggregated rows) in a plan.
- [ ] Turn batch mode on rowstore on and off and compare CPU time.
- [ ] List three things that can force a plan back into row mode.
<!-- explainer:end -->

## References

- *Analytics Optimization with Columnstore Indexes in Microsoft SQL Server* — Edward Pollack — BatCh exeCution H XE UTION › Batch Mode vs. Row Mode Performance
- *Analytics Optimization with Columnstore Indexes in Microsoft SQL Server* — Edward Pollack — Columnstore metadata › CHAPTER 7
- *Analytics Optimization with Columnstore Indexes in Microsoft SQL Server* — Edward Pollack — BatCh exeCution H XE UTION › How Does Batch Mode Work?
- *Analytics Optimization with Columnstore Indexes in Microsoft SQL Server* — Edward Pollack — BatCh exeCution H XE UTION › Batch Mode Execution
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 1: Getting Started with T-SQL › Batch Processing and the GO Command
- *T-SQL Fundamentals (4th ed.)* — Itzik Ben-Gan — Programmable objects
- *Beginning T-SQL* — Kathi Kellenberger, Lee Everest — WHILE
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 14: Unit Testing T-SQL with tSQLt › Running Multiple Tests and Test Suites
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 11: Automation with SSIS, SQL Agent, and PowerShell › Control Flow Tasks

