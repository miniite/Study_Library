---
id: "01_Databases_and_SQL/SQL_Language/Window_Functions/Ordering_Inside_and_Outside_OVER"
level: 4
type: "concept"
topic: "01_Databases_and_SQL/SQL_Language/Window_Functions"
tools: []
---

# Ordering Inside and Outside OVER

**L4 Advanced** · [↑ Window Functions](<_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** The ORDER BY inside OVER decides how the window function sees rows; the ORDER BY at the end of the query decides how the result is presented — they are independent and you usually need both.

### 1. The big picture
New SQL writers see two ORDER BY clauses and assume one implies the other. It doesn't. The ORDER BY **inside OVER** is part of the calculation: it defines "first", "previous", "running" and "rank". The ORDER BY **after FROM/WHERE** only sorts the finished rows for the client. A window function can order by one thing while the result is displayed in another order — and the engine gives no guarantee that the output is sorted by the window ordering unless you ask.

Understanding this removes the two classic bugs: "my running total is right but rows are shuffled" and "my rank changed between runs".

### 2. Core concepts and vocabulary
- **Window ordering** — ORDER BY inside OVER; defines sequence within each partition.
- **Presentation ordering** — the query's final ORDER BY; the only thing that guarantees output order.
- **Order-sensitive functions** — ROW_NUMBER, RANK, DENSE_RANK, NTILE, LAG, LEAD, FIRST_VALUE, LAST_VALUE, and aggregates with a frame. ORDER BY is *required* (ranking, LAG/LEAD) or changes the meaning (aggregates).
- **Order-insensitive uses** — `SUM(x) OVER (PARTITION BY k)`: no ORDER BY, window = whole partition.
- **Tie / peer** — rows equal on all window ORDER BY columns.
- **Tiebreaker** — extra column(s) that make the ordering unique.
- **Default frame** — with ORDER BY present and no frame: `RANGE BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW`; without ORDER BY: whole partition.

### 3. How it works, step by step
1. Rows are grouped by PARTITION BY.
2. Within each partition they are sorted by the window ORDER BY (ASC/DESC, NULL placement per engine).
3. The function is evaluated against that sorted sequence. ROW_NUMBER numbers it; LAG looks one row up; SUM with default frame adds from the start to the current row *and its peers*.
4. The rows are then handed on. Their physical order is whatever is convenient — often the window order, but **not promised**.
5. Only the final ORDER BY sets the output order.

Different functions can use different window orderings in the same SELECT:
```sql
SELECT orderid, custid, orderdate, val,
       ROW_NUMBER() OVER (PARTITION BY custid ORDER BY orderdate, orderid) AS seq_in_cust,
       RANK()       OVER (ORDER BY val DESC)                             AS value_rank
FROM Sales.Orders
ORDER BY custid, seq_in_cust;
```

### 4. Worked example
```sql
-- table: t(id, grp, d, v)  rows: (1,'x','2024-01-01',10),(2,'x','2024-01-01',20),(3,'x','2024-01-02',5)
SELECT id, d, v,
  SUM(v) OVER (PARTITION BY grp)                                    AS part_total,   -- no ORDER BY: 35 everywhere
  SUM(v) OVER (PARTITION BY grp ORDER BY d)                         AS run_default,  -- RANGE: 30, 30, 35
  SUM(v) OVER (PARTITION BY grp ORDER BY d, id ROWS UNBOUNDED PRECEDING) AS run_rows -- 10, 30, 35
FROM t
ORDER BY d, id;
```
Rows 1 and 2 are peers on `d`, so the default RANGE frame includes both for each: 30 and 30, not 10 and 30. Adding a unique tiebreaker (`id`) plus `ROWS` gives the "true" running total. Without the final `ORDER BY` the three result rows might come back in any order even though the running totals are correct.

### 5. Under the hood
- The optimizer plans a Sort (or uses an index that delivers order) on `(partition cols, order cols)`. Because the final ORDER BY is a separate requirement, a query ordering the output by the same columns reuses that sort; one ordering differently adds a **second** sort. Matching the two avoids it.
- ROW_NUMBER with a non-unique ORDER BY is legal: ties are broken by whatever order the sort emits, which can vary with parallelism, plan shape, statistics or data movement. The same query can legitimately return a different winner after an index rebuild.
- `ORDER BY (SELECT NULL)` in OVER is the T-SQL idiom for "number rows, I don't care in what order"; the result is explicitly arbitrary.
- NULL ordering differs: SQL Server sorts NULLs first in ASC; PostgreSQL/Oracle put them last (and support `NULLS FIRST/LAST`). That changes LAG/LEAD neighbours and ranks.

### 6. Performance and design implications
- Index `(partition_cols, order_cols) INCLUDE (needed)` lets the plan stream without sorting; in SQL Server the index key direction should match ASC/DESC (mixed directions need a matching index definition).
- If many window functions use different orderings you pay one sort each; group functions by identical OVER specification.
- Don't add a window ORDER BY "just in case": ordering an aggregate turns a cheap whole-partition total into a running calculation with a frame.
- Verify with `SET STATISTICS IO ON` that no `Worktable` reads appear (sign of a RANGE spool).

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `ORDER BY col [, tiebreaker]` in OVER | Defines window sequence | Any ranking / offset / running calc |
| Final `ORDER BY` | Defines output order | Always, if order matters to the consumer |
| `ROWS` vs `RANGE` | Physical vs peer-based frame | Running totals: use ROWS |
| `NULLS FIRST / LAST` | Control NULL position (not in T-SQL) | Portable ranking with NULLs |
| `ORDER BY (SELECT NULL)` | Arbitrary numbering | Surrogate numbering, no order meaning |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| Same ordering for window and output | Window order and output order differ — second sort |
| Add unique tiebreaker | You truly want ties to share a rank — use RANK/DENSE_RANK |
| Omit window ORDER BY for partition totals | Function requires it (ranking, LAG) — error or meaningless result |

### 9. Common mistakes
- Assuming output is sorted because OVER has ORDER BY → client sees random order → add a final ORDER BY.
- `ROW_NUMBER() OVER (PARTITION BY k)` (no ORDER BY) → error in SQL Server, arbitrary in others → always supply an order.
- ORDER BY on a non-unique column → nondeterministic row picked/numbered → add a unique tiebreaker.
- Reusing the default frame for running totals → peers get the same total, slow spool → `ROWS UNBOUNDED PRECEDING`.
- Ordering by a column alias in OVER → not allowed; window ORDER BY is evaluated in SELECT, but aliases from the same SELECT list are not visible → repeat the expression.

### 10. 🎯 Interview kit
**30-second answer:** The ORDER BY inside OVER orders rows within each partition for the calculation — rank, previous row, running total. The query's ORDER BY only controls presentation. They're independent, so you can sort a ranking descending and display by customer.

**Structure a longer answer:** clarify → mechanism (partition → window sort → evaluate → presentation sort) → trade-offs (second sort, ties, frames) → scenario (running balance with duplicate timestamps).

**Likely follow-ups:**
- *What happens without ORDER BY in ROW_NUMBER?* → error in SQL Server; arbitrary numbering elsewhere.
- *Does window ORDER BY sort the output?* → no, not guaranteed.
- *Why is my running total repeating on ties?* → default RANGE frame includes peers.
- *How do NULLs affect it?* → placement differs by engine; use NULLS LAST or COALESCE.

**Signals interviewers look for:** distinguishing the two orderings, spontaneously adding tiebreakers, naming the RANGE default pitfall.

### 11. Check yourself
- [ ] Write one query where the window order and output order differ and explain why both are needed.
- [ ] Demonstrate the peer behaviour of the default frame with duplicate order values.
- [ ] Explain why ROW_NUMBER over a non-unique key can change between runs.
- [ ] Predict where NULLs land for ASC in SQL Server vs PostgreSQL.
<!-- explainer:end -->

## Practice questions

- [Basic] [What does the OVER clause do in a window function and why is it required?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/Window_Functions/Q-44154_What_does_the_OVER_clause_do_in_a_window_function_and_why_is.md>)
- [Basic] [What does the ORDER BY inside an OVER clause control?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/Window_Functions/Q-44421_What_does_the_ORDER_BY_inside_an_OVER_clause_control.md>)
- [Basic] [What is the difference between putting SUM() OVER () and just SUM() with a GROUP BY in terms of how many rows you get back?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/Window_Functions/Q-44169_What_is_the_difference_between_putting_SUM_OVER_and_just_SUM.md>)
- [Basic] [What happens if you forget ORDER BY inside OVER when using ROW_NUMBER()?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/Window_Functions/Q-44179_What_happens_if_you_forget_ORDER_BY_inside_OVER_when_using_R.md>)
- [Intermediate] [What happens if you use a window aggregate like AVG() OVER () with no PARTITION BY and no ORDER BY?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/Window_Functions/Q-44379_What_happens_if_you_use_a_window_aggregate_like_AVG_OVER_wit.md>)
- [Intermediate] [What is the difference between SUM() OVER (ORDER BY x) and SUM() OVER (PARTITION BY x)?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/Window_Functions/Q-44408_What_is_the_difference_between_SUM_OVER_ORDER_BY_x_and_SUM_O.md>)
- [Intermediate] [Explain the difference between COUNT(*) OVER (ORDER BY dt) and COUNT(*) OVER () with a small example.](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/Window_Functions/Q-44192_Explain_the_difference_between_COUNT_OVER_ORDER_BY_dt_and_CO.md>)
- [Intermediate] [How would you use a named WINDOW clause to avoid repeating the same OVER definition across three window columns?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/Window_Functions/Q-44257_How_would_you_use_a_named_WINDOW_clause_to_avoid_repeating_t.md>)

## Related concepts

- [Window Functions Overview](<../SQL_Functions/Window_Functions_Overview.md>) *(SQL Functions)*
- [UDF Performance](<../SQL_Functions/UDF_Performance.md>) *(SQL Functions)*
- [GROUP BY vs PARTITION BY](<../SQL_Fundamentals/GROUP_BY_vs_PARTITION_BY.md>) *(SQL Fundamentals)*

## References

- *Expert T-SQL Window Functions in SQL Server* — Kathi Kellenberger, Clayton Groom, Ed Pollack — Understanding the OVER Clause
- *Expert T-SQL Window Functions in SQL Server* — Kathi Kellenberger, Clayton Groom, Ed Pollack — Using Window Aggregates
- *Expert T-SQL Window Functions in SQL Server* — Kathi Kellenberger, Clayton Groom, Ed Pollack — Adding ORDER BY to Window Aggregates
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 3: SELECT Statements and Filtering Logic › Window Function
- *T-SQL Fundamentals (4th ed.)* — Itzik Ben-Gan — T-SQL for data analysis
- *SQL Server Tutorial* (YouTube series) — Part 108: Over clause in SQL Server ([watch](https://www.youtube.com/watch?v=KwEjkpFltjc))
- *SQL Server Tutorial* (YouTube series) — Part 116: Window functions in SQL Server ([watch](https://www.youtube.com/watch?v=TzsrO4zTQj8))
- *SQL Server Tutorial* (YouTube series) — Part 118: LAST VALUE function in SQL Server ([watch](https://www.youtube.com/watch?v=zulLKbTK_j0))

