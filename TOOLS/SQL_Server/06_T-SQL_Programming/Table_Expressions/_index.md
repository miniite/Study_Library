---
id: "TOOLS/SQL_Server/06_T-SQL_Programming/Table_Expressions"
level: 4
type: "curriculum"
curriculum: "sql_server"
---

# Table Expressions

**L4 Advanced** · [↑ T-SQL Programming](<../_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** A table expression is a named query that the outer query treats like a table: derived tables, CTEs, views and inline table-valued functions. They exist to make code readable and reusable; SQL Server *inlines* them into the outer query, so they are not stored and not automatically faster or slower.

### 1. The big picture
Logical query processing has limits: you can't use a SELECT alias in WHERE, can't filter on a window function, and can't group by a computed column without repeating it. Table expressions solve this by letting you define a result in one query and reference it in another, as if it were a table.

SQL Server supports four kinds:

| Kind | Scope | Stored in metadata? | Parameters? |
|---|---|---|---|
| **Derived table** | One statement, defined inline in FROM | No | No (uses outer variables) |
| **CTE (common table expression)** | One statement, defined in `WITH` before it | No | No |
| **View** | Database object, reusable | Yes | No |
| **Inline table-valued function (iTVF)** | Database object, reusable | Yes | Yes |

The `APPLY` operator, closely related, lets a table expression reference columns from the outer row. Together these tools are how you structure complex T-SQL. Temp tables are *not* table expressions: they physically store data (compared in section 8).

### 2. Core concepts and vocabulary
- **Table expression**: a named query expression that represents a valid relational table.
- **Inlining (unnesting)**: the optimizer substitutes the table expression's definition into the outer query and optimizes the whole thing as one query. Nothing is materialized.
- **Derived table**: `FROM (SELECT …) AS alias`. Must have an alias, and every column must have a name.
- **CTE**: `WITH name [(cols)] AS (SELECT …) SELECT … FROM name`. Can be referenced several times in the outer query, and several CTEs can be chained, each referencing earlier ones.
- **Recursive CTE**: a CTE with an **anchor member** (the starting rows) and a **recursive member** (references the CTE itself), joined with `UNION ALL`.
- **MAXRECURSION**: safety limit on recursion depth. Default 100; `0` means no limit.
- **View**: a stored, named `SELECT`. Created with `CREATE VIEW`.
- **SCHEMABINDING**: view or function option that blocks changes to the referenced objects that would break it. Required for indexed views.
- **Inline TVF**: `CREATE FUNCTION … RETURNS TABLE AS RETURN (SELECT …)`. A parameterized view.
- **Multi-statement TVF (mTVF)**: declares a table variable, fills it in a body, and returns it. Not a table expression; it is *not* inlined.
- **APPLY**: `CROSS APPLY` runs the right-side expression for each left row and returns only rows that produced results (like inner join). `OUTER APPLY` also keeps left rows with no result (like left join).

Three requirements every table expression must meet, because it must be a relation:
1. **No guaranteed order**: `ORDER BY` is not allowed unless with `TOP`, `OFFSET-FETCH` or `FOR XML`, and then it only serves the filter.
2. **All columns must have names**: give aliases to computed columns.
3. **Column names must be unique**: `SELECT *` over a join with duplicate column names fails.

### 3. How it works, step by step
**Derived table, to reuse an alias:**
1. The inner query computes `YEAR(orderdate) AS orderyear`.
2. The outer query sees a table with a column `orderyear` and can use it in `WHERE`, `GROUP BY`, anywhere.
3. The optimizer inlines both into one plan, so `YEAR()` is computed once per row, just as if you'd repeated it.

**CTE, for readability:** a derived table nests (inside-out reading), a CTE chains (top-down reading). Compare:
```sql
-- Nested derived tables: read from the inside out
SELECT orderyear, numcusts
FROM (SELECT orderyear, COUNT(DISTINCT custid) AS numcusts
      FROM (SELECT YEAR(orderdate) AS orderyear, custid FROM Sales.Orders) AS d1
      GROUP BY orderyear) AS d2
WHERE numcusts > 70;

-- Chained CTEs: read top to bottom
WITH C1 AS (SELECT YEAR(orderdate) AS orderyear, custid FROM Sales.Orders),
     C2 AS (SELECT orderyear, COUNT(DISTINCT custid) AS numcusts FROM C1 GROUP BY orderyear)
SELECT orderyear, numcusts FROM C2 WHERE numcusts > 70;
```
Both produce the same plan.

**Multiple references.** A CTE can be referenced twice, for example to compare each year with the previous year by joining the CTE to itself. A derived table would have to be written out twice. **Important:** each reference is inlined separately, so the CTE's query runs once *per reference*. A CTE is a name for a query, not a cached result.

**Recursive CTE execution:**
1. Run the anchor member. Its rows are level 0.
2. Run the recursive member, joining the table to the rows produced by the *previous* iteration only.
3. Repeat step 2 until an iteration returns no rows.
4. Return the `UNION ALL` of all iterations.
5. If the depth exceeds `MAXRECURSION` (default 100), the statement fails with error 530.

### 4. Worked example
**Recursive org chart** with level and path:
```sql
WITH OrgChart AS
(
    -- anchor: the CEO
    SELECT empid, mgrid, firstname, 0 AS lvl,
           CAST(CONCAT('/', empid, '/') AS varchar(400)) AS path
    FROM HR.Employees
    WHERE mgrid IS NULL

    UNION ALL

    -- recursive member: direct reports of the previous level
    SELECT e.empid, e.mgrid, e.firstname, oc.lvl + 1,
           CAST(CONCAT(oc.path, e.empid, '/') AS varchar(400))
    FROM HR.Employees AS e
    JOIN OrgChart AS oc ON e.mgrid = oc.empid
)
SELECT REPLICATE('  ', lvl) + firstname AS employee, lvl, path
FROM OrgChart
ORDER BY path
OPTION (MAXRECURSION 50);
```
The `CAST` in both members matters: anchor and recursive columns must have *exactly* the same types, or you get error 240 ("Types don't match between the anchor and the recursive part").

**Inline TVF + APPLY: top 3 recent orders per customer:**
```sql
CREATE OR ALTER FUNCTION dbo.TopOrders (@custid int, @n int)
RETURNS TABLE
AS
RETURN
    SELECT TOP (@n) orderid, orderdate, empid
    FROM Sales.Orders
    WHERE custid = @custid
    ORDER BY orderdate DESC, orderid DESC;
GO

SELECT c.custid, c.companyname, t.orderid, t.orderdate
FROM Sales.Customers AS c
CROSS APPLY dbo.TopOrders(c.custid, 3) AS t;   -- OUTER APPLY keeps customers with no orders
```
With an index on `Sales.Orders (custid, orderdate DESC, orderid DESC)`, the plan is a scan of Customers with a nested loops seek + Top for each customer, which is very efficient.

**View with SCHEMABINDING:**
```sql
CREATE OR ALTER VIEW Sales.USACusts
WITH SCHEMABINDING
AS
SELECT custid, companyname, country
FROM Sales.Customers
WHERE country = N'USA'
WITH CHECK OPTION;   -- inserts/updates through the view must satisfy the WHERE
```

### 5. Under the hood
- **Inlining.** Look at the execution plan: there is no "CTE" or "view" operator, only base-table operators. Predicates from the outer query are pushed into the expression. A filter outside a view can still use an index inside it.
- **Exceptions to free pushdown**: `TOP`, `OFFSET`, window functions and `DISTINCT` inside the expression limit what can be pushed through them, because pushing a filter would change the result.
- **Recursive CTE plan.** Shows an *Index Spool (Lazy Spool)* storing each level in a worktable in tempdb, a *Concatenation* for the `UNION ALL`, and an *Assert* that enforces MAXRECURSION. Deep or wide hierarchies cause many spool reads. Index the join column (`mgrid`).
- **Views**: metadata stores the text; `SELECT *` in a view is expanded at creation time. Adding a column to the base table doesn't show up until `sp_refreshview` runs. Avoid `*` in views.
- **Inline vs. multi-statement TVFs.** iTVFs are inlined and get accurate estimates. mTVFs return a table variable; before SQL 2017 they were estimated at 1 row (100 rows in compat 120–130), which led to poor plans. SQL 2017+ has *interleaved execution* that runs the mTVF first and uses the real count, but mTVFs still can't push predicates in or run in parallel inside.
- **Indexed views**: a view with SCHEMABINDING and a unique clustered index is materialized and maintained on every base-table change. That is the one case where a "table expression" is stored (see the Programmable Objects and indexing lessons).

### 6. Performance and design implications
- Table expressions are free in themselves. Performance depends on the inlined query. Rewriting a derived table as a CTE changes readability, not speed.
- Referencing an expensive CTE several times repeats the work. If the inner query is costly and referenced more than once, materialize it into a `#temp` table instead. Temp tables also have statistics, which can improve estimates.
- Nested views (views on views on views) produce huge inlined queries, slow compilations and bad estimates. Keep nesting shallow.
- Prefer iTVFs over scalar UDFs and mTVFs. An iTVF is "a view with parameters" and optimizes like one.
- Watch for recursive CTEs over large graphs: they are row-by-row in disguise.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `WITH cte AS (…)` | Defines a CTE (previous statement must end with `;`) | Readability, multiple references |
| `OPTION (MAXRECURSION n)` | Recursion depth limit; 0 = none | Recursive CTEs |
| `CREATE OR ALTER VIEW … WITH SCHEMABINDING` | Protects dependencies | Production views, indexed views |
| `WITH CHECK OPTION` | Modifications through view must satisfy its filter | Updatable views |
| `sp_refreshview` | Refreshes view metadata | After base table changes |
| `CREATE FUNCTION … RETURNS TABLE AS RETURN` | Inline TVF | Parameterized reusable query |
| `CROSS APPLY` / `OUTER APPLY` | Correlated table expression | Top-N per group, TVF per row |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| **Derived table**: one-off reuse of an alias | Deep nesting → switch to CTEs |
| **CTE**: readability, chaining, self-reference, recursion | Expensive CTE referenced many times → temp table |
| **View**: shared logic, security layer (grant on view, not table) | Views on views; `SELECT *` in views |
| **iTVF**: reusable logic that needs parameters | — |
| **Temp table**: materialize once, index it, get statistics | Small, one-reference queries (extra I/O for nothing) |
| **Table variable**: very small sets, no recompiles | Large row counts (no column statistics) |

### 9. Common mistakes
- Believing a CTE is computed once → expensive query runs per reference → temp table.
- `ORDER BY` inside a view or CTE to sort output → not guaranteed → sort in the outer query.
- Missing `;` before `WITH` → syntax error 319 → terminate the previous statement.
- Mismatched types in recursive CTE members → error 240 → `CAST` both sides.
- Infinite loop in cyclic data → error 530 after 100 levels → track the path and stop on cycles.
- `SELECT *` in views → stale column list → list columns, use SCHEMABINDING.

### 10. 🎯 Interview kit
**30-second answer:** Table expressions are named queries used as tables: derived tables and CTEs live in one statement; views and inline TVFs are stored objects, and TVFs take parameters. SQL Server inlines them into the outer query, so they don't store data or speed things up by themselves. A CTE referenced twice runs twice. Recursive CTEs use an anchor and a recursive member with UNION ALL, limited by MAXRECURSION.

**Structure a longer answer:** what they are and the three rules (no order, named columns, unique names) → inlining → CTE vs. derived table vs. temp table → recursion mechanics → a case where materializing into a temp table fixed performance.

**Likely follow-ups:**
- *CTE vs. temp table?* → CTE is inlined, no stats, re-evaluated per reference; temp table is materialized, has stats and indexes.
- *Inline vs. multi-statement TVF?* → inline is optimized with the outer query; mTVF is a black box with poor estimates.
- *How does recursion stop?* → when the recursive member returns no rows, or MAXRECURSION is hit.
- *CROSS vs. OUTER APPLY?* → like inner vs. left join with a correlated table expression.

**Signals interviewers look for:** you know CTEs aren't materialized; you reach for APPLY for top-N-per-group; you know mTVF estimate problems.

### 11. Check yourself
- [ ] Rewrite a nested derived-table query as chained CTEs and confirm the plans are identical.
- [ ] Write a recursive CTE that returns a manager's whole subtree with level and path.
- [ ] Show with `STATISTICS IO` that a CTE referenced twice reads the table twice.
- [ ] Build an iTVF and use it with CROSS APPLY for top 3 per group.
- [ ] Explain the three requirements every table expression must meet.
<!-- explainer:end -->

## References

- *Beginning T-SQL* — Kathi Kellenberger, Lee Everest — Building on Subqueries, Common Table Expressions, and Unions › Using Derived Tables and Common Table Expressions
- *T-SQL Fundamentals (4th ed.)* — Itzik Ben-Gan — Table expressions
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 4: Joins, Set Operations, and Subqueries › Derived Tables and Common Table Expressions (CTEs)
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 4: Joins, Set Operations, and Subqueries › Derived Tables versus CTEs
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 4: Joins, Set Operations, and Subqueries › Common Table Expressions (CTEs)
- *Beginning T-SQL* — Kathi Kellenberger, Lee Everest — Writing Advanced Queries › Common Table Expressions
- *Beginning T-SQL* — Kathi Kellenberger, Lee Everest — Building on Subqueries, Common Table Expressions, and Unions › Using Common Table Expressions
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 6: CTEs, MERGE, Pivoting, JSON, and XML › Writing Recursive and Non-Recursive CTEs
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 6: CTEs, MERGE, Pivoting, JSON, and XML › Using CTEs vs. Temp Tables vs. Views
- *Beginning T-SQL* — Kathi Kellenberger, Lee Everest — Writing Advanced Queries › Advanced CTE Queries

