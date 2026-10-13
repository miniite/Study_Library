---
id: "TOOLS/Oracle/PL-SQL/Exception_Handling"
level: 4
type: "concept"
topic: "TOOLS/Oracle/PL-SQL"
tools: ["Oracle"]
---

# Exception Handling

**L4 Advanced** · [↑ PL-SQL](<_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** In PL/SQL an error becomes an *exception* that unwinds the block stack until a matching `WHEN` handler is found; good handling catches only what you can deal with, preserves the original error, and reports business errors to the caller with `RAISE_APPLICATION_ERROR`.

### 1. The big picture
Things go wrong at runtime: a `SELECT INTO` finds no row, a unique key is violated, a value is too large. In PL/SQL each such condition raises an **exception**. Execution of the current block stops and control jumps to its `EXCEPTION` section; if no handler fits, the exception is propagated to the enclosing block, then to the caller, and finally to the client as an `ORA-nnnnn` error. This lesson explains the types of exception, how to declare and raise your own, how to read the error information, how propagation and transactions interact, and the common patterns for production code (logging, error translation for application callers, batch error handling with `SAVE EXCEPTIONS`).

### 2. Core concepts and vocabulary
- **Predefined exception** — Oracle-named errors: `NO_DATA_FOUND` (ORA-01403), `TOO_MANY_ROWS` (ORA-01422), `ZERO_DIVIDE` (ORA-01476), `DUP_VAL_ON_INDEX` (ORA-00001), `INVALID_NUMBER` (ORA-01722), `VALUE_ERROR` (ORA-06502), `CURSOR_ALREADY_OPEN`, `LOGIN_DENIED`…
- **Non-predefined exception** — a standard Oracle error with a number but no name; you give it a name with `PRAGMA EXCEPTION_INIT(name, -error_number)`.
- **User-defined exception** — declared as `e_x EXCEPTION;`, raised with `RAISE e_x;`.
- **`WHEN OTHERS`** — catch-all handler; must be last.
- **`SQLCODE` / `SQLERRM`** — error number (negative; `+1` for user-defined; 100 for `NO_DATA_FOUND`) and message, available inside a handler. `SQLERRM` is truncated at 512 bytes.
- **`RAISE`** — raise a named exception, or (with no argument, inside a handler) re-raise the current one.
- **`RAISE_APPLICATION_ERROR(num, msg [, keep_errors])`** — raise a user error numbered -20000…-20999 with a message of up to 2048 bytes, that clients see as `ORA-20xxx: msg`.
- **`DBMS_UTILITY.FORMAT_ERROR_BACKTRACE`** — line-level call stack of where the error was raised; **`FORMAT_ERROR_STACK`** — full message stack.
- **Compile-time vs. runtime error** — syntax/semantic errors detected when creating the object (`PLS-00201` etc., see `USER_ERRORS`) vs. errors during execution (`ORA-`).

### 3. How it works, step by step
**Raising**
1. The runtime raises a predefined exception automatically, or you execute `RAISE name` or `RAISE_APPLICATION_ERROR`.
2. Normal execution stops; the remainder of the executable section is skipped.

**Handling**
3. Oracle looks for the first matching `WHEN` clause in the *current* block's exception section (`WHEN a OR b THEN`).
4. If found, its statements execute and the block completes **normally** (control returns to the caller of this block as if it had finished) unless the handler re-raises.
5. If not found, the exception propagates to the enclosing block *at the point of the inner block call*; the process repeats up to the outermost block, then to the client.
6. Exceptions raised in the **declaration section** or **inside an exception handler** cannot be handled by that same block's handlers; they propagate outward.

**Transaction effects**
7. A statement that fails is rolled back to its start (statement-level atomicity). Earlier statements in the transaction stay in place unless the caller rolls back. If an exception leaves the *outermost* PL/SQL call unhandled, the whole call counts as one failed statement and Oracle undoes the DML it did. If the exception is merely propagated out of an inner subprogram into a caller that handles it, Oracle does **not** undo that subprogram's earlier DML: the handler (or the caller) must decide to `ROLLBACK` or `ROLLBACK TO savepoint`.

**Sample structure**
```sql
BEGIN
  -- main work
EXCEPTION
  WHEN NO_DATA_FOUND THEN …      -- specific first
  WHEN DUP_VAL_ON_INDEX THEN …
  WHEN OTHERS THEN               -- last, log, then re-raise
     log_error(SQLCODE, SQLERRM, DBMS_UTILITY.FORMAT_ERROR_BACKTRACE);
     RAISE;
END;
```

### 4. Worked example
```sql
CREATE OR REPLACE PACKAGE BODY payments_api AS
  e_insufficient_funds EXCEPTION;
  PRAGMA EXCEPTION_INIT(e_insufficient_funds, -20001);   -- map to a custom number

  PROCEDURE transfer(p_from NUMBER, p_to NUMBER, p_amt NUMBER) IS
    v_bal accounts.balance%TYPE;
  BEGIN
    SELECT balance INTO v_bal FROM accounts WHERE id = p_from FOR UPDATE;   -- may raise NO_DATA_FOUND

    IF v_bal < p_amt THEN
       RAISE_APPLICATION_ERROR(-20001, 'Insufficient funds in account ' || p_from);
    END IF;

    UPDATE accounts SET balance = balance - p_amt WHERE id = p_from;
    UPDATE accounts SET balance = balance + p_amt WHERE id = p_to;
    IF SQL%ROWCOUNT = 0 THEN
       RAISE_APPLICATION_ERROR(-20002, 'Target account ' || p_to || ' does not exist');
    END IF;
  EXCEPTION
    WHEN NO_DATA_FOUND THEN
       RAISE_APPLICATION_ERROR(-20003, 'Source account ' || p_from || ' does not exist');
    WHEN e_insufficient_funds THEN
       RAISE;                                   -- let caller see ORA-20001 unchanged
    WHEN OTHERS THEN
       error_log.write(SQLCODE, SQLERRM, DBMS_UTILITY.FORMAT_ERROR_BACKTRACE);
       RAISE;                                   -- never swallow unknown errors
  END transfer;
END payments_api;
/
```
Application side (Java/.NET): catch the SQL exception, read the vendor error code (e.g. 20001), and map the -20000 series to business messages; treat everything else as a system failure. The application team should catch the *specific* custom codes you documented; `ORA-06512` lines in the message are just the PL/SQL stack.

**Handling a statement with a possibly-missing row** without an exception:
```sql
SELECT MAX(salary) INTO v_sal FROM employees WHERE employee_id = :id;  -- aggregate always returns one row
-- or wrap in a block:
BEGIN
  SELECT salary INTO v_sal FROM employees WHERE employee_id = :id;
EXCEPTION WHEN NO_DATA_FOUND THEN v_sal := NULL;
END;
```
**Batch with FORALL**
```sql
DECLARE
  TYPE t_ids IS TABLE OF NUMBER;
  v_ids t_ids := t_ids(1,2,3,4);
  e_bulk EXCEPTION;
  PRAGMA EXCEPTION_INIT(e_bulk, -24381);
BEGIN
  FORALL i IN 1..v_ids.COUNT SAVE EXCEPTIONS
    INSERT INTO target(id) VALUES (v_ids(i));
EXCEPTION
  WHEN e_bulk THEN
    FOR j IN 1..SQL%BULK_EXCEPTIONS.COUNT LOOP
      DBMS_OUTPUT.PUT_LINE('Row ' || SQL%BULK_EXCEPTIONS(j).ERROR_INDEX ||
                           ': ' || SQLERRM(-SQL%BULK_EXCEPTIONS(j).ERROR_CODE));
    END LOOP;
END;
/
```
Without `SAVE EXCEPTIONS`, the first failing row aborts the whole `FORALL` (earlier rows stay until rollback). With it, failing rows are collected in `SQL%BULK_EXCEPTIONS` and the loop continues; `ORA-24381` is raised at the end.

Seeing compilation errors:
```sql
CREATE OR REPLACE PROCEDURE p IS BEGIN x := 1; END;   -- PLS-00201 / PLS-00363
/
SHOW ERRORS
SELECT line, position, text FROM user_errors WHERE name = 'P' ORDER BY sequence;
```

### 5. Under the hood
- **Propagation is stack unwinding.** Each PL/SQL frame has an exception table; when an exception is raised Oracle scans the current frame, pops it if no handler matches and repeats. The back-trace is captured at the original raise point; `FORMAT_ERROR_BACKTRACE` retains it even after a `RAISE;` re-raise, whereas `RAISE_APPLICATION_ERROR` or `RAISE e` of a *new* exception resets the origin.
- **`SQLERRM` vs. `FORMAT_ERROR_STACK`.** In 12c+ the latter returns the complete chain (including `ORA-06512` entries), recommended for logging.
- **Exceptions in handlers.** An exception raised in a handler is not caught by sibling handlers, which is why logging code inside `WHEN OTHERS` should itself be safe (autonomous transaction, own handler).
- **Declaration errors.** `v NUMBER(2) := 100;` in `DECLARE` raises `VALUE_ERROR` that the same block cannot handle.
- **`WHEN OTHERS` and transaction state.** The statement is already rolled back when the handler runs; earlier statements are intact. If the handler swallows the error the transaction proceeds with partial work, which is how data inconsistencies get committed.
- **Autonomous transaction for logging.** `error_log.write` is declared with `PRAGMA AUTONOMOUS_TRANSACTION` and commits independently, so the log row survives a rollback of the main transaction.
- **Unhandled errors across the boundary.** The client receives the error and SQLCODE; `ORA-06512: at "OWNER.PKG", line 42` lines come from the call stack.

### 6. Performance and design implications
- Exception handling itself is cheap when not raised; raising millions of exceptions in a loop (using `NO_DATA_FOUND` as normal flow control) is measurable. Prefer a probe query (`MAX`, `EXISTS`, `COUNT` with `ROWNUM = 1`) for expected "not found".
- `SAVE EXCEPTIONS` lets bulk jobs finish in one pass and report bad rows rather than aborting or falling back to row-by-row.
- Use `DBMS_ERRLOG` (`LOG ERRORS INTO err$_t (…) REJECT LIMIT UNLIMITED`) for set-based DML where bad rows are diverted to an error table.
- Standardize an error package: constants for -20xxx codes, a `raise_error(code, msg)` helper, a logger. This keeps contracts with the application stable.
- Unit test with **utPLSQL**: assert on both state changes and expected exceptions (`--%throws(-20001)`), plus the case where nothing is raised.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `EXCEPTION WHEN … THEN` | Handler section | Anywhere you can act |
| `RAISE;` | Re-raise current exception | After logging/cleanup |
| `RAISE_APPLICATION_ERROR(-20xxx, msg)` | Application error to caller | Business rule violations |
| `PRAGMA EXCEPTION_INIT(name, -nnnnn)` | Name an ORA error | Handle e.g. ORA-00054 resource busy |
| `SQLCODE`, `SQLERRM` | Current error number/message | Logging |
| `DBMS_UTILITY.FORMAT_ERROR_BACKTRACE` | Line numbers | Diagnostics |
| `SAVE EXCEPTIONS` / `SQL%BULK_EXCEPTIONS` | Continue bulk DML past bad rows | Batch loads |
| `LOG ERRORS INTO` | SQL-level error logging | Set-based loads |
| `SHOW ERRORS`, `USER_ERRORS`, `PLSQL_WARNINGS` | Compile-time diagnostics | Deployment |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| Handle locally | You cannot restore a valid state or meaningfully continue |
| Let it propagate | A caller needs to decide (retry, rollback, message) |
| `RAISE_APPLICATION_ERROR` | The error is a system error you want to keep original (just `RAISE`) |
| `WHEN OTHERS` with log + `RAISE` | As a silent catch-all (`NULL;`) |
| `SAVE EXCEPTIONS` | Bad rows must abort the whole load (all-or-nothing) |
| `DBMS_ERRLOG` | Need per-row custom handling logic |

### 9. Common mistakes
- `WHEN OTHERS THEN NULL;` → hides failures, data inconsistency → log and re-raise.
- `WHEN OTHERS` placed before specific handlers → compile error `PLS-00370`; handlers after it unreachable.
- `SELECT INTO` without handling `NO_DATA_FOUND`/`TOO_MANY_ROWS` → unexpected runtime failures → handle or use cursor/aggregate.
- Using `SQLERRM` outside a handler or assuming it is complete → empty/truncated → use `FORMAT_ERROR_STACK`.
- `COMMIT` in a handler → persisting partial work → let the caller decide.
- Raising `RAISE_APPLICATION_ERROR` with numbers outside -20000…-20999 → `ORA-21000` → stay in range.
- Catching an exception and re-raising a *different* one without keeping the original → lost root cause → pass `keep_errors => TRUE` or log first.

### 10. 🎯 Interview kit
**30-second answer:** "An exception is a runtime error condition. The `EXCEPTION` section handles it by name — predefined ones like `NO_DATA_FOUND`, user-defined ones I declare and `RAISE`, and `OTHERS` as the last resort. If a block can't handle it, it propagates outward. I log with backtrace and re-raise, and use `RAISE_APPLICATION_ERROR` with -20000 to -20999 to give callers a meaningful business error."

**Structure a longer answer:** types → propagation → transaction effect → logging pattern → bulk handling → testing.

**Likely follow-ups:**
- *`RAISE` vs. `RAISE_APPLICATION_ERROR`?* → named/re-raise vs. custom code and message; the app should catch the custom numbers.
- *`SELECT INTO` returns more than one row?* → `TOO_MANY_ROWS`.
- *`PRAGMA EXCEPTION_INIT`?* → associates a name with an error code.
- *Can a handler catch errors raised in its own handler?* → no; propagates to the outer block.
- *How to continue after a row-level error in `FORALL`?* → `SAVE EXCEPTIONS`.
- *Compile vs. runtime error?* → `USER_ERRORS` vs. `ORA-` at execution.

**Signals interviewers look for:** you never swallow errors, you know where the transaction stands after an exception, and you design the error contract with the calling application.

### 11. Check yourself
- [ ] Write a block that catches `NO_DATA_FOUND`, `TOO_MANY_ROWS`, and re-raises everything else.
- [ ] Declare a user-defined exception and raise it from a nested block; predict where it is caught.
- [ ] Explain why an error in `DECLARE` is not caught by the same block.
- [ ] Use `SAVE EXCEPTIONS` and print failing indexes and messages.
- [ ] Explain the difference between `FORMAT_ERROR_STACK` and `FORMAT_ERROR_BACKTRACE`.
- [ ] Map `ORA-00054` to a named exception and handle it.
<!-- explainer:end -->

## Implementations (tools)

- [Oracle](<../_index.md>)

## Practice questions

- [Basic] [What is exception handling in PL/SQL and what does the EXCEPTION section do?](<../../../PRACTICE/TOOLS/Oracle/PL-SQL/Q-35712_What_is_exception_handling_in_PL_SQL_and_what_does_the_EXCEP.md>)
- [Intermediate] [What exception is raised when a SELECT INTO returns no rows and how do you handle it?](<../../../PRACTICE/TOOLS/Oracle/PL-SQL/Q-35710_What_exception_is_raised_when_a_SELECT_INTO_returns_no_rows_.md>)
- [Intermediate] [What is the difference between a predefined exception and a user-defined exception?](<../../../PRACTICE/TOOLS/Oracle/PL-SQL/Q-35713_What_is_the_difference_between_a_predefined_exception_and_a_.md>)
- [Intermediate] [How do you declare and raise a user-defined exception in PL/SQL?](<../../../PRACTICE/TOOLS/Oracle/PL-SQL/Q-35715_How_do_you_declare_and_raise_a_user-defined_exception_in_PL_.md>)
- [Intermediate] [What do the SQLCODE and SQLERRM functions return inside an exception handler?](<../../../PRACTICE/TOOLS/Oracle/PL-SQL/Q-35717_What_do_the_SQLCODE_and_SQLERRM_functions_return_inside_an_e.md>)
- [Intermediate] [How do you view compilation errors of a procedure using SHOW ERRORS or USER_ERRORS?](<../../../PRACTICE/TOOLS/Oracle/PL-SQL/Q-35771_How_do_you_view_compilation_errors_of_a_procedure_using_SHOW.md>)
- [Intermediate] [What is the difference between an anonymous block error at compile time and one at runtime?](<../../../PRACTICE/TOOLS/Oracle/PL-SQL/Q-35792_What_is_the_difference_between_an_anonymous_block_error_at_c.md>)
- [Intermediate] [How do you write PL/SQL unit tests with utPLSQL, and what would you assert for a procedure that mutates state and raises exceptions?](<../../../PRACTICE/TOOLS/Oracle/PL-SQL/Q-35672_How_do_you_write_PL_SQL_unit_tests_with_utPLSQL_and_what_wou.md>)

## Related concepts

- [Dynamic SQL](<../../../01_Databases_and_SQL/Database_Internals/DBMS_Concepts/Dynamic_SQL.md>) *(DBMS Concepts)*
- [PL/SQL Overview](<../Oracle_Platform/PLSQL_Overview.md>) *(Oracle Platform)*
- [CTEs in SQL Server](<../../SQL_Server/SQL_Server_Platform/CTEs_in_SQL_Server.md>) *(SQL Server Platform)*

## References

- *100 SQL Server Mistakes and How to Avoid Them* — Peter A. Carter — Error handling, testing, source control, and deployment › 7.1 #26 Writing code that doesn't handle errors
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 5: Views, Functions, and Stored Procedures › Error Handling with TRY...CATCH
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 5: Views, Functions, and Stored Procedures › Error Handling in Stored Procedures
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 5: Views, Functions, and Stored Procedures › TRY…CATCH Structure
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 5: Views, Functions, and Stored Procedures › Nested TRY…CATCH Blocks
- *Beginning T-SQL* — Kathi Kellenberger, Lee Everest — Managing Transactions › Error Handling
- *Beginning T-SQL* — Kathi Kellenberger, Lee Everest — Managing Transactions › Viewing Untrappable Errors
- *Beginning T-SQL* — Kathi Kellenberger, Lee Everest — Managing Transactions › Using RAISERROR
- *Beginning T-SQL* — Kathi Kellenberger, Lee Everest — Managing Transactions › Using THROW Instead of RAISERROR
- *T-SQL Fundamentals (4th ed.)* — Itzik Ben-Gan — Programmable objects
- *Mapping Data Flows in Azure Data Factory* — Mark Kromer — Monitor, Manage, and Optimize › Error Row Handling
- *SQL Server Integration Services Design Patterns* — Tim Mitchell et al. — Evolution of an SSIS Framework › Building Error Logging
- *SQL Server Tutorial* (YouTube series) — Part 55: Error handling in sql server 2000 ([watch](https://www.youtube.com/watch?v=xgpyqxKuta0))

