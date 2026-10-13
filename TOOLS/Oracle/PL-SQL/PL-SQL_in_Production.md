---
id: "TOOLS/Oracle/PL-SQL/PL-SQL_in_Production"
level: 4
type: "concept"
topic: "TOOLS/Oracle/PL-SQL"
tools: ["Oracle"]
---

# PL-SQL in Production

**L4 Advanced** · [↑ PL-SQL](<_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** Production PL/SQL is organized in packages with a narrow public API, processes data in bulk (`BULK COLLECT` + `FORALL`), keeps hot SQL free of per-row function calls, is instrumented for DBAs, scheduled with `DBMS_SCHEDULER`, and is protected by grants rather than trust.

### 1. The big picture
A procedure that works on 100 rows in a test schema can fail in production because of volume, concurrency, deployment, observability or security requirements. This lesson collects the practices that separate "works on my machine" PL/SQL from production code: package design, bulk processing and its effect on redo and latches, the true cost of per-row functions, reading plans of embedded SQL, restricting direct DML, conditional compilation, `NOCOPY`, instrumentation, scheduling, JSON handling, and wrapping.

### 2. Core concepts and vocabulary
- **Package specification / body** — the spec is the public contract (declarations); the body holds implementation and private items. A spec-only package (constants, types) needs no body.
- **Bulk binding** — `BULK COLLECT` fetches many rows into a collection with one engine switch; `FORALL` sends many DML executions with one switch.
- **`LIMIT`** — caps rows per `BULK COLLECT` to bound PGA memory.
- **Conditional compilation** — `$IF … $THEN … $END`, `$$flag` and `PLSQL_CCFLAGS` select code at compile time.
- **`NOCOPY`** — hint to pass `OUT`/`IN OUT` parameters by reference instead of copy-in/copy-out.
- **Instrumentation** — `DBMS_APPLICATION_INFO` (`MODULE`, `ACTION`, `CLIENT_INFO`), logging tables, `DBMS_SESSION.SET_IDENTIFIER`.
- **`DBMS_SCHEDULER`** — Oracle job scheduler: programs, schedules, jobs, chains, windows, event-based jobs.
- **API-only access** — revoking direct DML privileges on tables and exposing procedures that enforce rules.
- **Wrapping** — obfuscating PL/SQL source (`wrap` utility or `DBMS_DDL.CREATE_WRAPPED`).
- **JSON in SQL/PLSQL** — `JSON_TABLE`, `JSON_VALUE`, `JSON_QUERY`, `JSON_OBJECT`, `JSON_ARRAYAGG`, PL/SQL object type `JSON_OBJECT_T` (12.2+).

### 3. How it works, step by step
**Package structure**
1. Spec declares only what callers need: procedures, functions, public types/constants. Anything not in the spec is private.
2. Body implements and may hold state (cursors, caches). First reference in a session instantiates the package (runs its initialization block).
3. Deploying a new *body* does not invalidate callers; changing the *spec* invalidates dependents (11g+ is finer-grained: only dependents of changed elements).

**Bulk pattern**
1. Open a cursor on the source.
2. `FETCH c BULK COLLECT INTO l_rows LIMIT 1000;`
3. `EXIT WHEN l_rows.COUNT = 0;` (not `c%NOTFOUND`, which becomes true on the last partial batch and would drop it).
4. `FORALL i IN 1..l_rows.COUNT INSERT …` (use `SAVE EXCEPTIONS` if bad rows must not abort).
5. Commit per batch only if the business logic and restart design allow it.

**Locking down access (API-only writes)**
1. Tables owned by a data schema; no `INSERT/UPDATE/DELETE` granted to application users.
2. Application users get `EXECUTE` on package `orders_api` (definer's rights, owned by the data schema).
3. Optional: a statement-level or row-level trigger that checks a package flag (`orders_api.in_api`) and raises if DML is attempted outside the API; simpler and more robust is relying on grants alone.

### 4. Worked example
```sql
-- 1. Batch with bulk operations and instrumentation
CREATE OR REPLACE PACKAGE BODY archive_pkg AS
  c_batch CONSTANT PLS_INTEGER := 1000;

  PROCEDURE archive_old_orders(p_cutoff DATE) IS
    CURSOR c IS SELECT * FROM orders WHERE order_dt < p_cutoff;
    TYPE t_rows IS TABLE OF orders%ROWTYPE;
    l_rows t_rows;
    l_total PLS_INTEGER := 0;
  BEGIN
    DBMS_APPLICATION_INFO.SET_MODULE('ARCHIVE_PKG','archive_old_orders');
    OPEN c;
    LOOP
      FETCH c BULK COLLECT INTO l_rows LIMIT c_batch;
      EXIT WHEN l_rows.COUNT = 0;

      FORALL i IN 1..l_rows.COUNT
        INSERT INTO orders_arch VALUES l_rows(i);
      FORALL i IN 1..l_rows.COUNT
        DELETE FROM orders WHERE order_id = l_rows(i).order_id;

      l_total := l_total + l_rows.COUNT;
      DBMS_APPLICATION_INFO.SET_ACTION('archived ' || l_total);
    END LOOP;
    CLOSE c;
    COMMIT;
  EXCEPTION WHEN OTHERS THEN
    IF c%ISOPEN THEN CLOSE c; END IF;
    ROLLBACK;
    RAISE;
  END;
END archive_pkg;
/
```
Why this is faster than row-by-row: 1,000 rows cost *one* PL/SQL→SQL switch per statement instead of 1,000; `FORALL` generates redo in bulk (fewer redo *records* and `LGWR` hand-offs, fewer buffer-cache latch acquisitions because one execution touches many rows in the same block), and the cursor is parsed once. A pure SQL alternative (`INSERT … SELECT` then `DELETE`) is faster still, and `INSERT /*+ APPEND */` can use direct-path insert (minimal undo, but more redo only if `LOGGING`); use PL/SQL when row-level transformation or error isolation is required.

```sql
-- 2. Conditional compilation: instrumentation in dev/test only
ALTER SESSION SET PLSQL_CCFLAGS = 'debug:TRUE';
CREATE OR REPLACE PROCEDURE p IS
BEGIN
  $IF $$debug $THEN
     DBMS_OUTPUT.PUT_LINE('entering p');
  $END
  NULL;
END;
/
-- production build: ALTER SESSION SET PLSQL_CCFLAGS = 'debug:FALSE'; recompile → code is not in the object
```
```sql
-- 3. JSON in an integration layer
SELECT jt.*
FROM   api_payloads p,
       JSON_TABLE(p.body, '$.items[*]'
         COLUMNS (sku VARCHAR2(20) PATH '$.sku',
                  qty NUMBER       PATH '$.qty')) jt
WHERE  p.id = :id;

SELECT JSON_OBJECT('id' VALUE order_id, 'total' VALUE total) FROM orders WHERE order_id = :id;
```
```sql
-- 4. Scheduling with failure visibility
BEGIN
  DBMS_SCHEDULER.CREATE_JOB(
    job_name        => 'ARCHIVE_ORDERS_JOB',
    job_type        => 'PLSQL_BLOCK',
    job_action      => 'BEGIN archive_pkg.archive_old_orders(ADD_MONTHS(TRUNC(SYSDATE),-24)); END;',
    start_date      => SYSTIMESTAMP,
    repeat_interval => 'FREQ=DAILY;BYHOUR=2;BYMINUTE=0',
    enabled         => TRUE,
    auto_drop       => FALSE);
END;
/
SELECT job_name, status, error#, actual_start_date, run_duration, additional_info
FROM   dba_scheduler_job_run_details WHERE job_name = 'ARCHIVE_ORDERS_JOB'
ORDER  BY actual_start_date DESC;
```
Calendar strings (`FREQ=…;BYTIME`) handle time zones when `start_date` carries a region (`'Europe/Berlin'`) so daylight-saving shifts are applied by the scheduler. Chains (`DBMS_SCHEDULER.CREATE_CHAIN`) express dependencies between steps with conditional branching and failure handling.

### 5. Under the hood
- **Scalar functions per row.** A SELECT such as `SELECT id, tax(amount) FROM big` performs a SQL→PL/SQL switch per row (microseconds each, millions of rows → minutes), and any queries inside the function execute recursively once per call, each with its own read-consistent snapshot. Remedies: (1) *inline the logic as SQL/`CASE`*, (2) mark `DETERMINISTIC`/`RESULT_CACHE` or use a scalar-subquery cache, (3) `WITH FUNCTION`/`PRAGMA UDF` to cut call overhead, or (4) restructure into a join/pipelined function.
- **Redo and latch behavior of bulk vs. row-by-row.** Each row-by-row `INSERT` re-enters the SQL layer: separate redo change vectors, repeated buffer pinning (`cache buffers chains` latch), repeated undo record creation. A `FORALL` insert sends an array through one execution, which batches block visits and generates fewer, larger redo records; contention on shared pool, library cache and `log buffer` is lower. Numbers: 10–50× faster for 100k rows is common.
- **Execution plans of embedded SQL.** `EXPLAIN PLAN` shows the plan for the *text* with bind variables unpeeked; the actual runtime plan depends on peeked bind values, histograms, adaptive features and the session's NLS/optimizer settings. Read the runtime plan with `DBMS_XPLAN.DISPLAY_CURSOR(sql_id,child,'ALLSTATS LAST +PEEKED_BINDS')` after executing with `GATHER_PLAN_STATISTICS`; find the `SQL_ID` via `V$SQL` using `PROGRAM_ID`/`MODULE` or `V$SESSION.SQL_ID`.
- **`NOCOPY` semantics.** Normally an `OUT` or `IN OUT` parameter is copied in and copied back at normal exit, so on an exception the caller's variable is untouched. With `NOCOPY` the callee modifies the caller's variable directly (by reference): good for huge collections (avoids duplicating memory/time) but if the callee fails midway the caller's collection is left partially changed, and the hint is ignored in some situations (remote calls, implicit conversions, `FORALL` binds).
- **Conditional compilation** happens before parsing: unchosen branches are not in the compiled unit at all, so there is zero runtime cost; `PLSQL_CCFLAGS` is stored per object at compile time (`USER_PLSQL_OBJECT_SETTINGS`).
- **Wrapping** changes source representation (an encoded, compressed text), not encryption in the cryptographic sense: it prevents casual reading from `USER_SOURCE`, but deobfuscators exist, and `$IF` code can't be wrapped easily with variable names; keep the unwrapped source in version control.
- **Scheduler internals.** Job coordinator process (`CJQ0`) starts slaves (`Jnnn`); run history lives in `DBA_SCHEDULER_JOB_LOG` and `_RUN_DETAILS`; `max_job_slave_processes` limits concurrency.

### 6. Performance and design implications
- Design packages around business capabilities (`order_api`, `inventory_api`), expose the minimum, never expose raw tables to the app.
- Validate batch size (`LIMIT`) against PGA: 100–5,000 rows is typical; measure.
- Commit strategy: decide restartability up front (checkpoint table with last processed key), then commit per batch; never commit inside a loop fetching from a cursor on a table you are modifying (ORA-01555).
- Use `DBMS_APPLICATION_INFO` and `DBMS_SESSION.SET_IDENTIFIER` so `V$SESSION.MODULE/ACTION` identify which routine consumes resources; use `DBMS_MONITOR` to trace by module/action.
- For SQL embedded in PL/SQL, gather plan statistics and keep binds; check `V$SQL` for `executions`, `buffer_gets/executions`, and `loads`/`invalidations`.
- For JSON integration prefer SQL JSON functions (set-based) to string parsing in loops.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `BULK COLLECT INTO … LIMIT n` | Array fetch | Any multi-row processing |
| `FORALL … SAVE EXCEPTIONS` | Array DML, continue on error | Loads with bad rows |
| `DBMS_APPLICATION_INFO.SET_MODULE/SET_ACTION` | Label session activity | Observability |
| `DBMS_SCHEDULER.CREATE_JOB / CREATE_CHAIN` | Scheduling | Replace `DBMS_JOB`, cron |
| `PLSQL_CCFLAGS`, `$IF` | Conditional compilation | Debug code stripped from prod |
| `NOCOPY` | By-reference parameter | Large OUT collections |
| `wrap iname=x.sql` / `DBMS_DDL.CREATE_WRAPPED` | Obfuscate source | Shipped IP |
| `REVOKE INSERT,UPDATE,DELETE …` + `GRANT EXECUTE` | API-only access | Integrity enforcement |
| `PLSQL_WARNINGS='ENABLE:ALL'` | Compile-time warnings | CI quality gate |
| `utPLSQL` | Unit-test framework | Regression tests |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| Set-based SQL | Per-row logic with side effects/external calls |
| `BULK COLLECT`/`FORALL` | Source is small (overhead without benefit) or memory is constrained and no `LIMIT` |
| API-only (grants) | Tools/ETL need direct table access (use a separate privileged schema) |
| Trigger-based enforcement | Grants can do it; triggers hide logic |
| `NOCOPY` | Exception-safety matters more than speed |
| `DBMS_SCHEDULER` | Workflow needs cross-system orchestration (use an orchestrator, call procedures) |
| Wrapping | You need auditability or have untrusted deployment targets |

### 9. Common mistakes
- Row-by-row with commit per row → slow, non-restartable → bulk + batch commit.
- `BULK COLLECT` without `LIMIT` on big sets → PGA exhaustion (`ORA-04030`) → use `LIMIT`.
- `EXIT WHEN c%NOTFOUND` right after a bulk fetch → last partial batch lost → test `l_rows.COUNT = 0`.
- Calling `f(col)` in `WHERE` on big tables → full scans and switches → inline or index the expression.
- Using `EXPLAIN PLAN` and assuming it equals the runtime plan → misdiagnosis → `DISPLAY_CURSOR`.
- Debug `DBMS_OUTPUT` left in production code → buffer errors (`ORA-20000: buffer overflow`) and overhead → conditional compilation.
- Scheduling with `SYSDATE` and local time assumptions → DST drift → time zone-aware calendar.
- Relying on wrapping as security → exposure → keep secrets out of code.

### 10. 🎯 Interview kit
**30-second answer:** "In production I use packages so the public API is small and implementation can change without invalidating callers, process in bulk with `BULK COLLECT … LIMIT` and `FORALL` to cut context switches and redo overhead, avoid per-row functions in SQL, instrument with `DBMS_APPLICATION_INFO`, schedule via `DBMS_SCHEDULER`, and enforce an API-only write path through grants."

**Structure a longer answer:** structure (packages, rights) → data handling (bulk, commit strategy) → performance traps (functions in SQL, plans) → operability (instrumentation, scheduler, logging) → delivery (conditional compilation, tests, wrapping).

**Likely follow-ups:**
- *Spec vs. body?* → contract vs. implementation; invalidation behavior.
- *Why is `FORALL` faster?* → fewer context switches, fewer redo records and latch acquisitions, one parse.
- *Scalar UDF in SELECT slow — why/how fix?* → per-row switches + hidden queries; inline, cache, `WITH FUNCTION`.
- *Runtime plan vs. `EXPLAIN PLAN`?* → bind peeking and adaptive features; use `DISPLAY_CURSOR`.
- *`NOCOPY` risk?* → partial modification on exception.
- *Conditional compilation use case?* → keep debug instrumentation out of production builds.
- *Enforce API-only writes?* → revoke DML, grant `EXECUTE`, definer's rights packages.

**Signals interviewers look for:** you quantify, you know the failure modes (ORA-01555, PGA), and you treat operability as part of the design.

### 11. Check yourself
- [ ] Write a batch routine with `LIMIT`, `FORALL`, `SAVE EXCEPTIONS` and correct loop exit.
- [ ] Explain the redo/latch difference between `FORALL` and a row loop.
- [ ] Build a debug-only trace using `$IF $$debug`.
- [ ] Read a runtime plan with peeked binds and compare estimated vs. actual rows.
- [ ] Set up an API-only write path with grants.
- [ ] Create a scheduler job with a time-zone-aware calendar and check its run history.
<!-- explainer:end -->

## Implementations (tools)

- [Oracle](<../_index.md>)

## Practice questions

- [Basic] [What is a PL/SQL package and what are its two parts?](<../../../PRACTICE/TOOLS/Oracle/PL-SQL/Q-35726_What_is_a_PL_SQL_package_and_what_are_its_two_parts.md>)
- [Intermediate] [How would you loop through query results and print each row using an explicit cursor?](<../../../PRACTICE/TOOLS/Oracle/PL-SQL/Q-35793_How_would_you_loop_through_query_results_and_print_each_row_.md>)
- [Deep] [What is the difference between a strong REF CURSOR and a weak REF CURSOR?](<../../../PRACTICE/TOOLS/Oracle/PL-SQL/Q-35754_What_is_the_difference_between_a_strong_REF_CURSOR_and_a_wea.md>)
- [Deep] [Describe how you use DBMS_LOCK or an application-level lock to serialize a critical section across sessions in PL/SQL, and its pitfalls.](<../../../PRACTICE/TOOLS/Oracle/PL-SQL/Q-35621_Describe_how_you_use_DBMS_LOCK_or_an_application-level_lock_.md>)
- [Deep] [Describe a case where you chose a MERGE statement over separate INSERT/UPDATE logic in PL/SQL and how you handled the WHEN NOT MATCHED and DELETE clauses.](<../../../PRACTICE/TOOLS/Oracle/PL-SQL/Q-35651_Describe_a_case_where_you_chose_a_MERGE_statement_over_separ.md>)
- [Deep] [How do you implement optimistic locking in a PL/SQL update using a version column or ORA_ROWSCN, and how do you detect a lost update?](<../../../PRACTICE/TOOLS/Oracle/PL-SQL/Q-35686_How_do_you_implement_optimistic_locking_in_a_PL_SQL_update_u.md>)
- [Expert] [How would you decide between a NOCOPY hint on a large OUT collection parameter versus the default pass-by-value, and what risk does NOCOPY introduce on exception?](<../../../PRACTICE/TOOLS/Oracle/PL-SQL/Q-35603_How_would_you_decide_between_a_NOCOPY_hint_on_a_large_OUT_co.md>)
- [Expert] [Explain how BULK COLLECT INTO a record collection combined with FORALL INSERT differs from row-by-row inserts in redo generation and latch behavior.](<../../../PRACTICE/TOOLS/Oracle/PL-SQL/Q-35648_Explain_how_BULK_COLLECT_INTO_a_record_collection_combined_w.md>)

## Related concepts

- [Large Batch Updates](<../../../01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Large_Batch_Updates.md>) *(SQL Fundamentals)*
- [MERGE Pitfalls and Error Handling](<../../SQL_Server/T-SQL/MERGE_Pitfalls_and_Error_Handling.md>) *(T-SQL)*
- [SKIP LOCKED Job Queues](<../../../01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/SKIP_LOCKED_Job_Queues.md>) *(SQL Fundamentals)*
- [Dynamic SQL](<../../../01_Databases_and_SQL/Database_Internals/DBMS_Concepts/Dynamic_SQL.md>) *(DBMS Concepts)*
- [Bulk Collect and FORALL](<../../../01_Databases_and_SQL/Database_Objects/Cursors/Bulk_Collect_and_FORALL.md>) *(Cursors)*
- [Stored Procedure Performance and Design](<../../../01_Databases_and_SQL/Database_Objects/Stored_Procedures/Stored_Procedure_Performance_and_Design.md>) *(Stored Procedures)*

## References

- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 7: Dynamic SQL, Triggers, and Table-Valued Functions (TVFs) › Structure
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 15: Case Studies and the Future of T-SQL › Refactoring Legacy Code
- *Designing Data-Intensive Applications* — Martin Kleppmann — D
- *Data Quality Fundamentals* — Barr Moses, Lior Gavish, Molly Vorwerck — Building End-to-End Lineage › Data Lineage Design
- *100 SQL Server Mistakes and How to Avoid Them* — Peter A. Carter — Instance and database management › 9.9 #51 Failing to automate
- *Fuzzy Data Matching with SQL* — Jim Lehmer — Import Artifacts into SQL › And Now, for My Next Trick
- *SQL Server Integration Services Design Patterns* — Tim Mitchell et al. — Evolution of an SSIS Framework › Building Package Instance Logging

