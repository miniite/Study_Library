---
id: "01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Conditional_Aggregation"
level: 4
type: "concept"
topic: "01_Databases_and_SQL/SQL_Language/SQL_Fundamentals"
tools: []
---

# Conditional Aggregation

**L4 Advanced** · [↑ SQL Fundamentals](<_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** Conditional aggregation puts a `CASE` expression *inside* an aggregate (`SUM(CASE WHEN … THEN 1 ELSE 0 END)`) so one scan and one `GROUP BY` produce many differently-filtered metrics side by side — it is also the portable way to pivot.

### 1. The big picture
A typical monitoring or reporting need: per service, the total requests, the error requests, the slow requests, the distinct hosts that errored. The naive approach runs one query per metric (each re-scans the table) or joins several grouped subqueries. **Conditional aggregation** computes all metrics in a **single pass**: every row is read once and contributes to whichever aggregates its `CASE` says it belongs to.

It sits between plain aggregation (`COUNT/SUM` over a `GROUP BY`) and the more elaborate tools — `PIVOT`, window functions, and `FILTER` clauses. Learn it well: it appears in nearly every metrics query (error rate, completeness, funnel steps, A/B experiment arms) and is the answer to "pivot rows to columns when the column list is dynamic".

### 2. Core concepts and vocabulary
- **Aggregate** — a function collapsing a group into one value: `COUNT`, `SUM`, `AVG`, `MIN`, `MAX`.
- **`CASE` expression** — returns a value depending on conditions; with no `ELSE` it returns `NULL`.
- **`COUNT(*)` vs `COUNT(col)`** — `COUNT(*)` counts rows; `COUNT(col)` counts non-NULL values; aggregates other than `COUNT(*)` *ignore NULL*. This is the trick that makes `COUNT(CASE WHEN … THEN 1 END)` work.
- **`FILTER (WHERE …)`** — PostgreSQL/standard syntax: `COUNT(*) FILTER (WHERE level = 'ERROR')`. T-SQL has no `FILTER`; use `CASE`.
- **Pivot** — turning distinct row values into columns.
- **Rate / ratio** — a conditional count divided by a total; needs divide-by-zero protection.

### 3. How it works, step by step
1. The engine reads each row once (after the `WHERE` filter).
2. It assigns the row to a group by the `GROUP BY` key.
3. For every aggregate in the select list it evaluates the inner `CASE` for that row; `NULL` results are skipped by `SUM/AVG/COUNT(expr)`, `0/1` results are summed.
4. At the end of the input, each group emits one output row with all aggregates.

Three equivalent idioms:

| Goal | Pattern |
|---|---|
| Count rows meeting a condition | `SUM(CASE WHEN cond THEN 1 ELSE 0 END)` or `COUNT(CASE WHEN cond THEN 1 END)` |
| Sum a value only for some rows | `SUM(CASE WHEN cond THEN amount ELSE 0 END)` |
| Distinct count under a condition | `COUNT(DISTINCT CASE WHEN cond THEN host END)` |
| Ratio | `1.0 * SUM(CASE WHEN cond THEN 1 ELSE 0 END) / NULLIF(COUNT(*), 0)` |

### 4. Worked example
```sql
-- requests(ts, service, host, status, latency_ms)
SELECT  service,
        COUNT(*)                                                        AS total_requests,
        SUM(CASE WHEN status >= 500 THEN 1 ELSE 0 END)                  AS errors,
        1.0 * SUM(CASE WHEN status >= 500 THEN 1 ELSE 0 END)
            / NULLIF(COUNT(*), 0)                                       AS error_rate,
        COUNT(DISTINCT CASE WHEN status >= 500 THEN host END)           AS hosts_with_errors,
        AVG(CASE WHEN status < 500 THEN latency_ms END)                 AS avg_latency_ok,
        SUM(CASE WHEN latency_ms > 1000 THEN 1 ELSE 0 END)              AS slow_requests
FROM    requests
WHERE   ts >= DATEADD(MINUTE, -15, SYSUTCDATETIME())
GROUP BY service;
```
One scan of the last 15 minutes, five metrics. Notes on reading it:
- `1.0 *` forces decimal division (integer division would give 0).
- `NULLIF(COUNT(*),0)` turns a zero denominator into `NULL` instead of an error.
- `AVG(CASE … END)` averages only successful requests because the `CASE` yields `NULL` for others and `AVG` ignores NULL.

**Pivot a narrow table to wide** (events per type per day):
```sql
SELECT CAST(ts AS date) AS day,
       SUM(CASE WHEN event_type = 'click'    THEN 1 ELSE 0 END) AS clicks,
       SUM(CASE WHEN event_type = 'purchase' THEN 1 ELSE 0 END) AS purchases,
       SUM(CASE WHEN event_type = 'refund'   THEN 1 ELSE 0 END) AS refunds
FROM   events
GROUP  BY CAST(ts AS date);
```

### 5. Under the hood (optional at L4)
A conditional aggregate is just an ordinary aggregate whose argument is a scalar expression evaluated per row. In SQL Server the plan shows a single **Stream Aggregate** or **Hash Match (Aggregate)** with several `[Expr]` outputs; the `CASE`s are *compute scalar* work, which is cheap relative to I/O. `COUNT(DISTINCT …)` is more expensive: it needs a hash/sort per distinct aggregate, and several different `DISTINCT` aggregates in one query can multiply the work. `PIVOT` in T-SQL is rewritten by the optimizer into the same conditional-aggregate plan, so there is no performance reason to prefer it.

### 6. Performance and design implications
- It scans the table once regardless of the number of metrics — far better than N subqueries.
- Push time and partition predicates into `WHERE`, not into every `CASE`; a filter in `WHERE` can use an index seek, a `CASE` cannot.
- Only condition *aggregates* belong in `CASE`; filter out irrelevant rows in `WHERE` when *all* metrics need the same restriction.
- `LIKE '%timeout%'` (leading wildcard) is not sargable, so counting matches forces a scan of the text column; mitigate with a parsed/structured error-code column, a full-text index, or pre-classifying at ingest.
- Pre-aggregate large raw logs into per-minute rollups (see Incremental Rollups) and run conditional aggregation over the rollup.
- Check with `SET STATISTICS IO ON`: logical reads should equal one pass of the table/index.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `SUM(CASE WHEN … THEN 1 ELSE 0 END)` | Conditional row count | Any portable engine |
| `COUNT(*) FILTER (WHERE …)` | Same, standard SQL | PostgreSQL, SQLite, DuckDB |
| `COUNTIF(cond)` | Same | BigQuery |
| `COUNT_IF(cond)` | Same | Snowflake, Databricks |
| `NULLIF(x, 0)` | Avoid divide-by-zero | Ratios |
| `PIVOT (… FOR col IN (…))` | Static pivot | Known, fixed column list |
| Dynamic SQL + `STRING_AGG` | Generate the `CASE` list | Column set unknown at write time |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| Conditional aggregation: several metrics from one table, columns known | The set of columns changes constantly — generate SQL or pivot in the BI layer |
| `PIVOT`: you like the syntax and the list is fixed | You need more than one aggregate per pivoted column (clumsy) |
| Window functions (`COUNT() OVER`) | You want *one* row per group — use aggregation |
| Separate queries/CTEs | Metrics have *different* source tables or join paths |

### 9. Common mistakes
- Integer division (`SUM(CASE…)/COUNT(*)`) → rate is 0 or 1 → multiply by `1.0` or cast.
- Writing `ELSE 0` inside `AVG(CASE …)` → NULLs become zeros and pull the average down → omit `ELSE` so non-matching rows are ignored.
- `COUNT(CASE WHEN cond THEN 1 ELSE 0 END)` → counts *every* row, because `0` is not NULL → use `SUM`, or drop the `ELSE`.
- `COUNT(DISTINCT col)` ignores NULL → "distinct hosts" excludes rows with a NULL host; decide if that is intended.
- Putting a row-level filter in `HAVING` instead of `WHERE` → no index use, extra work.
- Comparing to `NULL` with `=` inside `CASE` → never true → use `IS NULL`.

### 10. 🎯 Interview kit
**30-second answer:** "Conditional aggregation puts a `CASE` inside `SUM` or `COUNT` so a single `GROUP BY` pass computes several filtered metrics at once — error counts, rates, pivoted columns — instead of several queries or joins."

**Structure a longer answer:** clarify (which metrics, which grain, NULL semantics) → core mechanism (one scan, `CASE` yields 1/0 or NULL, aggregates skip NULL) → trade-offs (vs `PIVOT`, `FILTER`, windows, dynamic columns) → scenario (per-service error rate over the last 15 minutes with divide-by-zero guard).

**Likely follow-ups:**
- *Why does `COUNT(CASE … ELSE 0 END)` over-count?* → 0 is not NULL.
- *How do you pivot when the values aren't known?* → dynamic SQL, or pivot in the BI/app layer.
- *Why can a `LIKE '%x%'` count be slow?* → not sargable; full scan; fix with structured columns or full-text.
- *`PIVOT` vs `CASE`?* → same plan; `CASE` is more flexible and portable.

**Signals interviewers look for:** one-pass thinking, correct NULL semantics, safe division, awareness of cost of `DISTINCT` and leading wildcards.

### 11. Check yourself
- [ ] Write per-service total, error count and error rate in one query without a subquery.
- [ ] Explain precisely why `COUNT(CASE … ELSE 0 END)` is wrong.
- [ ] Rewrite a `PIVOT` as conditional aggregation.
- [ ] State two reasons a conditional count with `LIKE '%…%'` is slow and one fix.
- [ ] Say when `AVG(CASE…)` needs no `ELSE`.
<!-- explainer:end -->

## Practice questions

- [Basic] [How would you write a query to compute the number of rows loaded per source file so you can spot a partial or empty file?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Q-40031_How_would_you_write_a_query_to_compute_the_number_of_rows_lo.md>)
- [Basic] [How do you write a query to count how many predictions the model logged per day from a predictions table for monitoring?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Q-40574_How_do_you_write_a_query_to_count_how_many_predictions_the_m.md>)
- [Basic] [How do you count the number of ERROR-level log lines per service in the last 15 minutes?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Q-40947_How_do_you_count_the_number_of_ERROR-level_log_lines_per_ser.md>)
- [Basic] [How would you compute the number of distinct hosts that emitted an ERROR log in the last hour?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Q-40965_How_would_you_compute_the_number_of_distinct_hosts_that_emit.md>)
- [Basic] [Write a query to count distinct services that generated an alert each day over the last week.](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Q-41033_Write_a_query_to_count_distinct_services_that_generated_an_a.md>)
- [Intermediate] [How would you write a query to find conversations that triggered a moderation flag and count how many distinct users they came from?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Q-40815_How_would_you_write_a_query_to_find_conversations_that_trigg.md>)
- [Intermediate] [How would you count HTTP status codes grouped into 2xx/3xx/4xx/5xx classes from an access-log table using CASE?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Q-41001_How_would_you_count_HTTP_status_codes_grouped_into_2xx_3xx_4.md>)
- [Intermediate] [How would you compute the count of log entries per service per severity in one query using conditional aggregation?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Q-41010_How_would_you_compute_the_count_of_log_entries_per_service_p.md>)

## Related concepts

- [Classic SQL Query Problems](<../../Database_Internals/DBMS_Concepts/Classic_SQL_Query_Problems.md>) *(DBMS Concepts)*
- [Counting with Joins](<../Aggregation/Counting_with_Joins.md>) *(Aggregation)*
- [Advanced Aggregation](<../Aggregation/Advanced_Aggregation.md>) *(Aggregation)*

## References

- *Hadoop: The Definitive Guide* — Tom White — MapReduce Features › Retrieving counters
- *Hadoop: The Definitive Guide* — Tom White — HBase › Tip
- *Data Pipelines with Apache Airflow* — Bas P. Harenslak, Julian de Ruiter — Templating tasks using the Airflow context
- *Databricks Certified Associate Developer for Apache Spark* — Saba Shah — Questions

