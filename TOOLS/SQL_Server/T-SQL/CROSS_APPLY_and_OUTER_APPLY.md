---
id: "TOOLS/SQL_Server/T-SQL/CROSS_APPLY_and_OUTER_APPLY"
level: 4
type: "concept"
topic: "TOOLS/SQL_Server/T-SQL"
tools: ["SQL_Server"]
---

# CROSS APPLY and OUTER APPLY

**L4 Advanced** · [↑ T-SQL](<_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** `APPLY` runs a table expression once for every row of the left input, letting the right side *reference the left row's columns* — something an ordinary `JOIN` cannot do.

### 1. The big picture
T-SQL's `FROM` clause supports four table operators: `JOIN`, `APPLY`, `PIVOT` and `UNPIVOT`. `JOIN` is standard SQL; `APPLY` is a T-SQL extension (SQL standard later added `LATERAL`, which is the same idea). A join treats its two inputs as independent sets and combines them using a predicate. `APPLY` treats the right input as a *function of the left row*: for each left row, evaluate the right-side expression with that row's values available, and return whatever rows come out.

That single difference unlocks four everyday patterns: **top-N per group**, **calling table-valued functions per row**, **unpacking per-row expressions** (`STRING_SPLIT`, `OPENJSON`, `VALUES` unpivots), and **reusing a computed expression** without repeating it. It sits between joins/subqueries (parent topic: querying multiple tables) and table-valued functions / window functions (neighbours that solve overlapping problems).

### 2. Core concepts and vocabulary
- **Table operator** — a clause in `FROM` that takes input table(s) and produces a table (`JOIN`, `APPLY`, `PIVOT`, `UNPIVOT`).
- **Left input / right input** — with `APPLY`, the left is evaluated once; the right is evaluated *per left row*.
- **Correlation** — the right side referencing columns of the left side (`d.DepartmentID`).
- **Derived table / inline table-valued function (iTVF)** — the typical right-hand side. A *multi-statement* TVF also works but is opaque to the optimizer.
- **`CROSS APPLY`** — like `INNER JOIN`: left rows whose right side returns no rows are dropped.
- **`OUTER APPLY`** — like `LEFT OUTER JOIN`: such left rows are kept, right-side columns are `NULL`.
- **Lateral join** — the general name (PostgreSQL/standard: `LATERAL`).

### 3. How it works, step by step
Logically, for `A CROSS APPLY f(A.x)`:
1. Take the next row `a` from `A`.
2. Evaluate the right expression with `a`'s column values substituted.
3. Join `a` to every row produced (a mini inner join on "same left row").
4. If zero rows were produced: `CROSS APPLY` emits nothing; `OUTER APPLY` emits `a` plus `NULL`s.
5. Repeat for all left rows; the union is the result.

Physically the optimizer is free to rewrite this. Simple iTVFs and correlated derived tables are usually **inlined and flattened** into ordinary joins; when it can't, you see a **Nested Loops** operator with an *Outer References* list — the "per row" execution made visible.

| | `INNER JOIN` | `CROSS APPLY` | `LEFT JOIN` | `OUTER APPLY` |
|---|---|---|---|---|
| Right side may reference left | no | **yes** | no | **yes** |
| Unmatched left rows | dropped | dropped | kept (NULLs) | kept (NULLs) |
| `ON` clause | required | none | required | none |

### 4. Worked example
Top 3 most recent orders per customer — including customers with none:

```sql
SELECT c.CustomerID, c.CompanyName, o.OrderID, o.OrderDate
FROM dbo.Customers AS c
OUTER APPLY (
    SELECT TOP (3) o.OrderID, o.OrderDate
    FROM dbo.Orders AS o
    WHERE o.CustomerID = c.CustomerID      -- correlation
    ORDER BY o.OrderDate DESC, o.OrderID DESC
) AS o;
```

`TOP (3)` with a correlated `WHERE` is evaluated per customer; a plain join cannot limit rows *per left row*. With `CROSS APPLY` the customers with no orders disappear; with `OUTER APPLY` they appear once with `NULL` order columns.

Unpacking a delimited column:

```sql
SELECT t.TicketID, s.value AS Tag
FROM dbo.Tickets AS t
CROSS APPLY STRING_SPLIT(t.Tags, ',') AS s;
```

Reusing a computed value (avoids repeating the expression in `SELECT`, `WHERE`, `ORDER BY`):

```sql
SELECT o.OrderID, v.NetTotal
FROM dbo.Orders AS o
CROSS APPLY (VALUES (o.Qty * o.UnitPrice * (1 - o.Discount))) AS v(NetTotal)
WHERE v.NetTotal > 1000;
```

Reading the plan: look for **Nested Loops → inner side → Index Seek** with *Outer References* = `c.CustomerID`. The execution count on the seek equals the number of left rows; if that's large, the inner side must be cheap (index on `(CustomerID, OrderDate DESC)`).

### 5. Under the hood
- Because `APPLY` has no `ON`, the join condition lives *inside* the right side. The optimizer decorrelates (turns it into a join/semi-join/`Segment`+`Top`) when it can; `TOP` with `ORDER BY` frequently becomes a **Top** operator under a nested loop, or, for `ROW_NUMBER` forms, a **Segment/Sequence Project** after a scan.
- **Scalar and multi-statement TVFs** are black boxes: fixed row guesses (multi-statement TVF: 1 row with the pre-2014 estimator, 100 rows with the 2014+ one), no parallelism for scalar UDFs before **scalar UDF inlining** (SQL Server 2019, compat level 150). Inline TVFs are expanded like views.
- Nested loops cost = left rows × cost of one inner execution. Hash/merge are not available for truly correlated inner sides, which is why indexing the correlation column matters so much.
- `OUTER APPLY` cannot be turned into an inner join, so it restricts join reordering slightly versus `CROSS APPLY`.

### 6. Performance and design implications
- **Index for the inner side**: `(correlation column, sort column) INCLUDE (needed columns)` turns top-N-per-group into N seeks per left row.
- **Few groups, many rows per group** → `APPLY` + `TOP` wins. **Many groups, few rows each** → `ROW_NUMBER() OVER (PARTITION BY …)` with one scan often wins. Test both.
- Measure with `SET STATISTICS IO ON` and actual execution plans (*Number of Executions* on inner operators).
- Prefer **inline** TVFs over scalar/multi-statement ones; a scalar UDF in a `WHERE` runs per row and (pre-2019) forces serial plans.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `CROSS APPLY` | per-row inner semantics | top-N, per-row function, row-level unpivot |
| `OUTER APPLY` | per-row, keep unmatched left rows | optional detail, "latest or none" |
| `CROSS APPLY (VALUES …)` | cheap unpivot / alias reuse | repeated expressions |
| `STRING_SPLIT`, `OPENJSON` | table-valued built-ins | shredding strings/JSON |
| `SET STATISTICS IO, TIME ON` | cost of per-row execution | tuning |
| Compat level 150+ | scalar UDF inlining | legacy scalar functions |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| Need top-N (or latest) per group, with a supporting index | The right side is a scalar/multi-statement UDF on large left input |
| Right side depends on the left row (function, split, JSON) | A plain join or `GROUP BY` solves it — `APPLY` is then needless |
| You want to keep parents without children (`OUTER APPLY`) | Left side is huge and no index supports the inner seek |
| Reusing an expression across clauses | You want portability to engines without lateral support |

### 9. Common mistakes
- Using `CROSS APPLY` where parents without children must be kept → rows silently vanish → use `OUTER APPLY`.
- `TOP` without `ORDER BY` in the right side → arbitrary rows, non-deterministic → always order, add a tie-breaker.
- Ordering by a non-unique column → ties give unstable results → append the key.
- No index on the correlation column → one scan per left row → add the covering index.
- Filtering `OUTER APPLY` results in `WHERE o.OrderID IS NOT NULL` → turns it into `CROSS APPLY`; say what you mean.

### 10. 🎯 Interview kit
**30-second answer:** `APPLY` is a lateral join: for each left row it evaluates a table expression that may reference that row. `CROSS APPLY` drops left rows with no right rows, `OUTER APPLY` keeps them with `NULL`s. Typical uses: top-N per group, table-valued functions, splitting strings/JSON.

**Structure a longer answer:** clarify (is the right side dependent on the left?) → mechanism (per-row evaluation, nested loops, outer references) → trade-offs vs `ROW_NUMBER` and plain joins → scenario (latest 3 orders per customer with a seek-supporting index).

**Likely follow-ups:**
- *Why can't a JOIN do this?* → a join's right input can't reference the left input.
- *APPLY vs ROW_NUMBER for top-N?* → APPLY with index scales with few groups; window scan better with many.
- *Does APPLY always run row by row?* → no, the optimizer often flattens it to a join.
- *Equivalent in other engines?* → `LATERAL` (PostgreSQL, standard), `CROSS JOIN LATERAL`.

**Signals interviewers look for:** can state the semantic difference, picks `OUTER APPLY` for optional rows, mentions indexing and execution counts, knows inline vs multi-statement TVF.

### 11. Check yourself
- [ ] Rewrite a "latest order per customer" query using `ROW_NUMBER` and then `APPLY`; compare IO.
- [ ] Explain why `CROSS APPLY` and `INNER JOIN` differ when the right side is correlated.
- [ ] Find *Outer References* and execution counts in an actual plan.
- [ ] Use `CROSS APPLY (VALUES …)` to unpivot three columns into rows.

<!-- related-deep-dives -->

### 🔗 Related deep dives
- [ANSI Core vs T-SQL Extensions (APPLY vs LATERAL)](<Dialects_and_Portability/02_ANSI_Core_vs_T-SQL_Extensions.md>)
- [Syntax Translation Cookbook (top-N per group)](<Dialects_and_Portability/03_Syntax_Translation_Cookbook.md>)
- [Series hub: T-SQL, ANSI SQL and Dialects](<Dialects_and_Portability/_index.md>)
<!-- explainer:end -->

## Implementations (tools)

- [SQL Server](<../_index.md>)

## Practice questions

- [Basic] [What are the different types of JOINs available in T-SQL?](<../../../PRACTICE/TOOLS/SQL_Server/T-SQL/Q-42911_What_are_the_different_types_of_JOINs_available_in_T-SQL.md>)
- [Basic] [What is a FULL OUTER JOIN and what rows does it return?](<../../../PRACTICE/TOOLS/SQL_Server/T-SQL/Q-42923_What_is_a_FULL_OUTER_JOIN_and_what_rows_does_it_return.md>)
- [Intermediate] [How does NULL behave in comparisons and how do you test for it in T-SQL?](<../../../PRACTICE/TOOLS/SQL_Server/T-SQL/Q-42869_How_does_NULL_behave_in_comparisons_and_how_do_you_test_for_.md>)
- [Intermediate] [What is a correlated subquery and how does it execute?](<../../../PRACTICE/TOOLS/SQL_Server/T-SQL/Q-42875_What_is_a_correlated_subquery_and_how_does_it_execute.md>)
- [Intermediate] [What is a self JOIN and give an example use case such as employee-manager relationships?](<../../../PRACTICE/TOOLS/SQL_Server/T-SQL/Q-42921_What_is_a_self_JOIN_and_give_an_example_use_case_such_as_emp.md>)
- [Intermediate] [How do you handle NULLs in aggregate and join logic so a report does not silently drop or double-count rows?](<../../../PRACTICE/TOOLS/SQL_Server/T-SQL/Q-42849_How_do_you_handle_NULLs_in_aggregate_and_join_logic_so_a_rep.md>)
- [Deep] [Explain the difference between nested loops merge and hash joins, and what row estimates cause the optimizer to pick each.](<../../../PRACTICE/TOOLS/SQL_Server/T-SQL/Q-42788_Explain_the_difference_between_nested_loops_merge_and_hash_j.md>)
- [Deep] [Explain the gaps-and-islands problem and how you would solve it with ROW_NUMBER differencing.](<../../../PRACTICE/TOOLS/SQL_Server/T-SQL/Q-42812_Explain_the_gaps-and-islands_problem_and_how_you_would_solve.md>)

## Related concepts

- [Join Types](<../../../01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Join_Types.md>) *(SQL Fundamentals)*
- [WHERE vs HAVING](<../../../01_Databases_and_SQL/Database_Internals/DBMS_Concepts/WHERE_vs_HAVING.md>) *(DBMS Concepts)*
- [Joins vs Subqueries and Join Types](<../../../01_Databases_and_SQL/SQL_Language/Joins_and_Subqueries/Joins_vs_Subqueries_and_Join_Types.md>) *(Joins and Subqueries)*

## References

- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 3: SELECT Statements and Filtering Logic › EXISTS, IN, and JOIN Filters
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 4: Joins, Set Operations, and Subqueries › Structure
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 4: Joins, Set Operations, and Subqueries › CROSS JOIN and SELF JOIN
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 4: Joins, Set Operations, and Subqueries › Conclusion
- *T-SQL Fundamentals (4th ed.)* — Itzik Ben-Gan — Joins

