---
id: "01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Attribution_with_Window_Functions"
level: 4
type: "concept"
topic: "01_Databases_and_SQL/SQL_Language/SQL_Fundamentals"
tools: []
---

# Attribution with Window Functions

**L4 Advanced** · [↑ SQL Fundamentals](<_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** Attribution means "pick the right row (first, last, most recent before X) per user and credit it" — `ROW_NUMBER()` over a *fully deterministic* `ORDER BY` does this correctly and in one scan, whereas joining back to `MIN(ts)`/`MAX(ts)` silently duplicates rows when timestamps tie.

### 1. The big picture
Marketing, experimentation and incident analysis all ask the same shape of question: *for each entity, which event should get the credit?* First-touch (the earliest marketing interaction), last-touch (the latest one before conversion), "the most recent deploy before this incident", "the order immediately before this one". Each is a **pick-one-row-per-group** problem, sometimes with a time bound.

SQL gives you three families of tools: (1) **window functions** (`ROW_NUMBER`, `FIRST_VALUE`, `LAG/LEAD`) that rank or look around rows without collapsing them; (2) **aggregate-then-join** (`MIN`/`MAX` in a subquery, joined back); and (3) **lateral / `CROSS APPLY`** per-row subqueries. This lesson explains when each is right, why the second is subtly wrong, and how indexing decides whether the first is cheap. It builds on GROUP BY vs PARTITION BY and on deduplication, which is the same "keep one row per key" idea.

### 2. Core concepts and vocabulary
- **Touch / event** — one row of an events fact (`user_id, ts, channel, …`).
- **First-touch / last-touch** — the earliest / latest touch per user (or per user before a conversion time).
- **Window function** — computes a value per row from a related set of rows (the *window*) defined by `OVER (PARTITION BY … ORDER BY …)`; does not collapse rows like `GROUP BY`.
- **Tie** — two rows with the same `ORDER BY` value inside one partition. Ranking among ties is *arbitrary* unless you add a tie-breaker.
- **Tie-breaker** — an extra, unique `ORDER BY` column (e.g. `event_id`) that makes the ordering total and the result reproducible.
- **`CROSS APPLY` / `LATERAL`** — a subquery evaluated once per outer row, able to reference that row's columns (`TOP (1) … ORDER BY …`).
- **POC index** — the index shape that serves a window function: **P**artition columns, then **O**rder columns, then **C**overering (`INCLUDE`) columns.

### 3. How it works, step by step
**First-touch per user with `ROW_NUMBER`:**
1. *Partition* the rows by `user_id` (conceptually: one bucket per user).
2. *Order* each bucket by `ts`, then by a unique tie-breaker (`event_id`).
3. Number the rows 1, 2, 3 … inside each bucket.
4. Keep `rn = 1` (first touch) — or `ORDER BY ts DESC, event_id DESC` and keep `rn = 1` for last touch.

Windows cannot be filtered in the same `SELECT` they are computed in (window functions are evaluated *after* `WHERE`/`GROUP BY`/`HAVING`), so wrap the numbering in a CTE or derived table and filter outside.

**Why `MIN`/`MAX` + join is subtly wrong:**
1. Subquery: `SELECT user_id, MIN(ts) AS first_ts FROM events GROUP BY user_id`.
2. Join back on `user_id AND ts = first_ts` to fetch the channel.
3. If a user has **two events with the same earliest `ts`** (batched ingestion, second-resolution timestamps, two pixels firing together) the join returns **both** rows → the user is counted twice, revenue is double-credited, and your attribution totals exceed 100%.

`ROW_NUMBER` always returns exactly one row per partition, because it *numbers* ties instead of matching them.

**Which ranking function?** `ROW_NUMBER` (always unique — use for "exactly one"), `RANK` (ties share a number and leave gaps — "all tied winners"), `DENSE_RANK` (ties share, no gaps). For "pick one row" you want `ROW_NUMBER` plus a tie-breaker.

### 4. Worked example
```sql
-- events(event_id, user_id, ts, channel); conversions(user_id, converted_at, revenue)

-- 1) First- and last-touch per user in ONE pass (no self-join)
WITH ranked AS (
    SELECT user_id, channel, ts,
           ROW_NUMBER() OVER (PARTITION BY user_id ORDER BY ts ASC,  event_id ASC)  AS rn_first,
           ROW_NUMBER() OVER (PARTITION BY user_id ORDER BY ts DESC, event_id DESC) AS rn_last
    FROM   events
)
SELECT user_id,
       MAX(CASE WHEN rn_first = 1 THEN channel END) AS first_touch_channel,
       MAX(CASE WHEN rn_last  = 1 THEN channel END) AS last_touch_channel
FROM   ranked
WHERE  rn_first = 1 OR rn_last = 1
GROUP  BY user_id;

-- 2) Last touch BEFORE conversion, within a 30-day lookback
WITH touches AS (
    SELECT c.user_id, c.revenue, e.channel,
           ROW_NUMBER() OVER (PARTITION BY c.user_id, c.converted_at
                              ORDER BY e.ts DESC, e.event_id DESC) AS rn
    FROM   conversions c
    JOIN   events e
           ON  e.user_id = c.user_id
           AND e.ts <= c.converted_at
           AND e.ts >  DATEADD(DAY, -30, c.converted_at)
)
SELECT channel, SUM(revenue) AS attributed_revenue
FROM   touches WHERE rn = 1
GROUP  BY channel;
```
Reading the result: the sum of `attributed_revenue` should equal total conversion revenue *for converters that had a touch*. If it is larger, you have a tie bug (duplicates); if smaller, some conversions had no touch in the lookback window (an `INNER JOIN` dropped them — switch to `LEFT JOIN` and bucket them as `'direct'`).

A **linear multi-touch** variant uses `COUNT(*) OVER (PARTITION BY user_id, converted_at)` to split revenue: `revenue * 1.0 / COUNT(*) OVER (…)` per touch.

### 5. Under the hood (optional at L4)
A window function with `PARTITION BY a ORDER BY b` needs the input *sorted by (a, b)*. The engine either reads an index already in that order (cheap, streaming **Window Spool/Segment** operators) or inserts a **Sort** operator (memory-hungry, can spill to tempdb). Kellenberger's tuning chapter shows the practical consequence: with an index on the join key, a plan may merge-join first and then sort for `ROW_NUMBER`; rewriting so the window is computed in a CTE *before* the join removed the Sort and cut that query's relative cost to ~12% of the batch. The lesson: apply the window to the narrowest table with the right index, *then* join.

### 6. Performance and design implications
- Build a **POC index** on the events table: `(user_id, ts, event_id) INCLUDE (channel)`. Both `ASC` first-touch and `DESC` last-touch can use it (the engine can scan an index backwards, though two windows with opposite directions may need a sort for one of them).
- Compute the window before joining to wide dimension tables; look for a `Sort` operator in the actual plan and for a spill warning.
- A self-join "find pairs within 5 minutes" is O(n²) because each row is compared with every other row; bound it with a range predicate on an indexed timestamp (`e2.ts BETWEEN e1.ts AND DATEADD(MINUTE,5,e1.ts)`) so each probe is an index seek over a small range.
- `CROSS APPLY (SELECT TOP (1) … WHERE d.ts <= i.start ORDER BY d.ts DESC)` wins when the outer set is small (e.g. 10k incidents) and the inner table is big and indexed on `(ts)`: it does N index seeks and never sorts the whole table. Windows win when you need *every* row's rank over most of the table.
- Measure with `SET STATISTICS IO, TIME ON` and the actual execution plan; compare logical reads, not just elapsed time.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `ROW_NUMBER() OVER (PARTITION BY … ORDER BY …)` | Unique 1..n per partition | Exactly-one-row picks, dedupe |
| `FIRST_VALUE / LAST_VALUE` | Value from the first/last row of the frame | Copy the first touch onto every row (watch the default frame for `LAST_VALUE`) |
| `LAG / LEAD(col, n, default)` | Value from a previous/next row | Time since previous event, "did channel change?" |
| `CROSS APPLY (SELECT TOP 1 …)` / `LATERAL` | Per-row top-1 subquery | Most recent change before each incident |
| `SET STATISTICS IO ON` | Logical reads per table | Compare window vs apply plans |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| `ROW_NUMBER` + filter: need one row for *most/all* groups, index on partition+order | Outer set is tiny and the inner table huge — a Sort of everything is wasteful |
| `CROSS APPLY TOP 1`: small driving set, big indexed lookup table, time-bounded lookback | No supporting index → N full scans |
| `RANK`: business wants all tied events credited | You need a single row → duplicates |
| `MIN`/`MAX` + join | Only safe if the pair (group key, value) is *unique*; otherwise duplicates |

### 9. Common mistakes
- Joining back on `MIN(ts)` → duplicate rows on ties → totals exceed 100%. Fix: `ROW_NUMBER` with a unique tie-breaker.
- Ordering by `ts` only → result changes between runs → non-reproducible reports. Fix: add `event_id`.
- Using `LAST_VALUE` with the default frame (`RANGE BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW`) → returns the current row. Fix: specify `ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING` or use `ROW_NUMBER DESC`.
- Filtering on a window result in `WHERE` → syntax error. Fix: CTE/derived table.
- `INNER JOIN` to touches drops conversions with no touch → revenue disappears. Fix: `LEFT JOIN`, label as direct.
- NULL timestamps sort first in `ASC` → "first touch" is a bad row. Fix: filter or `ORDER BY CASE WHEN ts IS NULL THEN 1 ELSE 0 END, ts`.

### 10. 🎯 Interview kit
**30-second answer:** "I'd number each user's events with `ROW_NUMBER() OVER (PARTITION BY user ORDER BY ts, event_id)` and keep row 1 for first touch (DESC for last touch). A MIN/MAX subquery joined back is wrong when two events share the timestamp, because the join returns both."

**Structure a longer answer:** clarify (tie rules, lookback window, what counts as a touch) → core mechanism (partition/order/number, filter in an outer query) → trade-offs (window vs `CROSS APPLY`, POC index, sort cost) → real scenario (conversion with 30-day lookback; check that attributed revenue reconciles to total).

**Likely follow-ups:**
- *When does `CROSS APPLY` beat a window?* → small driving set, big indexed inner table, top-N per outer row; seeks instead of a full sort.
- *Why is a self-join for "pairs within 5 minutes" O(n²)?* → every row is compared to every row; a bounded range predicate on an indexed timestamp turns it into seeks.
- *How do you avoid a Sort in the plan?* → index on (partition cols, order cols) and compute the window before joins.
- *Merge vs hash join?* → merge needs both inputs sorted (cheap with matching indexes); hash spills when the build side exceeds memory.

**Signals interviewers look for:** you name ties unprompted, you add a deterministic tie-breaker, you reconcile totals, you talk about the index that supports the window.

### 11. Check yourself
- [ ] Write first- and last-touch in one query and explain why it is tie-safe.
- [ ] Construct a 3-row example where the MIN-join returns a duplicate.
- [ ] Say which index supports `PARTITION BY user_id ORDER BY ts` and where to see the Sort in a plan.
- [ ] Decide between `CROSS APPLY` and a window for "latest deploy within 30 minutes before each incident" and justify.
- [ ] Explain why `LAST_VALUE` often "doesn't work".
<!-- explainer:end -->

## Practice questions

- [Intermediate] [How do you use a self-join to compute the day-over-day difference in a daily snapshot table when there's no window function support?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Q-40072_How_do_you_use_a_self-join_to_compute_the_day-over-day_diffe.md>)
- [Deep] [When would a LATERAL join (CROSS APPLY) to fetch the top-3 recent orders per customer beat a window-function approach, and what indexing does the lateral subquery require?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Q-39910_When_would_a_LATERAL_join_CROSS_APPLY_to_fetch_the_top-3_rec.md>)
- [Deep] [How would you join a table to itself to compare each row with the previous row without window functions?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Q-40211_How_would_you_join_a_table_to_itself_to_compare_each_row_wit.md>)
- [Deep] [Explain how a self-join on a huge table for 'find pairs of rows within 5 minutes of each other' becomes O(n^2), and how a range join with a bounded window plus an index on the timestamp makes it tractable.](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Q-40468_Explain_how_a_self-join_on_a_huge_table_for_find_pairs_of_ro.md>)
- [Deep] [How would you use SQL window functions to compute per-user first-touch and last-touch attribution features, and why is a MIN/MAX subquery join subtly wrong for ties?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Q-40700_How_would_you_use_SQL_window_functions_to_compute_per-user_f.md>)
- [Deep] [You must build a change-vs-incident correlation report: for each incident, find the most recent deploy/config-change within 30 minutes before start_time. Why is a LATERAL/CROSS APPLY top-1 join better than a window function here?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Q-41071_You_must_build_a_change-vs-incident_correlation_report_for_e.md>)
- [Expert] [Describe when the planner chooses a merge join, how adding matching indexes on both sides converted a spilling hash join into a cheap merge join, and when the sort cost isn't worth it.](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Q-39926_Describe_when_the_planner_chooses_a_merge_join_how_adding_ma.md>)
- [Expert] [A LEFT JOIN LATERAL (or CROSS APPLY) is proposed to fetch the top-3 recent orders per customer in one query. When does a lateral join beat a window-function approach, and what are its indexing requirements?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Q-40389_A_LEFT_JOIN_LATERAL_or_CROSS_APPLY_is_proposed_to_fetch_the_.md>)

## Related concepts

- [EXISTS Semi-joins and Predicate Placement](<../Joins_and_Subqueries/EXISTS_Semi-joins_and_Predicate_Placement.md>) *(Joins and Subqueries)*
- [UDF Performance](<../SQL_Functions/UDF_Performance.md>) *(SQL Functions)*

## References

- *Expert T-SQL Window Functions in SQL Server* — Kathi Kellenberger, Clayton Groom, Ed Pollack — Indexing to Improve the Performance of Window Functions
- *Designing Data-Intensive Applications* — Martin Kleppmann — Stream Joins › Stream-table join (stream enrichment)
- *Designing Data-Intensive Applications* — Martin Kleppmann — Stream Joins › Time-dependence of joins
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 4: Joins, Set Operations, and Subqueries › Advanced Considerations for CROSS JOIN and SELF JOIN
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 4: Joins, Set Operations, and Subqueries › SELF JOIN: Advanced Use Cases
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 4: Joins, Set Operations, and Subqueries › Combining with Other SQL Features
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 9: Execution Plans and Query Store › Option 2: Use Query Hints (Use with Caution).
- *Databricks Certified Associate Developer for Apache Spark* — Saba Shah — Joining DataFrames in Spark › Left joins
- *Databricks Certified Associate Developer for Apache Spark* — Saba Shah — Data-based optimizations in Apache Spark › Use case
- *Fuzzy Data Matching with SQL* — Jim Lehmer — CustomerCountByState View › J
- *Beginning T-SQL* — Kathi Kellenberger, Lee Everest — Joining Tables › Solutions to Exercise 5-2: Using OUTER JOIN
- *Beginning T-SQL* — Kathi Kellenberger, Lee Everest — Writing Advanced Queries › Using CROSS APPLY and OUTER APPLY

