---
id: "TOOLS/SQL_Server/T-SQL/Dialects_and_Portability/03_Syntax_Translation_Cookbook"
level: 4
type: "concept"
topic: "TOOLS/SQL_Server/T-SQL/Dialects_and_Portability"
tools: ["SQL_Server"]
---

# 3 · Syntax Translation Cookbook

**L4** · [↑ Series index](<_index.md>) · Prev: [ANSI Core vs T-SQL Extensions](<02_ANSI_Core_vs_T-SQL_Extensions.md>) · Next: [Same Syntax, Different Semantics](<04_Same_Syntax_Different_Semantics.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** Most dialect differences concentrate in about fifteen everyday tasks; learn the recipe for each in T-SQL, PostgreSQL, MySQL, Oracle and the cloud-warehouse dialects and you can read or port almost any query.

**Dialect columns:** **T** = T-SQL · **PG** = PostgreSQL · **MY** = MySQL 8 · **OR** = Oracle 12c+ · **SF** = Snowflake · **SP** = Spark SQL / Databricks. Where a cell says *same*, the T-SQL form works unchanged.

### 1. Recipe index

| # | Task | # | Task |
|---|---|---|---|
| R1 | Limit / paginate rows | R9 | Upsert |
| R2 | Concatenate strings | R10 | Get changed rows back from DML |
| R3 | Handle NULLs | R11 | UPDATE / DELETE with a join |
| R4 | Current date/time | R12 | Create table from query |
| R5 | Date arithmetic | R13 | Temp tables & variables |
| R6 | Type conversion | R14 | Identifiers: quoting and case |
| R7 | String aggregation | R15 | Auto-numbering keys |
| R8 | Conditional logic | R16 | Row numbering, top-N per group |

### R1 · Limit and paginate rows

| | Syntax |
|---|---|
| **T** | `SELECT TOP (10) * FROM t ORDER BY c;` &nbsp;·&nbsp; `ORDER BY c OFFSET 20 ROWS FETCH NEXT 10 ROWS ONLY;` (needs `ORDER BY`) |
| **PG** | `… ORDER BY c LIMIT 10 OFFSET 20;` or `FETCH FIRST 10 ROWS ONLY` |
| **MY** | `… ORDER BY c LIMIT 20, 10;` or `LIMIT 10 OFFSET 20` |
| **OR** | `… ORDER BY c OFFSET 20 ROWS FETCH NEXT 10 ROWS ONLY;` (12c+); older: nested `ROWNUM` |
| **SF** | `LIMIT 10 OFFSET 20`; also accepts `TOP 10` |
| **SP** | `LIMIT 10` (no `OFFSET` until recent versions) |

*Portable choice:* `OFFSET … FETCH` works on T-SQL 2012+, Oracle 12c+, PostgreSQL, Db2. Deep offsets are slow everywhere — use **keyset pagination**:

```sql
-- keyset page: pass the last seen (OrderDate, OrderID)
SELECT TOP (50) OrderID, OrderDate
FROM   Sales.Orders
WHERE  (OrderDate < @LastDate) OR (OrderDate = @LastDate AND OrderID < @LastID)
ORDER  BY OrderDate DESC, OrderID DESC;
```

### R2 · Concatenate strings

| | Syntax | NULL behaviour |
|---|---|---|
| **T** | `a + b`, `CONCAT(a, b)`, `CONCAT_WS(',', a, b)` (2017+) | `+` → NULL if any operand NULL; `CONCAT` treats NULL as `''` |
| **PG** | `a \|\| b`, `CONCAT()` | `\|\|` → NULL; `CONCAT` ignores NULL |
| **MY** | `CONCAT(a, b)` (`\|\|` is logical OR unless `PIPES_AS_CONCAT`) | NULL if any NULL |
| **OR** | `a \|\| b` | NULL treated as `''` |
| **SF/SP** | `a \|\| b`, `CONCAT()` | NULL propagates for `\|\|`/`CONCAT` |

Number + string: **T** implicitly converts the *string* to a number (`'1' + 1 = 2`; `'a' + 1` → conversion error); **PG** errors; **MY** converts. Always `CAST` explicitly.

### R3 · Handle NULLs

| Need | T | PG | MY | OR |
|---|---|---|---|---|
| Replace NULL | `ISNULL(x,0)` / `COALESCE` | `COALESCE` | `IFNULL` / `COALESCE` | `NVL` / `COALESCE` |
| NULL if equal | `NULLIF(a,b)` | same | same | same |
| NULL-safe equal | `a = b OR (a IS NULL AND b IS NULL)`; `IS NOT DISTINCT FROM` (2022+) | `IS NOT DISTINCT FROM` | `a <=> b` | `DECODE(a,b,1,0)=1` |

*Portable choice:* **`COALESCE`** — it is in the standard, takes any number of arguments, and picks the highest-precedence type (not the first argument's type, unlike `ISNULL`).

### R4 · Current date and time

| Meaning | T | PG | MY | OR | SF |
|---|---|---|---|---|---|
| Now, local, server | `SYSDATETIME()`, `GETDATE()`, `CURRENT_TIMESTAMP` | `now()`, `CURRENT_TIMESTAMP` | `NOW()` | `SYSTIMESTAMP`, `CURRENT_TIMESTAMP` | `CURRENT_TIMESTAMP()` |
| Now, UTC | `SYSUTCDATETIME()`, `GETUTCDATE()` | `now() AT TIME ZONE 'UTC'` | `UTC_TIMESTAMP()` | `SYS_EXTRACT_UTC(SYSTIMESTAMP)` | `SYSDATE()` (UTC by default) |
| Today (date only) | `CAST(SYSDATETIME() AS date)` | `CURRENT_DATE` | `CURDATE()` | `TRUNC(SYSDATE)` | `CURRENT_DATE` |
| With offset | `SYSDATETIMEOFFSET()` | `now()` (timestamptz) | n/a | `SYSTIMESTAMP` | `CURRENT_TIMESTAMP` |

`CURRENT_DATE` is **not** available in T-SQL.

### R5 · Date arithmetic

| Task | T | PG | MY | OR |
|---|---|---|---|---|
| Add 7 days | `DATEADD(day, 7, d)` | `d + INTERVAL '7 days'` | `d + INTERVAL 7 DAY` | `d + 7` |
| Difference in days | `DATEDIFF(day, a, b)` | `b::date - a::date` | `DATEDIFF(b, a)` | `b - a` |
| Month start | `DATEFROMPARTS(YEAR(d), MONTH(d), 1)` or `DATETRUNC(month, d)` (2022+) | `date_trunc('month', d)` | `DATE_FORMAT(d,'%Y-%m-01')` | `TRUNC(d,'MM')` |
| Month end | `EOMONTH(d)` | `(date_trunc('month', d) + INTERVAL '1 month - 1 day')` | `LAST_DAY(d)` | `LAST_DAY(d)` |
| Extract part | `DATEPART(month, d)`, `MONTH(d)` | `EXTRACT(MONTH FROM d)` | `EXTRACT(MONTH FROM d)`, `MONTH(d)` | `EXTRACT(MONTH FROM d)` |
| Format | `FORMAT(d,'yyyy-MM-dd')`, `CONVERT(char(10), d, 23)` | `to_char(d,'YYYY-MM-DD')` | `DATE_FORMAT(d,'%Y-%m-%d')` | `TO_CHAR(d,'YYYY-MM-DD')` |

> ⚠ **`DATEDIFF` semantic trap:** `DATEDIFF(year,'2023-12-31','2024-01-01')` returns **1** — it counts calendar boundaries crossed, not elapsed time. See note 4.

### R6 · Type conversion

| Task | T | PG | MY | OR |
|---|---|---|---|---|
| Cast | `CAST(x AS int)`, `CONVERT(int, x)` | `CAST(x AS int)`, `x::int` | `CAST(x AS SIGNED)` | `CAST(x AS NUMBER)`, `TO_NUMBER(x)` |
| Safe cast (NULL on failure) | `TRY_CAST`, `TRY_CONVERT`, `TRY_PARSE` | (no built-in; custom function) | (silently converts) | `CAST(x AS NUMBER DEFAULT NULL ON CONVERSION ERROR)` (12.2+) |
| Date → string | `CONVERT(varchar, d, 112)` | `to_char` | `DATE_FORMAT` | `TO_CHAR` |
| String → date | `CONVERT(date, '20240131', 112)` | `to_date('20240131','YYYYMMDD')` | `STR_TO_DATE` | `TO_DATE` |

*Portable choice:* `CAST(x AS <type>)` with ISO-8601 literals (`'2024-01-31'`, `'2024-01-31T14:30:00'`).

### R7 · String aggregation

| | Syntax |
|---|---|
| **T** | `STRING_AGG(name, ', ') WITHIN GROUP (ORDER BY name)` (2017+); before: `FOR XML PATH('')` + `STUFF` |
| **PG** | `string_agg(name, ', ' ORDER BY name)` |
| **MY** | `GROUP_CONCAT(name ORDER BY name SEPARATOR ', ')` |
| **OR** | `LISTAGG(name, ', ') WITHIN GROUP (ORDER BY name)` |
| **SF** | `LISTAGG(name, ', ') WITHIN GROUP (ORDER BY name)` |
| **SP** | `array_join(collect_list(name), ', ')` / `concat_ws` |

### R8 · Conditional logic

| Need | T | Others |
|---|---|---|
| Searched `CASE` | `CASE WHEN … THEN … ELSE … END` | same everywhere ✅ |
| Inline if | `IIF(cond, a, b)` (2012+) | MY `IF(cond,a,b)`; SF `IFF`; PG/OR use `CASE` |
| Pick by index | `CHOOSE(2,'a','b','c')` | MY `ELT`; others `CASE` |
| Boolean as value | not allowed: use `CASE WHEN a=b THEN 1 ELSE 0 END` | PG/MY: `SELECT a = b` valid |
| Boolean filter | `WHERE flag = 1` (`bit`) | PG: `WHERE flag` |

### R9 · Upsert

```sql
-- T-SQL: MERGE (read the pitfalls note before using in concurrent systems)
MERGE dbo.Stock WITH (HOLDLOCK) AS t
USING (VALUES (@sku, @qty)) AS s(sku, qty) ON t.sku = s.sku
WHEN MATCHED THEN UPDATE SET t.qty = t.qty + s.qty
WHEN NOT MATCHED THEN INSERT (sku, qty) VALUES (s.sku, s.qty);
```
```sql
-- PostgreSQL
INSERT INTO stock (sku, qty) VALUES (:sku, :qty)
ON CONFLICT (sku) DO UPDATE SET qty = stock.qty + EXCLUDED.qty;
-- MySQL
INSERT INTO stock (sku, qty) VALUES (?, ?)
ON DUPLICATE KEY UPDATE qty = qty + VALUES(qty);
-- Oracle / Snowflake / Db2: MERGE INTO … USING … ON … WHEN MATCHED … WHEN NOT MATCHED …
```
*T-SQL alternative without `MERGE`:*

```sql
BEGIN TRAN;
UPDATE dbo.Stock WITH (UPDLOCK, SERIALIZABLE) SET qty += @qty WHERE sku = @sku;
IF @@ROWCOUNT = 0
    INSERT dbo.Stock (sku, qty) VALUES (@sku, @qty);
COMMIT;
```
See [MERGE Pitfalls and Error Handling](<../MERGE_Pitfalls_and_Error_Handling.md>) and [MERGE and Upserts](<../../SQL_Server_Platform/MERGE_and_Upserts.md>).

### R10 · Get affected rows back

| | Syntax |
|---|---|
| **T** | `INSERT … OUTPUT inserted.id, inserted.name VALUES …` · `UPDATE … OUTPUT deleted.x, inserted.x …` |
| **PG / OR** | `INSERT … RETURNING id` |
| **MY** | no equivalent; `LAST_INSERT_ID()` for single row |
| **SF** | `RETURNING` not supported for generic DML; query using `RESULT_SCAN` or `INSERT … ` then `SELECT` |

`OUTPUT` is more powerful than `RETURNING`: it exposes both `inserted.` and `deleted.` images and can write into a table (`OUTPUT … INTO`), and `MERGE … OUTPUT $action` reports which branch ran.

### R11 · UPDATE / DELETE with a join

```sql
-- T-SQL
UPDATE o SET o.Status = c.DefaultStatus
FROM   Sales.Orders AS o JOIN Sales.Customers AS c ON c.CustomerID = o.CustomerID
WHERE  o.Status IS NULL;
-- PostgreSQL
UPDATE sales.orders o SET status = c.default_status
FROM   sales.customers c WHERE c.customer_id = o.customer_id AND o.status IS NULL;
-- MySQL
UPDATE sales.orders o JOIN sales.customers c ON c.customer_id = o.customer_id
SET o.status = c.default_status WHERE o.status IS NULL;
-- Portable (ANSI correlated subquery; watch for multiple matches → error / arbitrary)
UPDATE orders SET status = (SELECT c.default_status FROM customers c WHERE c.customer_id = orders.customer_id)
WHERE status IS NULL AND EXISTS (SELECT 1 FROM customers c WHERE c.customer_id = orders.customer_id);
```
T-SQL's `UPDATE … FROM` silently picks *one* match if the join is one-to-many (non-deterministic); `MERGE` raises an error instead — a semantic difference.

### R12 · Create a table from a query

| | Syntax |
|---|---|
| **T** | `SELECT … INTO dbo.NewTable FROM …;` |
| **PG / MY / OR / SF** | `CREATE TABLE NewTable AS SELECT …;` |
| **Synapse dedicated** | `CREATE TABLE … WITH (DISTRIBUTION = HASH(col)) AS SELECT …` (CTAS) |

### R13 · Temp tables and variables

| | Temp table | Variable |
|---|---|---|
| **T** | `#t`, `##t`; `DECLARE @t TABLE (…)` | `DECLARE @n int = 1; SET @n = 2;` |
| **PG** | `CREATE TEMP TABLE` | PL/pgSQL `DECLARE n int := 1;`; psql `\set` |
| **MY** | `CREATE TEMPORARY TABLE` | `SET @n = 1;`; `DECLARE` inside routines |
| **OR** | global temporary tables (persistent definition), private temp tables (18c+) | PL/SQL block only |
| **SF** | `CREATE TEMPORARY TABLE`; session variables `SET n = 1;` | |

See [Temp Tables vs Table Variables](<../Temp_Tables_vs_Table_Variables.md>) — T-SQL's table variable has no equivalent in PG/OR.

### R14 · Identifiers: quoting and case folding

| | Quote | Unquoted names are… |
|---|---|---|
| **T** | `[Name]` or `"Name"` (when `QUOTED_IDENTIFIER ON`) | case-**preserving**, comparison depends on database collation (usually case-insensitive) |
| **PG** | `"Name"` | folded to **lower** case → `"Name"` ≠ `name` |
| **OR** | `"Name"` | folded to **UPPER** case |
| **MY** | `` `Name` `` | table names follow file-system case rules on Linux (case-sensitive) |
| **SF** | `"Name"` | folded to **UPPER** case |

This is one of the most common reasons a pipeline "can't find the column": a T-SQL-created `CustomerID` queried in Postgres as `CustomerID` becomes `customerid` and doesn't match `"CustomerID"`.

### R15 · Auto-numbering

| | Syntax |
|---|---|
| **T** | `id int IDENTITY(1,1)`; `NEXT VALUE FOR dbo.seq` (2012+) |
| **PG** | `id int GENERATED ALWAYS AS IDENTITY`; `serial` |
| **MY** | `id INT AUTO_INCREMENT` |
| **OR** | `id NUMBER GENERATED AS IDENTITY` (12c+); `seq.NEXTVAL` |
| **SF** | `id INT AUTOINCREMENT`, `IDENTITY`; sequences |

T-SQL: identity values are **not rolled back** with the transaction (gaps) and `IDENTITY_INSERT` is required to supply explicit values.

### R16 · Row numbering and top-N per group

```sql
-- Portable (ANSI windows; works in T-SQL, PG, MY 8+, OR, SF, SP)
WITH ranked AS (
  SELECT CustomerID, OrderID, OrderDate,
         ROW_NUMBER() OVER (PARTITION BY CustomerID ORDER BY OrderDate DESC, OrderID DESC) AS rn
  FROM   Sales.Orders)
SELECT CustomerID, OrderID, OrderDate FROM ranked WHERE rn <= 3;
-- Snowflake/BigQuery shortcut:  … QUALIFY ROW_NUMBER() OVER (...) <= 3
-- T-SQL alternative:  CROSS APPLY (SELECT TOP 3 …)  — often faster with a supporting index
```

### 17. Worked translation — full batch

Take this Oracle query and produce the T-SQL equivalent:

```sql
-- Oracle
SELECT e.ename || ' (' || NVL(d.dname,'-') || ')' AS label,
       TRUNC(SYSDATE) - TRUNC(e.hiredate)         AS days_employed
FROM   emp e LEFT JOIN dept d ON d.deptno = e.deptno
WHERE  ROWNUM <= 5
ORDER  BY e.hiredate;      -- BUG: ROWNUM applied before ORDER BY
```
```sql
-- T-SQL  (note also the fix of the ROWNUM-before-ORDER BY semantics)
SELECT TOP (5)
       CONCAT(e.ename, ' (', COALESCE(d.dname,'-'), ')')                  AS label,
       DATEDIFF(day, CAST(e.hiredate AS date), CAST(SYSDATETIME() AS date)) AS days_employed
FROM   dbo.emp  AS e
LEFT JOIN dbo.dept AS d ON d.deptno = e.deptno
ORDER  BY e.hiredate;
```
Two lessons: (1) `||` → `CONCAT` or `+` with NULL-guards; (2) Oracle's `ROWNUM <= 5` filters **before** sorting, so a "faithful" port would be different from the "intended" result — dialects differ in *evaluation order* not just spelling.

### 18. Tools for mechanical translation

| Tool | Use |
|---|---|
| **sqlglot** (Python) — `sqlglot.transpile(sql, read="tsql", write="postgres")` | Parse and re-emit queries between ~20 dialects; good for bulk review; not guaranteed correct for semantics |
| **SQL Server Migration Assistant (SSMA)** | Convert schema + procedural code from Oracle, MySQL, Db2, Sybase, Access into T-SQL |
| **Babelfish Compass** | Reports which T-SQL constructs Babelfish for Aurora PostgreSQL does not support |
| **dbt cross-database macros** (`dateadd`, `datediff`, `concat`, `safe_cast`) | Write once, compile per adapter |

### 19. Common mistakes

- Translating syntax only and ignoring **evaluation-order** (`ROWNUM`, `LIMIT` without `ORDER BY`).
- Porting `+` concat without NULL handling.
- Assuming `GROUP BY alias` or `ORDER BY 1` works the same (T-SQL allows ordinal `ORDER BY`, not alias in `GROUP BY`).
- Forgetting identifier case folding when moving T-SQL DDL to PG/OR/SF.

### 20. Check yourself

- [ ] Produce R1, R3, R5, R9 from memory for T-SQL and one other dialect.
- [ ] Translate an `UPDATE … FROM` T-SQL statement to PostgreSQL and to ANSI subquery form.
- [ ] Explain why `UPDATE … FROM` and `MERGE` differ on one-to-many matches.
- [ ] Use sqlglot or by hand: port a Postgres `ON CONFLICT` upsert to T-SQL.

**Where this connects:** [MERGE Pitfalls](<../MERGE_Pitfalls_and_Error_Handling.md>) · [CROSS APPLY](<../CROSS_APPLY_and_OUTER_APPLY.md>) · [Temp Tables vs Table Variables](<../Temp_Tables_vs_Table_Variables.md>) · [Date and Time Types](<../../SQL_Server_Platform/Date_and_Time_Types.md>) · Next → [Same Syntax, Different Semantics](<04_Same_Syntax_Different_Semantics.md>)
<!-- explainer:end -->
