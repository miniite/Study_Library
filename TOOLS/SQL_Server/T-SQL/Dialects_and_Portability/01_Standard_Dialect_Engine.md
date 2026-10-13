---
id: "TOOLS/SQL_Server/T-SQL/Dialects_and_Portability/01_Standard_Dialect_Engine"
level: 4
type: "concept"
topic: "TOOLS/SQL_Server/T-SQL/Dialects_and_Portability"
tools: ["SQL_Server"]
---

# 1 · Standard, Dialect and Engine

**L4** · [↑ Series index](<_index.md>) · Next: [ANSI Core vs T-SQL Extensions](<02_ANSI_Core_vs_T-SQL_Extensions.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** "SQL" names a *standard*; "T-SQL" names *one product family's dialect* of it; the *engine* is the software that parses and executes that dialect — and most confusion comes from mixing those three up.

### 1. Three words people use as if they meant the same thing

| Word | What it actually is | Example |
|---|---|---|
| **Standard** | A written specification, maintained by ISO/IEC (and previously ANSI) — a document, not software | ISO/IEC 9075, revisions SQL-92, SQL:1999, SQL:2003, SQL:2008, SQL:2011, SQL:2016, SQL:2023 |
| **Dialect** | One vendor's *language*: grammar, built-in functions, data types and semantic defaults | T-SQL, PL/SQL + Oracle SQL, PostgreSQL SQL, MySQL SQL, Snowflake SQL |
| **Engine** | The program that implements a dialect: parser, optimizer, executor, storage | The SQL Server database engine (`sqlservr.exe`), PostgreSQL's `postgres` backend |

A useful analogy: the standard is like a *grammar textbook for "Romance languages"*; Spanish, Italian and French are the dialects; a native speaker's brain is the engine. A sentence that works in Italian may be rejected in Spanish — not because either is "wrong" but because each has its own rules on top of the shared roots.

> ⚠ **Nobody implements the whole standard.** The standard is organised into a mandatory *core* plus many optional feature packages. Vendors implement a core subset, add their own features, and implement some standard features with different behaviour. "ANSI compliant" on a datasheet is a marketing statement about a *level* (for example, SQL-92 *Entry*), not about the current standard.

### 2. What "T-SQL" means precisely

**T-SQL (Transact-SQL)** is the SQL dialect used by the Microsoft SQL Server family. It has two inseparable halves:

1. **The query/DDL/DML language** — `SELECT`, `INSERT`, `CREATE TABLE`, joins, CTEs, windows…
2. **The procedural extensions** — variables (`DECLARE @x`), `IF…ELSE`, `WHILE`, `TRY…CATCH`, stored procedures, functions, triggers, cursors, dynamic SQL.

Both halves are accepted by the same parser. There is no separate "SQL mode" and "T-SQL mode". That is the exact reason the answer to *"is every query in SQL Server T-SQL?"* is **yes**: whatever text the server accepts is T-SQL by definition, including the portable core.

The reverse is the interesting part: not every *standard* SQL statement is valid T-SQL, and not every T-SQL statement is valid standard SQL.

```
                     standard SQL statements
              ┌────────────────────────────────────┐
              │   ┌──────────────┐                 │
              │   │  portable    │  not implemented│
              │   │  core        │  by SQL Server  │
              │   │  (= valid in │  (e.g. FILTER   │
              │   │   both)      │   clause,       │
              │   └──────┬───────┘   INTERVAL,     │
              └──────────┼───────────row-values)───┘
                         │
        ┌────────────────┼────────────────────────┐
        │   T-SQL        │  extensions only        │
        │   statements   │  T-SQL has (TOP, APPLY, │
        │                │  OUTPUT, @vars, #temp)  │
        └─────────────────────────────────────────┘
```

### 3. A short history (and why it matters)

| Year | Event | Why you care |
|---|---|---|
| 1986 | First ANSI SQL standard (SQL-86) | Defines the idea of a vendor-neutral language |
| 1987–89 | Sybase builds *Sybase SQL Server* with "Transact-SQL"; Microsoft/Ashton-Tate license it and port it to OS/2 — **SQL Server 1.0 (1989)** | T-SQL's `@variables`, `#temp`, `GO`, `TOP`, `@@ROWCOUNT` come from this Sybase heritage |
| 1992 | SQL-92 | The baseline most vendors claim "entry level" against; explicit `JOIN … ON` syntax |
| 1994–96 | Microsoft and Sybase part ways; Sybase continues as SAP ASE, Microsoft as SQL Server | Two diverging T-SQL variants; modern "T-SQL" means Microsoft's |
| 2005 | CTEs, `PIVOT`, `APPLY`, `TRY…CATCH`, ranking window functions, `INTERSECT/EXCEPT` | Large catch-up to the standard (`SQL:1999/2003`) |
| 2008 | `MERGE`, row constructors, `GROUPING SETS`, `date`/`datetime2` | Mostly standard features |
| 2012 | `OFFSET/FETCH`, full window frames, `SEQUENCE`, `THROW`, `LAG/LEAD` | `TOP` is no longer the only paging option |
| 2016–2019 | JSON functions, temporal tables, `STRING_AGG`, graph tables | Some follow SQL:2011/2016, some are proprietary |
| 2022 | `IS [NOT] DISTINCT FROM`, `GREATEST/LEAST`, `WINDOW` clause, `DATETRUNC`, `GENERATE_SERIES` | More portability — but gated by compatibility level |

The history explains a design trait: T-SQL tends to **keep old syntax forever** (backward compatibility) and **add the standard form later**. So SQL Server frequently has *two* ways to do something (`TOP` and `OFFSET/FETCH`; `ISNULL` and `COALESCE`; `CONVERT` and `CAST`; `GETDATE()` and `CURRENT_TIMESTAMP`). The older proprietary form is usually the one that sticks in tutorials and legacy code.

### 4. Where T-SQL lives today — the "family"

"T-SQL" is not one fixed thing even inside Microsoft:

| Product | Language it accepts | Notes |
|---|---|---|
| SQL Server (on-prem / VM) | Full T-SQL | The reference surface |
| Azure SQL Database | Almost full T-SQL | PaaS; some instance-level features absent |
| Azure SQL Managed Instance | Near-full T-SQL | Closest PaaS match to SQL Server |
| Azure Synapse **dedicated** pool | T-SQL **subset** + MPP-specific DDL (`DISTRIBUTION`, `CTAS`) | Distributed engine; many features absent |
| Azure Synapse **serverless** pool | T-SQL subset for querying files (`OPENROWSET`) | Read-focused |
| Microsoft Fabric Warehouse / SQL endpoint | T-SQL subset | Evolving quickly |
| Babelfish for Aurora PostgreSQL | T-SQL *translated* onto a PostgreSQL engine | Shows that dialect ≠ engine |

Details and feature differences are in [T-SQL Across the Microsoft Family](<08_T-SQL_Across_the_Microsoft_Family.md>).

### 5. Dialect ≠ engine — a demonstration

Two facts show the dialect and engine are separable:

- **Same dialect, different engines.** SQL Server and Synapse dedicated pool both "speak T-SQL", but they are different engines (single-node B-tree row store vs distributed MPP). The same T-SQL text may be rejected, or accepted but execute with very different cost behaviour.
- **Different dialect, same engine.** PostgreSQL's engine can be driven through Babelfish with T-SQL. Conversely, one engine may accept two dialect "modes" (e.g. MySQL `ANSI_QUOTES`, Oracle's compatibility modes in some products).

