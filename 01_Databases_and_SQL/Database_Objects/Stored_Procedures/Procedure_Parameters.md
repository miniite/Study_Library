---
id: "01_Databases_and_SQL/Database_Objects/Stored_Procedures/Procedure_Parameters"
level: 4
type: "concept"
topic: "01_Databases_and_SQL/Database_Objects/Stored_Procedures"
tools: ["SQL_Server"]
---

# Procedure Parameters

**L4 Advanced** · [↑ Stored Procedures](<_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** A procedure's parameters are its contract — input values that arrive as *data* (never as executable SQL), optional defaults for backward compatibility, OUTPUT parameters and return codes for results, and table-valued parameters for sets; get the types, NULL handling and compatibility right and the procedure stays safe, fast and evolvable.

### 1. The big picture
A stored procedure is an API living inside the database. Its parameter list is the public signature: callers (applications, other procedures, jobs) depend on names, types, order, defaults and output behaviour. Good parameter design achieves four things:

1. **Safety** — values are passed as typed data and bound into the statement, which is why parameterised procedures eliminate classic SQL injection (the input is treated as data, not executable SQL).
2. **Clarity** — named, correctly typed parameters make calls readable and invalid input detectable.
3. **Evolvability** — you can add optional parameters without breaking existing callers.
4. **Performance** — parameter types and values influence plan reuse, implicit conversions and *parameter sniffing*.

### 2. Core concepts and vocabulary
- **Parameter** — a named, typed placeholder declared in the procedure header (`@CustomerID INT`).
- **Argument** — the actual value supplied by the caller.
- **IN / input parameter** — value passed *into* the procedure (default in T-SQL and MySQL). **OUT / OUTPUT** — value passed *back* to the caller. **INOUT** — both (MySQL, Oracle, PL/pgSQL).
- **Return code** — integer returned by `RETURN n` in T-SQL; status only, not for data.
- **Default value** — makes a parameter optional (`@Status VARCHAR(10) = 'Open'`).
- **Positional vs named arguments** — by order, or `@Name = value`; once you name one, you must name the rest (T-SQL).
- **Table-valued parameter (TVP)** — a user-defined table type passed as a read-only parameter (SQL Server); PostgreSQL arrays/`unnest`, Oracle collection types, JSON/XML parameters play the same role.
- **Parameter sniffing** — the optimizer compiles the plan using the parameter values of the first call and reuses it.
- **NULL semantics / three-valued logic** — comparisons with NULL evaluate to *unknown*, so `col = @p` is never true when `@p IS NULL`.
- **Implicit conversion** — the engine converts a value from one type to another to make a comparison work; can disable index seeks.
- **Optional-parameter ("catch-all") search procedure** — one procedure with many nullable filters.

### 3. How it works, step by step
**Calling and binding**
1. The caller sends the procedure name plus arguments (positional or named) — as RPC (`CALL`, `EXEC`, driver "callable statement") rather than concatenated text.
2. The engine matches arguments to parameters; missing ones take their defaults; if none exists → error ("expects parameter … which was not supplied"); too many → error.
3. Each argument is converted to the declared type; incompatible values fail (conversion error) and compatible ones are implicitly converted.
4. The statements inside run with the parameter values as bound variables. On first execution the optimizer *sniffs* them for cardinality estimates and caches the plan.
5. On exit OUTPUT parameters and the return code are sent back; result sets stream separately.

**Parameter modes by engine**
| Feature | SQL Server | MySQL | PostgreSQL (procedures) | Oracle |
|---|---|---|---|---|
| Input | default | `IN` (default) | `IN` (default) | `IN` |
| Output | `OUTPUT` | `OUT` | `OUT` / `INOUT` | `OUT` / `IN OUT` |
| Default values | `= value` | **not supported** — use NULL + `IFNULL` | `DEFAULT value` | `DEFAULT value` |
| Return value | integer status via `RETURN` | functions only | functions; procedures via OUT | functions |
| Set parameter | table-valued (READONLY) | temp table / JSON | array / `jsonb` | collection / `SYS_REFCURSOR` |
| Result set to caller | SELECT in body | SELECT in body | `refcursor` / `RETURNS SETOF` | ref cursor |

**Adding an optional parameter without breaking callers**
1. Append the new parameter at the **end** of the list with a default (`@IncludeClosed BIT = 0`).
2. Make the default reproduce the old behaviour exactly.
3. Keep existing parameter names/order; callers using positional calls keep working.
4. Deploy with `CREATE OR ALTER`; in engines without defaults (MySQL) create a new procedure (`usp_x_v2`) and make the old one a thin wrapper.
5. Track usage (Query Store, extended events, `sys.dm_exec_procedure_stats`) then retire the old signature.

### 4. Worked example
```sql
-- SQL Server
CREATE OR ALTER PROCEDURE dbo.usp_GetOrders
    @CustomerID   INT,
    @Status       VARCHAR(20) = NULL,        -- optional filter
    @From         DATE        = NULL,
    @PageSize     INT         = 50,
    @TotalRows    INT         OUTPUT         -- extra result via OUTPUT
AS
BEGIN
    SET NOCOUNT ON;

    IF @CustomerID IS NULL OR @PageSize NOT BETWEEN 1 AND 1000
    BEGIN
        THROW 50001, 'Invalid @CustomerID or @PageSize.', 1;   -- validate inputs first
    END

    SELECT @TotalRows = COUNT(*)
    FROM dbo.Orders
    WHERE CustomerID = @CustomerID
      AND (@Status IS NULL OR Status = @Status)
      AND (@From   IS NULL OR OrderDate >= @From);

    SELECT TOP (@PageSize) OrderID, OrderDate, Status, TotalAmount
    FROM dbo.Orders
    WHERE CustomerID = @CustomerID
      AND (@Status IS NULL OR Status = @Status)
      AND (@From   IS NULL OR OrderDate >= @From)
    ORDER BY OrderDate DESC, OrderID DESC
    OPTION (RECOMPILE);           -- catch-all pattern: plan per call (see below)

    RETURN 0;                      -- status code
END;

-- Calls
DECLARE @n INT;
EXEC dbo.usp_GetOrders @CustomerID = 102, @TotalRows = @n OUTPUT;             -- defaults used
EXEC dbo.usp_GetOrders @CustomerID = 102, @Status = 'Shipped', @TotalRows = @n OUTPUT;
```
Walkthrough:
- `@CustomerID` has no default, so omitting it is an error. Passing `NULL` explicitly is caught by validation.
- `(@Status IS NULL OR Status = @Status)` implements "optional filter". Because `NULL` is *unknown*, a plain `Status = @Status` with `@Status = NULL` would return nothing — the classic three-valued-logic bug.
- The `OPTION (RECOMPILE)` makes the optimizer treat the parameters as constants each call and simplify away the unused branches; without it a single cached plan must serve every combination and is often bad for most.
- The caller reads the output with `@TotalRows = @n OUTPUT`; forgetting the `OUTPUT` keyword at call time silently leaves the variable NULL — a classic bug.

Passing a list (set) in:
```sql
CREATE TYPE dbo.IdList AS TABLE (Id INT PRIMARY KEY);
GO
CREATE OR ALTER PROC dbo.usp_GetByIds @Ids dbo.IdList READONLY AS
    SELECT o.* FROM dbo.Orders o JOIN @Ids i ON i.Id = o.OrderID;
```
Equivalents: PostgreSQL `WHERE id = ANY($1::int[])`, SQL Server 2016+ `STRING_SPLIT(@csv, ',')` (easy, no statistics), MySQL `JSON_TABLE`/`FIND_IN_SET` (poor index use) or a temp table.

### 5. Under the hood
- **Injection safety is a binding property, not a procedure property.** Parameters are sent as typed values separate from the statement text. A procedure that builds `EXEC('SELECT … WHERE name = ''' + @name + '''')` re-introduces injection. Safe dynamic SQL uses `sp_executesql` with parameters and `QUOTENAME()` for identifiers.
- **Parameter sniffing.** On compile, the optimizer reads the actual parameter values and uses statistics histograms to estimate rows. Good: plan tailored to typical values. Bad: first call with a rare value (or a very common one) gives a plan wrong for the others → big variance in duration/reads for the same procedure. Mitigations: `OPTION (RECOMPILE)` (per statement), `OPTIMIZE FOR (@p = value | UNKNOWN)`, copying to local variables (forces density-average estimates), splitting into sub-procedures per case, plan guides/Query Store forcing, or in SQL Server 2022 Parameter Sensitive Plan optimization.
- **Local variables vs parameters.** `DECLARE @x = @p; … WHERE col = @x` hides the value from the optimizer (it uses average density) — sometimes a deliberate fix, sometimes an accident that costs accuracy.
- **Data type matching.** If `@Code NVARCHAR(20)` is compared to a `VARCHAR` indexed column, the column gets converted (data-type precedence) and the index can't seek (`CONVERT_IMPLICIT` in the plan warnings). Match types, length and collation. Also oversize parameters (`NVARCHAR(4000)` for short strings) inflate memory grants.
- **Wrong argument count/type.** T-SQL: error 8144 (too many) or 201 (missing); type mismatch → conversion error 245/8114 at runtime, not at creation time. MySQL: error 1318 "Incorrect number of arguments". Oracle can overload procedures by signature; T-SQL cannot (except function-numbering).
- **OUTPUT parameters** copy values back on completion; with an error before assignment the caller sees the *initial* value. Return codes `0` = success by convention; negative/positive values are user-defined. Neither replaces raising an error: callers that ignore them cause "silent failures" — raise with `THROW`/`SIGNAL`/`RAISE` for failures and use OUT values for results.
- **Plan cache and sets.** TVPs have no statistics (only cardinality of the table variable at compile time, often "1 row"), so large TVPs can mislead the optimizer; add `OPTION (RECOMPILE)` or insert into a `#temp` table with statistics for big sets. `STRING_SPLIT` returns an estimated fixed row count likewise.

### 6. Performance and design implications
- **Validate early**: check NULLs, ranges and enumerations at the top; fail fast with meaningful errors.
- **Use the narrowest correct type** and match column types to avoid implicit conversion.
- **Prefer explicit named parameters** in calls and keep a stable order; document defaults.
- **Catch-all procedures**: either `OPTION (RECOMPILE)`, separate statements per case (`IF @Status IS NULL … ELSE …`), or build parameterised dynamic SQL with only the active predicates (best for many optional filters, using `sp_executesql` to keep reuse).
- **Don't rely on OUTPUT for big data** — use result sets or TVPs/temp tables.
- **Sets**: TVP/array/JSON over comma-separated strings for anything beyond trivial; for very large inputs, bulk load into a staging table.
- **Observe**: `sys.dm_exec_procedure_stats`, Query Store `plan_id` variance per procedure, "Parameter List" in the plan (compiled vs runtime values).
- **Compatibility**: version procedure signatures; don't change types or order of existing parameters in place.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `CREATE [OR ALTER] PROC p @a INT, @b INT = 0 OUTPUT` | Declare parameters/defaults/outputs | Procedure definition |
| `EXEC p @a = 1, @b = @x OUTPUT` | Named call incl. output capture | Caller side |
| `RETURN n` | Integer status | Success/failure codes (not data) |
| `READONLY` table-type parameter | Pass a set | Bulk input, replacing CSV strings |
| `OPTION (RECOMPILE)`, `OPTIMIZE FOR` | Control sniffing | Variable-selectivity parameters |
| `sp_executesql @sql, N'@p INT', @p = …` | Parameterised dynamic SQL | Optional filters, dynamic columns |
| `sys.parameters`, `INFORMATION_SCHEMA.PARAMETERS` | Inspect signatures | Documentation, compatibility checks |
| `THROW` / `RAISERROR`, `SIGNAL SQLSTATE` (MySQL), `RAISE EXCEPTION` (PG) | Raise validation errors | Input validation |
| `DEFAULT` keyword at call | Use default explicitly | Skipping middle args positionally |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| **Default-valued optional parameter** — backward compatible extension | The default silently changes behaviour; or engine lacks defaults (MySQL) |
| **OUTPUT parameter** — one or two scalar results | Returning rows or many values (use result set/TVP/JSON) |
| **Return code** — coarse success/failure signalling | Used as the only error mechanism; callers ignore it |
| **TVP / array / JSON** — pass many values | Tiny lists (a few scalars) or you need statistics on a big set |
| **Catch-all + RECOMPILE** — few executions, many filter combos | Very high call rate (compile cost) → dynamic SQL per shape |
| **Local-variable copy** — stable but average plans | You need value-specific plans |
| **New procedure version** — signature must change incompatibly | Many versions accumulate; deprecate with usage tracking |

### 9. Common mistakes
- **Concatenating parameters into dynamic SQL** → injection and plan bloat → `sp_executesql` with parameters, `QUOTENAME`.
- **`WHERE col = @p` with NULLable optional filter** → returns nothing when `@p` is NULL → `(@p IS NULL OR col = @p)` or `col = ISNULL(@p, col)` (careful with NULL columns) or dynamic SQL.
- **Forgetting `OUTPUT` on the call** → caller variable stays NULL → add `OUTPUT` keyword.
- **Mismatched types (NVARCHAR vs VARCHAR/INT as string)** → implicit conversion, scans → align types.
- **Adding a parameter in the middle** → positional callers shift values silently → append at end, default value.
- **Ignoring return/OUT status in the app** → silent failures → raise errors for failures; enforce checks in the data-access layer and tests.
- **Parameter sniffing blindness** → "fast for some customers, slow for others" → check compiled vs runtime values, use RECOMPILE/OPTIMIZE FOR/split paths.
- **Passing CSV strings and splitting with a slow function** → row-by-row parsing → TVP or `STRING_SPLIT`/`unnest`.
- **Parameter defaults assumed in MySQL** → syntax error → use NULL and `COALESCE` inside.
- **Overloading TVP use for single values** → boilerplate and plan estimate problems → simple scalars.

### 10. 🎯 Interview kit
**30-second answer:** "Parameters are the procedure's contract: typed inputs bound as data — which is what makes them injection-safe — optional defaults, OUTPUT/return codes for results and status, and table-valued parameters for sets. I match types to columns to avoid implicit conversion, handle NULLs explicitly because of three-valued logic, add new parameters at the end with defaults to stay backward compatible, and watch parameter sniffing."

**Structure a longer answer:** define → modes (IN/OUT/INOUT/return) → safety (binding vs dynamic SQL) → compatibility (defaults, versioning) → performance (types, sniffing, sets) → error handling contract.

**Likely follow-ups:**
- *Add an optional parameter without breaking callers?* → append with default preserving old behaviour; wrapper/versioned procedure when defaults are unsupported; monitor callers.
- *What if you pass NULL to an IN parameter?* → value is NULL; comparisons become unknown; handle with `IS NULL` branches or validation.
- *Wrong number of parameters?* → error (missing without default; too many) — engine-specific message; types mismatches fail at run time.
- *Parameter sniffing incident?* → procedure plan built for a rare value, then reused for common; diagnose via plan parameter list/Query Store; fix with OPTIMIZE FOR/RECOMPILE/split.
- *App ignores OUT status?* → raise errors instead of codes, wrap in data-access layer, add tests/contract checks, log.
- *Passing lists efficiently?* → TVP (SQL Server), array/unnest or JSON (PostgreSQL), JSON_TABLE/temp table (MySQL).
- *How do you debug a wrong result?* → call with known inputs, `PRINT`/log intermediate values, run statements separately, compare with expected data, check NULL handling and implicit conversions.

**Signals interviewers look for:** you mention binding/injection, NULL logic, sniffing, defaults and evolution, and understand engine differences instead of generic definitions.

### 11. Check yourself
- [ ] Write a procedure with a required parameter, an optional parameter with default, and an OUTPUT parameter, and call it correctly.
- [ ] Explain why `WHERE Status = @Status` fails for NULL and show two corrections.
- [ ] Show how to add a new optional parameter without breaking a caller.
- [ ] Describe what parameter sniffing is and three ways to mitigate it.
- [ ] Choose a way to pass a list of IDs in SQL Server, PostgreSQL and MySQL and state one trade-off each.
<!-- explainer:end -->

## Implementations (tools)

- [SQL Server](<../../../TOOLS/SQL_Server/_index.md>)

## Practice questions

- [Basic] [What are the different types of parameters a stored procedure can accept?](<../../../PRACTICE/01_Databases_and_SQL/Database_Objects/Stored_Procedures/Q-41897_What_are_the_different_types_of_parameters_a_stored_procedur.md>)
- [Basic] [What happens if you call a stored procedure with the wrong number of parameters?](<../../../PRACTICE/01_Databases_and_SQL/Database_Objects/Stored_Procedures/Q-41911_What_happens_if_you_call_a_stored_procedure_with_the_wrong_n.md>)
- [Intermediate] [How would you validate input parameters inside a stored procedure before running the main logic?](<../../../PRACTICE/01_Databases_and_SQL/Database_Objects/Stored_Procedures/Q-41910_How_would_you_validate_input_parameters_inside_a_stored_proc.md>)
- [Intermediate] [Can you set default values for stored procedure parameters in MySQL? What are the alternatives?](<../../../PRACTICE/01_Databases_and_SQL/Database_Objects/Stored_Procedures/Q-41913_Can_you_set_default_values_for_stored_procedure_parameters_i.md>)
- [Intermediate] [What does an INOUT parameter do and when would you use it?](<../../../PRACTICE/01_Databases_and_SQL/Database_Objects/Stored_Procedures/Q-41919_What_does_an_INOUT_parameter_do_and_when_would_you_use_it.md>)
- [Intermediate] [How would you test a stored procedure to make sure it works correctly for different inputs?](<../../../PRACTICE/01_Databases_and_SQL/Database_Objects/Stored_Procedures/Q-41935_How_would_you_test_a_stored_procedure_to_make_sure_it_works_.md>)
- [Intermediate] [How do you handle NULL-related logic bugs inside procedures where a comparison silently returns no rows because of three-valued logic?](<../../../PRACTICE/01_Databases_and_SQL/Database_Objects/Stored_Procedures/Q-41803_How_do_you_handle_NULL-related_logic_bugs_inside_procedures_.md>)
- [Deep] [How do you handle the 'not found' condition when looping through a cursor?](<../../../PRACTICE/01_Databases_and_SQL/Database_Objects/Stored_Procedures/Q-41874_How_do_you_handle_the_not_found_condition_when_looping_throu.md>)

## Related concepts

- [Dynamic SQL](<../../Database_Internals/DBMS_Concepts/Dynamic_SQL.md>) *(DBMS Concepts)*
- [CTEs in SQL Server](<../../../TOOLS/SQL_Server/SQL_Server_Platform/CTEs_in_SQL_Server.md>) *(SQL Server Platform)*
- [Stored Procedure Performance and Design](<Stored_Procedure_Performance_and_Design.md>) *(Stored Procedures)*

## References

- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 7: Dynamic SQL, Triggers, and Table-Valued Functions (TVFs) › Parameterization in Stored Procedures
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 7: Dynamic SQL, Triggers, and Table-Valued Functions (TVFs) › Using Parameterized Stored Procedures
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 8: Query Optimization and Performance Tuning › Parameter Sniffing Awareness
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 9: Execution Plans and Query Store › Handling Parameter Sniffing
- *Beginning T-SQL* — Kathi Kellenberger, Lee Everest — Stored Procedures › Using Default Values with Parameters
- *Beginning T-SQL* — Kathi Kellenberger, Lee Everest — Stored Procedures › Using the OUTPUT Parameter
- *SQL Server Tutorial* (YouTube series) — Part 68: Optional parameters in sql server stored procedures ([watch](https://www.youtube.com/watch?v=CMyE4ETUwCk))
- *SQL Server Tutorial* (YouTube series) — Part 99: Table valued parameters in SQL Server ([watch](https://www.youtube.com/watch?v=Ewf5bivTKdI))

