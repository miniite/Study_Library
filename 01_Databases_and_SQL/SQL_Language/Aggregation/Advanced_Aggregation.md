---
id: "01_Databases_and_SQL/SQL_Language/Aggregation/Advanced_Aggregation"
level: 4
type: "concept"
topic: "01_Databases_and_SQL/SQL_Language/Aggregation"
tools: []
---

# Advanced Aggregation

**L4 Advanced** · [↑ Aggregation](<_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** Advanced aggregation is about shaping *when* and *where* you aggregate (before or after joins, in groups, across windows, incrementally, or approximately) so results are correct and the plan stays cheap.

### 1. The big picture
Basic `GROUP BY` collapses a table. Real reports need more: percentages of total next to each row, pivoted months across columns, the last non-null reading, retention cohorts, running totals, totals that don't double-count across two child tables, and nightly aggregates on a 200-million-row fact table that keep slowing down. The same few ideas answer most of these:

1. **Choose the grain** of every intermediate result, and aggregate to the grain you need *before* joining.
2. **Use conditional aggregation** (`SUM(CASE …)` / `FILTER`) to compute several measures in one pass, including pivots.
3. **Use window aggregates** when each row also needs its group's value.
4. **Know the physical operators** (sort, stream aggregate, hash aggregate) and what makes them spill.
5. **Pre-compute** (summary tables, indexed/materialized views, incremental maintenance) when the same aggregate is read repeatedly; **approximate** when exactness isn't needed.

### 2. Core concepts and vocabulary
- **Grain** — what one row of a result represents (one row per order line, per day-customer, ...).
- **Fan-out (cartesian inflation)** — joining two one-to-many children to one parent multiplies rows, so sums repeat.
- **Pre-aggregation** — aggregating a child to the join key's grain (in a derived table/CTE) before joining.
- **Conditional aggregation** — an aggregate over a `CASE` (or `FILTER`) expression; the basis of pivots.
- **Pivot / unpivot** — turning row values into columns / columns into rows.
- **Window aggregate** — `SUM(x) OVER (PARTITION BY …)`: aggregate result attached to every input row, rows not collapsed.
- **Running / moving aggregate** — window aggregate with an ordered frame.
- **Hash aggregate / stream aggregate** — physical operators; stream needs input sorted by group key, hash builds an in-memory table.
- **Spill** — the operator exceeds its memory grant and writes intermediate data to disk (tempdb / temp files).
- **Partial vs. final aggregate** — local aggregation on each parallel worker/node followed by a combining step.
- **Materialized / indexed view, summary table** — stored pre-computed aggregates.
- **Approximate aggregate** — HyperLogLog, sketches, sampling.
- **Last good value** — carrying the latest non-NULL value forward (a windowing problem, not a `MAX`).

### 3. How it works, step by step
**A. Aggregate before joining (fix fan-out)**
1. Identify the parent key you report at (`customer_id`).
2. For each child table, aggregate to that key in its own subquery (`SELECT customer_id, SUM(amount) …`).
3. Join the *aggregated* children to the parent (one row per key on every side) and compute.

**B. Pivot with conditional aggregation**
1. Group by the row key (`region`).
2. For each target column, `SUM(CASE WHEN month = 'Jan' THEN amount END)`.
3. Use a native `PIVOT` only when its limits suit you (static column list).
4. Unpivot (`CROSS APPLY (VALUES …)` / `UNPIVOT` / `UNION ALL`) when a wide table needs aggregating by the "column".

**C. Mixed grouped and detail output**
1. Need both rows and group total → `SUM(amount) OVER (PARTITION BY dept)`.
2. Need a smaller result → grouped `SUM`.
3. Need both → compute the group in a CTE and join, or use the window form.

**D. Cohort/retention**
1. Assign each user a cohort (signup date truncated to day/week).
2. Compute activity offsets `N = activity_date − signup_date` (once, not per user per N).
3. `COUNT(DISTINCT user_id)` grouped by `(cohort, N)`; divide by cohort size.
Naive queries explode by cross-joining users with a calendar or self-joining activity per N.

**E. Last non-null value**: `MAX`/`MIN` can't express it. Use a running window: for each row find the latest row with a non-NULL value (e.g. `MAX(CASE WHEN data IS NOT NULL THEN id END) OVER (ORDER BY id)` as a group key, then `MAX(data) OVER (PARTITION BY that_key)`), or `LAST_VALUE(data) IGNORE NULLS` where supported.

**F. Incremental maintenance**
1. Keep a summary table keyed by the grain (date, region).
2. For new/changed source rows, compute a delta and `MERGE`/upsert into the summary.
3. Periodically reconcile against a full recompute (control totals).

### 4. Worked example
**Fan-out and the fix**

```sql
-- Wrong: orders × payments multiplies; each order amount repeated per payment
SELECT c.customer_id, SUM(o.amount) AS ordered, SUM(p.amount) AS paid
FROM customer c
JOIN orders   o ON o.customer_id = c.customer_id
JOIN payments p ON p.customer_id = c.customer_id
GROUP BY c.customer_id;

-- Right: aggregate children to the customer grain first
SELECT c.customer_id,
       COALESCE(o.ordered,0) AS ordered,
       COALESCE(p.paid,0)    AS paid
FROM customer c
LEFT JOIN (SELECT customer_id, SUM(amount) AS ordered FROM orders   GROUP BY customer_id) o
       ON o.customer_id = c.customer_id
LEFT JOIN (SELECT customer_id, SUM(amount) AS paid    FROM payments GROUP BY customer_id) p
       ON p.customer_id = c.customer_id;
```
Check: customer with 3 orders and 4 payments → the wrong version shows each amount repeated 4× and 3× respectively.

**Pivot with conditional aggregation**

```sql
SELECT region,
       SUM(CASE WHEN sale_month = 1 THEN amount ELSE 0 END) AS jan,
       SUM(CASE WHEN sale_month = 2 THEN amount ELSE 0 END) AS feb,
       SUM(amount)                                        AS total
FROM sales
GROUP BY region;
```

**Percent of total with a window**

```sql
SELECT region, SUM(amount) AS region_total,
       100.0 * SUM(amount) / SUM(SUM(amount)) OVER () AS pct_of_all
FROM sales
GROUP BY region;
```
Here the window aggregate runs *over the grouped result*: the inner `SUM` is the group aggregate, the outer `SUM(...) OVER ()` is a window over those groups.

**Last good value**

```sql
SELECT id, data,
       MAX(data) OVER (PARTITION BY grp) AS last_good_value
FROM (
    SELECT id, data,
           MAX(CASE WHEN data IS NOT NULL THEN id END)
               OVER (ORDER BY id ROWS UNBOUNDED PRECEDING) AS grp
    FROM thetable) x;
-- NULLs in rows 3,4 get 1 from row 2; the NULL in row 6 gets 6 from row 5
```

**Reading a slow nightly job** (`SUM(amount) … GROUP BY region` on 200M rows): look at the plan. Is time in the scan (I/O: partitions, columnstore, narrower index), the sort (memory grant, spills to tempdb), or the hash aggregate (estimated 10 groups, actual 8M → under-sized memory grant → spill, then the downstream operators are also mis-sized)? Fix with statistics, a covering/columnstore index, pre-aggregation or incremental maintenance.

### 5. Under the hood
- **Operators**: *Sort* orders rows (memory-hungry; spills to tempdb when the grant is insufficient; avoidable via an index providing the order). *Aggregate* computes `SUM/COUNT/AVG`; as *stream aggregate* it needs ordered input and uses almost no memory; as *hash aggregate* it needs memory proportional to the number of groups. *Compute Scalar* evaluates per-row expressions (cheap, but multiplies with row count).
- **Partial/final aggregation**: in parallel plans each worker aggregates its slice (partial aggregate), possibly *before* a join (a "pushed-down" partial aggregate that shrinks the input), then a final aggregate combines partials. This is only valid for decomposable aggregates (`SUM`, `COUNT`, `MIN`, `MAX`; `AVG` as sum+count). `COUNT(DISTINCT)`, median and percentile are not decomposable and are costlier.
- **Cardinality misestimation**: if the optimizer thinks 10 groups but there are 8M, it grants too little memory → spill; picks nested loops or a wrong join order; histograms/density (statistics) drive the estimate for the grouping columns.
- **Why equivalent queries get different plans**: the optimizer explores rewrites (aggregate pushed below/above a join, `EXISTS` vs `JOIN` vs `IN`, derived table vs CTE) and picks the lowest *estimated* cost; different phrasing changes the search space and estimates. Steer with statistics updates, index/covering changes, rewriting (pre-aggregation), and hints only as a last resort.
- **Materialized/indexed views** persist the aggregate and are maintained by the engine on DML; the cost is write overhead and restrictions (deterministic expressions, no `AVG`, must include `COUNT_BIG(*)` in SQL Server).
- **Columnar/lakehouse storage** (columnstore, Parquet/Delta): aggregation queries read only needed columns, and benefit from partitioning/clustering aligned to the group keys; file size and layout matter as much as the SQL.
- **Approximate distinct**: HyperLogLog keeps a fixed-size sketch (KBs) with ~1–2% error and is mergeable across partitions; bitmap aggregation (roaring bitmaps) gives exact distinct counts for integer IDs and fast set operations at the cost of memory.

### 6. Performance and design implications
- **Reduce rows early**: filter and pre-aggregate before joining; project only needed columns.
- **Index for the group key** (stream aggregate, no sort) and make it covering for the measure; filtered/partial indexes for hot subsets (active rows).
- **Watch memory grants and spills**: look for spill warnings in the plan; update statistics; reduce group cardinality or key width.
- **Materialize when read-heavy**: dashboards hit thousands of times per minute should read a summary table or cache keyed by the grain with a cheap invalidation rule (partition/date based).
- **Incremental over full recompute**: maintain daily sums by merging new data; reconcile periodically.
- **Approximate when exactness isn't contractual**; always state the error.
- **Consistency strategy** for summaries: transactional update, nightly rebuild, or event-driven; decide which staleness is acceptable.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `SUM(CASE …)`, `FILTER (WHERE …)` | Conditional aggregation | Pivots, multi-metric rows |
| `GROUP BY ROLLUP / CUBE / GROUPING SETS` | Subtotals and grand totals in one pass | Reports with hierarchies |
| `SUM() OVER (PARTITION BY …)` | Window aggregate | Percent of total, running totals |
| `PIVOT` / `UNPIVOT` | Native pivoting | Static column sets |
| `APPROX_COUNT_DISTINCT`, HLL | Approximate distinct | Very large data |
| `MERGE` / `INSERT … ON CONFLICT` | Upsert into summary tables | Incremental maintenance |
| Indexed/materialized view | Engine-maintained aggregate | Stable, frequent queries |
| `EXPLAIN` / actual execution plan | See sort/hash/spill and estimates | Diagnosis |
| `UPDATE STATISTICS` / `ANALYZE` | Refresh estimates | After big data changes |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| Pre-aggregate children before the join | Single child table; the join is already one-to-one |
| Window aggregate | You only need one row per group; wasteful row inflation |
| Conditional aggregation pivot | Dynamic column lists (needs dynamic SQL) |
| Summary table / materialized view | Data changes constantly and the summary can't keep up |
| Incremental maintenance | Late-arriving or updated historical data without reconciliation |
| Approximate distinct | Billing, compliance, reconciliation |
| Hints to force a plan | Stale statistics are the real problem |

### 9. Common mistakes
- **Joining two child tables then summing** → inflated totals → pre-aggregate each child.
- **Average of averages** → biased → carry sum and count.
- **Grouping on high-cardinality keys unnecessarily** → huge hash tables, spills → group at the needed grain.
- **Over-materializing** → stale or inconsistent summaries → define a consistency plan.
- **Ignoring misestimates** → spill and bad join choices → update statistics, check the histogram.
- **Assuming covering indexes help every aggregate** → they help only when the measure columns are included and selectivity allows → check the plan.
- **Exact total counts for pagination** → slow → estimates or "load more".
- **Not reconciling a new pre-aggregated pipeline** → silent drift → compare row-for-row totals before cutover.

### 10. 🎯 Interview kit
**30-second answer:** I aggregate at the right grain: pre-aggregate children before joining to avoid fan-out, use conditional aggregation for pivots, window aggregates when I need detail plus totals, and look at the plan for sort/hash/spill problems. For repeated reads I pre-compute with summary tables or materialized views and maintain them incrementally, and I use approximate distinct counts when exactness isn't required.

**Structure a longer answer:** define the grain → show the naive mistake (fan-out) → correct pattern → plan/operators and statistics → persistence (materialization, incremental) → validation (control totals) → approximate options.

**Likely follow-ups:**
- *Windowed vs. grouped SUM?* → Grouped returns one row per group; windowed keeps all rows with the group value attached; use both when a report shows detail and totals.
- *How do you diagnose a slow `GROUP BY`?* → Plan: scan vs. sort vs. hash aggregate, spills, estimate vs. actual rows.
- *Why do equivalent aggregate queries differ?* → Different rewrites/estimates; fix statistics or rewrite.
- *How to validate an aggregate migration?* → Row-for-row comparison, control totals, checksums, parallel run before cutover.
- *Retention queries explode when?* → Cross joins with calendars or per-N self joins; compute offsets once.

**Signals interviewers look for:** grain awareness, plan literacy, pragmatic caching/materialization, correctness first (reconciliation), and knowing when approximation is appropriate.

### 11. Check yourself
- [ ] Explain fan-out with a two-child-table example and rewrite it correctly.
- [ ] Write a pivot with `SUM(CASE …)` and describe its limits.
- [ ] Write a percent-of-total query with a window over a grouped result.
- [ ] Describe how a hash aggregate spills and how to detect it in a plan.
- [ ] Describe how you'd incrementally maintain a daily sales summary and verify it.
- [ ] Explain why `MAX` can't solve the last-non-null-value problem and show a solution.
- [ ] State when an approximate distinct count is acceptable and what error you'd quote.
<!-- explainer:end -->

## Practice questions

- [Basic] [What is gRPC and what problem does it solve?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/Aggregation/Q-00913_What_is_gRPC_and_what_problem_does_it_solve.md>)
- [Intermediate] [Explain how you audit an aggregate report for correctness against source-of-truth: reconciliation queries, checksums, and control totals. What did you catch this way?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/Aggregation/Q-00812_Explain_how_you_audit_an_aggregate_report_for_correctness_ag.md>)
- [Deep] [How would you incrementally maintain a running daily SUM as new rows arrive, without rescanning the full fact table each night?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/Aggregation/Q-00721_How_would_you_incrementally_maintain_a_running_daily_SUM_as_.md>)
- [Deep] [Explain how a trigger that increments an aggregate counter can become a hot-row contention bottleneck, and how you would shard or batch it.](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/Aggregation/Q-00744_Explain_how_a_trigger_that_increments_an_aggregate_counter_c.md>)
- [Deep] [How do columnar stores like ClickHouse/Redshift make SUM and COUNT dramatically cheaper than a row store, and where does that advantage disappear?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/Aggregation/Q-00776_How_do_columnar_stores_like_ClickHouse_Redshift_make_SUM_and.md>)
- [Deep] [How would you compute year-over-year growth per category in one query, and why is a self-join to last year's grouped totals often the wrong tool versus LAG over grouped data?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/Aggregation/Q-00795_How_would_you_compute_year-over-year_growth_per_category_in_.md>)
- [Expert] [When a query joins to two child tables and then SUMs, why do you get a fan-out cartesian inflation, and how do you fix it with pre-aggregation before the join?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/Aggregation/Q-00749_When_a_query_joins_to_two_child_tables_and_then_SUMs_why_do_.md>)
- [Expert] [Explain how bitmap/roaring-bitmap aggregation (e.g. groupBitmap in ClickHouse) speeds up distinct-user analytics compared to COUNT(DISTINCT).](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/Aggregation/Q-00787_Explain_how_bitmap_roaring-bitmap_aggregation_e_g_groupBitma.md>)

## Related concepts

- [Partitioning and Partition Pruning](<../../Database_Internals/DBMS_Concepts/Partitioning_and_Partition_Pruning.md>) *(DBMS Concepts)*
- [Statistics and Sargability](<../../Query_Performance/Execution_Plans/Statistics_and_Sargability.md>) *(Execution Plans)*
- [Large Values and TOAST](<../SQL_Fundamentals/Large_Values_and_TOAST.md>) *(SQL Fundamentals)*
- [Aggregation Query Practice](<../SQL_Fundamentals/Aggregation_Query_Practice.md>) *(SQL Fundamentals)*
- [Clickstream Counting Pipeline](<../../../07_Distributed_Systems_and_Design/System_Design/System_Design_Core/Clickstream_Counting_Pipeline/_index.md>) *(System Design Core)*
- [Conditional Aggregation](<../SQL_Fundamentals/Conditional_Aggregation.md>) *(SQL Fundamentals)*

## References

- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 4: Joins, Set Operations, and Subqueries › Real-World Use Cases
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 9: Execution Plans and Query Store › Aggregate and Compute Operators
- *Expert T-SQL Window Functions in SQL Server* — Kathi Kellenberger, Clayton Groom, Ed Pollack — Solving Queries Using Accumulating Aggregates
- *Expert T-SQL Window Functions in SQL Server* — Kathi Kellenberger, Clayton Groom, Ed Pollack — Moving Totals and Simple Moving Averages
- *Expert T-SQL Window Functions in SQL Server* — Kathi Kellenberger, Clayton Groom, Ed Pollack — Summary › Contents
- *Expert Performance Indexing in SQL Server* — Jason Strate, Grant Fritchey — Histogram
- *Expert Performance Indexing in SQL Server* — Jason Strate, Grant Fritchey — Singleton Lookup
- *Expert Performance Indexing in SQL Server* — Jason Strate, Grant Fritchey — Page Compression › Performance Counters
- *Delta Lake: The Definitive Guide* — Denny Lee et al. — Performance Tuning Optimizing Your Data Pipelines with Delta Lake › Aggregations
- *Learning Spark (2nd ed.)* — Jules S. Damji, Brooke Wenig, Tathagata Das, Denny Lee — Apache Spark's Structured APIs › Aggregations
- *Learning Spark (2nd ed.)* — Jules S. Damji, Brooke Wenig, Tathagata Das, Denny Lee — Data Transformations › Stateful Streaming Aggregations
- *Mapping Data Flows in Azure Data Factory* — Mark Kromer — Introduction to Mapping Data Flows › Aggregate
- *Mapping Data Flows in Azure Data Factory* — Mark Kromer — Monitor, Manage, and Optimize › Profile Stats

