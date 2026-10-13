---
id: "TOOLS/SQL_Server/T-SQL/Dialects_and_Portability/11_Practice_Lab"
level: 5
type: "concept"
topic: "TOOLS/SQL_Server/T-SQL/Dialects_and_Portability"
tools: ["SQL_Server"]
---

# 11 · Practice Lab

**L5** · [↑ Series index](<_index.md>) · Prev: [Portable SQL and Migration Playbook](<10_Portable_SQL_and_Migration_Playbook.md>)

<!-- explainer:start -->
## 📘 Lab

> **In one line:** Twenty-nine exercises that make you *predict, translate, diagnose and design* — attempt each before opening the answer.

### Setup

You need a SQL Server (Developer, Express, or Azure SQL DB) and optionally PostgreSQL for the compare steps. Create the practice schema:

```sql
CREATE TABLE dbo.People (
    Id        int IDENTITY PRIMARY KEY,
    Name      varchar(30)  NULL,
    Email     varchar(100) NULL,
    Salary    decimal(10,2) NULL,
    Created   datetime      NOT NULL
);
INSERT dbo.People(Name, Email, Salary, Created) VALUES
 ('Smith',  'a@x.com', 100.505, '2024-03-31T23:59:59.999'),
 ('SMITH',  NULL,      NULL,    '2024-04-01T00:00:00.000'),
 ('',       'b@x.com', -5,      '2024-04-01T00:00:00.001'),
 ('Müller', 'c@x.com', 2500,    '2024-02-29T12:00:00'),
 (NULL,     NULL,      0,       '2024-01-01');
```

Use `SET STATISTICS IO ON` and the actual execution plan where asked.

---

## Part A — Predict the output (no running first)

**A1.** `SELECT CASE WHEN 'abc' = 'ABC' THEN 1 ELSE 0 END;`
**A2.** `SELECT LEN('hi   '), DATALENGTH('hi   '), 'hi' + NULL, CONCAT('hi', NULL);`
**A3.** `SELECT DATEDIFF(year, '20231231', '20240101'), DATEDIFF(day, '20240101 23:59', '20240102 00:01');`
**A4.** `DECLARE @v varchar = 'abcdef'; SELECT @v, CAST('abcdef' AS varchar);`
**A5.** `SELECT ISNULL(CAST(NULL AS varchar(2)), 'xyz'), COALESCE(CAST(NULL AS varchar(2)), 'xyz');`
**A6.** `SELECT COUNT(*), COUNT(Name), COUNT(DISTINCT Name) FROM dbo.People;`
**A7.** How many rows does `SELECT * FROM dbo.People WHERE Name = '';` return — and in Oracle?
**A8.** `SELECT CAST('2024-01-02' AS datetime)` after `SET LANGUAGE British;`

<details><summary>Answers A1–A8</summary>

- **A1** → `1` (case-insensitive default collation).
- **A2** → `2, 5 (varchar bytes), NULL, 'hi'`. (`LEN` ignores trailing spaces; `+` NULL propagates.)
- **A3** → `1, 1`.
- **A4** → `'a'` and `'abcdef'` (declaration length 1, `CAST` default 30).
- **A5** → `'xy'` and `'xyz'`.
- **A6** → `5, 4, 3`. Names are `'Smith','SMITH','','Müller',NULL`: `COUNT(*)=5`, `COUNT(Name)=4` (NULL skipped), `COUNT(DISTINCT Name)=3` because `Smith` and `SMITH` are equal under the CI collation → {Smith, '', Müller}.
- **A7** → `1` row (the `''`). In Oracle `''` is NULL, so `= ''` matches **0** rows.
- **A8** → `2024-02-01` (yyyy-mm-dd read as yyyy-dd-mm for `datetime` under `dmy`); use `'20240102'`.
</details>

---

## Part B — Translate

**B1.** Convert to PostgreSQL:
```sql
SELECT TOP (3) Name, ISNULL(Salary, 0) AS Salary, DATEADD(day, 30, Created) AS Due
FROM dbo.People ORDER BY Salary DESC;
```
**B2.** Convert this PostgreSQL upsert to T-SQL:
```sql
INSERT INTO people (email, name) VALUES ('a@x.com', 'Alice')
ON CONFLICT (email) DO UPDATE SET name = EXCLUDED.name
RETURNING id;
```
**B3.** Convert this MySQL to T-SQL:
```sql
SELECT GROUP_CONCAT(name ORDER BY name SEPARATOR ', ') FROM people WHERE created >= CURDATE() - INTERVAL 7 DAY LIMIT 1;
```
**B4.** Convert this T-SQL to Oracle:
```sql
SELECT p.Name, x.MaxSalary
FROM dbo.People p
CROSS APPLY (SELECT MAX(Salary) AS MaxSalary FROM dbo.People q WHERE q.Name = p.Name) x;
```
**B5.** Rewrite `UPDATE p SET Salary = Salary * 1.1 FROM dbo.People p JOIN dbo.Raise r ON r.Id = p.Id;` in ANSI/portable form.

