---
id: "TOOLS/Oracle/Oracle_Platform/PLSQL_Overview"
level: 4
type: "concept"
topic: "TOOLS/Oracle/Oracle_Platform"
tools: ["Oracle"]
---

# PL/SQL Overview

**L4 Advanced** · [↑ Oracle Platform](<_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** PL/SQL is Oracle's procedural language that lives inside the database: you group SQL and control flow into blocks, procedures, functions, packages and triggers, so logic runs next to the data with far fewer network round trips.

### 1. The big picture
SQL is declarative: you say *what* rows you want. Business processes also need variables, conditions, loops, error handling and reusable units. **PL/SQL** adds exactly that, sharing the type system of SQL and executing inside the database server. A client sends one call (`EXEC process_orders(:day)`) instead of hundreds of statements, and the logic is stored, compiled, versioned and secured in the database.

The language is built from one construct, the **block**, which can be anonymous (run once) or named (stored as a procedure, function, package or trigger). Everything else (cursors, collections, exceptions, packages) hangs off this. This lesson gives you the map: block structure, variable and type attributes, cursors, parameters, packages, triggers, errors, and the performance rule that dominates PL/SQL: *minimize switches between the PL/SQL and SQL engines*.

### 2. Core concepts and vocabulary
- **Block** — `DECLARE … BEGIN … EXCEPTION … END;` Declarative and exception sections are optional; `BEGIN … END` is mandatory and must contain at least one statement (`NULL;` counts).
- **Anonymous block** — unnamed, compiled and run each time, not stored. **Named block** — stored subprogram.
- **Procedure** — performs an action, may return values through `OUT` parameters. **Function** — returns exactly one value through `RETURN`, can be used in SQL expressions if it meets purity rules.
- **Package** — a container: **specification** (public interface: declarations) and **body** (implementation and private items). Package variables persist for the session.
- **Trigger** — stored block that fires on an event (DML, DDL, logon…).
- **Cursor** — handle to a query's result set. **Implicit cursor**: Oracle creates one for every DML and `SELECT INTO` (attributes via `SQL%ROWCOUNT`, `SQL%FOUND`). **Explicit cursor**: you declare, `OPEN`, `FETCH`, `CLOSE` it (attributes `%FOUND`, `%NOTFOUND`, `%ROWCOUNT`, `%ISOPEN`).
- **`%TYPE`** — declares a variable with the datatype of a column or another variable; **`%ROWTYPE`** — a record matching a row of a table or cursor.
- **Parameter modes** — `IN` (read-only, default), `OUT` (returned), `IN OUT`.
- **Exception** — a runtime error condition, handled in the `EXCEPTION` section; **`RAISE`** re-raises or raises a named exception, **`RAISE_APPLICATION_ERROR(-20000..-20999, msg)`** raises a custom ORA error to the caller.
- **Context switch** — transfer of control between the PL/SQL engine and the SQL engine.
- **Soft vs. hard parse** — reuse of a shared cursor from the shared pool vs. a full parse/optimize.

### 3. How it works, step by step
1. **Compile.** `CREATE OR REPLACE PROCEDURE …` is parsed and stored: source in `USER_SOURCE`, compile errors in `USER_ERRORS` (`SHOW ERRORS` in SQL*Plus), the compiled code (p-code or native machine code if `PLSQL_CODE_TYPE=NATIVE`) in the dictionary, dependencies in `USER_DEPENDENCIES`.
2. **Invoke.** A client or another block calls it; the server loads the code into the shared pool (library cache).
3. **Execute.** The PL/SQL engine runs procedural statements itself; each embedded SQL statement is handed to the SQL engine (a context switch), the results come back into PL/SQL variables.
4. **Handle errors.** If an exception occurs, control jumps to the `EXCEPTION` section of the current block; if no handler matches it propagates to the enclosing block and finally to the caller; uncaught exceptions roll back the failed statement (not the whole transaction unless the caller rolls back).
5. **Finish.** `COMMIT`/`ROLLBACK` are explicit; Oracle does not commit when a PL/SQL block ends.

**Block skeleton**
```sql
DECLARE
  v_name   employees.last_name%TYPE;
  v_emp    employees%ROWTYPE;
  c_limit  CONSTANT NUMBER := 100;
BEGIN
  SELECT * INTO v_emp FROM employees WHERE employee_id = 101;
  v_name := v_emp.last_name;
  IF v_emp.salary > c_limit THEN
     DBMS_OUTPUT.PUT_LINE(v_name || ' high');
  ELSIF v_emp.salary IS NULL THEN
     DBMS_OUTPUT.PUT_LINE('unknown');
  ELSE
     DBMS_OUTPUT.PUT_LINE(v_name || ' ok');
  END IF;
EXCEPTION
  WHEN NO_DATA_FOUND THEN DBMS_OUTPUT.PUT_LINE('no such employee');
  WHEN OTHERS THEN
     DBMS_OUTPUT.PUT_LINE(SQLCODE || ': ' || SQLERRM);
     RAISE;
END;
/
```
Variables are assigned with `:=`; equality tests use `=`. Parameters can be passed by position (`p(1, 'A')`), by name (`p(p_id => 1, p_kind => 'A')`) or mixed (positional first).

### 4. Worked example
```sql
CREATE OR REPLACE PACKAGE order_api AS
  -- specification: the public contract
  FUNCTION  order_total(p_order_id IN orders.order_id%TYPE) RETURN NUMBER;
  PROCEDURE close_order(p_order_id IN orders.order_id%TYPE,
                        p_status   OUT VARCHAR2);
END order_api;
/
CREATE OR REPLACE PACKAGE BODY order_api AS
  g_calls PLS_INTEGER := 0;                       -- private package state, per session

  FUNCTION order_total(p_order_id IN orders.order_id%TYPE) RETURN NUMBER IS
    v_total NUMBER;
  BEGIN
    g_calls := g_calls + 1;
    SELECT NVL(SUM(qty * price),0) INTO v_total
    FROM   order_items WHERE order_id = p_order_id;
    RETURN v_total;
  END;

  PROCEDURE close_order(p_order_id IN orders.order_id%TYPE,
                        p_status   OUT VARCHAR2) IS
    e_already_closed EXCEPTION;
    PRAGMA EXCEPTION_INIT(e_already_closed, -20010);
  BEGIN
    UPDATE orders SET status = 'CLOSED'
    WHERE  order_id = p_order_id AND status <> 'CLOSED';
    IF SQL%ROWCOUNT = 0 THEN
       RAISE_APPLICATION_ERROR(-20010, 'Order ' || p_order_id || ' is already closed');
    END IF;
    p_status := 'OK';
  END;
END order_api;
/
-- Calling
DECLARE v_s VARCHAR2(10); BEGIN order_api.close_order(1001, v_s); DBMS_OUTPUT.PUT_LINE(v_s); END;
/
SELECT order_id, order_api.order_total(order_id) FROM orders;   -- function in SQL
```
Notice: the spec can be recompiled independently; dependents are invalidated only when the *spec* changes (since 11g fine-grained dependency tracking), which is why packages beat standalone procedures for large systems. The last query calls the function once per row, each call running its own SQL: a *context switch + extra query per row* (see "Functions in SQL vs. PL/SQL" for the cure).

**Cursors**
```sql
-- implicit cursor FOR loop: open/fetch/close handled, bulk-fetches 100 rows at a time (since 10g)
FOR r IN (SELECT order_id FROM orders WHERE status = 'OPEN') LOOP
  order_api.close_order(r.order_id, v_s);
END LOOP;

-- explicit: four steps
CURSOR c IS SELECT … ;   -- DECLARE
OPEN c; LOOP FETCH c INTO rec; EXIT WHEN c%NOTFOUND; … END LOOP; CLOSE c;
```
Always use `EXIT WHEN c%NOTFOUND` immediately after `FETCH`, and always `CLOSE`: leaked cursors cause `ORA-01000: maximum open cursors exceeded`.

**Output in SQL*Plus:** `SET SERVEROUTPUT ON` is needed; `DBMS_OUTPUT` buffers messages until the block finishes, and is for debugging, not application logging.

### 5. Under the hood
- **Two engines.** The PL/SQL virtual machine (PVM) handles procedural statements; SQL statements are executed by the SQL executor. Each crossing costs a few microseconds; 1 million row-by-row iterations means 1 million crossings. **Bulk binding** (`BULK COLLECT`, `FORALL`) passes arrays across the boundary in one call.
- **Static SQL is bound automatically.** PL/SQL variables inside static SQL become bind variables, so soft-parse reuse is the default; dynamic SQL (`EXECUTE IMMEDIATE`) needs explicit `USING` binds or you reintroduce hard parses and SQL injection.
- **Definer's vs. invoker's rights.** By default a stored program runs with the privileges of its owner (`AUTHID DEFINER`), which lets users act on tables only through the API; `AUTHID CURRENT_USER` uses the caller's rights. Roles are *not* active inside definer's-rights code; privileges must be granted directly.
- **State.** Package-level variables are held in the session's UGA; with connection pools, state leaks between logical users unless reset. `PRAGMA SERIALLY_REUSABLE` changes the scope to a call.
- **Compilation.** PL/SQL optimizer levels (`PLSQL_OPTIMIZE_LEVEL` 0–3, default 2) can inline subprograms and reorder code; native compilation helps compute-heavy code, not SQL-bound code.
- **Dependencies and invalidation.** DDL on a referenced table invalidates dependent objects; they recompile on next use (and may fail with `ORA-04061` "existing state of … has been invalidated" for package state).

### 6. Performance and design implications
- Use **set-based SQL** first; PL/SQL loops that fetch rows and issue one DML per row are the usual cause of slowness ("slow-by-slow").
- When row-wise is unavoidable use `BULK COLLECT … LIMIT 100–1000` plus `FORALL` (`SAVE EXCEPTIONS` to continue past bad rows and inspect `SQL%BULK_EXCEPTIONS`).
- Keep transactions in the *caller*: stored procedures that commit inside make composition and testing hard.
- Reduce parsing: static SQL, binds, `OPEN-FOR USING`, `CURSOR_SHARING` only as a stopgap. Watch `parse count (hard)` in `V$SYSSTAT`.
- Instrument with `DBMS_APPLICATION_INFO.SET_MODULE/SET_ACTION` and `DBMS_PROFILER`/`DBMS_HPROF` to attribute time per line/subprogram.
- Prefer packages: they group related code, hide private helpers, and avoid cascading invalidation.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `CREATE [OR REPLACE] PROCEDURE/FUNCTION/PACKAGE` | Create stored code | Deploy logic |
| `SHOW ERRORS`, `USER_ERRORS` | Compile errors | After each deploy |
| `EXEC proc(args)` / `CALL proc(args)` | Invoke from SQL*Plus / SQL | Ad-hoc runs |
| `SET SERVEROUTPUT ON` | Display `DBMS_OUTPUT` | Debugging |
| `USER_SOURCE`, `USER_DEPENDENCIES`, `USER_OBJECTS.STATUS` | Source, deps, validity | Impact analysis |
| `ALTER PROCEDURE p COMPILE` / `UTL_RECOMP` | Recompile invalid code | After DDL |
| `PLSQL_WARNINGS`, `PLSQL_CCFLAGS` | Compile-time warnings / conditional compilation | Quality, debug code |
| `GRANT EXECUTE ON pkg TO role` | Privilege to run | Security |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| Plain SQL (set-based) | Logic needs branching per row that cannot be expressed in SQL |
| PL/SQL procedure/package | The logic is a single statement; app code is fine |
| Function used in SQL | It issues its own queries per row (prefer a join or `RESULT_CACHE`/scalar subquery caching) |
| Trigger | The rule can be a constraint or an explicit API call |
| Definer's rights | You need per-caller privileges (use invoker's rights) |
| Implicit cursor `FOR` loop | You need fine control (bulk limit, early exit) → explicit cursor with `BULK COLLECT LIMIT` |