So when you read an error message you should always ask: *which* parser rejected this text — the one on the machine that holds the data?

### 6. What "ANSI compliant" really tells you

| Claim | What it actually promises | What it does *not* promise |
|---|---|---|
| "Supports SQL-92 entry level" | A core of SELECT/INSERT/UPDATE/DELETE/DDL/transactions works as specified | Any of the later features: CTEs, windows, `MERGE`, JSON |
| "ANSI-compliant joins" | Explicit `JOIN … ON` syntax is supported | That `*=`-style old outer joins also work (SQL Server removed them for compatibility level 90+) |
| "Standard behaviour for NULL" | Three-valued logic, `IS NULL`, `COALESCE` | That `''` is distinct from `NULL` (it is in SQL Server; it is not in Oracle) |
| "Implements the standard datetime types" | `DATE`, `TIME`, `TIMESTAMP`-like types | A `TIMESTAMP` that means a point in time — in T-SQL `timestamp` is a **row-version counter** (a famous naming collision) |

> 🧠 **Takeaway.** The standard guarantees *less* than people hope: it constrains *syntax and core semantics*, not defaults like case-sensitivity, isolation level, date format, string padding or error behaviour. Those defaults are where "it ran fine on my old database" stories come from (see [Same Syntax, Different Semantics](<04_Same_Syntax_Different_Semantics.md>)).