<details><summary>Answers B1–B5</summary>

- **B1**
```sql
SELECT name, COALESCE(salary, 0) AS salary, created + INTERVAL '30 days' AS due
FROM people ORDER BY salary DESC LIMIT 3;   -- note: NULLS FIRST in DESC order in PG!
```
Add `NULLS LAST` to mimic T-SQL, where NULLs sort *lowest* (last for DESC).
- **B2**
```sql
MERGE dbo.People WITH (HOLDLOCK) AS t
USING (VALUES ('a@x.com','Alice')) AS s(Email, Name) ON t.Email = s.Email
WHEN MATCHED     THEN UPDATE SET t.Name = s.Name
WHEN NOT MATCHED THEN INSERT (Email, Name, Created) VALUES (s.Email, s.Name, SYSDATETIME())
OUTPUT inserted.Id;
```
(Requires a UNIQUE constraint on Email for correctness under concurrency; consider `UPDATE … IF @@ROWCOUNT = 0 INSERT` with `UPDLOCK, SERIALIZABLE`.)
- **B3**
```sql
SELECT STRING_AGG(Name, ', ') WITHIN GROUP (ORDER BY Name)
FROM dbo.People WHERE Created >= DATEADD(day, -7, CAST(SYSDATETIME() AS date));
```
`LIMIT 1` is unnecessary — the aggregate returns one row.
- **B4**
```sql
SELECT p.name, x.max_salary
FROM people p
CROSS JOIN LATERAL (SELECT MAX(salary) AS max_salary FROM people q WHERE q.name = p.name) x;  -- Oracle 12c+
```
Or simply `MAX(salary) OVER (PARTITION BY name)` — more portable.
- **B5**
```sql
UPDATE dbo.People
SET Salary = Salary * 1.1
WHERE EXISTS (SELECT 1 FROM dbo.Raise r WHERE r.Id = dbo.People.Id);
```
</details>

---

## Part C — Diagnose (stage and fix)

For each, name the **stage** (parse/bind/optimize/execute/client) and the fix.

**C1.** `SELECT * FROM dbo.People LIMIT 2;` → `Msg 102 … near '2'`
**C2.** `SELECT Name AS n FROM dbo.People WHERE n = 'Smith';` → `Msg 207`
**C3.** `DECLARE @d int = 1 WITH c AS (SELECT 1 x) SELECT * FROM c;` → `Msg 319`
**C4.** ADF Copy with partition option and query `SELECT … ORDER BY Id` → `Msg 1033`
**C5.** `SELECT Name, COUNT(*) FROM dbo.People;` → `Msg 8120`
**C6.** `SELECT 'abc' + 1;` → `Msg 245`
**C7.** A query is fast in SSMS (30 ms) but 40 s from the app with identical text and parameters.
**C8.** `SELECT * FROM dbo.People WHERE Email = N'a@x.com'` on a million-row table shows `CONVERT_IMPLICIT` and a scan.

<details><summary>Answers C1–C8</summary>

- **C1** parse — `LIMIT` consumed as table alias. Use `TOP (2)` or `OFFSET 0 ROWS FETCH NEXT 2 ROWS ONLY`.
- **C2** bind — alias not visible in `WHERE`. Repeat the expression, or wrap in a CTE/derived table.
- **C3** parse — previous statement must end with `;`. `DECLARE @d int = 1;` first.
- **C4** client wrapper/parse — derived table cannot have `ORDER BY`. Remove it or use a view/proc.
- **C5** bind (grouping rules) — add `GROUP BY Name`.
- **C6** execute (implicit conversion of `'abc'` to int, because int has higher precedence). Use `CONCAT('abc', 1)` or `'abc' + CAST(1 AS varchar(10))`.
- **C7** optimizer/plan cache — different `SET` options (e.g., `ARITHABORT`) give separate plans; parameter sniffing. Compare plans in Query Store, check `sys.dm_exec_plan_attributes`, test with `OPTION (RECOMPILE)`.
- **C8** optimizer/type precedence — `varchar` column vs `nvarchar` literal/parameter. Make the parameter `varchar` (or the column `nvarchar`), or use `CAST` on the parameter side.
</details>

