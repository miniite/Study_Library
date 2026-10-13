---
id: "TOOLS/SQL_Server/T-SQL/Dialects_and_Portability/02_ANSI_Core_vs_T-SQL_Extensions"
level: 5
type: "concept"
topic: "TOOLS/SQL_Server/T-SQL/Dialects_and_Portability"
tools: ["SQL_Server"]
---

# 2 · ANSI Core vs T-SQL Extensions

**L5** · [↑ Series index](<_index.md>) · Prev: [Standard, Dialect and Engine](<01_Standard_Dialect_Engine.md>) · Next: [Syntax Translation Cookbook](<03_Syntax_Translation_Cookbook.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** Every T-SQL construct falls into one of four buckets — portable core, T-SQL-only extension, standard feature SQL Server lacks, or standard name with different behaviour — and knowing the bucket tells you whether a statement will survive a move to another engine.

### 1. The four buckets

| Bucket | Symbol | Rule of thumb |
|---|---|---|
| **A. Portable core** | ✅ | Standard syntax, SQL Server implements it faithfully |
| **B. T-SQL extension** | ⚠ | Only the Microsoft family accepts it |
| **C. Standard feature SQL Server lacks** | ❌ | Valid elsewhere, a syntax or bind error in T-SQL |
| **D. Same name, different behaviour** | 🔀 | Parses everywhere, returns different data (see note 4) |

This note is a **catalogue**. Use it as a lookup when reading or porting SQL.

### 2. Bucket A — portable core (what you can rely on)

| Area | Constructs | Notes |
|---|---|---|
| Querying | `SELECT … FROM … WHERE … GROUP BY … HAVING … ORDER BY`, `DISTINCT`, `AS` aliases, `CASE`, `BETWEEN`, `IN`, `LIKE`, `EXISTS`, scalar and correlated subqueries | Bedrock |
| Joins | `INNER`, `LEFT/RIGHT/FULL OUTER`, `CROSS JOIN`, `ON` | Use explicit syntax; old `*=` outer join operators are removed |
| Set operators | `UNION [ALL]`, `INTERSECT`, `EXCEPT` | Oracle calls `EXCEPT` → `MINUS` |
| Table expressions | CTEs (`WITH`), derived tables, views | *Recursive* CTE: SQL Server omits the `RECURSIVE` keyword |
| Windows | `ROW_NUMBER`, `RANK`, `DENSE_RANK`, `NTILE`, `LAG`, `LEAD`, `SUM() OVER (… ROWS BETWEEN …)` | 2012+ for full frames |
| Aggregates | `COUNT`, `SUM`, `AVG`, `MIN`, `MAX`, `GROUPING SETS`, `ROLLUP`, `CUBE` | |
| DML | `INSERT … VALUES`, `INSERT … SELECT`, `UPDATE`, `DELETE`, `MERGE` | `MERGE` exists in the standard but has SQL Server-specific pitfalls |
| DDL | `CREATE/ALTER/DROP TABLE`, constraints (`PRIMARY KEY`, `FOREIGN KEY`, `UNIQUE`, `CHECK`, `DEFAULT`), `CREATE VIEW`, `CREATE INDEX` (index DDL itself is not in the standard but is universal) | |
| Functions | `CAST`, `COALESCE`, `NULLIF`, `CURRENT_TIMESTAMP`, `COUNT(DISTINCT x)`, `UPPER`, `LOWER`, `TRIM`(2017+), `SUBSTRING` (see ❌), `POWER`, `ABS`, `CEILING`, `FLOOR` | |
| Metadata | `INFORMATION_SCHEMA.TABLES / COLUMNS / ROUTINES …` | Thin compared with `sys.*` |
| Transactions | `BEGIN/COMMIT/ROLLBACK` (T-SQL spells `BEGIN TRANSACTION`), isolation levels | |

### 3. Bucket B — T-SQL extensions (⚠ rewrite when porting)

| Extension | What it does | Standard / other-engine equivalent |
|---|---|---|
| `TOP (n) [PERCENT] [WITH TIES]` | Limit rows | `FETCH FIRST n ROWS ONLY` (`WITH TIES` also standard), `LIMIT n` |
| `CROSS APPLY` / `OUTER APPLY` | Join to a correlated table expression | `LATERAL` (standard, Postgres, Oracle 12c+, MySQL 8.0.14+, Snowflake) |
| `OUTPUT` clause | Return rows affected by DML | `RETURNING` (Postgres/Oracle/MariaDB) |
| `PIVOT` / `UNPIVOT` | Rotate rows ↔ columns | Conditional aggregation; some engines have their own `PIVOT` |
| `UPDATE … FROM … JOIN`, `DELETE … FROM … JOIN` | Join-based DML | correlated subquery, `UPDATE … FROM` (Postgres), `MERGE` |
| `SELECT … INTO newtable` | Create-and-populate | `CREATE TABLE … AS SELECT` (CTAS) |
| `@variables`, `DECLARE`, `SET @x = …`, `SELECT @x = col` | Local variables | `DECLARE`/`SET` in PSM; different in each engine |
| `#temp`, `##globaltemp`, `@tablevar` | Session/batch-scoped tables | `CREATE TEMP TABLE`, global temporary tables |
| `IDENTITY(seed, incr)`, `@@IDENTITY`, `SCOPE_IDENTITY()` | Auto-numbering | `GENERATED … AS IDENTITY`, `AUTO_INCREMENT`, sequences |
| `BEGIN TRY … END TRY BEGIN CATCH … END CATCH`, `THROW`, `RAISERROR` | Error handling | `EXCEPTION WHEN …`, `DECLARE HANDLER` |
| `GO` | Batch separator | Not SQL at all — client directive |
| Table hints `WITH (NOLOCK, UPDLOCK, ROWLOCK, INDEX(…))` | Execution control | Engine-specific hints / `FOR UPDATE SKIP LOCKED` |
| Query hints `OPTION (RECOMPILE, MAXDOP 1, OPTIMIZE FOR …)` | Optimizer control | Engine-specific |
| `DATEADD`, `DATEDIFF`, `DATEPART`, `EOMONTH`, `FORMAT` | Date arithmetic | `INTERVAL` arithmetic, `EXTRACT`, `date_trunc` |
| `ISNULL`, `IIF`, `CHOOSE`, `CHARINDEX`, `LEN`, `STUFF`, `PATINDEX` | Convenience functions | `COALESCE`, `CASE`, `POSITION`, `CHAR_LENGTH`, … |
| `+` for string concatenation | Concat | `\|\|` (standard) |
| `[bracketed identifiers]` | Quoting | `"double-quoted"` (standard) |
| `SET` options: `NOCOUNT`, `XACT_ABORT`, `DATEFORMAT`, `LANGUAGE`, `ROWCOUNT` | Session behaviour | Engine-specific |
| `sp_*`, `xp_*` system procedures, `sys.*` DMVs | Admin & diagnostics | Engine-specific catalogs |
| `CREATE OR ALTER` (2016 SP1+), `DROP … IF EXISTS` (2016+) | Idempotent DDL | `CREATE OR REPLACE` |
| Computed columns `col AS (expr) [PERSISTED]` | Derived column | `GENERATED ALWAYS AS (expr) [STORED]` |
| `TRUNCATE TABLE` | Fast delete | Standard-ish; semantics (rollback, locks) vary |
| `MATCH(...)` graph syntax, `FOR XML`, `FOR JSON` | Graph, XML/JSON shaping | SQL/PGQ (2023), SQL/XML, SQL/JSON differ |

### 4. Bucket C — standard (or common) features SQL Server lacks

| Missing feature | Where it exists | T-SQL workaround |
|---|---|---|
| `LIMIT n [OFFSET m]` | Postgres, MySQL, SQLite, Snowflake, BigQuery | `TOP`, `OFFSET … FETCH` |
| `BOOLEAN` type & boolean expressions as values | Postgres, MySQL (alias), standard | `bit` + `CASE WHEN … THEN 1 ELSE 0 END` |
| `INTERVAL` type | Standard, Postgres, Oracle, MySQL | `DATEADD`/`DATEDIFF` with units |
| `CURRENT_DATE`, `LOCALTIMESTAMP` | Standard, Postgres, MySQL, Oracle | `CAST(SYSDATETIME() AS date)` |
| `EXTRACT(part FROM d)` | Standard | `DATEPART(part, d)` |
| `RETURNING` | Postgres, Oracle, MariaDB | `OUTPUT` |
| `INSERT … ON CONFLICT` / `ON DUPLICATE KEY` | Postgres / MySQL | `MERGE`, or `UPDATE` + `INSERT … WHERE NOT EXISTS` with correct locking |
| `NULLS FIRST / NULLS LAST` | Standard, Postgres, Oracle | `ORDER BY CASE WHEN col IS NULL THEN 1 ELSE 0 END, col` |
| `FILTER (WHERE …)` on aggregates | Standard, Postgres, SQLite | `SUM(CASE WHEN … THEN x END)` |
| `COUNT(DISTINCT x) OVER (…)` / `DISTINCT` in windowed aggregates | Oracle (some), Postgres (no), others vary | Pre-aggregate; `DENSE_RANK` trick; two-step CTE |
| `WITH RECURSIVE` keyword | Standard, Postgres, MySQL 8 | Omit the keyword in SQL Server |
| Row-value comparison `(a,b) IN ((1,2),(3,4))`, `(a,b) > (x,y)` | Standard, Postgres, MySQL | `EXISTS`, joined `VALUES`, expanded `AND/OR` |
| Array / composite types | Postgres, Snowflake, BigQuery | Child table, JSON, `STRING_SPLIT`, TVP |
| `CREATE TABLE … AS SELECT`, `CREATE TABLE … LIKE` | Postgres, MySQL, Oracle, Snowflake | `SELECT INTO`; scripting |
| `SELECT … FOR UPDATE [SKIP LOCKED]` | Postgres, Oracle, MySQL 8 | `WITH (UPDLOCK, ROWLOCK, READPAST)` |
| `SUBSTRING(s FROM n)` / two-argument `SUBSTRING` | Standard, Postgres, MySQL | `SUBSTRING(s, n, LEN(s))` — three arguments required |
| `ILIKE`, regular-expression operators (`~`, `REGEXP`) | Postgres, MySQL | `LIKE` with collation; `PATINDEX`; CLR; (very recent releases add regex functions — *verify* your version) |
| `DISTINCT ON (col)` | Postgres | `ROW_NUMBER() … WHERE rn = 1` |
| `GENERATE_SERIES` | Postgres | 2022+ has it; earlier: tally/numbers table |
| `QUALIFY` | Snowflake, BigQuery, Teradata | wrap windowed expression in CTE and filter |
| Named `WINDOW` clause | Standard, Postgres, MySQL 8 | 2022+ supports it (compat level 160) |

> 🧪 **Version drift.** Several rows above have *changed* in recent releases (e.g. `IS DISTINCT FROM`, `GENERATE_SERIES`, `WINDOW`, `GREATEST/LEAST` arrived in 2022). A "missing" feature on an old compatibility level may exist after an upgrade. Always check `SELECT @@VERSION` and `compatibility_level`.

### 5. Bucket D — standard names that behave differently (preview)

Covered fully in [note 4](<04_Same_Syntax_Different_Semantics.md>). The headline items:

| Item | T-SQL behaviour | Contrast |
|---|---|---|
| `''` vs `NULL` | Distinct values | Oracle treats `''` as `NULL` |
| `timestamp` type | Row-version counter (`binary(8)`) | Standard `TIMESTAMP` is a date-time |
| `UNIQUE` constraint and NULL | Allows only **one** NULL | Standard/Postgres/Oracle/MySQL allow many |
| `ISNULL(a,b)` | Result type = type of `a` | `COALESCE`: highest-precedence type |
| `varchar` without length | `1` in declarations, `30` in `CAST` | Most engines: unlimited or error |
| `DATEDIFF(year, …)` | Counts boundary crossings | Other engines: elapsed time |
| String `=` comparison | Trailing spaces ignored; case-insensitive by default collation | Postgres/Oracle: case-sensitive; `=` is exact |
| Date-literal parsing for `datetime` | Depends on `SET LANGUAGE`/`DATEFORMAT` | ISO 8601 `date` is safe |

### 6. Worked example — classify and port a real query

```sql
-- Original T-SQL (an order report)
SELECT TOP (10) WITH TIES
       c.CustomerID,
       ISNULL(c.Region, 'n/a')                       AS Region,
       DATEDIFF(day, MIN(o.OrderDate), GETDATE())    AS DaysSinceFirst,
       STRING_AGG(o.OrderID, ',')                    AS OrderIDs
FROM   Sales.Customers AS c
CROSS APPLY (SELECT TOP (3) OrderID, OrderDate
             FROM   Sales.Orders AS o2
             WHERE  o2.CustomerID = c.CustomerID
             ORDER  BY OrderDate DESC) AS o
GROUP  BY c.CustomerID, c.Region
ORDER  BY MIN(o.OrderDate);
```

**Classification line by line**

| Fragment | Bucket | Portable rewrite |
|---|---|---|
| `TOP (10) WITH TIES` | ⚠ B | `FETCH FIRST 10 ROWS WITH TIES` (standard) |
| `ISNULL(c.Region,'n/a')` | ⚠ B / 🔀 D | `COALESCE(c.Region,'n/a')` |
| `DATEDIFF(day, MIN(…), GETDATE())` | ⚠ B | `CURRENT_DATE - MIN(…)` (Postgres), `DATEDIFF('day', …)` (Snowflake), etc. |
| `STRING_AGG(o.OrderID, ',')` | ✅/⚠ | `string_agg` (Postgres), `LISTAGG` (Oracle/Snowflake), `GROUP_CONCAT` (MySQL) — cast the number to text first on some engines |
| `CROSS APPLY (SELECT TOP 3 …)` | ⚠ B | `CROSS JOIN LATERAL (SELECT … ORDER BY … LIMIT 3)` |
| `Sales.Customers AS c` | ✅ A | — |
| `GROUP BY` / `ORDER BY MIN(...)` | ✅ A | — |

**Portable (PostgreSQL / standard) form**

```sql
SELECT c.customer_id,
       COALESCE(c.region, 'n/a')               AS region,
       CURRENT_DATE - MIN(o.order_date)::date  AS days_since_first,
       STRING_AGG(o.order_id::text, ',')       AS order_ids
FROM   sales.customers AS c
CROSS  JOIN LATERAL (SELECT order_id, order_date
                     FROM   sales.orders o2
                     WHERE  o2.customer_id = c.customer_id
                     ORDER  BY order_date DESC
                     LIMIT  3) AS o
GROUP  BY c.customer_id, c.region
ORDER  BY MIN(o.order_date)
FETCH  FIRST 10 ROWS WITH TIES;      -- PostgreSQL 13+
```

The skeleton (select list, grouping, ordering) did not change — only the *extension* fragments were swapped. That is the pattern: **extensions cluster in row-limiting, null handling, date arithmetic, string aggregation and lateral joins**.

### 7. Under the hood — why these extensions exist at all

1. **Heritage.** `@vars`, `#temp`, `GO`, `TOP` came from Sybase before the standard defined equivalents; they were never removed because applications depend on them.
2. **Procedural tight coupling.** The standard's SQL/PSM arrived late and was rarely adopted; T-SQL grew its own procedural syntax, so batch, variable and error-handling forms are non-standard.
3. **Engine-specific concepts.** `NOLOCK`, `OUTPUT`, `OPTION`, plan-cache directives, `sys.dm_*` exist because they expose engine behaviour the standard deliberately says nothing about.
4. **Convenience.** Functions such as `DATEADD`, `CHARINDEX` and `LEN` were shortcuts before standard `INTERVAL` and `POSITION` were commonly implemented.

### 8. Performance and design implications

- **Isolate extensions.** Put `TOP`, `APPLY`, hints and date functions in views / inline TVFs / stored procedures; keep the *interface* (columns and parameters) portable.
- **`SET FIPS_FLAGGER`** (deprecated but functional): `SET FIPS_FLAGGER 'ENTRY'; <statement>` makes SQL Server emit a warning for each construct outside the chosen FIPS 127-2 level — a cheap first-pass portability audit.
- **Do not contort.** Replacing `APPLY` with a `ROW_NUMBER()` join *only* for portability can cost performance; weigh it.

### 9. Common mistakes

- Using `ISNULL` and `COALESCE` interchangeably → silent truncation (note 4).
- Writing `LIMIT` after years of MySQL → `Msg 102` near the number.
- Using `+` for strings with possibly-NULL columns → whole result NULL; use `CONCAT` / `CONCAT_WS`.
- Assuming `INFORMATION_SCHEMA` shows everything → it omits many SQL Server-specific objects.
- Using `GETDATE()` where a time zone is needed → `SYSDATETIMEOFFSET()`/`AT TIME ZONE`.

### 10. 🎯 Interview kit

**30-second answer:** "I split SQL into portable core, T-SQL extensions, features SQL Server lacks, and look-alikes with different behaviour. Extensions mostly cluster in row-limiting (`TOP`), null functions (`ISNULL`), dates (`DATEADD`), string aggregation, lateral joins (`APPLY`) and procedural code."

**Likely follow-ups:** *How would you paginate portably?* → `ORDER BY … OFFSET … FETCH` (SQL Server 2012+, Oracle 12c+, Postgres, DB2), or keyset pagination for large offsets. *`APPLY` vs `LATERAL`?* → same concept; `LATERAL` is standard, `APPLY` is Microsoft's spelling with `CROSS` = inner, `OUTER` = left. *Name three standard features SQL Server lacks.* → pick from bucket C.

### 11. Check yourself

- [ ] Classify each of `TOP`, `COALESCE`, `OUTPUT`, `OFFSET/FETCH`, `LIMIT`, `GROUP BY ROLLUP` into A/B/C.
- [ ] Rewrite `SELECT TOP 5 …` in three other dialects.
- [ ] Why is `''` vs `NULL` a bucket-D problem and not a syntax problem?
- [ ] Show how `SET FIPS_FLAGGER` helps and why it is only a first pass.

**Where this connects:** [T-SQL Basics](<../T-SQL_Basics.md>) · [CROSS APPLY and OUTER APPLY](<../CROSS_APPLY_and_OUTER_APPLY.md>) · [Window Functions](<../../06_T-SQL_Programming/Window_Functions/_index.md>) · Next → [Syntax Translation Cookbook](<03_Syntax_Translation_Cookbook.md>)
<!-- explainer:end -->