### 7. The practical classification you will use every day

When you meet a piece of SQL, label it with one of three tags:

| Tag | Meaning | Example in SQL Server | Port to other engine |
|---|---|---|---|
| ✅ **Portable core** | Standard syntax, standard-ish semantics | `SELECT … FROM … JOIN … GROUP BY … HAVING`, `CASE`, `COALESCE`, `CAST`, `UNION`, window `ROW_NUMBER() OVER (…)` | Usually works as is |
| ⚠ **Extension** | Exists only in T-SQL (or only in the Microsoft family) | `TOP`, `CROSS APPLY`, `OUTPUT`, `PIVOT`, `@@ROWCOUNT`, `#temp`, `WITH (NOLOCK)` | Needs rewrite |
| 🔀 **Standard name, different behaviour** | Looks standard, behaves differently | `''` vs NULL rules, `timestamp`, `UNIQUE` with NULLs, `ISNULL` return type | Looks fine, returns different data — most dangerous |

The next note catalogues these three groups.

### 8. Performance and design implications

- **The optimizer works on the logical operators, not the syntax.** Writing `TOP` vs `OFFSET/FETCH`, or a CTE vs derived table, rarely changes the plan. What changes plans: sargability, data types, statistics, indexes. Portability and performance are mostly independent concerns.
- **Dialect-specific hints** (`WITH (NOLOCK)`, `OPTION (RECOMPILE)`, `FORCESEEK`) are *execution* controls; they have no portable counterpart, and should be isolated in one layer (views/procedures) so the portable surface stays small.
- **A portable query is not automatically fast on every engine.** Each engine's cost model, index types and statistics differ; "portable" means "parses and returns the same rows", not "performs the same".

### 9. Common mistakes

- "SQL Server uses SQL, T-SQL is just stored procedures." → T-SQL is the *entire* dialect, queries included.
- "My query is ANSI so it'll work anywhere." → syntax may port; semantics, defaults and performance may not.
- "ADF/Power BI translates my SQL for me." → it does not; the target engine parses it (note 9).
- "`timestamp` stores a timestamp." → in SQL Server it is a binary row-version; use `datetime2`/`datetimeoffset`.

### 10. 🎯 Interview kit

**30-second answer:** "SQL is a standard; T-SQL is Microsoft's dialect that SQL Server and its family speak. It includes the portable core plus extensions like TOP, APPLY, OUTPUT and the procedural layer. Every query in SQL Server is parsed as T-SQL, but a lot of it is also valid ANSI SQL."

**Likely follow-ups:** *Who else uses T-SQL?* → SAP ASE (Sybase lineage) uses a related variant; Babelfish emulates SQL Server's T-SQL on PostgreSQL. *Does ANSI mode exist in SQL Server?* → No mode switch; there are session options (`ANSI_NULLS`, `QUOTED_IDENTIFIER`…) that make behaviour standard-conforming, and `SET FIPS_FLAGGER` to flag non-entry-level syntax (deprecated). *Why two ways to paginate?* → backward compatibility.

### 11. Check yourself

- [ ] Explain standard vs dialect vs engine in one sentence each.
- [ ] Give one T-SQL statement that is valid ANSI, one that is T-SQL-only, one that is valid ANSI but *not* valid T-SQL.
- [ ] Why does SQL Server often have two ways to do the same thing?
- [ ] Why is Synapse's T-SQL "different T-SQL" from SQL Server's?
- [ ] What does "ANSI SQL-92 entry level" guarantee, and what doesn't it?

**Where this connects:** [T-SQL Basics](<../T-SQL_Basics.md>) · [SQL Server platform](<../../_index.md>) · Next → [ANSI Core vs T-SQL Extensions](<02_ANSI_Core_vs_T-SQL_Extensions.md>)
<!-- explainer:end -->
