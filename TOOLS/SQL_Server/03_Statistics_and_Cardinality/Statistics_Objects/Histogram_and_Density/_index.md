---
id: "TOOLS/SQL_Server/03_Statistics_and_Cardinality/Statistics_Objects/Histogram_and_Density"
level: 5
type: "curriculum"
curriculum: "sql_server"
---

# Histogram and Density Vector

**L5 Expert** · [↑ Statistics Objects](<../_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** When the optimizer knows the value, it reads the histogram (EQ_ROWS for a step boundary, AVG_RANGE_ROWS inside a step, summed steps for ranges); when it doesn't know the value, it multiplies table rows by the density (1/distinct values).

### 1. The big picture
The *Statistics Objects* lesson showed what the histogram and density vector contain. This lesson shows exactly how the optimizer turns them into a number. Once you can do the arithmetic yourself, you can explain almost any misestimate: why a value that exists is estimated at 2.27 rows, why a parameter sniffed at compile time produced a plan for 72 rows, why yesterday's dates estimate 1 row, why `WHERE col = @localVariable` always gets the same estimate.

The histogram is a compressed picture of the data. With at most 200 steps for millions of distinct values, most values are *not* step boundaries. For them the optimizer can only assume they are average for their step. That single fact explains most "the stats are up to date but the estimate is still wrong" cases: skewed values hidden inside a step.

### 2. Core concepts and vocabulary
- **Step**: one histogram row. It covers the values greater than the previous step's `RANGE_HI_KEY` and up to and including its own `RANGE_HI_KEY`.
- **RANGE_HI_KEY**: the upper boundary value of the step (a real value from the data).
- **EQ_ROWS**: rows equal to `RANGE_HI_KEY`.
- **RANGE_ROWS**: rows strictly between the previous boundary and this one (excluding both).
- **DISTINCT_RANGE_ROWS**: number of distinct values strictly inside the step.
- **AVG_RANGE_ROWS**: `RANGE_ROWS / DISTINCT_RANGE_ROWS`, the assumed row count for any one value inside the step.
- **All density**: `1 / distinct values` for a column prefix. Estimated rows for an unknown equality value = `rows × density`.
- **Uniformity assumption**: inside a step, every distinct value is assumed to have the same number of rows.
- **Ascending key problem**: new values above the last `RANGE_HI_KEY` (new IDs, today's date) are not in the histogram until the next update.
- **Parameter sniffing**: the optimizer uses the actual parameter value at compile time to read the histogram; the plan is then reused for other values.

### 3. How it works, step by step
**3a. How the histogram is built**
1. SQL Server reads the leading column, either all rows (FULLSCAN) or a sample.
2. It sorts the values and walks them, computing for each candidate boundary how many rows equal it and how many lie in between.
3. It merges adjacent ranges until at most 200 steps remain (plus one extra step for NULL if NULLs exist). The algorithm tries to make *frequent* or *distinctive* values into boundaries, so they get exact `EQ_ROWS`, and to merge regions where values are similar in frequency.
4. With sampling, counts are scaled up from the sample, so `EQ_ROWS` and `RANGE_ROWS` become estimates (they can show fractions).

**3b. Equality with a known value (literal or sniffed parameter)**
Read the example histogram (from `Sales.SalesOrderDetail`, leading column `SalesOrderID`):

| Step | RANGE_HI_KEY | RANGE_ROWS | EQ_ROWS | DISTINCT_RANGE_ROWS | AVG_RANGE_ROWS |
|---|---|---|---|---|---|
| 1 | 43659 | 0 | 12 | 0 | 1 |
| 2 | 43692 | 282 | 28 | 32 | 8.8125 |
| 3 | 43898 | 716 | 28 | 205 | 3.4927 |

- `WHERE SalesOrderID = 43692` → the value **is a boundary** → estimate = `EQ_ROWS` = **28**.
- `WHERE SalesOrderID = 43670` → the value lies **inside step 2** (43660–43691) → estimate = `AVG_RANGE_ROWS` = **8.8125**, whether the real count is 1 or 50.
- `WHERE SalesOrderID = 43659` → step 1 boundary → **12**.

Step 2 says: between 43659 and 43692 (exclusive) there are 282 rows over 32 distinct order IDs, and 28 rows for order 43692 itself.

**3c. Range predicates**
`WHERE SalesOrderID BETWEEN 43660 AND 43898`:
- Fully covered steps contribute `RANGE_ROWS + EQ_ROWS`: step 2 = 282 + 28 = 310; step 3 = 716 + 28 = 744.
- Total ≈ 1,054. If a range ends *inside* a step, the optimizer takes a proportional share of that step's `RANGE_ROWS` (interpolation, assuming values are spread evenly).

**3d. Unknown value: use density**
When the value is not known at compile time:
- `WHERE SalesOrderID = @local` (a local variable, not a parameter),
- `OPTION (OPTIMIZE FOR UNKNOWN)`,
- or parameter sniffing disabled,

the estimate = `table rows × All density of the column`. With 121,317 rows and density 3.178e-05: 121,317 × 0.00003178 ≈ **3.86 rows**, for *every* value. This is the "average" value. It is great when data is uniform and poor when some values are huge.

For **inequalities with unknown values** (`> @local`), the optimizer has no density formula and uses a fixed guess (30 % of the rows for a single `>`/`<`). That is why local variables in range predicates produce strangely constant estimates.

**3e. Multi-column density**
For `WHERE SalesOrderID = @a AND SalesOrderDetailID = @b` with a statistic on both, the optimizer uses the density of the *prefix* `(SalesOrderID, SalesOrderDetailID)`: rows × 8.24e-06 ≈ 1. It does not multiply two independent selectivities, so correlation between the columns is captured automatically. This only works for equality on a left-based prefix.

**3f. GROUP BY / DISTINCT**
Estimated groups for `GROUP BY SalesOrderID` = `1 / density` = about 31,465. Grouping on `(SalesOrderID, SalesOrderDetailID)` uses that prefix's density.

**3g. Values outside the histogram**
If the value is greater than the last `RANGE_HI_KEY` (rows added since the last update):
- The **legacy CE** estimates about **1 row**.
- The **modern CE** (120+) estimates using the density (an average frequency), which is usually closer.
Both can be badly wrong after a large load of new key values. See *Cardinality Estimator* and *Statistics Maintenance*.

### 4. Worked example
```sql
-- 1) Look at the histogram as a rowset
SELECT h.step_number, h.range_high_key, h.range_rows, h.equal_rows,
       h.distinct_range_rows, h.average_range_rows
FROM sys.stats s
CROSS APPLY sys.dm_db_stats_histogram(s.object_id, s.stats_id) h
WHERE s.object_id = OBJECT_ID('Sales.SalesOrderDetail')
  AND s.name = 'PK_SalesOrderDetail_SalesOrderID_SalesOrderDetailID'
ORDER BY h.step_number;

-- 2) Find the step a value falls into (how the optimizer will estimate it)
DECLARE @KeyValue INT = 43670;
WITH h AS (
  SELECT hs.step_number, hs.range_high_key, hs.equal_rows, hs.average_range_rows
  FROM sys.stats s
  CROSS APPLY sys.dm_db_stats_histogram(s.object_id, s.stats_id) hs
  WHERE s.object_id = OBJECT_ID('Sales.SalesOrderDetail') AND s.stats_id = 1)
SELECT TOP (1) range_high_key,
       CASE WHEN CONVERT(INT, range_high_key) = @KeyValue THEN equal_rows
            ELSE average_range_rows END AS estimated_rows
FROM h WHERE CONVERT(INT, range_high_key) >= @KeyValue
ORDER BY step_number;

-- 3) Compare literal, parameter and local variable
SELECT * FROM Sales.SalesOrderDetail WHERE SalesOrderID = 43670;          -- histogram
DECLARE @v INT = 43670;
SELECT * FROM Sales.SalesOrderDetail WHERE SalesOrderID = @v;             -- density
EXEC sp_executesql N'SELECT * FROM Sales.SalesOrderDetail WHERE SalesOrderID = @p',
     N'@p INT', @p = 43670;                                               -- sniffed → histogram
```
**What to look at:** The literal and the `sp_executesql` call show the step estimate (8.8125 if 43670 is inside step 2). The local variable shows `rows × density` (≈ 3.86). The actual rows are the same in all three; only the estimate changes.

**Parameter sensitivity in one picture.** A procedure compiled for value 816 (inside a step, `AVG_RANGE_ROWS` = 2.27) gets a seek + lookup plan. Called later with 53465 (a boundary, `EQ_ROWS` = 72), it reuses the 2.27-row plan. Same histogram, two values, two very different "right" plans. That is the root of bad parameter sniffing, covered in *Parameter Sensitivity*.

### 5. Under the hood
- **Step selection.** The builder uses a variant of the *maxdiff* idea: it keeps boundaries where frequency changes most, so heavy hitters tend to become `RANGE_HI_KEY`s. A heavy value that ends up inside a step is averaged away. Filtered statistics or more targeted stats give it its own step.
- **200-step cap.** The limit is fixed. Large tables with millions of distinct values therefore rely heavily on `AVG_RANGE_ROWS`. Increasing the sample does not add steps; it only makes counts more accurate.
- **NULL step.** If NULLs exist, there is an additional step with `RANGE_HI_KEY = NULL`, so the visible count can be 201.
- **Sampling effects.** With a sample, `DISTINCT_RANGE_ROWS` is extrapolated. Distinct counts from samples are notoriously hard to estimate, so `AVG_RANGE_ROWS` can be off even when total rows are right.
- **String summaries.** For string columns, a separate trie-like summary (the *String Index* flag) estimates `LIKE '%abc%'` patterns. It only covers the first 80 characters of each value.
- **Density vector uses all rows of the prefix**, not the histogram, so it remains useful even for values the histogram does not show.

### 6. Performance and design implications
- **Literal/parameter = histogram; local variable/UNKNOWN = density.** Choose deliberately. `OPTIMIZE FOR UNKNOWN` trades precise estimates for stable plans.
- **Skewed data needs more than one statistic.** A filtered statistic per hot slice, or filtered indexes, gives the heavy values their own histogram.
- **Ascending keys** (identity, dates) go stale at the top end fastest. Update those statistics after big loads.
- **Check with numbers.** Estimate wrong? Find the step, read `EQ_ROWS`/`AVG_RANGE_ROWS`, compare to `COUNT(*)` for the value. If the step is accurate but the value is skewed, the fix is design (filtered stats, query split, recompile), not just a stats update.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `DBCC SHOW_STATISTICS (t, s) WITH HISTOGRAM` | Histogram only | Manual inspection |
| `sys.dm_db_stats_histogram` | Histogram as rows | Scripts, finding a value's step |
| `DBCC SHOW_STATISTICS (t, s) WITH DENSITY_VECTOR` | Prefix densities | Unknown-value and GROUP BY estimates |
| `OPTION (OPTIMIZE FOR (@p = value))` | Use the histogram for a chosen value | Stabilize on a representative value |
| `OPTION (OPTIMIZE FOR UNKNOWN)` | Use density | Stable average plan |
| `OPTION (RECOMPILE)` | Compile with current values every time | Highly skewed, infrequent queries |
| `UPDATE STATISTICS t s WITH FULLSCAN` | Exact counts | Skewed column, sample too small |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| Histogram estimates (literals, sniffed parameters) for precise plans | Skewed values reuse a plan made for another value |
| Density estimates (UNKNOWN) for stable average plans | A few values are huge; average plans fail for them |
| Filtered statistics for heavy slices | Parameterized queries that don't match the filter |
| FULLSCAN for skewed columns | Very large tables during busy hours |

### 9. Common mistakes
- Testing with a local variable and running in production with a parameter → different estimates and plans → test with `sp_executesql` or the real procedure.
- Assuming an up-to-date histogram means exact estimates → values inside steps are averaged → check the step for the value.
- Expecting a bigger sample to add steps → still 200 → use filtered stats.
- Reading `RANGE_ROWS` as including the boundary → it excludes it; the boundary is `EQ_ROWS`.

### 10. 🎯 Interview kit
**30-second answer:** The histogram has up to 200 steps on the first column. If the predicate value equals a step boundary, the estimate is `EQ_ROWS`; if it falls inside a step, it is `AVG_RANGE_ROWS`; ranges sum whole steps plus a share of partial ones. If the value is unknown at compile time, like a local variable, the optimizer uses rows × density, where density is 1/distinct values. Multi-column equality uses the density of the column prefix.

**Structure a longer answer:** clarify → histogram fields → known vs unknown values → worked numbers → skew, ascending keys, sniffing → fixes.

**Likely follow-ups:**
- *Why does a local variable give a different plan from a parameter?* → density vs histogram.
- *Why is the estimate 1 row for today's date?* → beyond the last step (legacy CE); stats need updating.
- *How do you estimate GROUP BY output?* → 1/density of the grouping prefix.

**Signals interviewers look for:** can compute an estimate from a histogram row; distinguishes `EQ_ROWS` from `AVG_RANGE_ROWS`; links density to local variables and `OPTIMIZE FOR UNKNOWN`.

### 11. Check yourself
- [ ] From the 3-step table above, estimate `= 43692`, `= 43700`, and `BETWEEN 43660 AND 43692`.
- [ ] Compute rows × density for a column and verify it against a local-variable plan.
- [ ] Find a value inside a step whose real count differs a lot from `AVG_RANGE_ROWS`.
- [ ] Explain why a 200-step histogram can't represent a skewed million-value column well, and two remedies.
<!-- explainer:end -->

## References

- *SQL Server 2025 Query Performance Tuning (7th ed.)* — Grant Fritchey — FULL_BOOK_NEEDS_MANUAL_SPLIT (cont.) › Chapter 12 Dealing With Index Fragmentation
- *Expert Performance Indexing in SQL Server* — Jason Strate, Grant Fritchey — Density Vector
- *Expert Performance Indexing in SQL Server* — Jason Strate, Grant Fritchey — Histogram
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 8: Query Optimization and Performance Tuning › Statistics

