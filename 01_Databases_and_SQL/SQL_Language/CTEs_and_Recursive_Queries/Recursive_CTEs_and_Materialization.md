---
id: "01_Databases_and_SQL/SQL_Language/CTEs_and_Recursive_Queries/Recursive_CTEs_and_Materialization"
level: 4
type: "concept"
topic: "01_Databases_and_SQL/SQL_Language/CTEs_and_Recursive_Queries"
tools: []
---

# Recursive CTEs and Materialization

**L4 Advanced** · [↑ CTEs and Recursive Queries](<_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** A CTE is a named, single-statement table expression; a *recursive* CTE is an anchor query plus a member that references the CTE itself, run repeatedly until it returns no rows — and whether a CTE is inlined or materialized decides whether it helps or hurts performance.

### 1. The big picture
A **Common Table Expression (CTE)** (`WITH name AS (…)`) gives a name to a query so the following statement can read it like a table. It is purely a *readability and structure* tool: it exists only for the one statement it prefixes, it is not stored, and unless the engine says otherwise it is not computed separately. Chained CTEs replace nested derived tables, and — unlike derived tables — can be referenced several times and can be **recursive**, which is the standard SQL way to walk hierarchies (org charts, folder trees, bill of materials) and graphs and to generate series.

The related performance question: does the engine *inline* the CTE (treat it like a view/macro and optimize the whole statement together) or *materialize* it (compute once, store, reuse)? The answer differs by engine and version and is the cause of many "same query, different speed" surprises.

### 2. Core concepts and vocabulary
- **CTE** — a temporary named result set defined by `WITH` and visible only to the immediately following statement.
- **Non-recursive CTE** — an ordinary query given a name; used for readability, reuse, and layering logic.
- **Recursive CTE** — a CTE whose definition has an **anchor member** and a **recursive member** joined by `UNION ALL`, the latter referencing the CTE.
- **Anchor member** — the seed query; runs once, cannot reference the CTE.
- **Recursive member** — runs repeatedly, each time seeing only the rows produced by the previous iteration.
- **Termination** — recursion stops when an iteration returns zero rows.
- **`MAXRECURSION`** — SQL Server query hint capping iterations (default 100; `0` = unlimited).
- **Inlining vs materialization** — expanding the CTE text into each reference vs computing it once into a worktable.
- **Optimization fence** — a boundary the planner cannot optimize across (old PostgreSQL CTEs; `MATERIALIZED`).
- **Cycle** — a node reachable from itself; makes naive recursion infinite.

### 3. How it works, step by step
**Non-recursive**
1. `WITH C1 AS (…), C2 AS (SELECT … FROM C1) SELECT … FROM C2;` — CTEs are separated by commas, never nested; each may reference earlier ones; the outer query sees all.
2. The statement before `WITH` must end with `;` (SQL Server) because `WITH` has other meanings.
3. Scope: one statement only. To share across statements use a view, a temp table or a table variable.
4. Multiple references: `C` referenced twice is two logical uses; whether it is *computed* twice depends on the engine (SQL Server re-evaluates each reference; PostgreSQL 12+ computes once only when referenced more than once or marked `MATERIALIZED`).

**Recursive**
1. Run the anchor; its rows are result set R0 and form the working set.
2. Run the recursive member with the CTE name bound to the *previous* iteration's rows only; the new rows are R1.
3. Append R1 to the result, repeat with R1 as input.
4. Stop when an iteration returns no rows (or error at `MAXRECURSION`).
5. The outer query reads the union of all Ri.

### 4. Worked example
```sql
-- Org chart: employee(empid, mgrid, name)
WITH emp_tree AS (
    SELECT empid, mgrid, name, 0 AS lvl,
           CAST(name AS VARCHAR(400)) AS path           -- anchor
    FROM employee WHERE mgrid IS NULL
    UNION ALL
    SELECT e.empid, e.mgrid, e.name, t.lvl + 1,
           CAST(t.path + ' > ' + e.name AS VARCHAR(400)) -- recursive member
    FROM employee AS e
    JOIN emp_tree AS t ON e.mgrid = t.empid
    WHERE t.lvl < 10                                      -- safety cap
)
SELECT * FROM emp_tree ORDER BY path
OPTION (MAXRECURSION 50);

-- Numbers 1..1000 (generator; a tally table is faster for big ranges)
WITH n AS (SELECT 1 AS i UNION ALL SELECT i + 1 FROM n WHERE i < 1000)
SELECT i FROM n OPTION (MAXRECURSION 1000);

-- Postgres: force/forbid materialization (12+)
WITH heavy AS MATERIALIZED     (SELECT … expensive …)
SELECT … FROM heavy a JOIN heavy b …;
WITH f AS NOT MATERIALIZED (SELECT * FROM big WHERE id = 42) SELECT * FROM f;
```
How to read it: level 0 is the root; each loop finds employees whose manager is a row from the previous level and adds one to `lvl`. `path` accumulates `CEO > VP > Manager`. The `lvl < 10` condition stops runaway recursion and `MAXRECURSION` is the engine-level backstop. In PostgreSQL and standard SQL the keyword is `WITH RECURSIVE`; SQL Server and Oracle infer recursion from the self-reference, a portability trap.

### 5. Under the hood   *(L4: optional, here for completeness)*
- **SQL Server** evaluates a recursive CTE as an *Index Spool/Table Spool* "Recursion" loop: a worktable stores the stack of intermediate results; the plan shows a *Table Spool (Lazy Spool)* with a *Concatenation* feeding the loop. Non-recursive CTEs are always inlined; there is no `MATERIALIZED` hint, so to force once-only evaluation use a temp table.
- **PostgreSQL ≤ 11** treated every CTE as an *optimization fence* (always materialized). **12+** inlines single-reference side-effect-free CTEs and materializes the rest by default; the planner can then push predicates in. `MATERIALIZED` restores the fence (useful for expensive computations used several times, or to stop a bad plan); `NOT MATERIALIZED` forces inlining.
- **Cardinality**: the optimizer has no statistics for the working set at each iteration and usually guesses a fixed size (SQL Server) — so row estimates for a recursive CTE are often wrong, which then cascades into bad join and memory-grant choices downstream.
- **Depth**: per iteration the engine only needs the previous iteration's rows; memory is about *breadth*, runtime about *total rows produced*.
- **Cycle protection**: standard SQL has `CYCLE … SET`; elsewhere carry a path/visited list and test `NOT path LIKE …` / array contains, or cap depth.

### 6. Performance and design implications
- Index the join column used in the recursive member (`employee.mgrid`); otherwise each iteration scans the table.
- Many stacked CTEs do not make a query slower by themselves, but if each is re-evaluated several times the cost multiplies; split into temp tables with statistics when a report has ~8+ CTEs and a messy plan (you gain stats and a stable plan, lose single-statement simplicity).
- Recursive CTEs are fine for shallow, controlled trees; for large hierarchies queried often consider materialized path, nested sets or closure tables, trading write cost for fast reads.
- A recursive CTE is a poor string splitter or number generator at scale — prefer `STRING_SPLIT` (SQL Server 2016+) and a numbers/tally table.
- Paginating `OFFSET … FETCH` over a large CTE recomputes the whole CTE each page → use keyset pagination.
- Diagnose CPU vs IO: actual plan + `SET STATISTICS IO, TIME ON`; high CPU with few logical reads points at recursion volume; large reads/spills point at the downstream join.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `WITH c AS (…)` | Defines a CTE for one statement | Layering and readability |
| `UNION ALL` in the CTE | Anchor + recursive member | Recursion; `UNION` also works but de-duplicates (cycle guard in some engines) |
| `OPTION (MAXRECURSION n)` | Cap iterations (SQL Server) | Always set explicitly for open hierarchies |
| `WITH RECURSIVE` | Required keyword in PostgreSQL/MySQL/SQLite | Portability |
| `MATERIALIZED` / `NOT MATERIALIZED` | PostgreSQL 12+ materialization control | Reuse vs predicate pushdown |
| `CYCLE col SET is_cycle USING path` | Standard cycle detection | PostgreSQL 14+ |
| `;WITH …` | Leading semicolon habit | Avoid "incorrect syntax near WITH" |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| CTE for readability / layered logic | You need the result in *later* statements — use a temp table |
| Recursive CTE for trees/graphs of unknown depth | Depth is known and tiny — a few self-joins may be simpler and faster |
| `MATERIALIZED` (PG) | You rely on filters pushing down — the fence blocks them |
| Temp table instead of many CTEs | You need a single atomic statement or an inline TVF/view |
| Closure/nested-set model | Hierarchy changes constantly — maintenance cost is high |

### 9. Common mistakes
- No termination condition / cycles in the data → "maximum recursion 100 has been exhausted" → add a depth cap, a visited path or fix the data, then set `MAXRECURSION` deliberately.
- Setting `MAXRECURSION 0` "to make the error go away" → real infinite loop on bad data → keep a cap.
- Expecting a CTE to be computed once → it runs once per reference in SQL Server → temp table.
- Assuming PostgreSQL still fences CTEs → pushdown now happens; old tuning tricks may be unnecessary or may change results of data-modifying CTEs.
- Forgetting the `;` before `WITH`.
- Type mismatch between anchor and recursive columns (`VARCHAR(50)` vs growing string) → truncation error → cast the anchor to the widest type.

### 10. 🎯 Interview kit
**30-second answer:** "A CTE is a named query scoped to one statement. A recursive CTE has an anchor and a recursive member combined with `UNION ALL`; it iterates, each pass using the previous pass's rows, until no rows return. In SQL Server I cap it with `MAXRECURSION`; in PostgreSQL I write `WITH RECURSIVE`."

**Structure a longer answer:** definition and scope → anchor/recursive mechanics → termination and cycles → inlining vs materialization (PG 12 change) → when to use temp tables or alternative hierarchy models.

**Likely follow-ups:**
- *CTE vs temp table vs view?* → CTE: one statement, no stats; temp table: persisted for the session with statistics and indexes; view: stored definition reused across statements.
- *Why did PostgreSQL stop fencing CTEs in 12?* → better plans through predicate pushdown/inlining; `MATERIALIZED` retains the old behaviour.
- *Maximum recursion exhausted — fix?* → find the cycle, add a depth/visited guard, then tune `MAXRECURSION`.
- *Top 3 per department?* → CTE with `ROW_NUMBER() OVER (PARTITION BY dept ORDER BY salary DESC)` then filter `rn <= 3`.
- *Why are recursive CTE row estimates wrong?* → no statistics for the iterated working set.

**Signals interviewers look for:** you name anchor/recursive/termination, know that scoping is one statement, and can discuss engine-specific materialization honestly.

### 11. Check yourself
- [ ] Write a recursive CTE listing all subordinates of manager 5 with their depth.
- [ ] Add cycle protection to it without relying on `MAXRECURSION`.
- [ ] Explain what is different about the CTE between PostgreSQL 11 and 12.
- [ ] Decide between 8 stacked CTEs and temp tables for a slow report, and justify.
- [ ] Explain why `MAXRECURSION` defaults to 100 and what raising it risks.
<!-- explainer:end -->

## Practice questions

- [Basic] [Explain scope and visibility of columns defined in one CTE when referenced by a later CTE in the same WITH clause.](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/CTEs_and_Recursive_Queries/Q-10251_Explain_scope_and_visibility_of_columns_defined_in_one_CTE_w.md>)
- [Intermediate] [How would you use a recursive CTE to split a comma-delimited string into rows, and why might a set-based split function be better?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/CTEs_and_Recursive_Queries/Q-10254_How_would_you_use_a_recursive_CTE_to_split_a_comma-delimited.md>)
- [Intermediate] [How would you refactor a query so a heavy CTE result is computed once and reused across three downstream aggregations?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/CTEs_and_Recursive_Queries/Q-10297_How_would_you_refactor_a_query_so_a_heavy_CTE_result_is_comp.md>)
- [Deep] [Why did Postgres change CTEs from an optimization fence to inlinable by default in 12+, and how did that affect your existing queries?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/CTEs_and_Recursive_Queries/Q-10246_Why_did_Postgres_change_CTEs_from_an_optimization_fence_to_i.md>)
- [Deep] [When a report uses 8 CTEs and the plan is a mess, how do you decide between splitting into temp tables versus keeping one statement?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/CTEs_and_Recursive_Queries/Q-10277_When_a_report_uses_8_CTEs_and_the_plan_is_a_mess_how_do_you_.md>)
- [Deep] [When would you deliberately keep a CTE as an optimization barrier in Postgres using MATERIALIZED, and what problem does that solve?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/CTEs_and_Recursive_Queries/Q-10301_When_would_you_deliberately_keep_a_CTE_as_an_optimization_ba.md>)
- [Deep] [How do you diagnose whether a slow multi-CTE query is CPU-bound in the recursion or I/O-bound in a downstream join?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/CTEs_and_Recursive_Queries/Q-10328_How_do_you_diagnose_whether_a_slow_multi-CTE_query_is_CPU-bo.md>)
- [Expert] [Explain how recursive CTE cardinality estimates are usually wrong and how bad estimates cascade into poor join plans downstream.](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/CTEs_and_Recursive_Queries/Q-10269_Explain_how_recursive_CTE_cardinality_estimates_are_usually_.md>)

## Related concepts

- [Recursive CTEs](<../SQL_Fundamentals/Recursive_CTEs.md>) *(SQL Fundamentals)*
- [Materialized CTEs](<../../../TOOLS/PostgreSQL/PostgreSQL_Platform/Materialized_CTEs.md>) *(PostgreSQL Platform)*
- [MySQL Optimizer Behavior](<../../../TOOLS/MySQL/MySQL_Platform/MySQL_Optimizer_Behavior.md>) *(MySQL Platform)*
- [Recursion and DP Judgment](<../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Recursion_and_DP_Judgment.md>) *(Data Structures and Algorithms)*
- [Query Rewrites and Plan Stability](<../../Query_Performance/Execution_Plans/Query_Rewrites_and_Plan_Stability.md>) *(Execution Plans)*

## References

- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 4: Joins, Set Operations, and Subqueries › Derived Tables versus CTEs
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 4: Joins, Set Operations, and Subqueries › Advanced Tips and Techniques
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 6: CTEs, MERGE, Pivoting, JSON, and XML › Structure
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 6: CTEs, MERGE, Pivoting, JSON, and XML › Writing Recursive and Non-Recursive CTEs
- *T-SQL Fundamentals (4th ed.)* — Itzik Ben-Gan — Table expressions
- *T-SQL Fundamentals (4th ed.)* — Itzik Ben-Gan — SQL Graph (cont.)
- *Beginning T-SQL* — Kathi Kellenberger, Lee Everest — Writing Advanced Queries › Advanced CTE Queries
- *Beginning T-SQL* — Kathi Kellenberger, Lee Everest — Writing Advanced Queries › Using Multiple CTEs
- *Beginning T-SQL* — Kathi Kellenberger, Lee Everest — Writing Advanced Queries › Referencing a CTE Multiple Times
- *Beginning T-SQL* — Kathi Kellenberger, Lee Everest — Writing Advanced Queries › Joining a CTE to Another CTE
- *100 SQL Server Mistakes and How to Avoid Them* — Peter A. Carter — Data types › 3.3 #8 Writing your own hierarchy code
- *Practical Graph Structures in SQL Server and Azure SQL* — Louis Davidson — SQL Graph Table Basics › Filtering for One Path
- *Practical Graph Structures in SQL Server and Azure SQL* — Louis Davidson — Tree Data Structures › Aggregating Child Activity at Every Level
- *SQL Server Tutorial* (YouTube series) — Part 49: CTE in sql server ([watch](https://www.youtube.com/watch?v=ZXB5b-7HJHk))
- *SQL Server Tutorial* (YouTube series) — Part 50: Updatable common table expressions in sql server ([watch](https://www.youtube.com/watch?v=byU7a0WKPXo))

