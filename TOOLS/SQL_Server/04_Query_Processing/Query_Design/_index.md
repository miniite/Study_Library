---
id: "TOOLS/SQL_Server/04_Query_Processing/Query_Design"
level: 4
type: "curriculum"
curriculum: "sql_server"
---

# Query Design for Performance

**L4 Advanced** · [↑ Query Processing and Tuning](<../_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** How you write a query decides what the optimizer *can* do. SARGable predicates, matching data types, small result sets and simple, honest logic let it use indexes and estimate well. Functions on columns, implicit conversions, nested views and needless work take those options away.

### 1. The big picture
Indexes and statistics give the optimizer *possibilities*. The query text decides whether it can use them. Two queries that return the same rows can differ by 1,000× in reads simply because one hides the column inside a function or compares it with the wrong data type. This lesson collects the query-writing rules that matter most for performance in SQL Server, and explains *why* each one matters in terms of the earlier lessons (seeks vs scans, histogram estimates, plan reuse).

The rules fall into five groups:
1. **Make predicates SARGable** so indexes can be used for seeks and histograms for estimates.
2. **Avoid implicit conversions** (match types).
3. **Do less work**: return only the rows and columns needed, avoid unnecessary sorts, DISTINCT and UNION.
4. **Keep the logic visible to the optimizer**: avoid deeply nested views, multi-statement functions and scalar UDFs where they hide work.
5. **Help plan reuse and stability**: parameterize, schema-qualify names, keep SET options consistent, use `SET NOCOUNT ON`.

### 2. Core concepts and vocabulary
- **SARGable (Search ARGument-able)**: a predicate the engine can use to *seek* an index range: `column operator expression`, where the column is bare. Examples: `=`, `>`, `<`, `>=`, `<=`, `BETWEEN`, `LIKE 'abc%'`, `IN (…)` (list of equalities).
- **Non-SARGable**: the column is wrapped or the pattern prevents a range: `YEAR(col) = 2024`, `col + 1 = 5`, `LEFT(col, 3) = 'abc'`, `LIKE '%abc'`, `ISNULL(col, 0) = 0`, `<>` / `NOT IN` (usually scans), comparisons that need an implicit conversion of the column.
- **Implicit conversion**: SQL Server converts one side automatically to match types. It follows **data type precedence**: the lower-precedence type is converted. If the *column* is converted (for example `varchar` column compared with an `nvarchar` value), the index can't be used for a seek on most collations.
- **Residual predicate**: a filter evaluated on every row read, rather than used to seek.
- **Row goal**: with `TOP`, `EXISTS`, `IN (subquery)` or `FAST n`, the optimizer plans to stop early, preferring plans that produce first rows quickly.
- **Nested view**: a view selecting from other views. Every layer gets expanded into one huge query that the optimizer may not be able to simplify.

### 3. How it works, step by step
**Why a function on a column blocks a seek:** an index on `OrderDate` is a B-tree sorted by `OrderDate`. To find `OrderDate` values in 2024, the engine can seek to `2024-01-01` and read until `2025-01-01`. To find rows where `YEAR(OrderDate) = 2024`, it would need an index sorted by `YEAR(OrderDate)`, which doesn't exist. So it must read every row, compute `YEAR()`, and filter. The histogram also describes `OrderDate`, not `YEAR(OrderDate)`, so the estimate becomes a guess.

**Rewrite rules:**
| Non-SARGable | SARGable rewrite |
|---|---|
| `WHERE YEAR(OrderDate) = 2024` | `WHERE OrderDate >= '20240101' AND OrderDate < '20250101'` |
| `WHERE CONVERT(date, OrderDateTime) = '20240315'` | `>= '20240315' AND < '20240316'` (note: casting `datetime` to `date` is a special case that *can* seek, but the range is still clearer) |
| `WHERE LEFT(LastName, 3) = 'Smi'` | `WHERE LastName LIKE 'Smi%'` |
| `WHERE Price * 1.2 > 120` | `WHERE Price > 100` |
| `WHERE ISNULL(Status, 'X') = 'X'` | `WHERE Status = 'X' OR Status IS NULL` |
| `WHERE DATEDIFF(day, OrderDate, GETDATE()) < 30` | `WHERE OrderDate > DATEADD(day, -30, GETDATE())` |
| `WHERE AccountNumber = 12345` (column is varchar) | `WHERE AccountNumber = '12345'` |
| `WHERE Email LIKE '%@contoso.com'` | Store reversed or domain column, or full-text search |

**Other query-design rules and their reasons:**
| Rule | Reason |
|---|---|
| Select only needed columns (no `SELECT *`) | Allows covering indexes; less I/O, memory and network; avoids Key Lookups |
| Filter early, return only needed rows | Fewer rows through every operator |
| `EXISTS` instead of `COUNT(*) > 0` | EXISTS stops at the first match (row goal) |
| `UNION ALL` instead of `UNION` when duplicates are impossible or acceptable | `UNION` adds a sort or hash to remove duplicates |
| Avoid unnecessary `DISTINCT` and `ORDER BY` | Each can add a Sort or Hash Aggregate |
| `NOT EXISTS` instead of `NOT IN (subquery)` on nullable columns | `NOT IN` with a NULL returns no rows and forces extra work |
| Don't nest views on views | The expanded query is huge; redundant joins may not be removed |
| Prefer inline TVFs to multi-statement TVFs | Inline TVFs are expanded like views; MSTVFs are black boxes |
| Avoid scalar UDFs in SELECT/WHERE on big sets (unless inlined) | Executed per row; block parallelism |
| Join on columns with matching types | Avoid conversions and enable merge/hash on the key |
| `SET NOCOUNT ON` in procedures | Removes "n rows affected" messages, which means less network chatter |
| Don't name procedures `sp_…` | SQL Server checks `master` first: an extra lookup and a risk of name clashes |
| Schema-qualify objects (`dbo.Orders`) | Plan reuse across users; faster name resolution |
| Avoid query hints unless proven | Freeze today's choice (see Query Optimizer) |

### 4. Worked example
```sql
-- Index for the demo
CREATE INDEX IX_SOH_OrderDate ON Sales.SalesOrderHeader (OrderDate);

SET STATISTICS IO ON;

-- 1. Non-SARGable
SELECT SalesOrderID, OrderDate
FROM Sales.SalesOrderHeader
WHERE YEAR(OrderDate) = 2013;
-- plan: Index Scan of IX_SOH_OrderDate (all rows read), Predicate = YEAR(...)

-- 2. SARGable
SELECT SalesOrderID, OrderDate
FROM Sales.SalesOrderHeader
WHERE OrderDate >= '20130101' AND OrderDate < '20140101';
-- plan: Index Seek, Seek Predicate on the range; reads drop sharply
```
`SalesOrderID` is the clustered key, so it is already in the nonclustered index, and both queries are covered. The only difference is the predicate shape.

**Implicit conversion trap:**
```sql
-- AccountNumber is nvarchar? varchar? Check first:
SELECT c.name, t.name AS type_name, c.max_length
FROM sys.columns AS c JOIN sys.types AS t ON t.user_type_id = c.user_type_id
WHERE c.object_id = OBJECT_ID(N'Sales.SalesOrderHeader') AND c.name = N'AccountNumber';

DECLARE @acct nvarchar(30) = N'10-4030-018749';
SELECT SalesOrderID FROM Sales.SalesOrderHeader WHERE AccountNumber = @acct;
```
If the column were `varchar` and the parameter `nvarchar`, the root SELECT shows a warning: *Type conversion in expression (CONVERT_IMPLICIT(nvarchar…, [AccountNumber])) may affect "SeekPlan"*. The fix is to declare the parameter with the column's type, **in the application too** (ORMs often send all strings as `nvarchar`).

**EXISTS vs COUNT:**
```sql
-- Does this customer have any orders?
IF (SELECT COUNT(*) FROM Sales.SalesOrderHeader WHERE CustomerID = 29825) > 0 …  -- counts all
IF EXISTS (SELECT 1 FROM Sales.SalesOrderHeader WHERE CustomerID = 29825) …     -- stops at first
```

### 5. Under the hood
- **Data type precedence** decides which side is converted. For example `int` beats `varchar`, so `varcharColumn = 12345` converts the **column** to int, row by row. That can't seek and fails at run time if any row holds non-numeric text. `nvarchar` beats `varchar`, so an `nvarchar` parameter converts a `varchar` column. With Windows collations SQL Server can still do a *dynamic seek* (range from `GetRangeThroughConvert`), but with SQL collations it scans, and estimates suffer either way.
- **Some functions are SARGable by design**: `CAST(datetimeCol AS date) = @d` uses an internal range (`GetRangeThroughConvert`). It's the exception, not the rule, and the estimates can still be off.
- **`LIKE 'abc%'`** becomes a range seek `>= 'abc' AND < 'abd'`. `LIKE @p` with a parameter is turned into a dynamic range at run time (LikeRangeStart/End).
- **`OR` across different columns** (`WHERE a = 1 OR b = 2`) can use *index union* (two seeks + concatenation + distinct) if both columns are indexed. Otherwise it scans. Rewriting as `UNION` of two queries sometimes helps.
- **Computed columns** fix unavoidable expressions: add `ALTER TABLE … ADD OrderYear AS YEAR(OrderDate)`, index it, and the optimizer can match `YEAR(OrderDate) = 2013` to that index (the expression must match). Statistics are created on the computed column too.
- **Nested views** are expanded inline at bind time. The optimizer can remove unused joins only when constraints prove they are redundant (trusted FKs, unique keys). Otherwise every join stays, and very large trees can hit an optimizer timeout.

### 6. Performance and design implications
- Check predicates in every new query: is the column bare? Do the types match?
- Look for the *Type conversion…* warning and residual predicates in plans. They are cheap to fix.
- Measure rewrites with logical reads and CPU time. Some "rules" (for example `EXISTS` vs `IN`) often produce identical plans in modern versions, so verify rather than assume.
- Design the schema for the query: correct data types (dates as `date`/`datetime2`, numbers as numeric types, not strings), computed columns for common expressions, constraints the optimizer can trust.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `SET STATISTICS IO, TIME ON` | Before/after numbers for rewrites | Every rewrite |
| Plan warning *Type conversion in expression* | Shows implicit conversions affecting seeks | Plan review |
| `ALTER TABLE … ADD col AS (expr) [PERSISTED]` + index | Indexable expression | Unavoidable function predicates |
| `SET NOCOUNT ON` | Suppresses row-count messages | Top of every procedure |
| `sys.dm_exec_query_stats` + `query_plan` LIKE `'%CONVERT_IMPLICIT%'` | Finds cached plans with conversions | Workload-wide search (run carefully) |
| `sys.sql_expression_dependencies` | Shows view-on-view nesting | Untangling nested views |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| Range predicates instead of functions | Readability suffers a little; comment the intent |
| Computed column + index | Function can't be removed from app queries; small write cost |
| Views for security/simplicity | Nesting more than one or two levels |
| Inline TVF | Logic needs multiple statements (then consider a temp table in a procedure) |
| `UNION ALL` | Duplicates must really be removed |
| Splitting a monster query using temp tables | Each step adds I/O; use it when the optimizer times out or estimates collapse |

### 9. Common mistakes
- Wrapping indexed columns in functions → scans and guessed estimates → move the function to the other side or use a computed column.
- Parameters typed differently from columns (often from ORMs) → implicit conversion, scan → match types and lengths.
- `SELECT *` in application queries → no covering indexes, more lookups, breaks when columns change → list columns.
- `NOT IN` against a nullable subquery → unexpected empty result → `NOT EXISTS`.
- Views built on views built on views → huge plans, timeouts, redundant joins → flatten, query base tables.
- `sp_` prefix on user procedures → extra master lookup → use another prefix or none.

### 10. 🎯 Interview kit
**30-second answer:** "I write predicates so the column stays bare and types match, so the optimizer can seek and use the histogram. For example, a date range instead of YEAR(col), and parameters typed like the column to avoid CONVERT_IMPLICIT. I return only needed rows and columns, prefer EXISTS, UNION ALL and inline TVFs, avoid nested views and scalar UDFs on large sets, and I measure every rewrite with logical reads and the plan."
**Structure a longer answer:** what SARGable means and why (B-tree order, histogram) → common anti-patterns with rewrites → implicit conversion and precedence → doing less work → verify with plans/IO.
**Likely follow-ups:** *Why does `varcharCol = N'abc'` hurt?* → nvarchar has higher precedence, so the column gets converted. *How do you index YEAR(OrderDate)?* → computed column + index, or rewrite as a range. *Is `IN` slower than `EXISTS`?* → often the same plan; `NOT IN` vs `NOT EXISTS` differs with NULLs.
**Signals interviewers look for:** explains *why* (index order, statistics) rather than reciting rules; knows type precedence; verifies with measurements.

### 11. Check yourself
- [ ] Rewrite five non-SARGable predicates from the table and confirm seeks in the plans.
- [ ] Reproduce a `CONVERT_IMPLICIT` warning and fix it.
- [ ] Create a computed column for `YEAR(OrderDate)`, index it, and show the original query using it.
- [ ] Show `NOT IN` returning no rows when the subquery contains a NULL.
- [ ] Explain why nested views can make the optimizer time out.
<!-- explainer:end -->

## References

- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 8: Query Optimization and Performance Tuning › Introduction
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 8: Query Optimization and Performance Tuning › Practical Example: Comparing Plans
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 8: Query Optimization and Performance Tuning › Conclusion
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 16: Best Practices for T-SQL Development and Database Design › Query Design Best Practices
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 8: Query Optimization and Performance Tuning › Best Practices Recap
- *SQL Server 2025 Query Performance Tuning (7th ed.)* — Grant Fritchey — FULL_BOOK_NEEDS_MANUAL_SPLIT (cont.) › Chapter 21 Intelligent Query Processing
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 3: SELECT Statements and Filtering Logic › Structure
- *SQL Server 2025 Query Performance Tuning (7th ed.)* — Grant Fritchey — FULL_BOOK_NEEDS_MANUAL_SPLIT
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 16: Best Practices for T-SQL Development and Database Design › Data Modeling for Performance
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 3: SELECT Statements and Filtering Logic › Performance Considerations

