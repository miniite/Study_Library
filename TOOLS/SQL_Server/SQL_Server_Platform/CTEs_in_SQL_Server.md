---
id: "TOOLS/SQL_Server/SQL_Server_Platform/CTEs_in_SQL_Server"
level: 4
type: "concept"
topic: "TOOLS/SQL_Server/SQL_Server_Platform"
tools: ["SQL_Server"]
---

# CTEs in SQL Server

**L4 Advanced** · [↑ SQL Server Platform](<_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** A common table expression (CTE) is a named, single-statement query result that makes complex T-SQL readable and lets you write recursion — but in SQL Server it is *not* materialized, so it behaves like an inline view, not a temp table.

### 1. The big picture
A CTE is declared with `WITH name AS (query)` immediately before one `SELECT`, `INSERT`, `UPDATE`, `DELETE` or `MERGE`. It lets you give a sub-result a name, reference it (even several times) in the statement that follows, and break a long query into readable steps instead of nesting derived tables.

CTEs sit between derived tables/views (inline, unnamed or permanent) and temporary tables (physically stored, with statistics). The most important fact for SQL Server users is that the optimizer **expands the CTE definition into the outer query**, exactly like a view. There is no "run this first, store it, reuse it" step. This drives both the performance behaviour and most interview questions.

CTEs have two flavours: **non-recursive** (readability) and **recursive** (hierarchies, graphs, number/date series).

### 2. Core concepts and vocabulary
- **CTE** — a named table expression valid only for the one statement that follows it.
- **Non-recursive CTE** — a plain named query; used to simplify or reuse logic inside one statement.
- **Anchor member** — the first, non-self-referencing `SELECT` in a recursive CTE; provides the starting rows.
- **Recursive member** — the `SELECT` that references the CTE's own name; run repeatedly against the rows produced by the previous iteration.
- **Termination** — recursion ends when an iteration returns zero rows.
- **MAXRECURSION** — query hint capping recursion depth (default 100; `0` = unlimited).
- **Inline expansion** — the optimizer substitutes the CTE text into the outer query; it is not stored.
- **Updatable CTE** — a CTE over a single base table can be the target of `UPDATE`/`DELETE`, changing the underlying table.

### 3. How it works, step by step
1. Write `WITH cte_name (optional_column_list) AS ( SELECT ... )`. If the statement before it is not terminated with `;`, the batch fails — hence the habit `;WITH`.
2. Follow it with exactly one outer statement that references `cte_name`.
3. The parser binds the name; the optimizer treats each reference as a copy of the defining query and merges it into one plan. If you reference the CTE twice, the definition is typically evaluated twice.
4. For a **recursive** CTE the engine runs the anchor once, then repeatedly runs the recursive member using only the *previous iteration's* output (via a spool in the plan) and `UNION ALL`s the results, until no rows come back or `MAXRECURSION` is exceeded.
5. Multiple CTEs are comma-separated under one `WITH`; later CTEs may reference earlier ones.

### 4. Worked example
```sql
-- Non-recursive: remove duplicates keeping the latest row per email
WITH ranked AS (
    SELECT CustomerID, Email,
           ROW_NUMBER() OVER (PARTITION BY Email ORDER BY CreatedAt DESC) AS rn
    FROM dbo.Customers
)
DELETE FROM ranked WHERE rn > 1;      -- deletes from dbo.Customers

-- Recursive: org chart with depth
WITH org AS (
    SELECT EmployeeID, ManagerID, Name, 0 AS Lvl
    FROM dbo.Employees WHERE ManagerID IS NULL           -- anchor
    UNION ALL
    SELECT e.EmployeeID, e.ManagerID, e.Name, o.Lvl + 1  -- recursive
    FROM dbo.Employees e
    JOIN org o ON e.ManagerID = o.EmployeeID
)
SELECT * FROM org ORDER BY Lvl, Name
OPTION (MAXRECURSION 50);
```
In the first query the CTE is only a window over `dbo.Customers`, so the `DELETE` touches the base table. In the second, iteration 0 returns the root, iteration 1 its direct reports, and so on; `Lvl` counts iterations. A cycle in the data (A manages B manages A) would loop until `MAXRECURSION` raises error 530.

### 5. Under the hood
- The plan for a referenced-twice CTE usually shows two separate scans/seeks of the base tables; there is no shared "CTE scan" operator for non-recursive CTEs.
- Recursive CTEs appear as an **Index Spool (Lazy Spool)** feeding a **Nested Loops** join, with a `Concatenation` of anchor and recursive branches. Each iteration is serial row-at-a-time work, so deep or wide recursions are CPU-bound and cannot parallelize well.
- Because the CTE is inlined, filters in the outer query are normally pushed into it (predicate pushdown), which is a benefit over a temp table that has already been filled.
- Cardinality estimates for recursive members are rough guesses; poor estimates often cause bad join choices further up.

### 6. Performance and design implications
- A CTE is **not** a performance feature. Rewriting a subquery as a CTE changes nothing in the plan.
- If the same expensive sub-result is used several times, or you need statistics/indexes on intermediate rows, **materialize into a `#temp` table** instead.
- Use recursion for hierarchies of modest depth; for big trees consider `hierarchyid`, a closure table, or iterating with a loop into a temp table.
- Measure with `SET STATISTICS IO, TIME ON` and the actual plan; check whether the base tables are accessed more than once.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `WITH cte AS (...)` | Declares a CTE for the next statement | Readability, step-wise logic |
| `UNION ALL` in anchor+recursive | Defines recursion | Hierarchies, series |
| `OPTION (MAXRECURSION n)` | Caps iterations (0 = unlimited) | Guard against cycles; deep trees |
| `ROW_NUMBER() ... DELETE FROM cte` | Updatable-CTE dedup idiom | Removing duplicates |
| `;WITH` | Terminates previous statement | Avoid "incorrect syntax near WITH" |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| CTE: one-off readability, recursion, dedup via window function | The CTE is referenced many times and is expensive (re-evaluated each time) |
| `#temp` table: reuse, need statistics/indexes, large intermediates | Tiny, single-use sub-results (extra I/O and recompiles for nothing) |
| View / inline TVF: reuse across statements | Logic only needed once |
| Derived table: simple one-level subquery | Needs recursion or multiple references |

### 9. Common mistakes
- Assuming a CTE is cached → it runs once per reference → use a temp table if you need that.
- Forgetting the `;` before `WITH` → syntax error → terminate previous statements.
- Recursion without a termination guarantee on cyclic data → error 530 or runaway → track a visited path or set `MAXRECURSION`.
- Using `UNION` instead of `UNION ALL` in the recursive CTE → invalid; recursion requires `UNION ALL` between members.
- Trying to use the CTE in a second statement → "Invalid object name" → CTE scope is one statement only.
- `ORDER BY` inside the CTE without `TOP` → error; order in the outer query.

### 10. 🎯 Interview kit
**30-second answer:** A CTE is a named query declared with `WITH` that exists for a single statement. It improves readability and enables recursion, but SQL Server inlines it like a view, so it isn't materialized or faster than a subquery.

**Structure a longer answer:** clarify (recursive or not?) → core mechanism (inlined, scope of one statement, anchor + recursive member) → trade-offs versus temp tables and views → real scenario (org chart, dedup with `ROW_NUMBER`).

**Likely follow-ups:**
- *CTE vs temp table?* → temp tables are stored, have statistics and indexes, survive multiple statements; CTE is inline and single-statement.
- *Can you update through a CTE?* → yes if it maps to one base table; the change hits the table.
- *How do you stop infinite recursion?* → `MAXRECURSION`, cycle detection, or a depth column in the predicate.
- *Is a CTE referenced twice executed twice?* → generally yes.

**Signals interviewers look for:** knowing CTEs are not materialized; recursion mechanics; choosing temp tables deliberately.

### 11. Check yourself
- [ ] Explain why referencing a CTE twice can double the work.
- [ ] Write a recursive CTE for an employee hierarchy with a depth column and cycle guard.
- [ ] Delete duplicates using a CTE and `ROW_NUMBER()`.
- [ ] State when you would replace a CTE with a `#temp` table and justify it with a plan or IO reading.
<!-- explainer:end -->

## Implementations (tools)

- [SQL Server](<../_index.md>)

## Practice questions

- [Basic] [What are the common aggregate functions in SQL Server and how does COUNT(*) differ from COUNT(column)?](<../../../PRACTICE/TOOLS/SQL_Server/SQL_Server_Platform/Q-41464_What_are_the_common_aggregate_functions_in_SQL_Server_and_ho.md>)
- [Basic] [How do you use the CASE expression to add conditional logic in a SELECT statement?](<../../../PRACTICE/TOOLS/SQL_Server/SQL_Server_Platform/Q-41472_How_do_you_use_the_CASE_expression_to_add_conditional_logic_.md>)
- [Basic] [What is a DEFAULT constraint and how does it behave when a column is omitted in an INSERT?](<../../../PRACTICE/TOOLS/SQL_Server/SQL_Server_Platform/Q-41504_What_is_a_DEFAULT_constraint_and_how_does_it_behave_when_a_c.md>)
- [Basic] [How do you declare and use a variable in T-SQL with the DECLARE and SET statements?](<../../../PRACTICE/TOOLS/SQL_Server/SQL_Server_Platform/Q-41548_How_do_you_declare_and_use_a_variable_in_T-SQL_with_the_DECL.md>)
- [Intermediate] [What is the difference between ISNULL() and COALESCE() in SQL Server?](<../../../PRACTICE/TOOLS/SQL_Server/SQL_Server_Platform/Q-41471_What_is_the_difference_between_ISNULL_and_COALESCE_in_SQL_Se.md>)
- [Intermediate] [What is a Common Table Expression (CTE) in SQL Server and why use one?](<../../../PRACTICE/TOOLS/SQL_Server/SQL_Server_Platform/Q-41509_What_is_a_Common_Table_Expression_CTE_in_SQL_Server_and_why_.md>)
- [Intermediate] [How do you handle errors in T-SQL using a TRY...CATCH block?](<../../../PRACTICE/TOOLS/SQL_Server/SQL_Server_Platform/Q-41526_How_do_you_handle_errors_in_T-SQL_using_a_TRY_CATCH_block.md>)
- [Deep] [What is a recursive CTE and give an example scenario where it is useful?](<../../../PRACTICE/TOOLS/SQL_Server/SQL_Server_Platform/Q-41510_What_is_a_recursive_CTE_and_give_an_example_scenario_where_i.md>)

## Related concepts

- [String Functions](<../../../01_Databases_and_SQL/SQL_Language/SQL_Functions/String_Functions.md>) *(SQL Functions)*
- [Dynamic SQL](<../../../01_Databases_and_SQL/Database_Internals/DBMS_Concepts/Dynamic_SQL.md>) *(DBMS Concepts)*
- [NULL vs Zero in Aggregates](<../../../01_Databases_and_SQL/SQL_Language/SQL_Functions/NULL_vs_Zero_in_Aggregates.md>) *(SQL Functions)*

## References

- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 1: Getting Started with T-SQL › Use PRINT or SELECT for Debugging
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 2: Data Types, NULLs and Conversions › Structure
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 2: Data Types, NULLs and Conversions › Implicit Type Conversion
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 2: Data Types, NULLs and Conversions › Type Precedence (Simplified Order)
- *T-SQL Fundamentals (4th ed.)* — Itzik Ben-Gan — Single-table queries (cont.)
- *Fuzzy Data Matching with SQL* — Jim Lehmer — Function Junction › Logical Functions: IIF
- *Fuzzy Data Matching with SQL* — Jim Lehmer — Function Junction › System Functions
- *Fuzzy Data Matching with SQL* — Jim Lehmer — CustomerCountByState View › W
- *Beginning T-SQL* — Kathi Kellenberger, Lee Everest — Thinking About Performance › Solutions to Exercise 4-1: Expressions Using Operators
- *Beginning T-SQL* — Kathi Kellenberger, Lee Everest — 12. Understanding T-SQL Programming Logic › Variables
- *Beginning T-SQL* — Kathi Kellenberger, Lee Everest — WHILE › Nesting WHILE Loops
- *Beginning T-SQL* — Kathi Kellenberger, Lee Everest — User-Defined Data Types › Table Types
- *Hadoop: The Definitive Guide* — Tom White — Hive › Operators and Functions
- *100 SQL Server Mistakes and How to Avoid Them* — Peter A. Carter — Data types › 3 Data types
- *100 SQL Server Mistakes and How to Avoid Them* — Peter A. Carter — Data types › 3.5 #10 Ignoring JSON
- *100 SQL Server Mistakes and How to Avoid Them* — Peter A. Carter — Error handling, testing, source control, and deployment › 7.3 #28 Not utilizing debugging functionality
- *100 SQL Server Mistakes and How to Avoid Them* — Peter A. Carter — Optimization › 10.2 #55 Not using instant file initialization
- *SQL Server Tutorial* (YouTube series) — Part 31: Inline table valued functions in sql server ([watch](https://www.youtube.com/watch?v=hs4mReAzESc))
- *SQL Server Tutorial* (YouTube series) — Part 32: Multi statement table valued functions in sql server ([watch](https://www.youtube.com/watch?v=EgYW7tsNP6g))
- *SQL Server Tutorial* (YouTube series) — Part 56: Error handling in sql server ([watch](https://www.youtube.com/watch?v=VLDirfx_OQg))
- *SQL Server Tutorial* (YouTube series) — Part 122: IIF function in SQL Server ([watch](https://www.youtube.com/watch?v=V8xA_JhpvCs))

