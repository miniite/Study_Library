---
id: "TOOLS/SQL_Server/T-SQL/Dialects_and_Portability/08_T-SQL_Across_the_Microsoft_Family"
level: 5
type: "concept"
topic: "TOOLS/SQL_Server/T-SQL/Dialects_and_Portability"
tools: ["SQL_Server", "Azure"]
---

# 8 · T-SQL Across the Microsoft Family

**L5** · [↑ Series index](<_index.md>) · Prev: [Procedural Code](<07_Procedural_Code_T-SQL_vs_PLSQL_vs_PLpgSQL.md>) · Next: [ADF and Orchestration Tools](<09_ADF_and_Orchestration_Tools.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** "It's T-SQL" tells you the *grammar family*, not what will run — SQL Server, Azure SQL Database, Managed Instance, Synapse (dedicated and serverless) and Fabric Warehouse each expose a different subset of T-SQL on a different engine, so you must know *which* one your connection points at.

> ⚠ **Volatility notice.** These platforms change monthly. Treat the tables below as the *shape* of the differences, and confirm specific features on the current Microsoft documentation page (each T-SQL reference article ends with an "Applies to" list). Items marked *verify* are especially likely to have changed.

### 1. The family at a glance

| Product | Engine style | Typical use | Distinguishing trait |
|---|---|---|---|
| **SQL Server** (box, VM) | Single-instance, symmetric multiprocessing, row/column store | OLTP, mixed, on-prem/IaaS | Full surface: Agent, CLR, linked servers, cross-DB, FILESTREAM |
| **Azure SQL Database** | PaaS single DB / elastic pool / serverless | Cloud OLTP apps | Per-database scope; no cross-DB queries (use elastic query); RCSI default |
| **Azure SQL Managed Instance** | PaaS, instance-scoped | "Lift-and-shift" of SQL Server | Near-full instance features (Agent, cross-DB, linked servers) |
| **Azure Synapse dedicated SQL pool** (ex-SQL DW) | **MPP**: control node + 60 distributions | Data warehouse | Distribution/CTAS design; many OLTP features absent |
| **Azure Synapse serverless SQL pool** | Distributed on-demand query over files | Lake querying | `OPENROWSET`, external tables; read-focused |
| **Microsoft Fabric Warehouse / SQL analytics endpoint** | Distributed over OneLake (Delta/Parquet) | Lakehouse analytics | T-SQL subset; endpoint is read-only over lakehouse |
| **Azure SQL Edge / others** | Edge/IoT | – | Deprecated/limited |

### 2. How to find out where you are

```sql
SELECT @@VERSION AS version_banner,
       SERVERPROPERTY('EngineEdition')     AS engine_edition,   -- 2/3/4=box, 5=Azure SQL DB, 6=Synapse dedicated, 8=MI, 11=Synapse serverless/Fabric (verify)
       SERVERPROPERTY('ProductMajorVersion') AS major,
       DB_NAME()                           AS db,
       (SELECT compatibility_level FROM sys.databases WHERE name = DB_NAME()) AS compat_level;
```

| `EngineEdition` | Meaning |
|---|---|
| 1–4 | Personal/Standard/Enterprise/Express (box product) |
| 5 | Azure SQL Database |
| 6 | Azure Synapse dedicated SQL pool |
| 8 | Azure SQL Managed Instance |
| 9 | Azure SQL Edge |
| 11 | Synapse serverless / Fabric (*verify for your service*) |

In ADF the same check can be run from a **Script** or **Lookup** activity to log which engine a linked service really resolves to — useful when one linked-service name is reused across environments.

### 3. Capability matrix (indicative)

| Capability | SQL Server | Azure SQL DB | Managed Instance | Synapse dedicated | Synapse serverless | Fabric Warehouse |
|---|---|---|---|---|---|---|
| Multi-statement T-SQL, variables, `TRY…CATCH` | ✅ | ✅ | ✅ | ⚠ subset | ⚠ subset | ⚠ subset |
| Stored procedures | ✅ | ✅ | ✅ | ✅ | ⚠ (limited) | ✅ |
| Triggers | ✅ | ✅ | ✅ | ❌ | ❌ | ❌ *(verify)* |
| `MERGE` | ✅ | ✅ | ✅ | ⚠ *(verify; historically restricted)* | ❌ | ⚠ *(verify)* |
| `IDENTITY` | ✅ | ✅ | ✅ | ✅ (not gap-free; not sequential) | ❌ | ⚠ *(verify)* |
| Enforced PK/FK/UNIQUE | ✅ | ✅ | ✅ | ❌ `NOT ENFORCED` only | ❌ | ⚠ not enforced *(verify)* |
| Recursive CTE | ✅ | ✅ | ✅ | ❌ *(historically)* | ❌ | ⚠ *(verify)* |
| Cursors | ✅ | ✅ | ✅ | ❌ *(historically)* | ❌ | ❌ |
| `SELECT … INTO` | ✅ | ✅ | ✅ | ❌ use `CTAS` | ❌ | ❌ use `CTAS` |
| Temp tables `#t` | ✅ | ✅ | ✅ | ✅ (distribution-aware) | ⚠ limited | ✅ |
| Cross-database queries | ✅ | ❌ (elastic query) | ✅ | ❌ | ✅ (3-part names across DBs in the pool) | ⚠ within workspace |
| SQL Agent, linked servers, CLR | ✅ | ❌ | ✅ | ❌ | ❌ | ❌ |
| `BULK INSERT` / `OPENROWSET` | ✅ files | ✅ blob only | ✅ | `COPY INTO`, PolyBase | `OPENROWSET` over lake | `COPY INTO` |
| Default isolation | locking RC | RCSI | locking RC *(verify)* | READ UNCOMMITTED (!) or RCSI per setting | snapshot-like | snapshot isolation |
| `NVARCHAR(MAX)`/LOB columns in columnstore | ✅ | ✅ | ✅ | ⚠ limits | – | ⚠ |

*Reading the matrix:* a ⚠/❌ is not necessarily permanent. Pipelines frequently fail at "works in dev SQL DB, breaks in Synapse prod" boundaries because the grammar still *parses* but a feature is not supported — the message is usually `Msg 103010/104xxx` style ("… is not supported in this version of SQL Server") rather than `Msg 102`.

### 4. MPP-specific T-SQL (dedicated pool)

Distribution design introduces syntax and semantics SQL Server does not have:

```sql
CREATE TABLE dbo.FactSales
(
   SaleID     bigint        NOT NULL,
   CustomerID int           NOT NULL,
   Amount     decimal(18,2) NOT NULL,
   SaleDate   date          NOT NULL
)
WITH ( DISTRIBUTION = HASH(CustomerID),     -- how rows are spread over 60 distributions
       CLUSTERED COLUMNSTORE INDEX );        -- default storage

CREATE TABLE dbo.FactSales_Agg
WITH (DISTRIBUTION = ROUND_ROBIN)
AS SELECT CustomerID, SUM(Amount) AS Total FROM dbo.FactSales GROUP BY CustomerID;   -- CTAS
```

| Concept | Meaning | Why a SQL Server query pattern may become slow |
|---|---|---|
| `DISTRIBUTION = HASH(col)` / `ROUND_ROBIN` / `REPLICATE` | Data placement | A join on non-distribution columns triggers a **shuffle (DMS)** |
| `CTAS` | Parallel create-table-as-select | Preferred over `SELECT INTO`, `INSERT … SELECT` for bulk |
| `RENAME OBJECT`, partition switching | Atomic swaps | Used for "blue/green" table loads |
| Statistics are manual-ish | `CREATE STATISTICS` / auto-create | Missing stats → poor plans |
| Result-set caching, workload groups | Throttling and caching | Behavioural differences vs OLTP |

Also: no scalar UDF inlining benefits as in the box engine, different limits on query text size, `OPTION (LABEL = '…')` to tag queries, and DMVs under `sys.dm_pdw_*` rather than `sys.dm_exec_*`.

### 5. Serverless SQL pool — querying files

```sql
SELECT TOP 10 *
FROM OPENROWSET(
        BULK 'https://myacct.dfs.core.windows.net/lake/sales/*.parquet',
        FORMAT = 'PARQUET') AS r;
```
- Column types come from file metadata; `WITH (…)` clause to impose types for CSV.
- Collation of string columns from Parquet often needs `UTF8` collations (`Latin1_General_100_BIN2_UTF8`) — otherwise silent mismatches or truncation of Unicode in `varchar`.
- Billed by data processed; `SELECT *` and unfiltered scans are *financial* mistakes, not only performance ones.

### 6. Fabric Warehouse and SQL analytics endpoint

- Storage is Delta/Parquet in OneLake; the SQL surface is T-SQL **subset** evolving rapidly (*verify* each statement).
- The *SQL analytics endpoint* over a lakehouse is **read-only** from T-SQL; writes go through Spark/dataflows. A pipeline that issues `INSERT` against an endpoint fails with a permission/feature error, not a syntax error.
- Collation default is case-sensitive (`Latin1_General_100_BIN2_UTF8`) in Fabric Warehouse — a shift from SQL Server's default CI collation (*verify*): joins and filters may behave differently after migration.

### 7. Babelfish — T-SQL without SQL Server

Babelfish for Aurora PostgreSQL adds a **TDS endpoint** and a **T-SQL parser** on PostgreSQL:

- Applications keep connecting with SQL Server drivers.
- `babelfishpg_tsql` translates T-SQL batches into PostgreSQL internals; unsupported items appear in the *Babelfish Compass* assessment report.
- Behaviour is T-SQL-flavoured (identifier case-insensitivity, `@@ROWCOUNT`, `#temp`) but the engine is PostgreSQL, so execution plans, locking (MVCC) and hints differ.

It is the clearest proof that **dialect ≠ engine**.

### 8. Worked example — one pipeline, three targets

A pipeline loads `dbo.DimCustomer` with this pre-copy script:

```sql
IF OBJECT_ID('dbo.DimCustomer_stage') IS NOT NULL DROP TABLE dbo.DimCustomer_stage;
SELECT * INTO dbo.DimCustomer_stage FROM dbo.DimCustomer WHERE 1 = 0;
```

| Target | Result | Why |
|---|---|---|
| SQL Server / Azure SQL DB | ✅ | `SELECT … INTO` supported |
| Synapse dedicated | ❌ `SELECT INTO` not supported *(older builds)* or distribution defaults unwanted | Use `CREATE TABLE … WITH (DISTRIBUTION = ROUND_ROBIN, HEAP) AS SELECT * FROM … WHERE 1=0` |
| Fabric Warehouse | ⚠ differs | Use `CREATE TABLE … AS SELECT` (CTAS) |

**Pattern for portability across the family:** isolate platform-specific DDL into *named procedures* (`etl.PrepareStage`) with one implementation per platform; the pipeline calls the procedure name only.

### 9. Under the hood

- **Same parser lineage, different binder surface:** the grammar is shared; *feature gates* are enforced after parsing, which is why you see "not supported" messages instead of syntax errors.
- **Control-plane differences:** on Azure SQL DB the instance-level objects (`sys.server_principals`, `msdb`, `master` writes) are absent or virtualised; scripts that touch `msdb.dbo.sysjobs` fail outside box/MI.
- **Versioning model:** Azure SQL DB / Synapse / Fabric are *evergreen* — features arrive without a version number; box SQL Server uses compatibility levels. Both can differ for the same "latest" label.

### 10. Performance and design implications

- Design for the *engine*, not the dialect: hash distribution and columnstore thinking in Synapse; B-tree and index seeks in OLTP SQL Server.
- Avoid row-by-row procedural code on MPP engines (it executes on the control node, not distributed).
- Treat cross-DB queries as an architectural decision on Azure SQL DB (not available), not a syntax issue.

### 11. Common mistakes

- Developing in Azure SQL DB and assuming the SQL will run unchanged in Synapse.
- Using `@@VERSION` strings to branch logic instead of `EngineEdition`.
- Deploying SSMS-generated scripts (with `USE`, `GO`, `ALTER DATABASE … SET`) to Azure SQL DB.
- Ignoring collation differences on serverless/Fabric reading Parquet.

### 12. 🎯 Interview kit

**30-second answer:** "T-SQL is shared by SQL Server, Azure SQL DB, Managed Instance, Synapse and Fabric, but each supports a different subset and runs on a different engine — Synapse dedicated is MPP with distributions and CTAS; serverless queries files with OPENROWSET; Fabric is lake-based. I check `SERVERPROPERTY('EngineEdition')` and the 'Applies to' list before assuming a feature exists."

**Likely follow-ups:** *Why can't I `SELECT INTO` in Synapse dedicated?* → use CTAS with a distribution. *What is Babelfish?* → T-SQL/TDS compatibility on Aurora PostgreSQL. *Why would the same query be slower in Synapse?* → data movement because join keys ≠ distribution key.

### 13. Check yourself

- [ ] Write the query that identifies which Microsoft engine you're connected to.
- [ ] Convert a `SELECT INTO` staging step into CTAS for a dedicated pool.
- [ ] Explain why unsupported features show as feature errors, not syntax errors.
- [ ] Name two features present in MI but not in Azure SQL DB.

**Where this connects:** [SQL Server platform](<../../SQL_Server_Platform/_index.md>) · [Azure Data Engineering](<../../../Azure/Azure_Data_Engineering/_index.md>) · [Azure Data Factory](<../../../Azure/Azure_Data_Factory/_index.md>) · Next → [ADF and Orchestration Tools](<09_ADF_and_Orchestration_Tools.md>)
<!-- explainer:end -->
