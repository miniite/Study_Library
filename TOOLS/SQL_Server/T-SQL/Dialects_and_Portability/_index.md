---
id: "TOOLS/SQL_Server/T-SQL/Dialects_and_Portability"
level: 4
type: "topic"
topic: "TOOLS/SQL_Server/T-SQL"
tools: ["SQL_Server"]
---

# T-SQL, ANSI SQL and Dialects — Deep-Dive Series

**L4–L5 Series** · 11 notes · [↑ T-SQL](<../_index.md>)

<!-- explainer:start -->
## 📚 What this series answers

> *Is T-SQL unique to SQL Server? Is every query I write in SQL Server "T-SQL" or "ANSI SQL"? Why do tools like Azure Data Factory accept one SQL pattern and reject another? What actually differs behind the syntax?*

This series takes those questions apart in layers, from "what is a dialect" down to "which stage of the engine produced this error message", and then back up to practical playbooks (writing portable SQL, migrating, orchestrating from ADF).

### The one picture to keep in your head

```
 ┌─ L1  THE STANDARD ───────────── ISO/IEC 9075 (SQL-92 … SQL:2023): a menu, never implemented in full
 │
 ├─ L2  THE DIALECT ────────────── T-SQL · PL/SQL · PostgreSQL SQL · MySQL SQL · Snowflake SQL …
 │        (grammar + functions + types + semantic defaults)
 │
 ├─ L3  THE ENGINE ─────────────── parser → binder → optimizer → executor → storage / lock manager
 │        (where 'Incorrect syntax', 'Invalid column', 'deadlock victim' come from)
 │
 ├─ L4  THE CLIENT / WRAPPER ───── SSMS (GO), sqlcmd, pyodbc, JDBC, ADF, Power BI, dbt
 │        (may split, wrap, rewrite or forward your text)
 │
 └─ L5  THE PLATFORM VARIANT ───── SQL Server · Azure SQL DB · Managed Instance · Synapse · Fabric
          (same dialect name, different supported surface)
```

Every confusing "this works there but not here" problem is one of those five layers disagreeing. The notes below are ordered so each layer is explained before you need it.

### Reading order

| # | Note | Layer | What you get |
|---|---|---|---|
| 1 | [Standard, Dialect and Engine](<01_Standard_Dialect_Engine.md>) | L1–L3 | Vocabulary, history, who owns "T-SQL", what ANSI conformance means |
| 2 | [ANSI Core vs T-SQL Extensions](<02_ANSI_Core_vs_T-SQL_Extensions.md>) | L2 | Catalogue: portable / extension / missing, with versions and a self-audit tool |
| 3 | [Syntax Translation Cookbook](<03_Syntax_Translation_Cookbook.md>) | L2 | Recipe-style rewrites between T-SQL, PostgreSQL, MySQL, Oracle, Snowflake/Spark |
| 4 | [Same Syntax, Different Semantics](<04_Same_Syntax_Different_Semantics.md>) | L2–L3 | The silent traps: NULL, `''`, padding, collation, dates, conversions |
| 5 | [Parser, Binder and Error Messages](<05_Parser_Binder_and_Error_Messages.md>) | L3 | Engine pipeline and an error-number → stage → fix catalogue |
| 6 | [Transactions and Concurrency Across Engines](<06_Transactions_and_Concurrency_Across_Engines.md>) | L3 | Locking vs MVCC, autocommit, error-abort rules, porting consequences |
| 7 | [Procedural Code: T-SQL vs PL/SQL vs PL/pgSQL](<07_Procedural_Code_T-SQL_vs_PLSQL_vs_PLpgSQL.md>) | L2 | Variables, loops, errors, cursors, dynamic SQL, triggers side by side |
| 8 | [T-SQL Across the Microsoft Family](<08_T-SQL_Across_the_Microsoft_Family.md>) | L5 | SQL Server vs Azure SQL vs MI vs Synapse vs Fabric, and how to tell where you are |
| 9 | [ADF and Orchestration Tools](<09_ADF_and_Orchestration_Tools.md>) | L4 | Why a query is accepted or rejected in ADF, Power BI, dbt, pyodbc |
| 10 | [Portable SQL and Migration Playbook](<10_Portable_SQL_and_Migration_Playbook.md>) | all | Design rules, type maps, tooling, checklist |
| 11 | [Practice Lab](<11_Practice_Lab.md>) | all | 29 exercises with worked answers |

**Short on time?** Read note 1 (15 min), the layer table in note 5, and note 9 — that answers the original "why does ADF reject my SQL" question end to end.

### How this connects to the rest of the T-SQL material

| If you are studying… | Use this series for… |
|---|---|
| [T-SQL Basics](<../T-SQL_Basics.md>) — logical query processing, NULLs | Which of those rules are *standard* and which are SQL Server's own (notes 2, 4) |
| [CROSS APPLY and OUTER APPLY](<../CROSS_APPLY_and_OUTER_APPLY.md>) | `APPLY` vs standard `LATERAL` and its translations (notes 2, 3) |
| [MERGE Pitfalls and Error Handling](<../MERGE_Pitfalls_and_Error_Handling.md>) | `MERGE` vs `ON CONFLICT` / `ON DUPLICATE KEY`; error behaviour (notes 3, 6, 7) |
| [Temp Tables vs Table Variables](<../Temp_Tables_vs_Table_Variables.md>) | What other engines offer instead of `@table` variables (note 7) |
| [Filtered Indexes and Plan Cache](<../Filtered_Indexes_and_Plan_Cache.md>) | `SET` options that must match for plan reuse; ANSI options (notes 2, 5) |
| [Logical Query Processing](<../../06_T-SQL_Programming/Logical_Query_Processing/_index.md>) | Where alias-reuse differences between dialects come from (notes 3, 5) |
| [Window Functions](<../../06_T-SQL_Programming/Window_Functions/_index.md>) | Windowing features SQL Server lacks (`FILTER`, `DISTINCT` in OVER) (note 2) |
| [Isolation and Row Versioning](<../../05_Concurrency/Isolation_and_Row_Versioning/_index.md>) | The SQL Server half of the concurrency comparison (note 6) |
| [Azure Data Factory](<../../../Azure/Azure_Data_Factory/_index.md>) | Query-text behaviour of Copy/Lookup/Script activities (note 9) |
| [Oracle PL/SQL](<../../../Oracle/PL-SQL/_index.md>), [PostgreSQL](<../../../PostgreSQL/_index.md>), [MySQL](<../../../MySQL/_index.md>), [Snowflake](<../../../Snowflake/_index.md>) | The "other side" of every comparison table |

### Conventions used in the notes

- **Version tags** like *(2012+)* mean "available from that SQL Server version". Where a feature depends on database compatibility level, the note says so.
- **✅ portable / ⚠ dialect-specific / ❌ missing** icons mark each construct's status.
- **Hedged statements** (marked *verify*) are facts that change between releases or platforms; check the current Microsoft documentation's "Applies to" banner before relying on them.
- All SQL is T-SQL unless the comment says otherwise. Examples use a small `Sales.Orders` / `Sales.Customers` schema in the style of the T-SQL Fundamentals books.
<!-- explainer:end -->
