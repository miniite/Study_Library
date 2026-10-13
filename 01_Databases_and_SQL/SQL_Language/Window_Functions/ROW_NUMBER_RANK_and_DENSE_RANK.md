---
id: "01_Databases_and_SQL/SQL_Language/Window_Functions/ROW_NUMBER_RANK_and_DENSE_RANK"
level: 4
type: "concept"
topic: "01_Databases_and_SQL/SQL_Language/Window_Functions"
tools: []
---

# ROW_NUMBER RANK and DENSE_RANK

**L4 Advanced** · [↑ Window Functions](<_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** ROW_NUMBER, RANK and DENSE_RANK all number rows within an ordered partition; they differ only in how they treat ties.

### 1. The big picture
Ranking is the most common use of window functions: "latest row per key", "top 3 per department", "second-highest salary", pagination. Three functions cover it (plus NTILE for buckets). Choosing the right one is purely a question of **what you want to happen when two rows tie** on the ORDER BY value.

They arrived in SQL Server 2005 and require an `ORDER BY` inside OVER; they do not use frames. Together with PARTITION BY they are the building block for deduplication and top-N-per-group, the two lessons that follow.

### 2. Core concepts and vocabulary
- **ROW_NUMBER()** — unique, consecutive 1, 2, 3 … even for ties; ties are broken arbitrarily.
- **RANK()** — ties share a number; the next rank *skips* (1, 2, 2, 4).
- **DENSE_RANK()** — ties share a number; the next rank does *not* skip (1, 2, 2, 3).
- **NTILE(n)** — splits the partition into n roughly equal buckets, numbering the buckets.
- **Tie** — rows equal on all window ORDER BY columns.
- **Tiebreaker** — extra ORDER BY column(s) making rows unique.
- **Gap** — missing numbers after a tie, produced by RANK.

### 3. How it works, step by step
1. Partition rows by the PARTITION BY columns (or treat everything as one partition).
2. Sort each partition by the OVER ORDER BY.
3. Walk the sorted rows with a counter:
   - ROW_NUMBER: counter + 1 for every row.
   - RANK: value = 1 + number of rows strictly before this row's peer group.
   - DENSE_RANK: value = 1 + number of distinct order values before this one.
4. Counters restart at 1 at each partition boundary.

| salary | ROW_NUMBER | RANK | DENSE_RANK |
|---|---|---|---|
| 300 | 1 | 1 | 1 |
| 300 | 2 | 1 | 1 |
| 200 | 3 | 3 | 2 |
| 100 | 4 | 4 | 3 |

### 4. Worked example
```sql
SELECT empid, deptid, salary,
       ROW_NUMBER() OVER (PARTITION BY deptid ORDER BY salary DESC)          AS rn,
       RANK()       OVER (PARTITION BY deptid ORDER BY salary DESC)          AS rnk,
       DENSE_RANK() OVER (PARTITION BY deptid ORDER BY salary DESC)          AS drnk
FROM dbo.Employees;
```
Second-highest salary **per department** (value, not employee):
```sql
WITH r AS (
  SELECT deptid, salary,
         DENSE_RANK() OVER (PARTITION BY deptid ORDER BY salary DESC) AS d
  FROM dbo.Employees)
SELECT DISTINCT deptid, salary FROM r WHERE d = 2;
```
DENSE_RANK is right because two people at the top salary must not push the "second highest" away. With RANK the value for `d = 2` would not exist after a tie at the top; with ROW_NUMBER you might return the same salary again.

Pagination (pre-OFFSET/FETCH style):
```sql
WITH n AS (SELECT *, ROW_NUMBER() OVER (ORDER BY EmployeeID) AS rn FROM dbo.Employees)
SELECT * FROM n WHERE rn BETWEEN 11 AND 20;   -- page 2, page size 10
```

### 5. Under the hood
- A **Segment** operator marks partition boundaries; **Sequence Project (Compute Scalar)** assigns the number. Both are streaming once input is sorted, so cost ≈ the sort.
- RANK/DENSE_RANK additionally compare each row with the previous one on the ORDER BY columns to decide whether to advance.
- ROW_NUMBER with ties is **nondeterministic**: the survivor depends on sort algorithm, parallel degree and input order. This is documented behavior, not a bug.
- Window functions can be filtered only after evaluation, so the engine computes the number for *all* rows before `WHERE rn = 1` discards most. For top-1 per group, an index on `(partition, order)` plus `ROW_NUMBER` is usually fine; for few groups with huge partitions `CROSS APPLY (SELECT TOP 1 ...)` can be cheaper.

### 6. Performance and design implications
- Cost is dominated by the sort on `(partition cols, order cols)`: provide an index in that order with the needed columns INCLUDEd.
- Because of the sort, ranking a 100M-row table can spill to tempdb; pre-filter with WHERE where logic allows.
- ROW_NUMBER is slightly cheaper than RANK/DENSE_RANK (no tie comparison) — irrelevant except at huge scale.
- Pagination with ROW_NUMBER scans and numbers everything before the page; for deep pages prefer keyset pagination (`WHERE key > @last ORDER BY key`).

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `ROW_NUMBER()` | Unique sequence | Dedup, pagination, pick exactly one row |
| `RANK()` | Ties share, gaps | Competition-style ranking |
| `DENSE_RANK()` | Ties share, no gaps | "Nth highest value" |
| `NTILE(n)` | Bucket number | Quartiles, batch splitting |
| `ORDER BY a DESC, b` | Tiebreaker | Deterministic result |
| `TOP (n) WITH TIES` | Alternative to RANK=1 | Overall top n including ties |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| ROW_NUMBER: you need exactly one row per group | You must keep all tied rows |
| RANK: ranking where "3rd place" after a tie should be 4th | You need the Nth *distinct* value |
| DENSE_RANK: Nth highest distinct value | Gaps carry meaning (competition ranking) |
| NTILE: equal-size buckets | Bucket sizes must be exact — NTILE gives sizes differing by up to 1 |

### 9. Common mistakes
- Using ROW_NUMBER for "top 3 salaries" → ties at the cut-off arbitrarily dropped → use RANK or DENSE_RANK (decide on business rule).
- Non-unique ORDER BY with ROW_NUMBER → different row each run → add a unique tiebreaker.
- Using RANK for "second highest" → no row 2 after a tie → DENSE_RANK.
- Filtering the window function in WHERE → error → CTE/subquery.
- Forgetting PARTITION BY → one global ranking instead of per group.
- Assuming NTILE(4) on 10 rows gives 2.5 each → buckets of 3,3,2,2 (earlier buckets get the extra rows).

### 10. 🎯 Interview kit
**30-second answer:** All three number rows within an ordered partition. ROW_NUMBER gives unique numbers, RANK gives ties the same number and skips the next, DENSE_RANK gives ties the same number without skipping.

**Structure a longer answer:** clarify tie semantics → show the 300/300/200/100 table → pick function → tiebreaker → index/sort cost.

**Likely follow-ups:**
- *Nth highest salary?* → DENSE_RANK = N (distinct); or OFFSET/FETCH over DISTINCT.
- *ROW_NUMBER vs RANK for dedup?* → ROW_NUMBER, because RANK would keep ties.
- *Is ROW_NUMBER deterministic?* → only with a unique ORDER BY.
- *How to number rows without ordering?* → `ORDER BY (SELECT NULL)`, explicitly arbitrary.

**Signals interviewers look for:** you ask about ties, you add tiebreakers, you know which function to use for which question.

### 11. Check yourself
- [ ] Produce the table of ROW_NUMBER/RANK/DENSE_RANK for 300, 300, 200, 100 from memory.
- [ ] Write "third-highest salary per department" and test it with ties.
- [ ] Split 10 rows with NTILE(4) and list the bucket sizes.
- [ ] Explain what index would eliminate the sort for `PARTITION BY dept ORDER BY salary DESC`.
<!-- explainer:end -->

## Practice questions

- [Basic] [What is the FIRST_VALUE() function and when would you use it?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/Window_Functions/Q-44099_What_is_the_FIRST_VALUE_function_and_when_would_you_use_it.md>)
- [Basic] [Explain the output of RANK() OVER (ORDER BY score DESC) on a table with tied scores.](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/Window_Functions/Q-44400_Explain_the_output_of_RANK_OVER_ORDER_BY_score_DESC_on_a_tab.md>)
- [Basic] [Write a query to rank students by marks so that ties share the same rank and no numbers are skipped.](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/Window_Functions/Q-44163_Write_a_query_to_rank_students_by_marks_so_that_ties_share_t.md>)
- [Intermediate] [How would you remove duplicate rows keeping only the first occurrence using ROW_NUMBER()?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/Window_Functions/Q-44177_How_would_you_remove_duplicate_rows_keeping_only_the_first_o.md>)
- [Intermediate] [How do you use a window function result to update a table, for example ranking rows and updating a rank column?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/Window_Functions/Q-44384_How_do_you_use_a_window_function_result_to_update_a_table_fo.md>)
- [Intermediate] [How do you use ROW_NUMBER() to pick one representative row per duplicate group before deleting the rest?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/Window_Functions/Q-44438_How_do_you_use_ROW_NUMBER_to_pick_one_representative_row_per.md>)
- [Intermediate] [How would you assign a version number per key so the newest row gets version 1 (for SCD-style processing)?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/Window_Functions/Q-44215_How_would_you_assign_a_version_number_per_key_so_the_newest_.md>)
- [Intermediate] [Write a query using DENSE_RANK to build a leaderboard where tied scores share a rank and the next distinct score is one higher.](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/Window_Functions/Q-44252_Write_a_query_using_DENSE_RANK_to_build_a_leaderboard_where_.md>)

## Related concepts

- [Window Functions Overview](<../SQL_Functions/Window_Functions_Overview.md>) *(SQL Functions)*
- [Classic SQL Query Problems](<../../Database_Internals/DBMS_Concepts/Classic_SQL_Query_Problems.md>) *(DBMS Concepts)*
- [T-SQL Basics](<../../../TOOLS/SQL_Server/T-SQL/T-SQL_Basics.md>) *(T-SQL)*

## References

- *Expert T-SQL Window Functions in SQL Server* — Kathi Kellenberger, Clayton Groom, Ed Pollack — Uncovering Special Case Windows
- *Expert T-SQL Window Functions in SQL Server* — Kathi Kellenberger, Clayton Groom, Ed Pollack — Using ROWNUMBER
- *Expert T-SQL Window Functions in SQL Server* — Kathi Kellenberger, Clayton Groom, Ed Pollack — Understanding RANK and DENSERANK
- *Expert T-SQL Window Functions in SQL Server* — Kathi Kellenberger, Clayton Groom, Ed Pollack — Solving Queries with Ranking Functions
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 3: SELECT Statements and Filtering Logic › Pagination with ROWNUMBER()
- *Beginning T-SQL* — Kathi Kellenberger, Lee Everest — Administrative Functions › The TOP Keyword
- *Beginning T-SQL* — Kathi Kellenberger, Lee Everest — Discovering Windowing Functions › Ranking Functions and the OVER Clause
- *T-SQL Fundamentals (4th ed.)* — Itzik Ben-Gan — Set operators
- *SQL Server Tutorial* (YouTube series) — Part 109: Row Number function in SQL Server ([watch](https://www.youtube.com/watch?v=cvrwOoGwgz8))
- *SQL Server Tutorial* (YouTube series) — Part 110: Rank and Dense Rank in SQL Server ([watch](https://www.youtube.com/watch?v=5-La_uSNkKU))
- *SQL Server Tutorial* (YouTube series) — Part 111: Difference between rank dense rank and row number in SQL ([watch](https://www.youtube.com/watch?v=MZTSHDFuCUk))

