---
id: "TOOLS/SQL_Server/T-SQL/Dialects_and_Portability/04_Same_Syntax_Different_Semantics"
level: 5
type: "concept"
topic: "TOOLS/SQL_Server/T-SQL/Dialects_and_Portability"
tools: ["SQL_Server"]
---

# 4 · Same Syntax, Different Semantics

**L5** · [↑ Series index](<_index.md>) · Prev: [Syntax Translation Cookbook](<03_Syntax_Translation_Cookbook.md>) · Next: [Parser, Binder and Error Messages](<05_Parser_Binder_and_Error_Messages.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** The most expensive dialect bugs are not syntax errors — the statement parses everywhere and quietly returns different data, because defaults for NULLs, strings, dates, rounding and conversions differ between engines.

### 1. Why this matters more than syntax

A syntax error stops you at once. A semantic difference ships to production. When you move SQL between engines — or when a tool such as ADF moves *data* between them — these are the differences that produce wrong totals, missing rows and "off by one day" reports.

Each section below shows: the statement, the T-SQL result, the contrast, and the defence.

### 2. NULL and empty string

```sql
SELECT CASE WHEN '' IS NULL THEN 'null' ELSE 'not null' END;   -- T-SQL: not null
```

| Engine | `'' IS NULL` | Consequence |
|---|---|---|
| SQL Server, PostgreSQL, MySQL | **false** — `''` is a value | `WHERE col = ''` and `WHERE col IS NULL` select different rows |
| Oracle | **true** — zero-length string is stored as NULL (for `VARCHAR2`) | `WHERE col = ''` never matches; `''` inserts become NULL |

**Defence:** normalise explicitly: `NULLIF(LTRIM(RTRIM(col)), '')` when you mean "treat blank as missing".

**Other NULL rules** that are standard but still catch people:

- `NULL = NULL` → UNKNOWN, not TRUE (so no row is returned). With `SET ANSI_NULLS OFF` (deprecated) T-SQL would return TRUE; never use it.
- `NOT IN (subquery_with_NULL)` returns no rows in every engine → use `NOT EXISTS`.
- Aggregates ignore NULLs (`COUNT(col)` vs `COUNT(*)`); `SUM` of only NULLs is NULL, not 0.
- **`UNIQUE` constraint:** SQL Server treats NULLs as *equal* — only **one** NULL allowed. Standard-compliant engines (PG, Oracle, MySQL) allow many. In SQL Server, emulate the standard with a **filtered unique index**:

```sql
CREATE UNIQUE INDEX ux_Email ON dbo.Users(Email) WHERE Email IS NOT NULL;
```
See [Filtered Indexes and Plan Cache](<../Filtered_Indexes_and_Plan_Cache.md>).

### 3. Strings

#### 3.1 Trailing spaces
```sql
SELECT CASE WHEN 'a' = 'a   ' THEN 'equal' ELSE 'different' END;  -- equal (T-SQL, ANSI padding)
SELECT LEN('a   '), DATALENGTH('a   ');                          -- 1 , 4
SELECT CASE WHEN 'a' LIKE 'a   ' THEN 1 ELSE 0 END;              -- 0  (LIKE does not pad!)
```
`=` pads the shorter operand; `LIKE` does not; `LEN` ignores trailing spaces; `DATALENGTH` counts bytes. PostgreSQL's `varchar`/`text` compare exactly; `char(n)` is blank-padded for comparison.

#### 3.2 Case sensitivity and accents
SQL Server's default collation (`SQL_Latin1_General_CP1_CI_AS`, or `Latin1_General_100_CI_AS` for newer installs) is **Case-Insensitive, Accent-Sensitive**.

| Comparison `'Smith' = 'SMITH'` | T-SQL (CI) | PostgreSQL | Oracle | MySQL (`utf8mb4_0900_ai_ci`) |
|---|---|---|---|---|
| default | TRUE | FALSE | FALSE | TRUE |

Effects: `GROUP BY name` merges `Smith` and `SMITH` in SQL Server but keeps them apart in PostgreSQL; `UNIQUE` index treats them as duplicates; joins between a CI and a CS collation raise **Msg 468** (collation conflict).

```sql
SELECT DATABASEPROPERTYEX(DB_NAME(), 'Collation') AS db_collation,
       SERVERPROPERTY('Collation')                 AS server_collation;
```

#### 3.3 Length defaults — the `varchar` trap
```sql
DECLARE @s varchar = 'abcdef';     SELECT @s;                    -- 'a'     (length 1 in declarations)
SELECT CAST('abcdef' AS varchar);                               -- 'abcdef' (length 30 in CAST)
DECLARE @t varchar(3) = 'abcdef';                               -- silently truncated to 'abc'
```
Always give a length. In PostgreSQL, `varchar` without length is unlimited.

#### 3.4 Unicode literals
`'日本'` in a `varchar` column with a Latin collation becomes `'??'`; use `N'日本'` and `nvarchar`. In PostgreSQL/Snowflake/MySQL-utf8mb4 all strings are Unicode by default.

#### 3.5 `+` is overloaded
```sql
SELECT '1' + 1;        -- 2     (string → int, because int has higher type precedence)
SELECT 'a' + 1;        -- Msg 245: Conversion failed when converting the varchar value 'a' to int
SELECT '1' + '1';      -- '11'
```
PostgreSQL coerces an *untyped literal* `'1'` to integer (so `'1' + 1` = 2) but raises an error for `'a' + 1` and for a `text` *column* plus an integer; MySQL silently converts (`'a' + 1` = 1 with a warning).

### 4. Numbers

```sql
SELECT 5/2 AS int_div, 5/2.0 AS dec_div, 5/2e0 AS float_div;   -- 2 | 2.500000 | 2.5
SELECT 1/0;                                                       -- Msg 8134: Divide by zero error
SELECT 2147483647 + 1;                                            -- Msg 8115: Arithmetic overflow
SELECT CAST('' AS int);                                           -- 0 !   (PG: error)
```

| Behaviour | T-SQL | PostgreSQL | MySQL | Oracle |
|---|---|---|---|---|
| `int / int` | integer division | integer division | decimal (`DIV` for int) | decimal |
| divide by zero | error | error | `NULL` (default sql_mode warns) | error |
| empty string → int | `0` | error | `0` + warning | NULL |
| decimal result scale | derived by precision rules (min scale 6) | arbitrary precision | fixed | NUMBER arbitrary |

**Defence:** make at least one operand `decimal`/`float` deliberately; use `NULLIF(denom,0)` for safe division; use `TRY_CAST` for dirty data.

**Rounding.** `ROUND(2.5,0)` → `3` for `decimal` (half away from zero) in T-SQL; PostgreSQL `numeric` the same, but `double precision` rounds half-to-even; Python's `round` is banker's. ETL pipelines that move numbers across engines should specify type and rounding mode.

### 5. Dates and times — the largest bug farm

#### 5.1 Types
| Concept | T-SQL | PG | MY | OR |
|---|---|---|---|---|
| Date only | `date` | `date` | `DATE` | `DATE` **includes time** |
| Date-time | `datetime2(p)` (preferred), `datetime` (legacy, ±3.33 ms) | `timestamp` | `DATETIME`, `TIMESTAMP` | `TIMESTAMP` |
| With zone | `datetimeoffset` | `timestamptz` | n/a (`TIMESTAMP` is UTC-converted) | `TIMESTAMP WITH TIME ZONE` |
| Row version | **`timestamp`/`rowversion`** | n/a | n/a | n/a |

#### 5.2 Literal parsing depends on session language (for `datetime`/`smalldatetime`)
```sql
SET LANGUAGE British;                         -- DATEFORMAT dmy
SELECT CAST('2024-01-02' AS datetime);        -- 2024-02-01 (!)  yyyy-mm-dd read as yyyy-dd-mm
SELECT CAST('20240102'   AS datetime);        -- 2024-01-02  safe
SELECT CAST('2024-01-02' AS date);            -- 2024-01-02  safe (date/datetime2 are language-neutral)
SELECT CAST('2024-01-02T00:00:00' AS datetime); -- safe (ISO 8601 with T)
```
**Defence:** `yyyymmdd` or `yyyy-mm-ddThh:mi:ss`, and prefer `date`/`datetime2`.

#### 5.3 Rounding of `datetime`
```sql
SELECT CAST('2024-01-01 23:59:59.999' AS datetime);   -- 2024-01-02 00:00:00.000  (rounds to .000/.003/.007)
SELECT CAST('2024-01-01 23:59:59.999' AS datetime2);  -- 2024-01-01 23:59:59.9990000
```
So `BETWEEN '20240101' AND '20240101 23:59:59.999'` on a `datetime` column includes **the next day's midnight**. Use half-open ranges: `>= '20240101' AND < '20240102'`.

#### 5.4 `DATEDIFF` counts boundaries
```sql
SELECT DATEDIFF(year,  '2023-12-31', '2024-01-01');   -- 1  (one year boundary crossed, one day elapsed)
SELECT DATEDIFF(month, '2024-01-31', '2024-02-01');   -- 1
SELECT DATEDIFF(day,   '2024-01-01 23:59', '2024-01-02 00:01');  -- 1
```
Postgres subtraction/`age()` and Python return *elapsed* intervals. For elapsed age in years in T-SQL: compare `DATEADD(year, DATEDIFF(year, birth, today), birth) > today` and subtract 1.

#### 5.5 Week numbering
`DATEPART(week, d)` depends on `SET DATEFIRST` (default 7 = Sunday for us_english) and is *not* ISO. Use `DATEPART(iso_week, d)`. PostgreSQL's `EXTRACT(week …)` is ISO.

Further reading: [Date and Time Types](<../../SQL_Server_Platform/Date_and_Time_Types.md>).

### 6. Implicit conversion and data-type precedence

T-SQL applies a **data-type precedence list**: when two types meet, the lower-precedence one is converted to the higher. Rough order (high → low): `datetimeoffset, datetime2, datetime, smalldatetime, date, time, float, real, decimal/numeric, money, bigint, int, smallint, tinyint, bit, nvarchar, nchar, varchar, char, binary`.

```sql
-- OrderNumber is varchar(20) with an index
SELECT * FROM dbo.Orders WHERE OrderNumber = N'A1001';   -- N'' literal is nvarchar → higher precedence
```
The **column** is converted to `nvarchar` for every row → with a *SQL collation* the index seek can become a scan; the plan shows a `CONVERT_IMPLICIT` warning. With Windows collations the engine can often still seek, but cardinality estimates degrade. The same applies to `int` column vs `varchar` parameter (the column is fine; *parameter* is converted), but `varchar` column vs `int` parameter → column converted.

PostgreSQL is much stricter: `text = integer` is an error, forcing you to cast — which prevents this class of bug.

**Defence:** match parameter types to column types; inspect actual plans for `CONVERT_IMPLICIT`; ORM / ADF parameters often default to `nvarchar`.

### 7. ISNULL vs COALESCE and other "same job" functions

```sql
DECLARE @s varchar(3);
SELECT ISNULL(@s, 'abcdef')   AS isnull_r,     -- 'abc'    (type = varchar(3), truncated)
       COALESCE(@s, 'abcdef') AS coalesce_r;   -- 'abcdef' (type by precedence)
```

| Function | Quirk |
|---|---|
| `ISNULL(a,b)` | two args only; returns type of `a`; result nullability computed differently (matters for `SELECT INTO`/computed columns) |
| `COALESCE(a,b,c…)` | standard; evaluates args possibly more than once for subqueries (can produce nondeterminism) |
| `LEN` | excludes trailing spaces; counts characters not bytes |
| `SUBSTRING(s, 0, n)` | position `0` consumes one of the `n` characters: returns `n-1` characters |
| `CHARINDEX` | start index 1-based; returns 0 when not found (not NULL, not -1) |
| `STRING_SPLIT` | returns rows, ordering unguaranteed until `enable_ordinal` in recent versions |
| `ROUND(123.456, -1)` | rounds *left* of the decimal point |
| `FORMAT()` | CLR-based, 10–100× slower than `CONVERT` — avoid in large sets |

### 8. Identity, sequence and ordering guarantees

- **No implicit order.** A `SELECT` without `ORDER BY` has *no* guaranteed order in any engine; T-SQL usually returns clustered-key order for a simple scan, and developers start to depend on it. Parallelism or an index change breaks that.
- **`TOP` without `ORDER BY`** is arbitrary rows. `TOP … WITH TIES` requires `ORDER BY`.
- **Identity gaps:** rolled-back inserts consume identity values; server restart can jump (cache); `IDENTITY` is not gap-free in any engine.
- **`ROW_NUMBER()` requires `ORDER BY`** inside `OVER` in T-SQL (`ORDER BY (SELECT NULL)` as a placeholder is allowed and produces arbitrary numbering).

### 9. Booleans and three-valued logic in practice

T-SQL has **no BOOLEAN data type**; predicates cannot be selected as values. A `bit` column holds `0/1/NULL`. Many SQL Server shops model flags as `bit`, `char(1)` ('Y'/'N') or `tinyint`; moving to PG's `boolean` requires explicit mapping and awareness that `NULL` ≠ `false`.

```sql
-- T-SQL
SELECT CASE WHEN a = b THEN 1 ELSE 0 END AS same;   -- cannot write: SELECT a = b
-- PG: SELECT a = b AS same;  → true / false / NULL
```

### 10. Schema-qualified resolution and case

- SQL Server resolves unqualified names using the *default schema of the user*, then `dbo`. Different users may resolve the same name to different objects.
- Object-name case sensitivity depends on **database collation**, not the engine: on a CS collation `Select * from orders` fails if the table is `Orders`.
- Variable and parameter names (`@x`) follow *server* collation rules in some contexts, particularly `tempdb` objects — a famous source of collation-conflict errors with temp tables in CS databases.

### 11. Worked example — same ETL, three engines, three answers

Source table in SQL Server:

| id | name | flag | amount | created |
|---|---|---|---|---|
| 1 | `Smith` | 1 | `100.505` | `2024-03-31 23:59:59.999` |
| 2 | `SMITH` | NULL | `5` | `2024-04-01 00:00:00.000` |
| 3 | `''` | 0 | `-5` | `2024-04-01 00:00:00.001` |

```sql
-- Report: rows per name, total amount (2dp), rows created in March
SELECT name, COUNT(*) AS cnt, ROUND(SUM(amount),2) AS tot
FROM   t WHERE created >= '2024-03-01' AND created < '2024-04-01'
GROUP  BY name;
```

| Engine / column type | What happens | Result of the March report |
|---|---|---|
| SQL Server, `created` is `datetime` | Row 1's `23:59:59.999` is **rounded to `2024-04-01 00:00:00.000` at insert**, so it falls *outside* March; `Smith`/`SMITH` would be merged in a group (CI) | 0 March rows; the "March" row has silently moved to April |
| SQL Server, `created` is `datetime2(3)` | Exact value stored | 1 row (`Smith`, 100.51 after `ROUND`) |
| PostgreSQL, `timestamp(3)` | Exact value; `Smith` and `SMITH` are **different** groups | same 1 March row, but any grouping over all rows shows 2 name groups instead of 1 |
| Oracle (`DATE` column, `VARCHAR2` name) | `DATE` has no fractional seconds (`.999` lost); `''` stored as NULL | 'blank name' row falls into the NULL group |

The *query text is identical*; the answers differ because of **type precision (rounding), collation, and empty-string semantics**. A data validation step after any cross-engine move should compare **row counts and aggregate checksums per group**, not just totals.

### 12. Performance and design implications

- Implicit conversions are both a correctness and a performance issue (see §6) — check plans, not just results.
- CI collations make `=` and `LIKE` lookups case-insensitive without a function — an advantage for SARGability that CS engines lack (they need `lower(col)` indexes).
- Prefer set-based, deterministic expressions; avoid depending on unspecified order.

### 13. Common mistakes

- Migrating `datetime` as-is to an engine with `timestamp(6)` and "gaining" or "losing" milliseconds.
- Trusting `BETWEEN` for date ranges.
- Assuming `LEN` = size in bytes (it is characters, trimmed).
- Comparing data across engines without comparing collations/time zones.
- Using `ISNULL` to build strings in `SELECT INTO`: the result column length equals the first argument's.

### 14. 🎯 Interview kit

**30-second answer:** "Syntax errors are cheap; semantic differences are dangerous. In T-SQL `''` isn't NULL, equality ignores trailing spaces, default collation is case-insensitive, `DATEDIFF` counts boundaries, `datetime` rounds to 3.33 ms, `ISNULL` takes the first argument's type, and `UNIQUE` allows only one NULL."

**Likely follow-ups:** *Why did my index seek become a scan?* → implicit conversion due to type precedence. *How would you validate a migration?* → per-group counts, sums, hash of sorted columns, edge-case rows (NULL, blanks, min/max dates). *How do you make dates safe?* → `date`/`datetime2` and ISO literals.

### 15. Check yourself

- [ ] Predict the output of `SELECT DATEDIFF(year,'20231231','20240101'), LEN('a  '), 5/2, ISNULL(CAST(NULL AS varchar(2)),'xyz')`.
- [ ] Why does a `BETWEEN` on a `datetime` column leak the next day?
- [ ] Rewrite a `UNIQUE (Email)` to allow many NULLs in SQL Server.
- [ ] Find the `CONVERT_IMPLICIT` in an execution plan and fix it.

**Where this connects:** [T-SQL Basics](<../T-SQL_Basics.md>) · [Filtered Indexes and Plan Cache](<../Filtered_Indexes_and_Plan_Cache.md>) · [Date and Time Types](<../../SQL_Server_Platform/Date_and_Time_Types.md>) · Next → [Parser, Binder and Error Messages](<05_Parser_Binder_and_Error_Messages.md>)
<!-- explainer:end -->
