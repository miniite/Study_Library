---
id: "TOOLS/SQL_Server/T-SQL/Dialects_and_Portability/09_ADF_and_Orchestration_Tools"
level: 4
type: "concept"
topic: "TOOLS/SQL_Server/T-SQL/Dialects_and_Portability"
tools: ["SQL_Server", "Azure"]
---

# 9 · ADF and Orchestration Tools

**L4** · [↑ Series index](<_index.md>) · Prev: [T-SQL Across the Microsoft Family](<08_T-SQL_Across_the_Microsoft_Family.md>) · Next: [Portable SQL and Migration Playbook](<10_Portable_SQL_and_Migration_Playbook.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** Azure Data Factory, Power BI, dbt and drivers do not understand SQL — they carry your text to a database engine (sometimes wrapping it first), so "ADF accepts T-SQL and rejects other patterns" really means "the engine behind this connector speaks T-SQL, and a wrapper may restrict what is legal around your query".

### 1. The mental model: who owns the text?

```
 you ──► [ tool: ADF / Power BI / dbt / pyodbc ] ──► [ driver ] ──► [ ENGINE: parser… ]
          may: split, wrap, template,                  may: rewrite
               append predicates, parametrise                escapes, parameters

   Only the ENGINE decides "valid / invalid SQL".
   The tool decides *what text the engine receives*.
```

Three independent facts therefore decide success:

1. **Which engine** is the connector pointing at? (dialect)
2. **How does the tool mutate your text** before sending? (wrapper)
3. **What shape of result does the tool expect?** (contract: one result set, column types, row limits)

### 2. Why "ADF accepts T-SQL and rejects other SQL"

| Observation | Real explanation |
|---|---|
| A T-SQL `TOP` works, `LIMIT` fails | The linked service targets SQL Server/Azure SQL/Synapse: the T-SQL parser decides |
| The same pipeline works with `LIMIT` against Snowflake | The Snowflake engine's parser decides; ADF never translated anything |
| ADF "accepts" a query that the Oracle connector rejects | Oracle: trailing `;` is invalid (`ORA-00911`); T-SQL tolerates `;` |
| An `ORDER BY` query rejected only when partitioning is on | ADF wraps the query as a subquery → `Msg 1033` |
| `GO` rejected | `GO` is not T-SQL (client directive) |
| Works in SSMS, fails in ADF | SSMS rewrites batches (`GO`), keeps session state; ADF sends one batch via a non-interactive identity |

So: **"accepts T-SQL pattern, rejects other"** = *the target database is a T-SQL engine*. Change the target, the accepted dialect changes with it.

### 3. Activities and what they do with SQL text

| Activity / component | How SQL is used | Constraints worth knowing |
|---|---|---|
| **Copy – source (Query)** | Sent as the source `SELECT`; ADF reads one result set | Single `SELECT`; no `GO`; with *partition option* ADF appends predicates (`?AdfDynamicRangePartitionCondition`) or wraps → no bare `ORDER BY`, no leading `WITH` unless you place the placeholder carefully |
| **Copy – source (Stored procedure)** | `EXEC proc @p=…` with parameters | Proc must return one result set; `SET NOCOUNT ON` recommended |
| **Copy – sink pre-copy script** | Runs before load, any T-SQL batch | No `GO`; failures abort copy |
| **Copy – sink (stored procedure + table type)** | Rows passed as a TVP | Table type must match schema |
| **Copy – sink Upsert** | Uses staging + keys | Key columns required; engine-specific implementation |
| **Lookup** | `SELECT` / proc; returns first row or all (limited rows/size) | Large results fail; use for small config only |
| **Script** | One or more T-SQL statements (NonQuery/Query) | Multi-statement ok; **no `GO`**; returns limited result |
| **Stored Procedure** | `EXEC` with typed parameters | Best for logic; no wrapper issues |
| **Mapping Data Flow (source "Query")** | SQL to source engine | Rest of flow is **not SQL** (expression language on Spark) |
| **Pipeline expressions** | `@concat()`, `@pipeline().parameters.x`, `@formatDateTime()` | Evaluated *before* the SQL is sent; not T-SQL |

### 4. Dynamic content — two languages braided together

```text
@concat('SELECT OrderID, OrderDate FROM Sales.Orders WHERE OrderDate >= ''',
        formatDateTime(pipeline().parameters.WindowStart, 'yyyy-MM-dd'), '''')
```

What happens: ADF evaluates the **expression** → produces a string → sends it to the engine, which parses it as T-SQL.

Traps:

| Trap | Why | Fix |
|---|---|---|
| Quote doubling | In an ADF expression `'` is escaped as `''`; in T-SQL string literals also `''` → quadruples confusing | Build with named parameters (Script activity parameters / stored procedure parameters) |
| `@` handling | A string beginning with `@` is evaluated as an expression; literal `@` at start needs `@@` | Put `SET NOCOUNT ON` first; avoid starting scripts with `@var` |
| Date format | `formatDateTime(x,'yyyy-MM-dd')` on a `datetime` column in a non-English session may mis-parse | `yyyyMMdd` or typed parameter |
| SQL injection | Concatenating user-controlled parameters | Whitelist table names; use `QUOTENAME`; stored procedures |
| Case/identifier | Postgres/Snowflake fold names; T-SQL doesn't | Quote identifiers consistently |

### 5. The wrapper problem in detail

When a Copy activity partitions a source (Physical partitions / Dynamic range), ADF can transform your query into something like:

```sql
SELECT * FROM ( <your query> ) AS adf_src
WHERE  <partition column> BETWEEN @lower AND @upper;
```
(Exact shapes vary by connector and version — check the activity's "Query to use" documentation.)

Everything illegal inside a derived table is now illegal in your query:

| Your query contains | Result | Fix |
|---|---|---|
| `ORDER BY` without `TOP`/`OFFSET` | `Msg 1033` | remove it (copy doesn't need order) |
| `WITH cte AS (…) SELECT …` | `Incorrect syntax near the keyword 'WITH'` | use the documented `?AdfDynamicRangePartitionCondition` placeholder placement, or make it a **view** |
| `DECLARE @x … ; SELECT …` | syntax error inside subquery | move to a stored procedure source |
| `SELECT … INTO #t` then select | not a single `SELECT` | proc |
| `;` at the end | `Incorrect syntax near ';'` in some wrappers | remove |
| duplicate column names | `The column 'x' was specified multiple times for 'adf_src'` | alias columns |

> **Rule of thumb:** *whatever isn't a single clean `SELECT` should live in the database as a view or stored procedure.* The tool then only needs `SELECT * FROM v` or `EXEC p`.

### 6. Type mapping — the silent half of a "query accepted" pipeline

Even a valid query goes through **ADF interim types** between source and sink:

| SQL Server type | Interim (ADF) type | Typical sink surprise |
|---|---|---|
| `datetime`, `datetime2` | `DateTime` | Parquet `timestamp` precision/time-zone assumptions |
| `datetimeoffset` | `DateTimeOffset` | Becomes UTC-normalised or string in some sinks |
| `decimal(38,18)` | `Decimal` | Precision > sink max (e.g. 38) fails; Parquet decimal scale loss |
| `bit` | `Boolean` | Becomes 0/1 or true/false by sink |
| `uniqueidentifier` | `Guid`/`String` | Case differences |
| `varchar/nvarchar(max)` | `String` | Truncation at sink column length |
| `money` | `Decimal` | scale 4 |
| `time` | `TimeSpan` | Unsupported in some sinks (Parquet, Snowflake mapping) |
| `hierarchyid`, `geography`, `sql_variant`, `xml` | unsupported / string | Cast in the query: `CAST(col AS nvarchar(max))`, `.STAsText()` |

**Defence:** make types explicit in the source query (`CAST(x AS datetime2(3))`), and validate the sink schema; do not rely on schema inference for production loads.

### 7. Other engines behind ADF — the accepted dialect flips

| Source linked service | Accepted dialect | Notable restrictions |
|---|---|---|
| SQL Server / Azure SQL / Synapse / Fabric | T-SQL | wrappers (above) |
| Oracle | Oracle SQL | no trailing `;`; `SYSDATE`, `ROWNUM`, `||` |
| PostgreSQL | PostgreSQL SQL | quoted-identifier case folding; `LIMIT` |
| MySQL | MySQL SQL | backticks; `LIMIT`; `||` is OR |
| Snowflake | Snowflake SQL | `QUALIFY`, `LIMIT`/`TOP`, uppercase folding |
| BigQuery, Databricks (via connector) | their dialects | backticks; project/dataset naming |
| SAP, Dynamics, REST | not SQL | OData/REST query parameters |

The **query to use** box therefore must be written in the *source's* dialect, and the **pre-copy script** in the *sink's* dialect.

### 8. Other tools, same pattern

| Tool | What it does to your SQL | Failure signature | Remedy |
|---|---|---|---|
| **Power BI** (Import with native query / DirectQuery) | Wraps as subquery; may add `TOP`/filters (query folding) | `Msg 1033`, CTE errors, "temp tables not supported" | Use views/procs; avoid `ORDER BY`; `Value.NativeQuery` options |
| **dbt (SQL Server / Synapse adapters)** | Wraps model SQL in `CREATE VIEW/TABLE AS` or `SELECT INTO`; CTEs allowed but leading `ORDER BY` not | `ORDER BY invalid in views…`, `CREATE VIEW must be first statement` | Remove `ORDER BY`, use macros (`{{ dbt.dateadd() }}`) for cross-dialect functions |
| **pyodbc / JDBC / ADO.NET** | One batch per call; parameter markers `?`/`@p` | `GO` errors, multiple result sets need `nextset()` | Split batches; `SET NOCOUNT ON` |
| **SSIS** | OLE DB source — parses query for metadata (`sp_describe_first_result_set`) | "invalid object" at design time for temp tables | Use procs with `WITH RESULT SETS`; see [SSIS](<../../SSIS/_index.md>) |
| **Spark / Databricks JDBC** | Wraps `(query) alias` → `SELECT * FROM (query) t` | same as Power BI | Provide subquery-safe SQL; use `query` option |
| **Azure Synapse pipelines / Fabric Data Factory** | Same as ADF; dedicated pool has additional `COPY INTO` staging | feature errors, not syntax errors | Check platform matrix (note 8) |

### 9. Worked example — fixing a query so it is accepted everywhere

Original author query (works in SSMS):

```sql
DECLARE @since date = '20240101';

WITH recent AS (
    SELECT CustomerID, OrderID, OrderDate,
           ROW_NUMBER() OVER (PARTITION BY CustomerID ORDER BY OrderDate DESC) AS rn
    FROM   Sales.Orders
    WHERE  OrderDate >= @since
)
SELECT CustomerID, OrderID, OrderDate
FROM   recent
WHERE  rn = 1
ORDER  BY CustomerID;
```

**Why it fails in an ADF Copy source with partitioning:** `DECLARE` + `;` + CTE + `ORDER BY` are all illegal when wrapped.

**Step 1 — move logic into the database:**

```sql
CREATE OR ALTER VIEW etl.vLatestOrderPerCustomer AS
WITH recent AS (
    SELECT CustomerID, OrderID, OrderDate,
           ROW_NUMBER() OVER (PARTITION BY CustomerID ORDER BY OrderDate DESC) AS rn
    FROM   Sales.Orders)
SELECT CustomerID, OrderID, OrderDate FROM recent WHERE rn = 1;
```

**Step 2 — the copy source becomes trivially wrapper-safe:**

```sql
SELECT CustomerID, OrderID, CAST(OrderDate AS datetime2(3)) AS OrderDate
FROM   etl.vLatestOrderPerCustomer
WHERE  OrderDate >= '@{formatDateTime(pipeline().parameters.Since,'yyyyMMdd')}'
```

**Step 3 — if parameters are needed server-side, use a procedure:**

```sql
CREATE OR ALTER PROCEDURE etl.GetLatestOrders @Since date
AS
BEGIN
    SET NOCOUNT ON;
    SELECT CustomerID, OrderID, OrderDate FROM etl.vLatestOrderPerCustomer WHERE OrderDate >= @Since;
END;
```
Stored Procedure source with parameter `Since` typed `Date`.

The *same view* can be exposed to Power BI and dbt — one T-SQL asset, no wrapper conflicts.

### 10. Debugging checklist

1. **Read the inner exception** in the activity output: the SQL error number maps to a stage ([note 5](<05_Parser_Binder_and_Error_Messages.md>)).
2. **Reproduce with the exact text:** copy the *evaluated* query from the activity input JSON (not the expression) into SSMS/Azure Data Studio, connect as the linked-service identity.
3. **Wrap it yourself:** run `SELECT * FROM ( <query> ) AS t;` — if that fails, the wrapper is the cause.
4. **Check engine and version:** `SELECT @@VERSION, SERVERPROPERTY('EngineEdition')` via a Lookup activity.
5. **Check types:** `sys.dm_exec_describe_first_result_set(N'<query>', NULL, 0)`.
6. **Check identity/permissions:** managed identity must have `SELECT`/`EXECUTE`.

### 11. Performance and design implications

- Push filters and aggregations **into the source query**; ADF itself is not a SQL engine and Copy simply streams rows.
- Prefer **incremental loads** with a watermark column (indexed) and parameterised procs.
- Use **partition options** only on columns with a supporting index and relatively even distribution; a wrapped `WHERE col BETWEEN` on a non-indexed column multiplies full scans.
- Use **staging + `COPY INTO`/PolyBase** for Synapse sinks; row-by-row `INSERT` copy is orders of magnitude slower.
- Keep query logic in version-controlled database objects, not in pipeline JSON.

### 12. Common mistakes

- Putting `GO` / `USE` in script blocks.
- Concatenating pipeline parameters into SQL without quoting.
- Writing source queries in the dialect of the *sink* or of the developer's local database.
- Using `SELECT *` in a source — a column added upstream silently changes the pipeline schema.
- Assuming the activity timeout is the SQL timeout (two different settings).

### 13. 🎯 Interview kit

**30-second answer:** "ADF doesn't parse or translate SQL; it forwards it to the engine of the linked service. A T-SQL source needs T-SQL, a Postgres source needs Postgres SQL. Additional errors come from wrapping — partitioned copy turns my query into a derived table so ORDER BY, CTE, DECLARE break — so I put logic in views or stored procedures and keep the source query a single simple SELECT."

**Likely follow-ups:** *How do you pass parameters safely?* → stored procedure activity or Script parameters, not `@concat`. *Why a Lookup returns only some rows?* → row/size limits; use Copy for volume. *How do you port a pipeline from SQL Server to Snowflake?* → change linked service + rewrite source/pre-copy SQL in Snowflake dialect; type mappings; identifier casing.

### 14. Check yourself

- [ ] Explain, in your own words, why `LIMIT` fails against SQL Server in ADF but works against Postgres.
- [ ] List four constructs that break under partition wrapping and the fix.
- [ ] Rewrite a CTE-with-`DECLARE` source query so it is wrapper-safe.
- [ ] Describe how you would debug an `Incorrect syntax near 'ORDER'` failure in a Copy activity.
- [ ] Name three interim-type surprises when copying SQL Server → Parquet.

**Where this connects:** [Azure Data Factory](<../../../Azure/Azure_Data_Factory/_index.md>) · [SSIS](<../../SSIS/_index.md>) · [Power BI](<../../../Power_BI/_index.md>) · [dbt](<../../../dbt/_index.md>) · Next → [Portable SQL and Migration Playbook](<10_Portable_SQL_and_Migration_Playbook.md>)
<!-- explainer:end -->
