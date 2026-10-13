---
id: "TOOLS/SQL_Server/T-SQL/T-SQL_Basics"
level: 4
type: "concept"
topic: "TOOLS/SQL_Server/T-SQL"
tools: ["SQL_Server"]
---

# T-SQL Basics

**L4 Advanced** · [↑ T-SQL](<_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** T-SQL is SQL Server's dialect of SQL — declarative set-based querying plus procedural extensions (variables, control flow, error handling) — and writing it well means thinking in sets and in *logical query processing order*.

### 1. The big picture
SQL is declarative: you describe *what* rows you want, and the optimizer decides *how* to fetch them. T-SQL (Transact-SQL) adds Microsoft's extensions: variables, `IF`/`WHILE`, `TRY…CATCH`, temp tables, stored procedures, functions and many built-ins. It is based on relational theory (sets, predicates, three-valued logic), but implements it with deviations — notably **duplicates allowed** (tables are multisets) and **`NULL`s** with three-valued logic.

The single most useful mental model is that a query is *written* in one order (`SELECT` first) but *logically processed* in another. Nearly every beginner "why can't I use my alias there?" question is answered by that order. Everything else in this topic (joins, subqueries, windows, DML, transactions) builds on it.

### 2. Core concepts and vocabulary
- **Batch** — one or more statements sent together, ended by `GO` (a client directive, not T-SQL).
- **Predicate** — expression that evaluates to `TRUE`, `FALSE` or `UNKNOWN`.
- **Three-valued logic** — `NULL` comparisons give `UNKNOWN`; `WHERE` keeps only `TRUE`.
- **Set-based vs row-based** — one statement operating on whole sets vs cursor/loop per row.
- **Logical query processing** — the conceptual evaluation order of clauses.
- **Identifier** — object name; delimit with `[ ]` when irregular.
- **Collation** — rules for comparing/sorting character data (affects case/accent sensitivity).
- **Data types** — `INT`, `DECIMAL(p,s)`, `DATE`/`DATETIME2`, `VARCHAR`/`NVARCHAR`, `BIT`, `UNIQUEIDENTIFIER`.
- **Variable** — `DECLARE @x INT = 5;` (scoped to the batch).

### 3. How it works, step by step
**Logical processing order of a `SELECT`:**
1. `FROM` (incl. joins, `APPLY`) → 2. `WHERE` → 3. `GROUP BY` → 4. `HAVING` → 5. `SELECT` (expressions, aliases, window functions) → 6. `DISTINCT` → 7. `ORDER BY` → 8. `TOP`/`OFFSET-FETCH`.

Consequences:
- Aliases defined in `SELECT` are visible only in `ORDER BY`, not in `WHERE`/`GROUP BY`/`HAVING`.
- `WHERE` filters rows *before* grouping; `HAVING` filters *groups* after.
- Window functions are evaluated in the `SELECT` phase, so they can't appear in `WHERE` (wrap in a CTE).
- `ORDER BY` returns a cursor-like ordered result, not a relation; a table/view/subquery has no guaranteed order.

**Key operators and functions:** `LIKE`, `IN`, `BETWEEN`, `EXISTS`, `CASE`, `COALESCE`, `ISNULL`, `NULLIF`, string/date functions, `CAST`/`TRY_CAST`/`CONVERT`.

### 4. Worked example
```sql
-- Setup
CREATE TABLE dbo.Orders (
    OrderID INT IDENTITY PRIMARY KEY,
    CustomerID INT NOT NULL,
    OrderDate DATE NOT NULL,
    Amount DECIMAL(10,2) NULL
);

-- Revenue per customer in 2025, only customers above 1000
SELECT   CustomerID,
         COUNT(*)                      AS Orders,
         SUM(COALESCE(Amount, 0))      AS Revenue
FROM     dbo.Orders
WHERE    OrderDate >= '20250101' AND OrderDate < '20260101'   -- sargable range
GROUP BY CustomerID
HAVING   SUM(COALESCE(Amount, 0)) > 1000
ORDER BY Revenue DESC;                                         -- alias legal here
```
Walkthrough: `FROM` reads the table; `WHERE` discards non-2025 rows (and can use an index on `OrderDate` because the predicate is a plain range); `GROUP BY` forms one group per customer; `HAVING` removes small groups; `SELECT` computes aggregates and aliases; `ORDER BY` can use the `Revenue` alias.

Procedural side:
```sql
DECLARE @id INT = 42, @total DECIMAL(12,2);
BEGIN TRY
    SELECT @total = SUM(Amount) FROM dbo.Orders WHERE CustomerID = @id;
    IF @total IS NULL PRINT 'no orders';
END TRY
BEGIN CATCH
    SELECT ERROR_NUMBER() AS Num, ERROR_MESSAGE() AS Msg;
END CATCH;
```

### 5. Under the hood
- The parser/binder checks syntax and names; the optimizer builds a plan from the *logical* tree, reordering freely as long as the result is equal — so logical order ≠ execution order.
- Plans are cached by text/parameters; a variable's value is *not* sniffed (unlike a parameter), giving average-density estimates.
- `NULL` is a mark, not a value: `NULL = NULL` is `UNKNOWN`. `GROUP BY` and `DISTINCT` treat NULLs as equal; `UNIQUE` constraints allow one NULL; aggregates ignore NULLs (except `COUNT(*)`).
- Implicit conversions follow *data type precedence*: the lower-precedence side is converted — comparing `VARCHAR` column to `NVARCHAR` literal converts the column and can kill an index seek.

### 6. Performance and design implications
- Write **sargable** predicates: `col >= @a AND col < @b`, not `YEAR(col) = 2025`.
- Select only needed columns; `SELECT *` blocks covering indexes and breaks on schema change.
- Prefer set-based statements to cursors/loops; reserve loops for batching large DML.
- Use correct types and match parameter types to column types.
- Measure: `SET STATISTICS IO, TIME ON`, actual execution plan, Query Store.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `SET NOCOUNT ON` | suppress "rows affected" messages | in procedures |
| `SET XACT_ABORT ON` | any run-time error aborts and rolls back the transaction | transactional code |
| `TRY_CAST`, `TRY_CONVERT` | return NULL instead of error | dirty input |
| `OFFSET … FETCH` | standard paging | needs `ORDER BY` |
| `ISNULL` vs `COALESCE` | 2-arg/typed-by-first vs ANSI multi-arg | prefer `COALESCE` for portability |
| `SELECT … INTO` | create+load table | staging |
| `@@ROWCOUNT`, `SCOPE_IDENTITY()` | last count / identity | after DML |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| Set-based single statement | Row-by-row logic needs external calls |
| `DATETIME2`/`DATE` for new work | Legacy `DATETIME` rounding (3 ms) matters |
| `NVARCHAR` for international text | Pure ASCII keys at huge scale (2× storage) |
| Temp tables for large intermediate sets | Table variables for big sets (no statistics until recompile) |

### 9. Common mistakes
- `WHERE col = NULL` → always UNKNOWN, no rows → use `IS NULL`.
- `NOT IN` with a nullable subquery → empty result → `NOT EXISTS`.
- `SELECT TOP` without `ORDER BY` → arbitrary rows → add `ORDER BY`.
- Relying on `ORDER BY` of a view/subquery → no guarantee → order in the outermost query.
- `BETWEEN` on datetimes with end-of-day literals → missed rows → half-open ranges.
- `COUNT(col)` instead of `COUNT(*)` → NULLs silently skipped.
- Integer division (`5/2 = 2`) → wrong ratios → cast first.

### 10. 🎯 Interview kit
**30-second answer:** T-SQL is SQL Server's SQL plus procedural features. Queries are declarative and logically processed `FROM → WHERE → GROUP BY → HAVING → SELECT → ORDER BY`; it uses three-valued logic with `NULL`.

**Structure a longer answer:** clarify → processing order → NULL semantics → set-based vs procedural → a performance example (sargability).

**Likely follow-ups:** *WHERE vs HAVING?* → rows vs groups, timing. *Why alias fails in WHERE?* → SELECT is evaluated after. *DELETE vs TRUNCATE?* → logged row-by-row vs deallocation, identity reset, permissions. *Temp table vs table variable?* → statistics, scope, recompiles.

**Signals interviewers look for:** NULL fluency, logical order recall, sargability, set-based thinking.

### 11. Check yourself
- [ ] Recite logical processing order and say where each alias becomes visible.
- [ ] Predict the result of `NULL = NULL`, `NULL IS NULL`, `NOT IN (1, NULL)`.
- [ ] Rewrite `WHERE YEAR(d)=2025` as a sargable predicate.
- [ ] Explain `XACT_ABORT` and `NOCOUNT`.

<!-- related-deep-dives -->

### 🔗 Related deep dives
- [Standard, Dialect and Engine](<Dialects_and_Portability/01_Standard_Dialect_Engine.md>)
- [ANSI Core vs T-SQL Extensions](<Dialects_and_Portability/02_ANSI_Core_vs_T-SQL_Extensions.md>)
- [Same Syntax, Different Semantics](<Dialects_and_Portability/04_Same_Syntax_Different_Semantics.md>)
- [Parser, Binder and Error Messages](<Dialects_and_Portability/05_Parser_Binder_and_Error_Messages.md>)
- [Series hub: T-SQL, ANSI SQL and Dialects](<Dialects_and_Portability/_index.md>)
<!-- explainer:end -->

## Implementations (tools)

- [SQL Server](<../_index.md>)

## Practice questions

- [Basic] [What is the difference between the BIT data type and using an INT flag to store true or false values in T-SQL?](<../../../PRACTICE/TOOLS/SQL_Server/T-SQL/Q-42864_What_is_the_difference_between_the_BIT_data_type_and_using_a.md>)
- [Basic] [How do you use the ORDER BY clause to sort results in ascending and descending order?](<../../../PRACTICE/TOOLS/SQL_Server/T-SQL/Q-42844_How_do_you_use_the_ORDER_BY_clause_to_sort_results_in_ascend.md>)
- [Basic] [How do you write a SELECT statement to fetch only the top 10 rows in T-SQL?](<../../../PRACTICE/TOOLS/SQL_Server/T-SQL/Q-42889_How_do_you_write_a_SELECT_statement_to_fetch_only_the_top_10.md>)
- [Basic] [What is a stored procedure and why would you use one in T-SQL?](<../../../PRACTICE/TOOLS/SQL_Server/T-SQL/Q-42933_What_is_a_stored_procedure_and_why_would_you_use_one_in_T-SQ.md>)
- [Intermediate] [What is a trigger in T-SQL and when would you use one?](<../../../PRACTICE/TOOLS/SQL_Server/T-SQL/Q-42882_What_is_a_trigger_in_T-SQL_and_when_would_you_use_one.md>)
- [Intermediate] [What is the TOP clause with WITH TIES and when is it useful?](<../../../PRACTICE/TOOLS/SQL_Server/T-SQL/Q-42901_What_is_the_TOP_clause_with_WITH_TIES_and_when_is_it_useful.md>)
- [Intermediate] [How would you debug a stored procedure that returns incorrect results as a fresher?](<../../../PRACTICE/TOOLS/SQL_Server/T-SQL/Q-42948_How_would_you_debug_a_stored_procedure_that_returns_incorrec.md>)
- [Deep] [Why can't you use a column alias defined in the SELECT list inside the WHERE clause?](<../../../PRACTICE/TOOLS/SQL_Server/T-SQL/Q-42943_Why_can_t_you_use_a_column_alias_defined_in_the_SELECT_list_.md>)

## Related concepts

- [MERGE and Upserts](<../SQL_Server_Platform/MERGE_and_Upserts.md>) *(SQL Server Platform)*
- [CTEs in SQL Server](<../SQL_Server_Platform/CTEs_in_SQL_Server.md>) *(SQL Server Platform)*
- [Dynamic SQL](<../../../01_Databases_and_SQL/Database_Internals/DBMS_Concepts/Dynamic_SQL.md>) *(DBMS Concepts)*
- [Scalar Subqueries](<../../../01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Scalar_Subqueries.md>) *(SQL Fundamentals)*
- [UDF Performance](<../../../01_Databases_and_SQL/SQL_Language/SQL_Functions/UDF_Performance.md>) *(SQL Functions)*
- [Date and Time Types](<../SQL_Server_Platform/Date_and_Time_Types.md>) *(SQL Server Platform)*

## References

- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 1: Getting Started with T-SQL › Introduction
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 1: Getting Started with T-SQL › T-SQL vs. ANSI SQL
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 1: Getting Started with T-SQL › T-SQL Extensions
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 1: Getting Started with T-SQL › CRUD in Action (Combined Workflow)
- *T-SQL Fundamentals (4th ed.)* — Itzik Ben-Gan — Background to T-SQL querying and programming
- *T-SQL Fundamentals (4th ed.)* — Itzik Ben-Gan — Single-table queries (cont.)
- *T-SQL Fundamentals (4th ed.)* — Itzik Ben-Gan — Set operators
- *T-SQL Fundamentals (4th ed.)* — Itzik Ben-Gan — T-SQL for data analysis
- *100 SQL Server Mistakes and How to Avoid Them* — Peter A. Carter — T-SQL development › 5 T-SQL development
- *Beginning T-SQL* — Kathi Kellenberger, Lee Everest — Writing Simple SELECT Queries › Using the SELECT Statement
- *Beginning T-SQL* — Kathi Kellenberger, Lee Everest — Writing Simple SELECT Queries › Formatting T-SQL Code
- *Beginning T-SQL* — Kathi Kellenberger, Lee Everest — Writing Simple SELECT Queries › Why Use Aliases?
- *Beginning T-SQL* — Kathi Kellenberger, Lee Everest — 12. Understanding T-SQL Programming Logic
- *Expert Performance Indexing in SQL Server* — Jason Strate, Grant Fritchey — T-SQL Scripts

