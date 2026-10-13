---
id: "TOOLS/SQL_Server/T-SQL/Dialects_and_Portability/10_Portable_SQL_and_Migration_Playbook"
level: 5
type: "concept"
topic: "TOOLS/SQL_Server/T-SQL/Dialects_and_Portability"
tools: ["SQL_Server"]
---

# 10 · Portable SQL and Migration Playbook

**L5** · [↑ Series index](<_index.md>) · Prev: [ADF and Orchestration Tools](<09_ADF_and_Orchestration_Tools.md>) · Next: [Practice Lab](<11_Practice_Lab.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** Portability is a *design decision made in layers* — keep a thin, standard-looking contract at the edge (views, procedures, data types) and quarantine dialect-specific code behind it — and migration is a repeatable pipeline of inventory → translate → verify, not a find-and-replace.

### 1. Decide first: do you need portability at all?

| Situation | Recommendation |
|---|---|
| Single-engine product, SQL Server for 10+ years | Use T-SQL idiomatically; don't contort. Isolate hints and platform features |
| Multi-engine product / SaaS shipped to customers | Portable core + engine-specific adapter layer; CI on each engine |
| Data platform with many tools reading the DB (BI, ADF, dbt) | Contract = views + procedures with stable shapes |
| Planned migration | Inventory dialect usage now; avoid *new* extensions in code you'll port |
| Analytics SQL consumed by dbt across warehouses | Use dbt cross-database macros and adapter dispatch |

**Cost-benefit:** "100 % ANSI" is rarely worth it — it forfeits engine strengths. Aim for **"portable where cheap, isolated where not"**.

### 2. The layered contract

```
┌────────────────────────────────────────────────────────────┐
│ Consumers: apps, ADF, Power BI, dbt, notebooks              │  ← only see simple SELECT / EXEC
├────────────────────────────────────────────────────────────┤
│ CONTRACT LAYER:  schema "api"/"etl"/"rpt"                   │  ← views, procedures, TVFs
│   stable column names, types, parameters, result shapes     │     (versioned: v1, v2)
├────────────────────────────────────────────────────────────┤
│ LOGIC LAYER: business rules (set-based SQL)                 │  ← mostly ANSI-flavoured
├────────────────────────────────────────────────────────────┤
│ PLATFORM LAYER: hints, partitioning, distribution, temp     │  ← dialect-specific; small
│   tables, error handling, identity/sequence, jobs           │
├────────────────────────────────────────────────────────────┤
│ STORAGE: tables, constraints, indexes                       │
└────────────────────────────────────────────────────────────┘
```

Benefits:
- Tools never see CTEs/hints/`GO` → no wrapper breakage.
- Porting means re-implementing the *platform layer* and re-running contract tests.
- Result-set shape changes become explicit versioning events.

### 3. Rules for portable-leaning T-SQL

| Do | Avoid / isolate |
|---|---|
| `COALESCE`, `NULLIF`, `CASE` | `ISNULL`, `IIF`, `CHOOSE` (fine inside platform layer) |
| `CAST(x AS type)` | `CONVERT(type, x, style)` for formatting (isolate) |
| `CURRENT_TIMESTAMP` | `GETDATE()`, `SYSDATETIME()` |
| `OFFSET … FETCH` | `TOP` (acceptable for simple limits) |
| Explicit `JOIN … ON`, ANSI aliases `AS` | old-style joins `FROM a, b WHERE …` |
| `INFORMATION_SCHEMA` | `sys.*`, `sp_help*` in logic layer |
| `||`-agnostic: `CONCAT()` | `+` for strings with NULLs |
| `EXISTS`/`NOT EXISTS` | `NOT IN` with nullable subquery |
| `date`/`datetime2`/`decimal(p,s)`/`varchar(n)`/`nvarchar(n)` | `datetime`, `money`, `text`/`ntext`/`image`, `timestamp` |
| Surrogate keys as `bigint`/`int` identity behind an interface | Relying on identity side-effects (`@@IDENTITY`) |
| ISO-8601 literals (`'2024-01-31'` with `date`, or `yyyymmdd`) | Locale-dependent literals |
| `UNION ALL` unless de-dup needed | `UNION` by habit (sort cost) |
| Window functions in standard form | `PIVOT`, `FOR XML PATH` string concat (isolate) |
| Parameterised queries | String-built SQL |

### 4. Data type mapping tables

#### 4.1 Oracle → SQL Server

| Oracle | SQL Server | Notes |
|---|---|---|
| `NUMBER(p,0)` | `int`/`bigint`/`decimal(p,0)` | choose by precision (≤9 → int, ≤18 → bigint) |
| `NUMBER(p,s)` | `decimal(p,s)` | `NUMBER` without precision → `float` or `decimal(38,10)` — decide by data |
| `VARCHAR2(n BYTE/CHAR)` | `varchar(n)` / `nvarchar(n)` | Byte vs char semantics; Unicode |
| `CHAR(n)` | `char(n)` | padded |
| `DATE` | `datetime2(0)` | contains time part! |
| `TIMESTAMP(p)` | `datetime2(p)` | max p = 7 |
| `TIMESTAMP WITH TIME ZONE` | `datetimeoffset(p)` | |
| `CLOB` / `NCLOB` | `varchar(max)` / `nvarchar(max)` | |
| `BLOB`/`RAW` | `varbinary(max)` / `varbinary(n)` | |
| `''` as NULL | rewrite NULL checks | semantics change (note 4) |
| Sequences | `SEQUENCE` or `IDENTITY` | |
| `ROWNUM`, `ROWID` | `ROW_NUMBER()`, PK | |
| `DECODE`, `NVL`, `NVL2` | `CASE`, `ISNULL`/`COALESCE`, `CASE` | |
| `CONNECT BY` | recursive CTE | |
| `MINUS` | `EXCEPT` | |
| Packages | schemas + naming | state variables need tables/session context |

#### 4.2 PostgreSQL → SQL Server

| PostgreSQL | SQL Server | Notes |
|---|---|---|
| `boolean` | `bit` | NULL handling |
| `smallint/integer/bigint` | `smallint/int/bigint` | |
| `numeric(p,s)` | `decimal(p,s)` | |
| `text`, `varchar` | `varchar(max)`/`nvarchar(max)` | *max* types are LOB: no index key |
| `timestamp`, `timestamptz` | `datetime2`, `datetimeoffset` | |
| `uuid` | `uniqueidentifier` | |
| `bytea` | `varbinary(max)` | |
| `jsonb`/`json` | `nvarchar(max)` + `ISJSON`/`OPENJSON` (native `json` type in newest releases — *verify*) | no GIN-style index |
| arrays | child table / JSON | |
| `serial`/identity | `IDENTITY` | |
| `ILIKE` | CI collation + `LIKE` | |
| `generate_series` | `GENERATE_SERIES` (2022+) or tally table | |
| `LATERAL` | `CROSS/OUTER APPLY` | |
| `ON CONFLICT` | `MERGE` / guarded UPDATE+INSERT | |
| `RETURNING` | `OUTPUT` | |
| `DISTINCT ON` | `ROW_NUMBER() = 1` | |

#### 4.3 SQL Server → PostgreSQL / Snowflake (reverse)

| SQL Server | PostgreSQL | Snowflake |
|---|---|---|
| `bit` | `boolean` | `BOOLEAN` |
| `datetime2(7)` | `timestamp(6)` (max 6 → truncation) | `TIMESTAMP_NTZ(9)` |
| `datetimeoffset` | `timestamptz` (offset not preserved) | `TIMESTAMP_TZ` |
| `nvarchar(max)` | `text` | `VARCHAR` (16 MB) |
| `money` | `numeric(19,4)` | `NUMBER(19,4)` |
| `uniqueidentifier` | `uuid` | `VARCHAR(36)` |
| `rowversion` | no equivalent (use `xmin`/trigger) | no equivalent |
| `IDENTITY` | `GENERATED … AS IDENTITY` | `AUTOINCREMENT` |
| `#temp` | `TEMP TABLE` | `TEMPORARY TABLE` |

### 5. The migration pipeline

```
 1 INVENTORY ──► 2 CLASSIFY ──► 3 TRANSLATE ──► 4 VERIFY ──► 5 CUT OVER ──► 6 STABILISE
```

| Phase | Activities | Tools |
|---|---|---|
| 1. Inventory | Extract all SQL: procedures, views, jobs, ADF/SSIS/BI queries, application code (ORM, inline SQL) | `sys.sql_modules`, Query Store, Extended Events trace, repo grep, Babelfish Compass, SSMA assessment |
| 2. Classify | Tag each unit with: ✅ portable / ⚠ extension / ❌ unsupported (bucket A/B/C from note 2); risk score by usage frequency | spreadsheet, sqlglot parse-and-count |
| 3. Translate | Automated first pass + manual for procedural code; schema (types, constraints, indexes) | SSMA, sqlglot, ora2pg/pgloader/AWS SCT/Azure DMS, manual |
| 4. Verify | Row counts, checksums per group/partition, sample diff, boundary values, performance baselines, plan regressions | dbt tests, `EXCEPT` diffs, data-diff tooling |
| 5. Cut over | Dual-run / CDC replication → freeze → switch → rollback plan | Azure DMS, replication, ADF |
| 6. Stabilise | Query Store/pg_stat_statements comparison, index tuning, plan forcing, retry logic | engine DMVs |

#### Verification queries (portable idea, T-SQL shown)

```sql
-- 1. Structure
SELECT COUNT(*) AS row_count, MIN(created) AS min_created, MAX(created) AS max_created,
       SUM(CAST(amount AS decimal(38,6))) AS sum_amount,
       COUNT(DISTINCT customer_id) AS distinct_cust,
       SUM(CASE WHEN notes IS NULL THEN 1 ELSE 0 END) AS null_notes,
       SUM(CASE WHEN notes = '' THEN 1 ELSE 0 END)   AS blank_notes   -- Oracle migration check!
FROM   dbo.Orders;

-- 2. Hash per chunk (compare with the same computation on the other engine)
SELECT customer_id % 100 AS bucket,
       COUNT(*)           AS n,
       CHECKSUM_AGG(CHECKSUM(order_id, customer_id, amount)) AS chk
FROM   dbo.Orders GROUP BY customer_id % 100;
```
Note: `CHECKSUM` is engine-specific — compare using a *deterministic* function you control (e.g., computing `SUM(order_id)`, `SUM(LEN(notes))`, or an MD5/SHA of a canonical string produced identically on both sides).

### 6. Testing strategy

| Test | Purpose |
|---|---|
| **Contract tests** | Each view/proc returns the same column names, order, types |
| **Golden-set tests** | Fixed input data → expected output committed to repo, run on every engine |
| **Edge-case tests** | NULL, `''`, whitespace, case variants, min/max date, DST boundary, negative numbers, decimals at scale limits, Unicode, long strings |
| **Concurrency tests** | Deadlock/retry, isolation anomalies (note 6) |
| **Plan/performance gates** | Query Store regression alert; row/IO budget |

### 7. Idempotent deployment

```sql
-- T-SQL (2016 SP1+)
CREATE OR ALTER VIEW  api.vOrders AS SELECT …;
CREATE OR ALTER PROCEDURE etl.LoadOrders AS …;
DROP TABLE IF EXISTS dbo.Staging;
-- PostgreSQL: CREATE OR REPLACE VIEW/FUNCTION; DROP TABLE IF EXISTS
-- Oracle: CREATE OR REPLACE VIEW/PROCEDURE (no IF EXISTS before 23c)
```
Use a migration tool (Flyway, Liquibase, DbUp, SSDT/SqlPackage, dbt) so each engine has its own versioned scripts plus a shared test suite.

### 8. Worked example — migrating one Oracle report to SQL Server

**Source (Oracle)**

```sql
SELECT d.dname,
       COUNT(*)                               AS emp_cnt,
       LISTAGG(e.ename, ', ') WITHIN GROUP (ORDER BY e.ename) AS names,
       MAX(NVL(e.comm, 0))                    AS max_comm
FROM   emp e JOIN dept d ON d.deptno = e.deptno
WHERE  e.hiredate >= ADD_MONTHS(TRUNC(SYSDATE,'YYYY'), -12)
GROUP  BY d.dname
HAVING COUNT(*) > 2
ORDER  BY emp_cnt DESC
FETCH FIRST 10 ROWS ONLY;
```

**Inventory → classification**

| Fragment | Class | Action |
|---|---|---|
| `LISTAGG … WITHIN GROUP` | ⚠ | `STRING_AGG(… , ', ') WITHIN GROUP (ORDER BY …)` (2017+) |
| `NVL` | ⚠ | `COALESCE` |
| `ADD_MONTHS(TRUNC(SYSDATE,'YYYY'),-12)` | ⚠ | `DATEADD(year, -1, DATEFROMPARTS(YEAR(SYSDATETIME()),1,1))` |
| `FETCH FIRST 10 ROWS ONLY` | ✅ | works in 2012+ with `OFFSET 0 ROWS` |
| `emp`, `dept` | – | schema-qualify; case |

**Target (T-SQL)**

```sql
SELECT d.dname,
       COUNT(*)                                                   AS emp_cnt,
       STRING_AGG(CAST(e.ename AS nvarchar(max)), N', ')
              WITHIN GROUP (ORDER BY e.ename)                    AS names,
       MAX(COALESCE(e.comm, 0))                                   AS max_comm
FROM   dbo.emp  AS e
JOIN   dbo.dept AS d ON d.deptno = e.deptno
WHERE  e.hiredate >= DATEADD(year, -1, DATEFROMPARTS(YEAR(SYSDATETIME()), 1, 1))
GROUP  BY d.dname
HAVING COUNT(*) > 2
ORDER  BY emp_cnt DESC
OFFSET 0 ROWS FETCH NEXT 10 ROWS ONLY;
```

**Verify semantics** (what a text translation misses):
- Oracle `DATE` includes time — T-SQL `hiredate` should be `datetime2(0)`, not `date`.
- Oracle `LISTAGG` overflow errors at 4000 bytes; `STRING_AGG` returns `nvarchar(max)` only if input is MAX typed (hence the cast) — otherwise 8000/4000 limit raises Msg 9829.
- `COUNT(*) > 2` identical, but grouping on `dname` is case-**sensitive** in Oracle and case-**insensitive** in default SQL Server → groups may merge.

### 9. Under the hood: what makes migrations expensive

1. **Implicit behaviour** — defaults for NULL/blank, collations, rounding, truncation, isolation, error handling. No syntax shows them.
2. **Procedural code** — different languages with different transaction ownership.
3. **Performance cliffs** — hints, plan shapes, indexing strategies and statistics differ; a correct query can be 100× slower.
4. **Operational surface** — jobs, backup, HA, security model, monitoring all change.
5. **Data semantics** — time zones, encodings, numeric scale, binary types.

### 10. Performance and design implications

- Capture a **baseline workload** (top-N queries, durations, I/O) in the source before migrating; compare on the target using the same parameter sets.
- Review indexes: a "missing index" in the target often corresponds to an implicit index on the source engine (e.g., Oracle index-organised tables vs clustered index).
- Re-examine batch sizes and commit frequency; they interact with locking/MVCC differences.

### 11. Common mistakes

- Migrating schema and code but not **collation/time-zone** assumptions.
- Using string-based diff tools that hide numeric-precision/trailing-space differences.
- Trying to retain Oracle/PG idioms in T-SQL that don't fit (cursors, nested transactions).
- Not versioning the contract layer.
- Believing automated translation (SSMA, sqlglot) is "done" — treat it as a draft.

### 12. 🎯 Interview kit

**30-second answer:** "I decide how much portability is worth, put a stable contract layer of views and procedures in front of dialect-specific code, use ISO types and standard functions where cheap, and for migrations I inventory and classify SQL, translate, then verify with per-group counts, checksums and edge cases — the risky parts are semantics and performance, not syntax."

**Likely follow-ups:** *How would you verify a migration?* → counts, aggregates, hashes, sampled diffs, edge cases, plan baselines. *What do you do about procedural code?* → re-implement in target language with the transaction ownership model; test heavily. *What tools?* → SSMA, DMS, sqlglot, Babelfish Compass, dbt tests.

### 13. Check yourself

- [ ] Draw the layered contract and say what goes in each layer.
- [ ] Map five Oracle types and five PostgreSQL types to SQL Server and flag the semantic hazards.
- [ ] Write a verification query set for a migrated table.
- [ ] Explain why "automated conversion" doesn't remove the need for verification.

**Where this connects:** [Syntax Translation Cookbook](<03_Syntax_Translation_Cookbook.md>) · [Same Syntax, Different Semantics](<04_Same_Syntax_Different_Semantics.md>) · [dbt](<../../../dbt/_index.md>) · [Oracle](<../../../Oracle/_index.md>) · [PostgreSQL](<../../../PostgreSQL/_index.md>) · Next → [Practice Lab](<11_Practice_Lab.md>)
<!-- explainer:end -->
