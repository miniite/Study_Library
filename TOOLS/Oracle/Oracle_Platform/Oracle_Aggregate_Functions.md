---
id: "TOOLS/Oracle/Oracle_Platform/Oracle_Aggregate_Functions"
level: 4
type: "concept"
topic: "TOOLS/Oracle/Oracle_Platform"
tools: ["Oracle"]
---

# Oracle Aggregate Functions

**L4 Advanced** · [↑ Oracle Platform](<_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** Aggregate functions collapse many rows into one value per group (`GROUP BY`), and analytic (window) functions compute similar values *without* collapsing rows, using `OVER (PARTITION BY … ORDER BY … frame)`; Oracle adds its own details for NULLs, dates, `ROWNUM` and histograms.

### 1. The big picture
Reporting questions ("revenue per region per month", "top 3 salaries per department", "days between order and shipment") need two families of tools. **Aggregates** reduce a set of rows to a single row per group. **Analytic functions** keep every row and attach a computed column derived from a "window" of related rows. Both are evaluated *after* `WHERE` and (for aggregates) `GROUP BY`/`HAVING`, which explains most of the rules about what you can reference where.

Oracle specifics you must know: how `NULL` behaves in aggregates, how `DATE` arithmetic returns numbers of days, how `ROWNUM` is assigned *before* `ORDER BY`, and how column histograms let the optimizer cope with skewed data.

### 2. Core concepts and vocabulary
- **Aggregate function** — `COUNT`, `SUM`, `AVG`, `MIN`, `MAX` (also `LISTAGG`, `MEDIAN`, `STDDEV`) reduce a group to one value; most ignore `NULL`s.
- **`GROUP BY`** — partitions rows into groups of equal values; every non-aggregated select-list column must appear in it.
- **`HAVING`** — filter applied to groups *after* aggregation (`WHERE` filters rows *before*).
- **Analytic function** — a function with an `OVER` clause that returns a value per input row.
- **`PARTITION BY`** — splits the window into independent partitions (like `GROUP BY` but rows are kept).
- **Window frame** — which rows relative to the current row are included: `ROWS BETWEEN 2 PRECEDING AND CURRENT ROW` (physical rows) or `RANGE BETWEEN INTERVAL '7' DAY PRECEDING AND CURRENT ROW` (logical value range).
- **`ROWNUM`** — pseudocolumn numbering rows *as they are returned from the access step*, before sorting.
- **`ROW_NUMBER()`** — analytic function numbering rows *within the sorted window*.
- **Histogram** — statistics object describing value frequency distribution of a column for the optimizer.

### 3. How it works, step by step
**Logical order of a grouped query**
1. `FROM` / joins produce the rows.
2. `WHERE` removes rows.
3. `GROUP BY` forms groups.
4. Aggregates are computed per group.
5. `HAVING` removes groups.
6. Analytic functions are evaluated over the result of the previous steps.
7. `SELECT` list, `DISTINCT`, then `ORDER BY`.
8. `FETCH FIRST` / row limiting (12c+), or outer-query `ROWNUM` filtering.

**NULL handling**
- `COUNT(*)` counts rows; `COUNT(col)` counts non-NULL values; `COUNT(DISTINCT col)` counts distinct non-NULL values.
- `SUM`, `AVG`, `MIN`, `MAX` skip NULLs; if *all* inputs are NULL the result is NULL (`COUNT` returns 0). `AVG(col)` divides by the number of non-NULLs, not rows, so use `AVG(NVL(col,0))` if NULL means zero.

**Date arithmetic**
- `date1 - date2` returns a **number of days** (fractional if times differ). `TRUNC()` both sides for whole days.
- `ADD_MONTHS(d, n)` moves by months (and keeps last-day-of-month alignment: `ADD_MONTHS('31-JAN-2026',1)` is 28-FEB-2026).
- `MONTHS_BETWEEN(d1, d2)` returns fractional months (positive when `d1` is later).
- `LAST_DAY(d)` returns the last day of d's month.
- Timestamps subtract to an `INTERVAL DAY TO SECOND`; use `EXTRACT(DAY FROM …)` to get parts.

**Row numbering and top-N**
- `WHERE ROWNUM <= 5 ORDER BY salary DESC` takes any five rows and then sorts them: wrong. Sort first in an inline view and filter outside, or use `FETCH FIRST 5 ROWS ONLY` (12c+) or `ROW_NUMBER()`/`RANK()`.

### 4. Worked example
```sql
-- Sample: EMP(empno, deptno, sal, comm, hiredate)

-- 1. Aggregates and HAVING
SELECT deptno,
       COUNT(*)            AS headcount,
       COUNT(comm)         AS with_comm,      -- ignores NULL comm
       ROUND(AVG(sal),2)   AS avg_sal,
       MAX(sal)            AS max_sal
FROM   emp
GROUP  BY deptno
HAVING COUNT(*) >= 3
ORDER  BY avg_sal DESC;

-- 2. Top 3 earners per department (analytic)
SELECT *
FROM (
  SELECT e.*,
         DENSE_RANK() OVER (PARTITION BY deptno ORDER BY sal DESC) AS rnk
  FROM emp e
)
WHERE rnk <= 3;

-- 3. Running total and moving average
SELECT empno, hiredate, sal,
       SUM(sal) OVER (PARTITION BY deptno ORDER BY hiredate
                      ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW) AS running_total,
       AVG(sal) OVER (PARTITION BY deptno ORDER BY hiredate
                      ROWS BETWEEN 2 PRECEDING AND CURRENT ROW)         AS moving_avg3,
       LAG(sal)  OVER (PARTITION BY deptno ORDER BY hiredate)           AS prev_sal
FROM emp;

-- 4. Date functions
SELECT hiredate,
       TRUNC(SYSDATE) - TRUNC(hiredate)           AS days_employed,
       MONTHS_BETWEEN(SYSDATE, hiredate)/12       AS years_employed,
       ADD_MONTHS(hiredate, 6)                    AS probation_end,
       LAST_DAY(hiredate)                         AS month_end
FROM emp;
```
In (2) `RANK` leaves gaps after ties (1,1,3), `DENSE_RANK` does not (1,1,2), `ROW_NUMBER` breaks ties arbitrarily (1,2,3). In (3) a frame with `ROWS` counts physical rows; with the default `RANGE UNBOUNDED PRECEDING … CURRENT ROW` (what you get when you specify `ORDER BY` and no frame) rows with equal sort keys are *peers* and get the same running total, which surprises people.

### 5. Under the hood
- **Aggregation methods.** The plan shows `HASH GROUP BY` (hash table of groups, no sort; default since 10g) or `SORT GROUP BY` (sort-based, used when the output must be ordered or an index delivers order — `SORT GROUP BY NOSORT` with an index).
- **Analytic functions** appear as `WINDOW SORT` (or `WINDOW BUFFER`) steps; each distinct `PARTITION BY/ORDER BY` combination can require its own sort, so reuse the same window spec across expressions (or use the `WINDOW` clause in 21c+/standard syntax) to avoid repeated sorts. A `WINDOW SORT PUSHED RANK` step shows Oracle optimizing `rank <= n` filters to keep only n rows per partition.
- **`ROWNUM` evaluation.** Oracle increments ROWNUM only when a row passes the predicates at its own query level; a condition such as `ROWNUM > 1` can never be true, so it returns no rows. That is also why `WHERE ROWNUM = 1` is cheap: it produces a `COUNT STOPKEY` and stops fetching.
- **Histograms.** `DBMS_STATS.GATHER_TABLE_STATS … METHOD_OPT => 'FOR ALL COLUMNS SIZE AUTO'` creates histograms for columns that are both skewed and used in predicates. Types: *frequency* (≤ number of buckets distinct values), *top-frequency*, *hybrid* and *height-balanced* (pre-12c). `USER_TAB_HISTOGRAMS` shows them. They improve cardinality estimates for `WHERE status = 'X'` when 99 % of rows are `'DONE'`. Cost: extra parse work; and with bind variables the first peeked value can fix a bad plan for later values (see adaptive cursor sharing in the SQL and Indexes lesson).
- **COUNT(*) vs. COUNT(1)** are identical in Oracle; the optimizer may satisfy either from the smallest index that has no NULL-able gaps (a bitmap or a NOT NULL B-tree index).

### 6. Performance and design implications
- Push filters into `WHERE` rather than `HAVING` where they do not depend on aggregates: fewer rows enter grouping.
- `DISTINCT` and `GROUP BY` without aggregates yield the same result and normally the same `HASH UNIQUE`/`HASH GROUP BY` plan; use `GROUP BY` only when you aggregate. `DISTINCT` applies to the whole select list.
- Analytic functions replace self-joins and correlated subqueries (previous row, running total, top-N per group) with a single pass plus a sort.
- Pre-aggregate with a materialized view for dashboards that repeat the same `GROUP BY`; `QUERY REWRITE` can let the optimizer use it transparently.
- Date arithmetic on indexed columns: keep the column bare (`hiredate >= :d AND hiredate < :d + 1`) rather than `TRUNC(hiredate) = :d`, which disables a normal index (or create a function-based index).
- Check plans via `DBMS_XPLAN.DISPLAY_CURSOR(format=>'ALLSTATS LAST')` after running with `/*+ GATHER_PLAN_STATISTICS */`.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `GROUP BY ROLLUP / CUBE / GROUPING SETS` | Subtotals and grand totals in one query | Reports with hierarchy |
| `LISTAGG(col, ',') WITHIN GROUP (ORDER BY …)` | Concatenate group values | Replace string-aggregation hacks |
| `RANK / DENSE_RANK / ROW_NUMBER() OVER` | Ranking | Top-N, de-duplication |
| `LAG / LEAD` | Previous/next row value | Differences between consecutive rows |
| `FETCH FIRST n ROWS ONLY` (12c+) | Standard top-N | Replaces ROWNUM wrapper |
| `PIVOT / UNPIVOT` | Rotate rows ↔ columns | Cross-tab reports |
| `DBMS_STATS.GATHER_TABLE_STATS` | Refresh stats/histograms | After large loads |
| `NVL / COALESCE` | NULL substitution | Make NULL count as 0 in `AVG` |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| Aggregate + `GROUP BY` | You also need the detail rows → use analytic |
| Analytic function | A simple aggregate is enough (extra sort/buffer) |
| `ROW_NUMBER` | You want ties kept → `RANK`/`DENSE_RANK` |
| `RANGE` frame | Duplicate order keys would make "peers" jump the total unexpectedly → use `ROWS` |
| Histogram | Column is evenly distributed or heavily bind-peeked and stability matters more |

### 9. Common mistakes
- `ROWNUM <= 3 ORDER BY …` → random three rows sorted → sort in a subquery first or `FETCH FIRST`.
- `COUNT(col)` expecting row count → NULLs excluded → `COUNT(*)`.
- Selecting a non-grouped column → `ORA-00979: not a GROUP BY expression` → add to `GROUP BY` or aggregate.
- Putting an aggregate in `WHERE` → `ORA-00934` → use `HAVING`.
- `AVG` over a nullable column read as "average of all rows" → biased high → decide on `NVL` explicitly.
- `date1 - date2` formatted as a date → nonsense → it is a number of days.
- `UNION` where `UNION ALL` suffices → extra sort/distinct → use `UNION ALL`.

### 10. 🎯 Interview kit
**30-second answer:** "Aggregates like `COUNT`, `SUM`, `AVG` collapse each `GROUP BY` group to one row and ignore NULLs, with `HAVING` filtering groups. Analytic functions use `OVER (PARTITION BY … ORDER BY …)` to compute the same kind of values while keeping each row — for ranking, running totals, and `LAG/LEAD`."

**Structure a longer answer:** logical evaluation order → NULL rules → grouped example → analytic example with frame → pitfalls (ROWNUM, ties) → performance (hash group by, window sort).

**Likely follow-ups:**
- *`COUNT(*)` vs. `COUNT(col)` vs. `COUNT(DISTINCT col)`?* → rows / non-null / distinct non-null.
- *`DISTINCT` vs. `GROUP BY`?* → same dedup, `GROUP BY` allows aggregates and `HAVING`.
- *`ROWNUM` vs. `ROW_NUMBER()`?* → pseudocolumn assigned pre-sort vs. analytic over an ordered window.
- *`RANK` vs. `DENSE_RANK`?* → gaps vs. no gaps after ties.
- *What do histograms do?* → describe skew so cardinality estimates (and thus plans) are right.
- *Days between two dates?* → subtract; `TRUNC` for whole days; `MONTHS_BETWEEN` for months.

**Signals interviewers look for:** correct NULL semantics, the `ROWNUM` trap, ability to write top-N-per-group, and awareness of how the optimizer sees skew.

### 11. Check yourself
- [ ] Predict `COUNT(*)`, `COUNT(comm)`, `AVG(comm)` for a column with values 10, NULL, 30.
- [ ] Write top 2 salaries per department with and without ties handled.
- [ ] Explain why `WHERE ROWNUM > 1` returns nothing.
- [ ] Compute last day of the month six months after a hire date.
- [ ] Show how to pivot monthly totals into 12 columns.
- [ ] Explain when a histogram changes a plan.
<!-- explainer:end -->

## Implementations (tools)

- [Oracle](<../_index.md>)

## Practice questions

- [Basic] [What is the GROUP BY clause and how does it work with aggregate functions in Oracle?](<../../../PRACTICE/TOOLS/Oracle/Oracle_Platform/Q-35220_What_is_the_GROUP_BY_clause_and_how_does_it_work_with_aggreg.md>)
- [Basic] [What are the common aggregate functions in Oracle such as COUNT, SUM, AVG, MIN, and MAX?](<../../../PRACTICE/TOOLS/Oracle/Oracle_Platform/Q-35221_What_are_the_common_aggregate_functions_in_Oracle_such_as_CO.md>)
- [Intermediate] [How do you calculate the number of days between two dates in Oracle?](<../../../PRACTICE/TOOLS/Oracle/Oracle_Platform/Q-35216_How_do_you_calculate_the_number_of_days_between_two_dates_in.md>)
- [Intermediate] [What do the ADD_MONTHS, MONTHS_BETWEEN, and LAST_DAY date functions do in Oracle?](<../../../PRACTICE/TOOLS/Oracle/Oracle_Platform/Q-35217_What_do_the_ADD_MONTHS_MONTHS_BETWEEN_and_LAST_DAY_date_func.md>)
- [Intermediate] [What are the cursor attributes %FOUND, %NOTFOUND, %ROWCOUNT, and %ISOPEN in Oracle?](<../../../PRACTICE/TOOLS/Oracle/Oracle_Platform/Q-35265_What_are_the_cursor_attributes_FOUND_NOTFOUND_ROWCOUNT_and_I.md>)
- [Intermediate] [What is the difference between the DISTINCT and GROUP BY clauses for removing duplicates in Oracle?](<../../../PRACTICE/TOOLS/Oracle/Oracle_Platform/Q-35282_What_is_the_difference_between_the_DISTINCT_and_GROUP_BY_cla.md>)
- [Deep] [What is the difference between ROWNUM and ROW_NUMBER() in Oracle?](<../../../PRACTICE/TOOLS/Oracle/Oracle_Platform/Q-35152_What_is_the_difference_between_ROWNUM_and_ROW_NUMBER_in_Orac.md>)
- [Deep] [What is the difference between COUNT(*), COUNT(column), and COUNT(DISTINCT column) in Oracle?](<../../../PRACTICE/TOOLS/Oracle/Oracle_Platform/Q-35222_What_is_the_difference_between_COUNT_COUNT_column_and_COUNT_.md>)

## Related concepts

- [Window Functions Overview](<../../../01_Databases_and_SQL/SQL_Language/SQL_Functions/Window_Functions_Overview.md>) *(SQL Functions)*
- [UDF Performance](<../../../01_Databases_and_SQL/SQL_Language/SQL_Functions/UDF_Performance.md>) *(SQL Functions)*
- [GROUP BY vs PARTITION BY](<../../../01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/GROUP_BY_vs_PARTITION_BY.md>) *(SQL Fundamentals)*

