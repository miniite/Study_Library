---
id: "01_Databases_and_SQL/Database_Objects/Cursors/Cursors_vs_Set-based_SQL"
level: 4
type: "concept"
topic: "01_Databases_and_SQL/Database_Objects/Cursors"
tools: ["SQL_Server"]
---

# Cursors vs Set-based SQL

**L4 Advanced** · [↑ Cursors](<_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** SQL engines are built to optimise whole-set statements, so replacing a cursor loop with a single `UPDATE … JOIN`, `INSERT … SELECT`, window function or `STRING_AGG` lets the optimizer choose scans, joins and parallelism once instead of paying statement overhead per row.

### 1. The big picture
The core idea: you describe *what* result you want and let the engine decide *how*. A cursor inverts that — you take over the "how", row by row, and the engine can't optimise across iterations. Jeff Moden's term for this is **RBAR** — "row by agonizing row". Each iteration carries the full overhead of running a stand-alone statement: plan lookup, lock acquisition, a log record, possibly a round trip. A loop over a million rows is a million statements.

"Set-based thinking" means expressing the logic as relational operations over entire sets: joins, aggregates, window functions, `CASE`, `MERGE`, recursive CTEs. Nearly every cursor in a typical application is a set operation in disguise. The skill covered here is *recognising the pattern* a cursor implements (running total, per-group top-N, concatenation, hierarchy walk, conditional update, per-parent summary) and *translating* it while proving the output is identical.

### 2. Core concepts and vocabulary
- **Set-based** — one statement operates on all qualifying rows; the optimizer picks the algorithm.
- **Procedural / RBAR** — logic expressed as a loop acting on one row at a time.
- **Window function** — aggregate/ranking computed over a frame of rows without collapsing them (`SUM() OVER (PARTITION BY … ORDER BY …)`, `ROW_NUMBER`, `LAG`/`LEAD`).
- **Running total / running balance** — cumulative sum per ordered partition.
- **`STRING_AGG` / `FOR XML PATH` / `GROUP_CONCAT` / `LISTAGG`** — set-based string concatenation.
- **Recursive CTE** — query that references itself to walk hierarchies or generate sequences.
- **`UPDATE … FROM … JOIN`**, **`MERGE`**, **`INSERT … SELECT`** — set-based DML forms.
- **`CROSS APPLY` / `LATERAL`** — per-row subquery inside a set operation (the set-based home for "top N per group").
- **Parity test** — comparing row counts and checksums between old and new implementation before cutover.

### 3. How it works, step by step
**Why one statement beats a loop**
1. *Plan once.* The statement is compiled once; a loop re-executes a cached plan N times (cheap but not free) plus per-statement setup.
2. *Choose algorithms by volume.* For millions of rows the optimizer can use a hash join and a single scan; a loop is stuck with N index seeks.
3. *Batch the I/O and log.* Many rows are modified within one operation, with fewer log flushes and better page locality.
4. *Parallelise.* A set statement may run on multiple cores; a loop cannot.

**Translating a cursor — recipe**
1. Read the loop body and write down, in words, what it does to each row: inputs, conditions, outputs.
2. Identify state carried between iterations (a running counter, previous value, accumulated string, hierarchy parent). State carried across rows ⇒ window function, aggregate or recursive CTE.
3. Replace the per-row lookups with joins; replace `IF` with `CASE`.
4. Write the set query, producing the same columns and ordering.
5. Validate: compare row counts, aggregate checksums (`SUM`, `CHECKSUM_AGG`, hashes) and `EXCEPT` both ways on a copy of production data.
6. Compare plans and runtimes; then cut over.

| Cursor pattern | Set-based replacement |
|---|---|
| Update each row from computed value | `UPDATE t SET col = expr` (with `CASE`) |
| Update parent from child rows | `UPDATE p SET x = c.agg FROM p JOIN (SELECT parent_id, SUM(...) agg FROM child GROUP BY parent_id) c ON …` |
| Running balance | `SUM(amount) OVER (PARTITION BY acct ORDER BY ts ROWS UNBOUNDED PRECEDING)` |
| Compare with previous row | `LAG()` / `LEAD()` |
| Concatenate values | `STRING_AGG(col, ',') WITHIN GROUP (ORDER BY …)` |
| Top-N per group | `ROW_NUMBER() OVER (PARTITION BY g ORDER BY …) <= N` or `CROSS APPLY (SELECT TOP (N) …)` |
| De-duplicate keep newest | `DELETE FROM (SELECT ROW_NUMBER() OVER (PARTITION BY key ORDER BY ts DESC) rn …) WHERE rn > 1` |
| Hierarchy traversal | Recursive CTE |
| Pivot rows to columns | `PIVOT` / conditional aggregation |
| Nested cursors (orders → lines) | One join + `GROUP BY` / window |

### 4. Worked example
Problem: per account, compute a running balance and flag transactions that push it negative.

```sql
-- Cursor version (RBAR)
DECLARE @acct int, @prev int = -1, @bal money = 0, @id int, @amt money;
DECLARE c CURSOR LOCAL FAST_FORWARD FOR
    SELECT TxID, AccountID, Amount FROM dbo.Tx ORDER BY AccountID, TxDate, TxID;
OPEN c; FETCH NEXT FROM c INTO @id, @acct, @amt;
WHILE @@FETCH_STATUS = 0
BEGIN
    IF @acct <> @prev BEGIN SET @bal = 0; SET @prev = @acct; END
    SET @bal += @amt;
    UPDATE dbo.Tx SET RunningBalance = @bal, Overdrawn = CASE WHEN @bal < 0 THEN 1 ELSE 0 END
    WHERE TxID = @id;
    FETCH NEXT FROM c INTO @id, @acct, @amt;
END
CLOSE c; DEALLOCATE c;

-- Set-based version
WITH r AS (
  SELECT TxID,
         SUM(Amount) OVER (PARTITION BY AccountID ORDER BY TxDate, TxID
                           ROWS UNBOUNDED PRECEDING) AS Bal
  FROM dbo.Tx
)
UPDATE t
SET    RunningBalance = r.Bal,
       Overdrawn      = CASE WHEN r.Bal < 0 THEN 1 ELSE 0 END
FROM   dbo.Tx t JOIN r ON r.TxID = t.TxID;
```
Note the explicit `ROWS UNBOUNDED PRECEDING`: the default frame is `RANGE`, which treats ties on the ORDER BY value as peers and uses an on-disk worktable in SQL Server; `ROWS` is both faster and gives exactly the cursor's row-by-row semantics. Parity check:

```sql
-- same rows, same numbers?
SELECT COUNT(*), SUM(RunningBalance), SUM(CAST(Overdrawn AS int)) FROM dbo.Tx;      -- new
SELECT * FROM dbo.Tx_old EXCEPT SELECT * FROM dbo.Tx;                               -- should be empty
SELECT * FROM dbo.Tx EXCEPT SELECT * FROM dbo.Tx_old;                               -- should be empty
```
Typical result on a few million rows: minutes → seconds; the plan shows one scan, one `Segment`/`Window Spool`, one `Update`.

### 5. Under the hood
- **Why the loop is slow.** In SQL Server, `FETCH` plus the body is executed through the same machinery as an ad-hoc batch step: the optimizer's cached plan is looked up, locks are taken and released, and a log record is written *per row*. With default cursor options the cursor can also materialise a tempdb worktable. In Oracle each SQL statement in a PL/SQL loop is a SQL-to-PL/SQL context switch.
- **Windowed operators.** A window aggregate sorts (or uses an index in the right order) once, then streams through computing the aggregate incrementally — O(n) after the sort. Without a suitable index the sort may spill; an index on `(AccountID, TxDate, TxID) INCLUDE (Amount)` eliminates it.
- **String aggregation.** `STRING_AGG` is a streaming aggregate; the older `FOR XML PATH('')` trick does a correlated subquery per group. Engines differ in limits: SQL Server `STRING_AGG` (2017+) returns `varchar(8000)` unless the input is `varchar(max)`; MySQL `GROUP_CONCAT` is truncated by `group_concat_max_len` (default 1024); PostgreSQL `string_agg` has no practical limit; Oracle `LISTAGG` errors on overflow past 4000 bytes unless `ON OVERFLOW TRUNCATE`.
- **Recursive CTEs** iterate level by level (anchor, then recursive member joined to the previous level). Risks: cycles in the data (add a depth guard / cycle detection), SQL Server's default `MAXRECURSION` of 100, and exponential growth in poorly bounded graphs.
- **Old MySQL.** Before MySQL 8.0 there were no window functions or CTEs; the set-based alternatives are self-joins, correlated subqueries or user variables (`@running := @running + x`, order-dependent and officially unreliable). Upgrading to 8.0 is the real fix.
- **Rounding/ordering differences.** Cursors accumulate in a specified type and order; set queries may sum in a different order, giving floating-point or `ROUND` timing differences. Use `DECIMAL`, round at the same place, and make `ORDER BY` deterministic.
- **When set-based is *not* faster.** Massive single-statement updates can bloat the log and escalate locks; the right tool is *batched set-based* — `UPDATE TOP (10000) … WHERE processed = 0` in a loop with commits — not a row cursor.

### 6. Performance and design implications
- Estimate effort: loop cost ∝ rows × per-statement overhead; set cost ∝ rows × per-row work + one-off planning.
- Index to support the window `PARTITION BY`/`ORDER BY` and join keys; check the plan for `Sort` and `Spool` warnings.
- Keep transactions bounded: for very large updates, chunk by key ranges and commit between chunks to protect the log and avoid long blocking.
- Check that a set-based replacement doesn't change semantics: NULL handling, duplicate rows, tie ordering, rows affected by triggers (a set update fires an `UPDATE` trigger once with many rows — trigger code written for single-row `inserted` breaks).
- Keep readability: a 40-line CTE chain may be harder than a loop for maintainers; name CTEs for the business rule and comment edge cases.
- Prove it: row counts, `EXCEPT` both directions, aggregate checksums, plus a handful of manually verified sample entities.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `UPDATE t SET … FROM t JOIN …` | Set-based multi-table update | Cursor updating from lookup |
| `SUM() OVER (… ROWS UNBOUNDED PRECEDING)` | Running total, row-accurate frame | Balances, cumulative metrics |
| `LAG()/LEAD()` | Access neighbouring rows | Row-to-row comparisons, gaps |
| `ROW_NUMBER()/RANK()/DENSE_RANK()` | Ranking / dedup / top-N per group | Replace nested loops |
| `STRING_AGG`, `GROUP_CONCAT`, `LISTAGG`, `string_agg` | Concatenate | Cursor building strings |
| `WITH cte AS (… UNION ALL …) OPTION (MAXRECURSION n)` | Recursive CTE | Hierarchies, number series |
| `MERGE` / `INSERT … ON CONFLICT` | Upsert in one pass | Cursor-driven insert-or-update |
| `EXCEPT` / `CHECKSUM_AGG` / hash aggregates | Result parity checks | Migration validation |
| `SET STATISTICS IO, TIME ON` | Compare logical reads/CPU | Benchmark old vs new |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| **Set-based** — any transformation, aggregation, ranking or join | Each row requires an external side effect (sending mail, calling a service) |
| **Window function** — logic depends on neighbouring/cumulative rows | Engine lacks windows (MySQL < 8.0) — upgrade or use alternative |
| **Recursive CTE** — arbitrary-depth hierarchy | Cyclic data without a guard; very deep trees with a low recursion limit |
| **Batched set-based DML** — very large changes needing small transactions | Single giant statement that would bloat log or block users |
| **Cursor retained** — per-object dynamic DDL, procedure per row, ordered legacy logic with unavoidable state | The only reason is "it's how we always did it" |

### 9. Common mistakes
- **Cursor to `UPDATE` rows by key** → N statements → single `UPDATE … WHERE`/`JOIN`.
- **Default `RANGE` frame in running totals** → spools to disk and tie-handling surprise → use `ROWS`.
- **Row-by-row trigger logic** → breaks when a set update fires the trigger once → write triggers set-based on `inserted`/`deleted`.
- **No deterministic ORDER BY** → set result differs run to run, "doesn't match the cursor" → add tie-breakers.
- **Nested cursors** → N×M statements → one join with `GROUP BY`.
- **Replacing with `WHILE` loop counters** → same RBAR cost, different syntax → true set-based rewrite.
- **Skipping validation** → silent behaviour change in totals → parity checks before cutover.
- **Huge single-statement rewrite** → log/lock problems → batch by key range.
- **Using `FOR XML PATH` concatenation on 2017+** → unnecessary complexity and XML-entity escaping bugs → `STRING_AGG`.

### 10. 🎯 Interview kit
**30-second answer:** "Cursors process rows one at a time and pay statement overhead per row; the optimizer can't plan across iterations. Set-based SQL expresses the whole transformation in one statement — joins, `CASE`, window functions, `STRING_AGG`, recursive CTEs — so the engine can scan once, pick hash/merge joins, batch log writes and parallelise. I translate the cursor's logic, then prove parity with row counts and `EXCEPT` before cutover."

**Structure a longer answer:** clarify what the cursor computes → identify carried state → map to a set construct → write and index → validate (counts, checksums, edge cases, triggers) → compare plans/timings → deploy with fallback.

**Likely follow-ups:**
- *Running balance without cursor?* → `SUM() OVER … ROWS UNBOUNDED PRECEDING`; check ties and NULLs.
- *Top-N per group?* → `ROW_NUMBER` filter or `CROSS APPLY`/`LATERAL`; cursor is poor because it loops groups.
- *Summary column for millions of parents?* → one grouped subquery join update, chunked by parent key; or compute on read/indexed view.
- *Concatenate strings across rows in each engine?* → `STRING_AGG`/`string_agg`/`GROUP_CONCAT`/`LISTAGG` and their limits.
- *Hierarchy rewrite risks?* → cycles, recursion limits, exponential blow-up, ordering of traversal.
- *When is a cursor actually right?* → per-object admin tasks, calling procedures with side effects per row, sometimes streaming/ETL glue.
- *Totals differ slightly?* → data type/rounding, summation order, ties in ordering.

**Signals interviewers look for:** you reason in sets first, name the per-row overhead, can write the window/CTE alternatives fluently, and insist on validation.

### 11. Check yourself
- [ ] Rewrite a cursor that applies a 10% raise to one region as a single `UPDATE`.
- [ ] Write a running total and a `LAG`-based delta query, and state which frame you used.
- [ ] Replace a string-building cursor with `STRING_AGG`, noting the length limit.
- [ ] Design a parity test between a legacy cursor report and your rewrite.
- [ ] Name two situations where a cursor is still justified.
<!-- explainer:end -->

## Implementations (tools)

- [SQL Server](<../../../TOOLS/SQL_Server/_index.md>)

## Practice questions

- [Intermediate] [How would you rewrite a simple cursor that increments a salary column into a single set-based UPDATE?](<../../../PRACTICE/01_Databases_and_SQL/Database_Objects/Cursors/Q-10515_How_would_you_rewrite_a_simple_cursor_that_increments_a_sala.md>)
- [Intermediate] [How would you convert a T-SQL cursor loop that concatenates strings into a set-based STRING_AGG?](<../../../PRACTICE/01_Databases_and_SQL/Database_Objects/Cursors/Q-10528_How_would_you_convert_a_T-SQL_cursor_loop_that_concatenates_.md>)
- [Intermediate] [Describe how you would migrate a legacy T-SQL cursor report to a windowed set-based query and validate row-count and checksum parity before cutover.](<../../../PRACTICE/01_Databases_and_SQL/Database_Objects/Cursors/Q-10369_Describe_how_you_would_migrate_a_legacy_T-SQL_cursor_report_.md>)
- [Intermediate] [How would you convert a cursor that builds a comma-separated string across rows into a set-based aggregation, and what limits does each engine impose?](<../../../PRACTICE/01_Databases_and_SQL/Database_Objects/Cursors/Q-10400_How_would_you_convert_a_cursor_that_builds_a_comma-separated.md>)
- [Intermediate] [How do you migrate a MySQL cursor-based procedure to a set-based statement when MySQL lacks some window functions in older versions?](<../../../PRACTICE/01_Databases_and_SQL/Database_Objects/Cursors/Q-10432_How_do_you_migrate_a_MySQL_cursor-based_procedure_to_a_set-b.md>)
- [Deep] [What is meant by set-based versus procedural thinking and how do cursors relate to it?](<../../../PRACTICE/01_Databases_and_SQL/Database_Objects/Cursors/Q-10460_What_is_meant_by_set-based_versus_procedural_thinking_and_ho.md>)
- [Deep] [Why might a set-based UPDATE using a JOIN outperform a cursor that updates rows one at a time?](<../../../PRACTICE/01_Databases_and_SQL/Database_Objects/Cursors/Q-10516_Why_might_a_set-based_UPDATE_using_a_JOIN_outperform_a_curso.md>)
- [Deep] [How would you test that a cursor-based procedure and its set-based rewrite produce identical results?](<../../../PRACTICE/01_Databases_and_SQL/Database_Objects/Cursors/Q-10533_How_would_you_test_that_a_cursor-based_procedure_and_its_set.md>)

## Related concepts

- [Stored Procedure Performance and Design](<../Stored_Procedures/Stored_Procedure_Performance_and_Design.md>) *(Stored Procedures)*
- [UDF Performance](<../../SQL_Language/SQL_Functions/UDF_Performance.md>) *(SQL Functions)*
- [PL-SQL in Production](<../../../TOOLS/Oracle/PL-SQL/PL-SQL_in_Production.md>) *(PL-SQL)*

## References

- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 15: Case Studies and the Future of T-SQL › Query Refactoring
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 16: Best Practices for T-SQL Development and Database Design › Use Set-Based Logic, Avoid RBAR (Row-By-Agonizing-Row)
- *SQL Server Tutorial* (YouTube series) — Part 64: Replacing cursors using joins in sql server ([watch](https://www.youtube.com/watch?v=4LbPZXeOARY))