### 9. Common mistakes
- Row-by-row processing with a commit per row → slow and not atomic → set-based or bulk with one commit.
- `WHEN OTHERS THEN NULL` → swallowed errors → log and re-raise.
- Forgetting `CLOSE` or `EXIT WHEN %NOTFOUND` → cursor leak or infinite loop → use cursor `FOR` loops where possible.
- Using `SELECT INTO` where 0 or >1 rows are possible → `NO_DATA_FOUND`/`TOO_MANY_ROWS` → handle or use cursor.
- Hard-coding column types → breaks after DDL → `%TYPE`/`%ROWTYPE`.
- String-building dynamic SQL without binds → hard parse and injection → `USING`.

### 10. 🎯 Interview kit
**30-second answer:** "PL/SQL is Oracle's procedural extension to SQL. Code is organized in blocks — declare, begin, exception, end — and stored as procedures, functions, packages and triggers that run inside the database, so I avoid network round trips. Packages split spec and body; cursors let me process result sets; exceptions handle errors; and the key performance idea is minimizing context switches with bulk operations."

**Structure a longer answer:** block anatomy → stored units and when to use each → cursors and `%TYPE` → error handling → performance (context switches, bulk, binds) → security (definer's rights).

**Likely follow-ups:**
- *Procedure vs. function?* → action with OUT params vs. one return value usable in SQL.
- *Package spec vs. body?* → public contract vs. implementation; recompiling the body doesn't invalidate dependents.
- *Implicit vs. explicit cursor?* → Oracle-managed for DML/`SELECT INTO` vs. user-controlled.
- *`%TYPE` vs. `%ROWTYPE`?* → column/variable type vs. whole row.
- *`RAISE` vs. `RAISE_APPLICATION_ERROR`?* → re-raise/raise named exception vs. custom -20000..-20999 error with message.
- *Soft vs. hard parse?* → cached vs. full compile.
- *Row-level vs. statement-level trigger?* → per row with `:OLD/:NEW` vs. once per statement.

**Signals interviewers look for:** you reach for set-based SQL first, you can explain context switches and bulk binding, and you understand how packages and rights models support maintainability.

### 11. Check yourself
- [ ] Write an anonymous block with all four sections.
- [ ] Explain why a block that ends without `COMMIT` leaves data uncommitted.
- [ ] Convert a row-by-row loop into `BULK COLLECT` + `FORALL`.
- [ ] Describe what is invalidated when a package body vs. spec changes.
- [ ] Explain when `%ROWCOUNT` of an implicit cursor is 0 and how to use it.
- [ ] List the four steps of using an explicit cursor.
<!-- explainer:end -->

## Implementations (tools)

- [Oracle](<../_index.md>)

## Practice questions

- [Basic] [What are DDL, DML, DCL, and TCL categories of SQL commands in Oracle?](<../../../PRACTICE/TOOLS/Oracle/Oracle_Platform/Q-35235_What_are_DDL_DML_DCL_and_TCL_categories_of_SQL_commands_in_O.md>)
- [Basic] [What is the basic block structure of a PL/SQL program with DECLARE, BEGIN, EXCEPTION, and END sections?](<../../../PRACTICE/TOOLS/Oracle/Oracle_Platform/Q-35259_What_is_the_basic_block_structure_of_a_PL_SQL_program_with_D.md>)
- [Basic] [What is a trigger in Oracle and when would you use one?](<../../../PRACTICE/TOOLS/Oracle/Oracle_Platform/Q-35267_What_is_a_trigger_in_Oracle_and_when_would_you_use_one.md>)
- [Intermediate] [What is the difference between an anonymous PL/SQL block and a named block in Oracle?](<../../../PRACTICE/TOOLS/Oracle/Oracle_Platform/Q-35260_What_is_the_difference_between_an_anonymous_PL_SQL_block_and.md>)
- [Intermediate] [What is a cursor in Oracle PL/SQL and what is the difference between an implicit and explicit cursor?](<../../../PRACTICE/TOOLS/Oracle/Oracle_Platform/Q-35264_What_is_a_cursor_in_Oracle_PL_SQL_and_what_is_the_difference.md>)
- [Intermediate] [What is a package in Oracle PL/SQL and what is the difference between the package specification and body?](<../../../PRACTICE/TOOLS/Oracle/Oracle_Platform/Q-35271_What_is_a_package_in_Oracle_PL_SQL_and_what_is_the_differenc.md>)
- [Intermediate] [What do the RAISE and RAISE_APPLICATION_ERROR statements do in Oracle PL/SQL?](<../../../PRACTICE/TOOLS/Oracle/Oracle_Platform/Q-35273_What_do_the_RAISE_and_RAISE_APPLICATION_ERROR_statements_do_.md>)
- [Intermediate] [What is the %TYPE attribute in Oracle PL/SQL and why is it useful?](<../../../PRACTICE/TOOLS/Oracle/Oracle_Platform/Q-35278_What_is_the_TYPE_attribute_in_Oracle_PL_SQL_and_why_is_it_us.md>)

## Related concepts

- [Dynamic SQL](<../../../01_Databases_and_SQL/Database_Internals/DBMS_Concepts/Dynamic_SQL.md>) *(DBMS Concepts)*
- [Functions in SQL vs PL/SQL](<../PL-SQL/Functions_in_SQL_vs_PLSQL.md>) *(PL-SQL)*
- [PL-SQL in Production](<../PL-SQL/PL-SQL_in_Production.md>) *(PL-SQL)*

