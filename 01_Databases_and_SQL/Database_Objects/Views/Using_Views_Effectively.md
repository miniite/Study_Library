---
id: "01_Databases_and_SQL/Database_Objects/Views/Using_Views_Effectively"
level: 4
type: "concept"
topic: "01_Databases_and_SQL/Database_Objects/Views"
tools: []
---

# Using Views Effectively

**L4 Advanced** · [↑ Views](<_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** A view is a saved query that you use like a table; it gives you abstraction and security for free, no performance on its own, and it stays cheap only if you keep it simple, non-nested, and let the optimizer expand it into the outer query.

### 1. The big picture
Views let you name a query. Application developers `SELECT` from `Sales.vOrderDetail` instead of repeating a ten-table join; an analyst gets only the columns and rows they are allowed to see; a legacy table can be renamed behind a view that keeps the old shape. None of this requires copying data: a standard (non-materialized) view stores *only its definition*.

What a view does *not* do is make the query faster. When you query a view, the engine substitutes the view's `SELECT` into your statement (view *expansion* / *inlining*), optimizes the whole thing as one query, and runs it. Performance therefore depends on how the combined query looks — which is why "just add another view on top" can silently produce plans with dozens of joins.

Materialized (indexed) views are a separate topic in the following lessons; here we focus on ordinary views and how to use them well.

### 2. Core concepts and vocabulary
- **View** — a named, stored `SELECT`; queried like a table.
- **View expansion (inlining)** — the optimizer replaces the view reference with its definition before optimization; there is no separate "view execution".
- **Updatable view** — a view through which `INSERT/UPDATE/DELETE` are permitted: single base table, no aggregates, `DISTINCT`, `TOP`, `UNION`, etc.; otherwise needs an `INSTEAD OF` trigger.
- **`WITH CHECK OPTION`** — rejects a modification made through the view if the resulting row would no longer be visible in the view.
- **`SCHEMABINDING`** — binds the view to the underlying objects so they cannot be dropped or altered in ways that break it; required for indexed views and recommended for views that use functions.
- **`WITH ENCRYPTION`** — obfuscates the definition (weak protection, hinders support).
- **Partitioned view** — a `UNION ALL` of tables with non-overlapping `CHECK` constraints, treated as one table.
- **Ownership chaining** — if view and base tables have the same owner, a user needs `SELECT` on the view only.
- **Nested view** — a view that references other views.

### 3. How it works, step by step
1. You create the view: `CREATE VIEW` stores the text in the catalog (`sys.sql_modules`) and records dependencies. No rows are read.
2. A query references the view: parsing/binding resolves the name and replaces it with the stored `SELECT` (as a derived table).
3. The optimizer simplifies: it removes joins that are not needed (join elimination thanks to trusted foreign keys), pushes predicates down into the view's tables, and chooses indexes — all as if you had written the long query yourself.
4. A plan is compiled for the *entire* statement and cached under the *outer* statement's text.
5. Permissions are checked using ownership chaining: if the chain is unbroken, only the view permission matters.

| Use | What it gives | What it does not |
|---|---|---|
| Abstraction | Hide joins, rename/reshape columns, stable interface | Speed |
| Security | Column/row restriction via select list and `WHERE` | Protection from side channels if not combined with permissions |
| Compatibility | Keep old name/shape after schema change | Free evolution — the view still must be maintained |
| Reuse | One definition of "active customer" | Immunity to bad plans |

### 4. Worked example
```sql
-- Reporting view: stable interface, restricted columns
CREATE VIEW Sales.vOrderSummary
WITH SCHEMABINDING
AS
SELECT o.OrderID, o.OrderDate, c.CustomerName, o.Status,
       SUM(l.Quantity * l.UnitPrice) AS OrderValue
FROM Sales.Orders AS o
JOIN Sales.Customers AS c ON c.CustomerID = o.CustomerID
JOIN Sales.OrderLines AS l ON l.OrderID = o.OrderID
GROUP BY o.OrderID, o.OrderDate, c.CustomerName, o.Status;
GO

-- Security: give analysts the view only
GRANT SELECT ON Sales.vOrderSummary TO ReportingRole;
DENY  SELECT ON Sales.Customers     TO ReportingRole;   -- ownership chaining still lets the view work

-- Row-restricting, updatable view with CHECK OPTION
CREATE VIEW Sales.vOpenOrders
AS SELECT OrderID, CustomerID, Status, OrderDate
   FROM Sales.Orders
   WHERE Status = 'Open'
WITH CHECK OPTION;
GO
UPDATE Sales.vOpenOrders SET Status = 'Closed' WHERE OrderID = 1;  -- fails: row would leave the view
```
**What happens at query time:** `SELECT OrderID FROM Sales.vOrderSummary WHERE OrderDate >= '2026-01-01'` — the predicate on `OrderDate` is pushed *below* the aggregate (it is a grouping column), so only recent orders are aggregated; the optimizer works on the combined query. Check with `SET STATISTICS IO ON` and the actual plan: you should see seeks on `Orders`, not a full aggregate followed by a filter. If a predicate references an aggregate (`OrderValue > 1000`) it must run *after* aggregation — expected, not a bug.

### 5. Under the hood
- **Plan caching.** Plans are cached per outer statement; changing the view's definition (`ALTER VIEW`) invalidates plans of statements that use it. Frequent definition changes in a hot path cause recompilation churn.
- **`SELECT *` is frozen.** At `CREATE VIEW` time `*` is expanded into the column list; later added columns do not appear, and dropped/renamed columns can break the view (`sp_refreshview` rebinds). Always list columns; use `SCHEMABINDING` so base changes that break it are rejected.
- **`ORDER BY` in a view.** Not allowed without `TOP`/`OFFSET`, and even then it does not guarantee order of the outer query — order belongs in the outer `SELECT`.
- **Predicate pushdown and non-inlinable shapes.** Views with `TOP`, window functions, `DISTINCT`, or `UNION` are still expanded but some predicates cannot be pushed below them without changing results; this is the usual cause of "the view is slow but the same SQL inline is fast".
- **Nested views.** Each layer is expanded; ten layers of "just add a column" yield plans with the same table joined repeatedly. The optimizer's search budget is finite, so very large expansions lead to "good enough plan found" with poor choices.
- **Updating through views.** The engine maps modified columns to the underlying single table; multi-table views need `INSTEAD OF` triggers; partitioned views can be updated if rules (primary keys include partition column, `CHECK` constraints) are met.

### 6. Performance and design implications
- A view is as fast as the equivalent inline query — no better. Measure the *outer* query's plan.
- Keep views thin and single-purpose; avoid nesting beyond one or two levels.
- Do not put expensive row-by-row scalar UDFs or multi-statement table-valued functions in views: they inhibit parallelism and cardinality estimation.
- Add the *indexes on base tables* the common view predicates need — a view cannot be indexed unless materialized.
- Prefer inline table-valued functions when you need a *parameterized* view.
- Do not use a view as a performance wall you cannot see through: monitor the underlying statements with `sys.dm_exec_query_stats` and Query Store.
- For heavy aggregations that are read far more than written, consider an indexed/materialized view or a summary table.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `CREATE/ALTER VIEW` | Define or change | Always list columns |
| `WITH SCHEMABINDING` | Lock dependencies | Indexed views, protect from breaking changes |
| `WITH CHECK OPTION` | Enforce view predicate on writes | Row-restricting updatable views |
| `sp_refreshview` | Refresh view metadata | After base-table column changes |
| `sys.sql_expression_dependencies` | Who depends on what | Impact analysis |
| `sys.views`, `OBJECT_DEFINITION` | Inspect definitions | Review |
| `GRANT SELECT … ON view` | Security layer | Reporting roles |
| `CREATE UNIQUE CLUSTERED INDEX` on a view | Materialize it | Heavy aggregations (see next lessons) |

### 8. Trade-offs and decisions
| Choose a view when | Avoid / watch out when |
|---|---|
| Hiding join complexity or giving a stable interface to apps | You will stack views on views — flatten them |
| Restricting columns/rows for a role | You rely on the view as the *only* security layer for sensitive data without permission hygiene |
| Providing backwards compatibility during a migration | You need parameters — use an inline TVF |
| Reusing a shared business definition | The definition changes frequently and many reports depend on it silently |
| Making a partitioned (`UNION ALL`) table appear as one | Native partitioning is available and simpler |

### 9. Common mistakes
- Assuming a view caches results → each query re-runs the underlying work → use a materialized view/summary table if you need that.
- `SELECT *` in a view → new/changed columns not reflected or break the view → explicit columns + `SCHEMABINDING`.
- `ORDER BY` in views → ignored/rejected, confused expectations → order in the outer query.
- Deep nesting → bloated plans → flatten, test with actual plans.
- Joining a view to itself or to the base tables it already joins → redundant work → query base tables or reshape the view.
- Granting users `SELECT` on both view and tables "to be safe" → defeats the purpose → grant only the view.
- Forgetting `WITH CHECK OPTION` → rows "disappear" from the view after an update → add it for row-restricted updatable views.

### 10. 🎯 Interview kit
**30-second answer:** A view is a stored query used like a table. It stores no data (unless indexed/materialized); the optimizer inlines its definition into the outer query. Use it for abstraction and security, not for speed, and keep it simple to avoid nested, slow plans.

**Structure a longer answer:** clarify (standard vs materialized? engine?) → mechanism (expansion, predicate pushdown, permissions through ownership chaining) → trade-offs (nesting, `SELECT *`, `ORDER BY`, updatability) → scenario (security view for support staff; indexed view for a dashboard).

**Likely follow-ups:**
- *View vs table?* → definition vs storage; no indexes on a plain view.
- *Can you update through a view?* → yes if single table and no aggregates/DISTINCT/etc., else `INSTEAD OF` trigger; `WITH CHECK OPTION` guards.
- *View vs stored procedure vs inline TVF?* → composability and parameters.
- *Why is my view slow?* → look at the expanded plan, non-sargable predicates, nested views, scalar UDFs.
- *View vs CTE?* → a view is persistent and reusable; a CTE exists for one statement.
- *Security?* → grant on the view only; ownership chaining; `WITH CHECK OPTION`.

**Signals interviewers look for:** that you know a view does not materialize data, that you think in terms of plans, and that you use views deliberately for security and compatibility.

### 11. Check yourself
- [ ] Explain what happens, in order, when a query references a view.
- [ ] Show why `SELECT *` in a view is fragile and how `SCHEMABINDING` helps.
- [ ] Build a row-restricted updatable view and demonstrate `WITH CHECK OPTION`.
- [ ] Take a three-level nested view stack and describe how you would flatten it.
- [ ] State when you would use an inline TVF or an indexed view instead of a plain view.
<!-- explainer:end -->

## Practice questions

- [Basic] [How do you drop a view and what happens to the underlying tables when you do?](<../../../PRACTICE/01_Databases_and_SQL/Database_Objects/Views/Q-43844_How_do_you_drop_a_view_and_what_happens_to_the_underlying_ta.md>)
- [Basic] [What is a view in a relational database and why is it called a virtual table?](<../../../PRACTICE/01_Databases_and_SQL/Database_Objects/Views/Q-44043_What_is_a_view_in_a_relational_database_and_why_is_it_called.md>)
- [Intermediate] [What is the difference between a view and a stored procedure that returns a result set?](<../../../PRACTICE/01_Databases_and_SQL/Database_Objects/Views/Q-43987_What_is_the_difference_between_a_view_and_a_stored_procedure.md>)
- [Intermediate] [How do you document and version-control view definitions in a team environment?](<../../../PRACTICE/01_Databases_and_SQL/Database_Objects/Views/Q-44039_How_do_you_document_and_version-control_view_definitions_in_.md>)
- [Deep] [Why might filtering on a view column still cause a full scan of the underlying table?](<../../../PRACTICE/01_Databases_and_SQL/Database_Objects/Views/Q-43994_Why_might_filtering_on_a_view_column_still_cause_a_full_scan.md>)
- [Deep] [A view with WITH CHECK OPTION is rejecting valid inserts after a base-table trigger fires; how would you debug the interaction between the check option and the trigger?](<../../../PRACTICE/01_Databases_and_SQL/Database_Objects/Views/Q-43854_A_view_with_WITH_CHECK_OPTION_is_rejecting_valid_inserts_aft.md>)
- [Deep] [Explain how statistics on the base tables (not the view) drive the plan for a view query, and what happens when those stats go stale on a hot table.](<../../../PRACTICE/01_Databases_and_SQL/Database_Objects/Views/Q-43903_Explain_how_statistics_on_the_base_tables_not_the_view_drive.md>)
- [Expert] [Explain how a view definition can silently degrade an execution plan because the optimizer materializes an inline view instead of merging it; how do you detect this in the plan?](<../../../PRACTICE/01_Databases_and_SQL/Database_Objects/Views/Q-43858_Explain_how_a_view_definition_can_silently_degrade_an_execut.md>)

## Related concepts

- [Materialized View Refresh and Maintenance](<../Materialized_Views/Materialized_View_Refresh_and_Maintenance.md>) *(Materialized Views)*
- [MySQL Optimizer Behavior](<../../../TOOLS/MySQL/MySQL_Platform/MySQL_Optimizer_Behavior.md>) *(MySQL Platform)*
- [Query Rewrites and Plan Stability](<../../Query_Performance/Execution_Plans/Query_Rewrites_and_Plan_Stability.md>) *(Execution Plans)*
- [Views and Materialized Views](<../../Database_Internals/DBMS_Concepts/Views_and_Materialized_Views.md>) *(DBMS Concepts)*
- [Schemas and Databases](<../../../TOOLS/MySQL/MySQL_Platform/Schemas_and_Databases.md>) *(MySQL Platform)*
- [Scalar Subqueries](<../../SQL_Language/SQL_Fundamentals/Scalar_Subqueries.md>) *(SQL Fundamentals)*

## References

- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 5: Views, Functions, and Stored Procedures › Introduction
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 5: Views, Functions, and Stored Procedures › Structure
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 5: Views, Functions, and Stored Procedures › Views in SQL Server
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 5: Views, Functions, and Stored Procedures › Historical Context of Views
- *Hadoop: The Definitive Guide* — Tom White — Hive › Views
- *Beginning T-SQL* — Kathi Kellenberger, Lee Everest — Views
- *Beginning T-SQL* — Kathi Kellenberger, Lee Everest — Views › Creating Views
- *Beginning T-SQL* — Kathi Kellenberger, Lee Everest — Views › Avoiding Common Problems with Views
- *Beginning T-SQL* — Kathi Kellenberger, Lee Everest — Views › Manipulating Data with Views
- *Expert Performance Indexing in SQL Server* — Jason Strate, Grant Fritchey — Page Compression › Indexed Views
- *Databricks Certified Data Engineer Associate Study Guide* — Derar Alhussein — Mastering Relational Entities in Databricks › Exploring Views
- *Databricks Certified Data Engineer Associate Study Guide* — Derar Alhussein — Mastering Relational Entities in Databricks › Comparison of View Types
- *Databricks Certified Data Engineer Associate Study Guide* — Derar Alhussein — Mastering Relational Entities in Databricks › Creation syntax
- *Databricks Certified Data Engineer Associate Study Guide* — Derar Alhussein — Chapter 6. Building Production Pipelines › Live views
- *T-SQL Fundamentals (4th ed.)* — Itzik Ben-Gan — Table expressions
- *SQL Server Tutorial* (YouTube series) — Part 39: Views in sql server ([watch](https://www.youtube.com/watch?v=VQpmOmZO2mo))
- *SQL Server Tutorial* (YouTube series) — Part 40: Updatable views in sql server ([watch](https://www.youtube.com/watch?v=xRIMMZsIt2k))
- *SQL Server Tutorial* (YouTube series) — Part 42: View limitations in sql server ([watch](https://www.youtube.com/watch?v=NlJvzTpVX8Y))
- *SQL Server Tutorial* (YouTube series) — Part 48: Derived tables and common table expressions in sql server ([watch](https://www.youtube.com/watch?v=FwcAkH8UyEA))