---

## Part D — Semantics and design

**D1.** The same report returns 1 group in SQL Server and 2 groups in PostgreSQL for `GROUP BY Name`. Explain and list two fixes.

**D2.** Make `UNIQUE (Email)` allow many NULLs in SQL Server.

**D3.** `WHERE Created BETWEEN '20240301' AND '20240331 23:59:59.999'` on a `datetime` column returns rows from April 1. Why? Rewrite it.

**D4.** Design the contract layer for these consumers: ADF nightly load (incremental), Power BI dashboard, dbt staging. Which objects, which parameters?

**D5.** Your pipeline copies from SQL Server to Snowflake; `Name` values lose `Müller`. Where could it break?

**D6.** Write a retry wrapper (T-SQL pseudo or application pseudo) for deadlock victims (1205).

**D7.** Compare how a work-queue is implemented on SQL Server vs PostgreSQL.

**D8.** A query uses `WITH (NOLOCK)` everywhere. Propose a safer plan and its risks.

<details><summary>Answers D1–D8</summary>

- **D1** CI vs CS collation. Fix: make the *business key* explicit — `GROUP BY LOWER(name)` in PG or `COLLATE Latin1_General_CS_AS` in SQL Server (decide which is correct); or use `citext`/a case-insensitive collation on PG.
- **D2** `CREATE UNIQUE INDEX ux_People_Email ON dbo.People(Email) WHERE Email IS NOT NULL;`
- **D3** `datetime` stores to ~3.33 ms; `.999` rounds up to the next day's midnight. Use half-open: `Created >= '20240301' AND Created < '20240401'`.
- **D4** `etl.vOrders_Delta` view (or `etl.GetOrdersDelta @Since datetime2` proc) with stable types and a watermark column; `rpt.vOrderSummary` for BI (no ORDER BY, explicit aggregations); `stg.vOrders` for dbt (plain SELECT). Version with `_v1`.
- **D5** Candidates: `varchar` column with non-UTF8 code page → data already lost; ADF interim type collation; driver encoding; Snowflake file-format encoding; use `nvarchar` and UTF-8 end to end; test with `SELECT UNICODE(SUBSTRING(Name,2,1))`.
- **D6**
```sql
DECLARE @try int = 0, @max int = 3;
WHILE @try < @max
BEGIN
  BEGIN TRY
    EXEC dbo.DoWork;  BREAK;
  END TRY
  BEGIN CATCH
    IF ERROR_NUMBER() = 1205 AND @try < @max - 1
    BEGIN SET @try += 1; WAITFOR DELAY '00:00:00.200'; END
    ELSE THROW;
  END CATCH
END
```
(Retries must wrap the *whole* transaction.)
- **D7** SQL Server: `UPDLOCK, READPAST, ROWLOCK` with `OUTPUT`; PostgreSQL: `FOR UPDATE SKIP LOCKED` with `RETURNING`.
- **D8** Enable `READ_COMMITTED_SNAPSHOT`, remove hints; risks: tempdb/PVS usage, long transactions, code relying on blocking semantics; test with realistic concurrency.
</details>

---

## Part E — Capstone mini-project (60–90 min)

1. Pick one real T-SQL stored procedure from your own work (or `AdventureWorks`).
2. Classify every statement into **A/B/C/D** buckets (note 2).
3. Produce a PostgreSQL translation of the *query part* (leave the procedural wrapper).
4. List **semantic differences** you expect (note 4) and write the verification queries (note 10).
5. Move the logic behind a view/proc contract usable by ADF with partitioning on (note 9).
6. Record: what would you *not* port, and why?

**Grading rubric (self-assess)**

| Criterion | ✓ |
|---|---|
| All dialect-specific constructs identified | |
| Semantic differences predicted, not just syntax | |
| Verification queries cover NULL/blank/dates/collation | |
| Contract layer is wrapper-safe | |
| Performance risks noted (types, hints, plans) | |

**Where this connects:** [Series index](<_index.md>) · [T-SQL Basics](<../T-SQL_Basics.md>) · [MERGE Pitfalls](<../MERGE_Pitfalls_and_Error_Handling.md>) · [Temp Tables vs Table Variables](<../Temp_Tables_vs_Table_Variables.md>)
<!-- explainer:end -->
