---
id: "01_Databases_and_SQL/Database_Internals/DBMS_Concepts/Dynamic_SQL"
level: 4
type: "concept"
topic: "01_Databases_and_SQL/Database_Internals/DBMS_Concepts"
tools: ["Oracle", "SQL_Server", "MySQL"]
---

# Dynamic SQL

**L4 Advanced** · [↑ DBMS Concepts](<_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** Dynamic SQL builds a statement as a string at run time and executes it; use `sp_executesql` with real parameters and `QUOTENAME` for identifiers so you get plan reuse and immunity from SQL injection, and use it only where static SQL truly cannot express the query.

### 1. The big picture
Static SQL fixes table names, column lists and the shape of the query at compile time. Some needs cannot be expressed that way: a `PIVOT` over columns you only learn from the data, a search screen with ten optional filters, a maintenance script that must run `BACKUP DATABASE` or `ALTER INDEX` for each object found in the catalog, a query against a table name passed in as a parameter.

Dynamic SQL solves these by *constructing the text* and executing it. The power brings two risks: **SQL injection** (user input becomes code) and **plan-cache/compile cost** (every distinct string is a new statement). Done right — parameterized and with quoted identifiers — it is safe and fast; done by naive concatenation it is the most common route to a breach.

### 2. Core concepts and vocabulary
- **Dynamic SQL** — SQL text assembled and executed at run time.
- **`EXEC (@sql)`** — executes a string; no parameters; each unique string is a separate ad hoc plan.
- **`sp_executesql`** — executes a Unicode string with a *declared parameter list*, enabling parameterized plans.
- **SQL injection** — attacker-supplied text alters the statement's structure.
- **Parameterization** — values passed as typed parameters, never concatenated into text.
- **`QUOTENAME()`** — safely brackets an identifier (`[` `]`), doubling embedded `]`.
- **Whitelisting** — validating identifiers/options against known-good lists (`sys.tables`, `IN (…)`).
- **Ad hoc plan / plan cache bloat** — thousands of single-use plans from literal-embedded strings.
- **`optimize for ad hoc workloads`** — server option storing a small plan stub for first execution.
- **Batch scope** — a variable declared outside is not visible inside the dynamic batch; dynamic SQL runs in its own scope and security context (caller's permissions, not ownership-chained).

### 3. How it works, step by step
1. Build a Unicode string: `DECLARE @sql NVARCHAR(MAX) = N'SELECT … WHERE CustomerID = @cust';`
2. Add *structure* (identifiers, optional clauses) by concatenation — but only from validated/quoted sources.
3. Pass *values* as parameters: `EXEC sp_executesql @sql, N'@cust INT', @cust = @CustomerID;`
4. SQL Server parses/compiles the text (first time) and caches the plan keyed by text + parameter definitions; subsequent calls with the same text reuse it with different values.
5. Results/output return through result sets or `OUTPUT` parameters.

| Technique | Parameters | Plan reuse | Injection risk |
|---|---|---|---|
| `EXEC('… ' + @val)` | None (concatenation) | Poor (per literal) | High |
| `sp_executesql` with params | Yes | Good | Low (for values) |
| Static SQL with `OR @p IS NULL` | Yes | One plan for all shapes (sniffing issues) | None |
| ORM / client parameterized query | Yes | Good | Low |

### 4. Worked example
```sql
-- 1. Optional-filter search, safe and plan-friendly
CREATE OR ALTER PROCEDURE Sales.SearchOrders
    @CustomerID INT = NULL,
    @FromDate   DATE = NULL,
    @Status     VARCHAR(10) = NULL
AS
BEGIN
    SET NOCOUNT ON;
    DECLARE @sql NVARCHAR(MAX) = N'
        SELECT OrderID, CustomerID, OrderDate, Status, OrderTotal
        FROM Sales.Orders
        WHERE 1 = 1';
    IF @CustomerID IS NOT NULL SET @sql += N' AND CustomerID = @CustomerID';
    IF @FromDate   IS NOT NULL SET @sql += N' AND OrderDate >= @FromDate';
    IF @Status     IS NOT NULL SET @sql += N' AND Status = @Status';
    SET @sql += N' OPTION (RECOMPILE);';   -- optional: tune per shape of parameters

    EXEC sys.sp_executesql @sql,
         N'@CustomerID INT, @FromDate DATE, @Status VARCHAR(10)',
         @CustomerID, @FromDate, @Status;      -- unused parameters are fine
END;
GO

-- 2. Identifier from outside: whitelist + QUOTENAME
DECLARE @tbl SYSNAME = N'Orders', @sql NVARCHAR(MAX);
IF NOT EXISTS (SELECT 1 FROM sys.tables WHERE name = @tbl AND schema_id = SCHEMA_ID('Sales'))
    THROW 50001, 'Unknown table', 1;
SET @sql = N'SELECT COUNT(*) FROM Sales.' + QUOTENAME(@tbl) + N';';
EXEC sys.sp_executesql @sql;

-- 3. Output parameter from dynamic SQL
DECLARE @cnt INT;
EXEC sys.sp_executesql N'SELECT @c = COUNT(*) FROM Sales.Orders WHERE Status = @s',
     N'@c INT OUTPUT, @s VARCHAR(10)', @c = @cnt OUTPUT, @s = 'Open';

-- 4. Generated pivot columns from data
DECLARE @cols NVARCHAR(MAX) =
  (SELECT STRING_AGG(QUOTENAME(Region), N',') WITHIN GROUP (ORDER BY Region)
   FROM (SELECT DISTINCT Region FROM Sales.Customers) r);
DECLARE @pv NVARCHAR(MAX) = N'
  SELECT * FROM (SELECT Region, YEAR(OrderDate) AS Yr, OrderTotal FROM Sales.vOrdersByRegion) s
  PIVOT (SUM(OrderTotal) FOR Region IN (' + @cols + N')) p;';
EXEC sys.sp_executesql @pv;

-- Injection demo: what NOT to do
-- SET @sql = 'SELECT * FROM Users WHERE Name = ''' + @name + '''';   -- @name = ' OR 1=1; DROP TABLE Users;--
```
**Reading it:** values (`@CustomerID`…) are *parameters*, so even `'; DROP TABLE …` is just a string compared to a column. Identifiers cannot be parameters, so they are whitelisted and wrapped with `QUOTENAME`. Each distinct combination of optional filters produces one cached plan (at most 2³ here).

### 5. Under the hood
- **Plan cache keys.** The cache key is the exact text (case/whitespace-sensitive) plus SET options. `EXEC('… WHERE id = 5')` and `… id = 6` are two plans. With `sp_executesql` and `@id`, both reuse the same plan (and inherit *parameter sniffing* behavior).
- **Simple/forced parameterization** can auto-parameterize some literal queries but is not a substitute for explicit parameters.
- **Why data types matter.** Declare parameters with the column's type (`VARCHAR` vs `NVARCHAR`); mismatches cause implicit conversions that prevent index seeks (the same effect as the data-conversion plan issue in indexing).
- **Scope and permissions.** Dynamic batches cannot see local variables/temp tables created *inside* them after they finish (temp tables created inside vanish); `#temp` tables created outside *are* visible inside. Permissions are checked against the caller (ownership chaining does not extend into the string), so the caller needs rights on underlying objects unless `EXECUTE AS` is used.
- **Injection mechanisms.** Classic string-breaking, second-order injection (malicious data stored then concatenated later), and injection through identifiers, `ORDER BY` columns, and `LIKE` patterns. Escaping quotes (`REPLACE(@s,'''','''''')`) is a fallback, not a defense for identifiers.
- **Other engines.** Oracle: `EXECUTE IMMEDIATE … USING`; PostgreSQL: `EXECUTE format('… %I …', ident) USING val`; MySQL: `PREPARE/EXECUTE`. Same principles: bind values, quote identifiers.

### 6. Performance and design implications
- Parameterized dynamic SQL ≈ static SQL cost after first compile; the extra cost is string building (negligible) and optimizer time for each *shape*.
- Avoid constructing literals into text: plan cache bloat, compile CPU, and `optimize for ad hoc workloads` only softens it.
- For "kitchen-sink" searches, generating only the needed predicates gives better plans than `(@p IS NULL OR col = @p)` which yields one compromise plan; add `OPTION (RECOMPILE)` when parameter distributions vary wildly.
- Keep dynamic SQL small and isolated; log the generated text when debugging (`PRINT @sql`) and test with edge inputs.
- Index design must account for all generated shapes.
- Use least privilege: the executing login should not be `sysadmin`; consider module signing for needed rights.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `sp_executesql @stmt, @params, …` | Parameterized dynamic execution | Default choice |
| `EXEC (@sql)` | Unparameterized execution | Rare admin scripts with trusted input |
| `QUOTENAME(name[, quote])` | Safe identifier quoting | Every dynamic identifier |
| `STRING_AGG`, `FORMATMESSAGE` | Build lists/text | Generated column lists |
| `OPTION (RECOMPILE)` | Plan per execution | Variable shapes |
| `sys.sp_executesql` + `OUTPUT` params | Return values | Counters, ids |
| `EXECUTE AS` / module signing | Controlled permissions | Restricted callers |
| `optimize for ad hoc workloads` | Smaller single-use plan footprint | Servers with legacy ad hoc code |

### 8. Trade-offs and decisions
| Choose dynamic SQL when | Avoid / watch out when |
|---|---|
| Structure (columns, tables, clauses) varies at run time | A static query with `CASE`/`COALESCE` or separate procedures is clear enough |
| Admin tasks iterate over catalog objects | The logic could use a set-based catalog query without strings |
| Pivot/unpivot with unknown elements | Users provide free-form SQL fragments |
| Optional-filter search needing good plans | Permissions chain relies on the calling procedure |

### 9. Common mistakes
- Concatenating user input → SQL injection → `sp_executesql` parameters.
- Not quoting identifiers → injection and failures on odd names → `QUOTENAME`.
- `VARCHAR` text with `N''` literals mismatch / non-Unicode `@sql` for `sp_executesql` → error or truncation → `NVARCHAR(MAX)`.
- Forgetting parameter list updates when adding filters → "must declare scalar variable" → pass all parameters consistently.
- Detailed error messages exposed to users → information leak → log internally, return generic.
- Assuming temp tables created inside persist → missing object → create outside or use `##`/permanent staging.
- Overusing dynamic SQL for convenience → untestable code → use static SQL where possible.

### 10. 🎯 Interview kit
**30-second answer:** Dynamic SQL is building a statement as a string and executing it at run time. I use `sp_executesql` with typed parameters for values and `QUOTENAME` plus whitelists for identifiers, which prevents injection and lets plans be reused; I only use it when the query shape genuinely varies.

**Structure a longer answer:** clarify (what varies: values or structure?) → static alternatives → safe construction → plan caching and permissions → example (search screen, dynamic pivot).

**Likely follow-ups:**
- *`EXEC` vs `sp_executesql`?* → parameters, plan reuse, injection safety, output params.
- *Can identifiers be parameters?* → no; whitelist and `QUOTENAME`.
- *How do you prevent injection beyond parameters?* → least privilege, validation, error handling, logging.
- *Why is my dynamic query not using the cached plan?* → text differs; literals concatenated.
- *Dynamic SQL and permissions?* → caller context; `EXECUTE AS`/signing.

**Signals interviewers look for:** security-first construction, plan-cache awareness, and restraint.

### 11. Check yourself
- [ ] Rewrite a concatenated `WHERE name = '…'` query with `sp_executesql`.
- [ ] Explain why a table name cannot be a parameter and how to handle it.
- [ ] Build a dynamic pivot safely.
- [ ] Describe how plan cache differs between `EXEC('… 5')` and parameterized execution.
- [ ] List three defenses beyond parameterization.
<!-- explainer:end -->

## Implementations (tools)

- [Oracle](<../../../TOOLS/Oracle/_index.md>)
- [SQL Server](<../../../TOOLS/SQL_Server/_index.md>)
- [MySQL](<../../../TOOLS/MySQL/_index.md>)

## Practice questions

- [Basic] [What are the different types of SQL commands (DDL, DML, DCL, TCL)?](<../../../PRACTICE/01_Databases_and_SQL/Database_Internals/DBMS_Concepts/Q-22154_What_are_the_different_types_of_SQL_commands_DDL_DML_DCL_TCL.md>)
- [Basic] [What are string functions like SUBSTRING CONCAT UPPER and TRIM and give a use case for each?](<../../../PRACTICE/01_Databases_and_SQL/Database_Internals/DBMS_Concepts/Q-22374_What_are_string_functions_like_SUBSTRING_CONCAT_UPPER_and_TR.md>)
- [Intermediate] [What is a database trigger and give a practical use case for one.](<../../../PRACTICE/01_Databases_and_SQL/Database_Internals/DBMS_Concepts/Q-22182_What_is_a_database_trigger_and_give_a_practical_use_case_for.md>)
- [Intermediate] [What is the difference between a stored procedure parameter that is IN OUT or INOUT?](<../../../PRACTICE/01_Databases_and_SQL/Database_Internals/DBMS_Concepts/Q-22411_What_is_the_difference_between_a_stored_procedure_parameter_.md>)
- [Intermediate] [What is the difference between a BEFORE trigger and an AFTER trigger?](<../../../PRACTICE/01_Databases_and_SQL/Database_Internals/DBMS_Concepts/Q-22415_What_is_the_difference_between_a_BEFORE_trigger_and_an_AFTER.md>)
- [Intermediate] [How do you propagate a custom error from PL/SQL to the calling application using RAISE_APPLICATION_ERROR and named exceptions?](<../../../PRACTICE/01_Databases_and_SQL/Database_Internals/DBMS_Concepts/Q-22459_How_do_you_propagate_a_custom_error_from_PL_SQL_to_the_calli.md>)
- [Deep] [What is the PL/SQL BULK COLLECT clause and how does it reduce context switches between the SQL and PL/SQL engines?](<../../../PRACTICE/01_Databases_and_SQL/Database_Internals/DBMS_Concepts/Q-22449_What_is_the_PL_SQL_BULK_COLLECT_clause_and_how_does_it_reduc.md>)
- [Deep] [What is the difference between the PRAGMA EXCEPTION_INIT declaration and just checking SQLCODE after the fact?](<../../../PRACTICE/01_Databases_and_SQL/Database_Internals/DBMS_Concepts/Q-22460_What_is_the_difference_between_the_PRAGMA_EXCEPTION_INIT_dec.md>)

## Related concepts

- [Stored Procedure Performance and Design](<../../Database_Objects/Stored_Procedures/Stored_Procedure_Performance_and_Design.md>) *(Stored Procedures)*
- [PL-SQL in Production](<../../../TOOLS/Oracle/PL-SQL/PL-SQL_in_Production.md>) *(PL-SQL)*
- [Functions in SQL vs PL/SQL](<../../../TOOLS/Oracle/PL-SQL/Functions_in_SQL_vs_PLSQL.md>) *(PL-SQL)*

## References

- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 7: Dynamic SQL, Triggers, and Table-Valued Functions (TVFs) › Comparing spexecutesql with EXEC()
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 7: Dynamic SQL, Triggers, and Table-Valued Functions (TVFs) › Best Practices Recap
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 7: Dynamic SQL, Triggers, and Table-Valued Functions (TVFs) › Conclusion
- *T-SQL Fundamentals (4th ed.)* — Itzik Ben-Gan — Programmable objects
- *Databricks Certified Data Engineer Associate Study Guide* — Derar Alhussein — Transforming Data with Apache Spark › Example 2: database
- *Beginning T-SQL* — Kathi Kellenberger, Lee Everest — WHILE
- *Beginning T-SQL* — Kathi Kellenberger, Lee Everest — Writing Advanced Queries › The OUTPUT Clause
- *Fuzzy Data Matching with SQL* — Jim Lehmer — Code-Generating Code
- *Data Engineering Best Practices* — Luiz Fernando F. Dos Santos — Data Models › Naming convention for other elements
- *SQL Server Tutorial* (YouTube series) — Part 144: exec vs sp executesql in sql server ([watch](https://www.youtube.com/watch?v=6ZoXXJ3hvRU))

