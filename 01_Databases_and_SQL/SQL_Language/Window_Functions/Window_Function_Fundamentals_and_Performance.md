---
id: "01_Databases_and_SQL/SQL_Language/Window_Functions/Window_Function_Fundamentals_and_Performance"
level: 4
type: "concept"
topic: "01_Databases_and_SQL/SQL_Language/Window_Functions"
tools: []
---

# Window Function Fundamentals and Performance

**L4 Advanced** · [↑ Window Functions](<_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** A window function computes a value for every row by looking at a set of related rows (its window), without collapsing the rows the way GROUP BY does.

### 1. The big picture
Plain aggregates answer "what is the total per customer?" but throw the detail rows away. Before window functions, if you wanted each order *and* the customer's total next to it, you needed a self-join or a correlated subquery that re-read the table. Window functions (SQL:2003, in SQL Server since 2005 for ranking and 2012 for full frames) solve this: the aggregate is computed over a *window* of rows and attached to each row as one more column.

Everything about a window function is controlled by the `OVER` clause. There is no window function without `OVER`. This lesson is the foundation for the whole Window Functions topic: ranking (ROW_NUMBER/RANK), offset (LAG/LEAD), framing (running totals, moving averages) and the patterns built from them (dedup, top-N per group, sessionization).

### 2. Core concepts and vocabulary
- **Window** — the set of rows a function sees for the current row. Different rows can have different windows.
- **OVER clause** — defines the window. Up to three parts: `PARTITION BY`, `ORDER BY`, and a frame.
- **PARTITION BY** — splits the result set into independent groups; the function restarts in each. Omitted = the whole result set is one partition.
- **Window ORDER BY** — orders rows *inside* the partition. It gives meaning to rank, to LAG/LEAD, and to running calculations. It is **not** presentation ordering of the final result.
- **Frame** — a sub-range of the ordered partition, e.g. `ROWS BETWEEN 2 PRECEDING AND CURRENT ROW`. Only certain functions use it (aggregates, FIRST_VALUE/LAST_VALUE).
- **Ranking functions** — ROW_NUMBER, RANK, DENSE_RANK, NTILE. Require ORDER BY, ignore frames.
- **Offset functions** — LAG, LEAD (fetch another row), FIRST_VALUE, LAST_VALUE.
- **Window aggregate** — SUM/AVG/MIN/MAX/COUNT used with OVER.
- **Table expression** — a CTE, derived table or view, needed to filter on a window function's result.

### 3. How it works, step by step
1. The engine runs FROM, WHERE, GROUP BY and HAVING first, producing the "underlying result set".
2. In the SELECT phase it takes that set, splits it by PARTITION BY and sorts each partition by the window ORDER BY.
3. For each row it determines the frame (default or explicit), evaluates the function over it, and appends the value as a column.
4. Only then do DISTINCT and the final ORDER BY run.

Consequence of step 1–4: window functions are legal only in **SELECT** and **ORDER BY**. You cannot write `WHERE ROW_NUMBER() OVER (...) = 1`; you wrap the query in a CTE and filter outside.

| Query shape | Rows returned | Detail columns allowed |
|---|---|---|
| `SELECT dept, SUM(sal) ... GROUP BY dept` | one per group | only grouped columns |
| `SELECT *, SUM(sal) OVER (PARTITION BY dept)` | one per input row | all columns |

A window aggregate can even sit on top of an aggregate query: `SUM(COUNT(*)) OVER (ORDER BY m)` first groups, then runs the window over the grouped rows.

### 4. Worked example
```sql
CREATE TABLE emp (id INT PRIMARY KEY, dept VARCHAR(10), sal INT);
INSERT INTO emp VALUES (1,'A',100),(2,'A',300),(3,'A',300),(4,'B',200),(5,'B',50);

SELECT id, dept, sal,
       SUM(sal) OVER ()                         AS company_total,
       SUM(sal) OVER (PARTITION BY dept)        AS dept_total,
       sal * 100.0 / SUM(sal) OVER (PARTITION BY dept) AS pct_of_dept,
       AVG(sal) OVER (PARTITION BY dept)        AS dept_avg,
       sal - AVG(sal) OVER (PARTITION BY dept)  AS diff_from_avg,
       ROW_NUMBER() OVER (PARTITION BY dept ORDER BY sal DESC, id) AS rn
FROM emp
ORDER BY dept, rn;
```
All five rows come back. `company_total` is 950 on every row (empty OVER = one big window). `dept_total` is 700 for A and 250 for B. `rn` restarts at 1 in each department. With GROUP BY you would get two rows and lose the employees.

### 5. Under the hood
- A window function is typically implemented by a **Segment** operator (detects partition boundaries) and a **Sequence Project** (ranking) or **Window Spool** (framed aggregates) in SQL Server plans. Input must arrive sorted by (partition columns, order columns).
- That sort is the main cost. If an index already provides `(PARTITION BY cols, ORDER BY cols)` order, the **Sort** operator disappears. Design an index with partition columns first, order columns next, then INCLUDE the columns the query returns.
- Framed aggregates with `RANGE` (the default when ORDER BY is present and no frame is written) use an on-disk spool in SQL Server; `ROWS` uses an in-memory fast path. This is why `ROWS` is dramatically faster for running totals.
- Several functions with the *same* OVER specification share one sort and one pass; different specifications each need their own sort.

### 6. Performance and design implications
- Compare plans: look for `Sort` operators, `Window Spool` with "Actual Rows" much larger than input, and spills to tempdb (warning icon on Sort).
- Prefer one pass with identical windows; use the `WINDOW` clause (SQL Server 2022+, PostgreSQL, MySQL 8) to name a shared specification.
- Filtering before the window (WHERE) shrinks the input — but changes the answer: the window only sees surviving rows.
- A window function replaces a correlated subquery that is O(n²) by an O(n log n) sort-and-scan. Typically 10–100× faster on large tables.
- Wide partitions (one customer with millions of rows) concentrate work; skew in distributed engines (Spark, Snowflake) shows up as one slow task.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `OVER ()` | Whole result set is one window | Grand total beside each row |
| `PARTITION BY` | Restart per group | Per-customer / per-day calculations |
| `ORDER BY` in OVER | Orders rows inside partition | Rank, LAG, running total |
| `ROWS BETWEEN ...` | Physical row frame | Running and moving aggregates |
| `RANGE BETWEEN ...` | Logical value frame (peers) | Rare; ties share a frame |
| `WINDOW w AS (...)` | Name a spec | Same window used by many functions |
| `SET STATISTICS IO ON` | Shows spool reads (Worktable) | Detect RANGE spool cost |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| You need detail rows *and* group/ordered context | You only need group totals — plain GROUP BY is simpler |
| Replacing self-joins/subqueries | The partition is enormous and not indexed — big sort |
| One pass over the data | Result needs filtering on the window value — must wrap in CTE |

### 9. Common mistakes
- Putting a window function in WHERE/GROUP BY/HAVING → error "Windowed functions can only appear in SELECT or ORDER BY" → compute in a CTE then filter.
- Forgetting that window ORDER BY is not output ordering → rows return in arbitrary order → add an outer ORDER BY.
- Relying on the default frame with ORDER BY (`RANGE UNBOUNDED PRECEDING`) → slow, and ties give identical running totals → write `ROWS` explicitly.
- Assuming window functions see rows removed by WHERE *after* the fact → the window is built from post-WHERE rows; pre-filter carefully.
- Non-unique ORDER BY → nondeterministic rank/ROW_NUMBER → add a tiebreaker.

### 10. 🎯 Interview kit
**30-second answer:** A window function does a calculation across a set of rows related to the current row — defined by OVER with optional PARTITION BY, ORDER BY and frame — and returns one value per input row, so unlike GROUP BY it does not collapse rows.

**Structure a longer answer:** clarify (what do we need beside each row?) → core mechanism (partition, order, frame, evaluated after WHERE/GROUP BY, only in SELECT/ORDER BY) → trade-offs (sort cost, index support, ROWS vs RANGE) → scenario (top earner per department, running balance).

**Likely follow-ups:**
- *Why can't I use a window function in WHERE?* → logical processing order: windows are evaluated at SELECT; use a CTE/subquery.
- *Difference between SUM() OVER () and GROUP BY?* → row count preserved vs reduced.
- *What does omitting PARTITION BY do?* → the whole result set is one partition.
- *How would you make it faster?* → supporting index (partition cols, order cols), ROWS frames, fewer distinct window specs.

**Signals interviewers look for:** you know the logical processing order, you use `ROWS` deliberately, you think about the index and the sort, and you handle ties.

### 11. Check yourself
- [ ] Explain the three parts of OVER and what happens when each is omitted.
- [ ] Rewrite a "salary vs department average" query with a self-join and with a window function; compare plans.
- [ ] Show why `WHERE rn = 1` needs a CTE.
- [ ] Find the Sort and Window Spool operators in a plan and say what index would remove the sort.
<!-- explainer:end -->

## Practice questions

- [Basic] [Can a single query mix a GROUP BY aggregate and a window function together? Explain what each operates on.](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/Window_Functions/Q-44378_Can_a_single_query_mix_a_GROUP_BY_aggregate_and_a_window_fun.md>)
- [Intermediate] [Since which version did MySQL support window functions, and how did people emulate them before?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/Window_Functions/Q-44424_Since_which_version_did_MySQL_support_window_functions_and_h.md>)
- [Intermediate] [Explain how Snowflake's QUALIFY clause is compiled and why it can avoid an extra subquery scan compared to filtering a window result in an outer WHERE.](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/Window_Functions/Q-44274_Explain_how_Snowflake_s_QUALIFY_clause_is_compiled_and_why_i.md>)
- [Deep] [How can you compute a rank that is reused across a query using a named window?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/Window_Functions/Q-44409_How_can_you_compute_a_rank_that_is_reused_across_a_query_usi.md>)
- [Deep] [Explain why LAST_VALUE with the default frame gives the current row and walk through the exact frame you'd set to get the true partition last value.](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/Window_Functions/Q-44084_Explain_why_LAST_VALUE_with_the_default_frame_gives_the_curr.md>)
- [Deep] [In Spark SQL a window without PARTITION BY forces all data into a single partition; how do you diagnose the resulting single-task skew from the Spark UI and what redesign avoids it?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/Window_Functions/Q-44267_In_Spark_SQL_a_window_without_PARTITION_BY_forces_all_data_i.md>)
- [Deep] [How do you decide between materializing a window-computed metric into a summary table versus recomputing it on each query for a metric read thousands of times per minute?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/Window_Functions/Q-44336_How_do_you_decide_between_materializing_a_window-computed_me.md>)
- [Expert] [Explain how a window function's frame boundary of 'GROUPS' differs from ROWS and RANGE, and give a case where GROUPS is the right choice.](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/Window_Functions/Q-44104_Explain_how_a_window_function_s_frame_boundary_of_GROUPS_dif.md>)

## Related concepts

- [UDF Performance](<../SQL_Functions/UDF_Performance.md>) *(SQL Functions)*
- [Filtered Indexes and Plan Cache](<../../../TOOLS/SQL_Server/T-SQL/Filtered_Indexes_and_Plan_Cache.md>) *(T-SQL)*
- [Window Functions Overview](<../SQL_Functions/Window_Functions_Overview.md>) *(SQL Functions)*
- [GROUP BY vs PARTITION BY](<../SQL_Fundamentals/GROUP_BY_vs_PARTITION_BY.md>) *(SQL Fundamentals)*
- [T-SQL Basics](<../../../TOOLS/SQL_Server/T-SQL/T-SQL_Basics.md>) *(T-SQL)*
- [Partitioning and Data Skipping in Lakes](<../../Query_Performance/Indexing/Partitioning_and_Data_Skipping_in_Lakes.md>) *(Indexing)*

## References

- *Expert T-SQL Window Functions in SQL Server* — Kathi Kellenberger, Clayton Groom, Ed Pollack — Discovering Window Functions
- *Expert T-SQL Window Functions in SQL Server* — Kathi Kellenberger, Clayton Groom, Ed Pollack — Thinking About the Window
- *Expert T-SQL Window Functions in SQL Server* — Kathi Kellenberger, Clayton Groom, Ed Pollack — Dividing Windows with Partitions
- *Expert T-SQL Window Functions in SQL Server* — Kathi Kellenberger, Clayton Groom, Ed Pollack — Creating Custom Window Aggregate Functions
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 3: SELECT Statements and Filtering Logic › Window Functions
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 3: SELECT Statements and Filtering Logic › Categories of Window Functions
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 3: SELECT Statements and Filtering Logic › Window Frames
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 3: SELECT Statements and Filtering Logic › Best Practices
- *Mapping Data Flows in Azure Data Factory* — Mark Kromer — Introduction to Mapping Data Flows › Window
- *Mapping Data Flows in Azure Data Factory* — Mark Kromer — Monitor, Manage, and Optimize › Rank and Surrogate Key
- *Beginning T-SQL* — Kathi Kellenberger, Lee Everest — Discovering Windowing Functions › What Is a Windowing Function?
- *Beginning T-SQL* — Kathi Kellenberger, Lee Everest — Discovering Windowing Functions › Dividing the Window into Partitions
- *Beginning T-SQL* — Kathi Kellenberger, Lee Everest — Discovering Windowing Functions › Defining the Window with Framing
- *Beginning T-SQL* — Kathi Kellenberger, Lee Everest — Discovering Windowing Functions › Understanding the Difference Between ROWS and RANGE
- *Learning Spark (2nd ed.)* — Jules S. Damji, Brooke Wenig, Tathagata Das, Denny Lee — Spark SQL and DataFrames Interacting with External Data Sources › Windowing
- *Databricks Certified Associate Developer for Apache Spark* — Saba Shah — Advanced Spark SQL operations › Understanding window functions

