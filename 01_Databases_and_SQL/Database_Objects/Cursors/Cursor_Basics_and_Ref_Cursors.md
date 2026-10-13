---
id: "01_Databases_and_SQL/Database_Objects/Cursors/Cursor_Basics_and_Ref_Cursors"
level: 4
type: "concept"
topic: "01_Databases_and_SQL/Database_Objects/Cursors"
tools: ["Oracle", "SQL_Server"]
---

# Cursor Basics and Ref Cursors

**L4 Advanced** · [↑ Cursors](<_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** A cursor is a server- or client-side handle on a query's result set that lets procedural code walk it one row (or small batch) at a time — you DECLARE, OPEN, FETCH in a loop, then CLOSE/DEALLOCATE; a *ref cursor* (cursor variable) is the same handle held in a variable so it can be passed around and bound to a query chosen at run time.

### 1. The big picture
SQL is set-oriented: one statement describes a whole result and the engine processes it as a unit. Procedural code (T-SQL, PL/SQL, PL/pgSQL, application languages) is row-oriented: it wants "the next row" so it can branch, call something, or build output. A **cursor** bridges the two: the query is evaluated by the engine, and the cursor remembers *where you are* in the result so you can fetch rows incrementally.

Cursors exist for good reasons — streaming a huge result without loading it all into memory, row-by-row administrative actions (running a command per database or table), returning a result from a procedure to a caller, and logic that really cannot be expressed as a set operation. But they are also the single most common source of slow database code: every FETCH and every statement inside the loop carries per-statement overhead, so a loop over a million rows behaves like a million separate statements. This lesson covers how cursors work and the vocabulary; the companion lessons cover *when to replace them* (set-based SQL), *batching* (bulk collect), and *consistency* (what a cursor sees while data changes).

### 2. Core concepts and vocabulary
- **Cursor** — a named pointer to the current row of a query's result set.
- **Implicit cursor** — one the engine creates for you for a single SQL statement (Oracle: every DML and `SELECT INTO`; attributes `SQL%FOUND`, `SQL%ROWCOUNT`). **Explicit cursor** — one you declare, open and fetch yourself.
- **Server cursor** (T-SQL `DECLARE … CURSOR`) vs **client cursor** (created by the driver/data-access layer, e.g. ADO.NET `DataReader`, JDBC `ResultSet`).
- **Lifecycle** — DECLARE (bind to a SELECT and set characteristics) → OPEN (execute/position before first row) → FETCH (move + read) → CLOSE (release the result/locks) → DEALLOCATE (remove the definition, T-SQL).
- **Fetch status** — loop-control signal: `@@FETCH_STATUS = 0` (T-SQL), `%FOUND`/`%NOTFOUND` (PL/SQL), `NOT FOUND` condition handler (MySQL).
- **FORWARD_ONLY vs SCROLL** — can you only move to the next row, or also `PRIOR`, `FIRST`, `LAST`, `ABSOLUTE n`, `RELATIVE n`.
- **READ ONLY vs updatable** (`FOR UPDATE`, `WHERE CURRENT OF`) — whether you can modify the row the cursor sits on.
- **STATIC / KEYSET / DYNAMIC / FAST_FORWARD** — T-SQL cursor types describing how the result set is materialised and how sensitive it is to later changes.
- **Cursor FOR loop** (PL/SQL) — construct that opens, fetches and closes automatically.
- **Ref cursor / cursor variable** (`SYS_REFCURSOR`, `REF CURSOR`, `refcursor`) — a variable that points to a cursor; can be opened `FOR` any compatible query and returned to callers.
- **Strongly vs weakly typed ref cursor** — return type declared (compiler checks) vs any query allowed.
- **RBAR** — "row by agonizing row", the derogatory name for loops that process rows individually.

### 3. How it works, step by step
**Explicit cursor lifecycle (T-SQL)**
1. `DECLARE c CURSOR [LOCAL|GLOBAL] [FORWARD_ONLY|SCROLL] [STATIC|KEYSET|DYNAMIC|FAST_FORWARD] [READ_ONLY|…] FOR SELECT …` — nothing runs yet; the definition is compiled.
2. `OPEN c` — the query is executed (a STATIC/KEYSET cursor builds a worktable in tempdb at this moment); the position is *before* the first row.
3. `FETCH NEXT FROM c INTO @vars` — advances and copies column values to variables; sets `@@FETCH_STATUS`.
4. Loop: `WHILE @@FETCH_STATUS = 0 BEGIN … FETCH NEXT … END`.
5. `CLOSE c` — releases the result set and locks; `DEALLOCATE c` — frees the definition. Without these, resources (tempdb, memory, locks) leak until session end.

**PL/SQL equivalent**
```
OPEN c;  LOOP FETCH c INTO v;  EXIT WHEN c%NOTFOUND;  …  END LOOP;  CLOSE c;
-- or simply: FOR r IN c LOOP … END LOOP;   -- engine opens/fetches/closes
```
Opening a cursor that is already open raises `ORA-01003`-style errors; fetching from an unopened cursor raises "cursor not open".

**Ref cursor**
1. Declare a variable of cursor type.
2. `OPEN rc FOR SELECT …` (the SELECT can be chosen by `IF` or built as dynamic SQL).
3. Hand `rc` to another routine or return it as an OUT parameter / function result; the caller FETCHes from it.
4. The *owner of the final fetch* must CLOSE it.

| | Fixed cursor | Ref cursor / cursor variable |
|---|---|---|
| Query bound | at compile time | at OPEN time |
| Passable as parameter / return value | no | yes |
| Typical use | loops inside one routine | returning result sets to clients, generic reporting procs |
| Compile-time checking | full | strong type: yes; weak: no |

### 4. Worked example
```sql
-- SQL Server: the "textbook" cursor and why it is usually wrong
DECLARE @OrderID int, @Total money;
DECLARE c CURSOR LOCAL FAST_FORWARD FOR
    SELECT OrderID, Total FROM dbo.Orders WHERE Status = 'Pending';
OPEN c;
FETCH NEXT FROM c INTO @OrderID, @Total;
WHILE @@FETCH_STATUS = 0
BEGIN
    IF @Total > 1000
        UPDATE dbo.Orders SET Status = 'Review' WHERE OrderID = @OrderID;
    ELSE
        UPDATE dbo.Orders SET Status = 'Processed' WHERE OrderID = @OrderID;
    FETCH NEXT FROM c INTO @OrderID, @Total;
END
CLOSE c; DEALLOCATE c;

-- The set-based equivalent: one statement, one plan
UPDATE dbo.Orders
SET Status = CASE WHEN Total > 1000 THEN 'Review' ELSE 'Processed' END
WHERE Status = 'Pending';
```
Run both with `SET STATISTICS TIME, IO ON`; for 100,000 rows the cursor version issues 100,000 single-row updates (each with its own plan lookup, lock acquisition and log record), the set version a single scan and a batched update. A legitimate cursor, by contrast:

```sql
-- Administrative: run a command per database — set-based SQL cannot do this
DECLARE @db sysname, @sql nvarchar(max);
DECLARE d CURSOR LOCAL FAST_FORWARD FOR SELECT name FROM sys.databases WHERE state = 0;
OPEN d; FETCH NEXT FROM d INTO @db;
WHILE @@FETCH_STATUS = 0
BEGIN
    SET @sql = N'DBCC CHECKDB(' + QUOTENAME(@db) + N') WITH NO_INFOMSGS;';
    EXEC (@sql);
    FETCH NEXT FROM d INTO @db;
END
CLOSE d; DEALLOCATE d;
```
And a ref cursor returning a result set from PL/SQL:
```sql
CREATE OR REPLACE PROCEDURE get_orders(p_status IN VARCHAR2, p_rc OUT SYS_REFCURSOR) AS
BEGIN
  OPEN p_rc FOR SELECT order_id, total FROM orders WHERE status = p_status;
END;
```
The client fetches from `p_rc`, then closes it; the server procedure never fetches.

### 5. Under the hood
- **Per-iteration cost.** The book's framing: each FETCH plus each statement in the loop costs as much as running it stand-alone — parse/plan-cache lookup, permission checks, lock acquire/release, a log record, and a client/engine "context switch" in PL/SQL (SQL ↔ PL/SQL engine). One million iterations ≈ one million statements.
- **Cursor characteristics** are the main driver of overhead (Fritchey): *location* (client vs server), *concurrency* (read-only, optimistic, scroll locks) and *type* (forward-only, static, keyset, dynamic). The cheapest is `LOCAL FAST_FORWARD` (forward-only, read-only, with performance optimisations).
- **Where the rows live.** STATIC copies the whole result to a tempdb worktable at OPEN (stable but pays up-front I/O and space). KEYSET copies only the keys; non-key columns are read live. DYNAMIC re-evaluates the query as you move — no worktable but the heaviest locking and the one that sees every change.
- **Client cursors** push the work out of the server (fewer server resources, better scalability) at the cost of network traffic and client memory; **server cursors** keep rows on the server and allow fetching small chunks, but hold server resources and often a transaction/lock for as long as they're open.
- **Fetch size.** Drivers fetch rows in batches (JDBC `setFetchSize`, psycopg2 named cursors `itersize`); the batch size decides how many rows cross the network per round trip. Too small → many round trips; too large → client memory pressure.
- **Ref cursors** are pointers to *open* server-side cursor state (in Oracle, an entry in the session's open-cursor cache and PGA); they stay valid only within the session (and often the transaction, unless declared `WITH HOLD`), and each unclosed one counts toward the open-cursor limit (`ORA-01000`).
- **Implicit cursors** in PL/SQL include the single-row `SELECT INTO`, which raises `NO_DATA_FOUND`/`TOO_MANY_ROWS` rather than setting `%NOTFOUND`; this is a classic interview contrast with explicit cursor fetches.

### 6. Performance and design implications
- Default to **set-based SQL**; reach for a cursor only when each row needs something SQL can't do set-wise (dynamic DDL, calling a procedure per row, row-level external effects).
- If you must loop: `LOCAL FAST_FORWARD` (T-SQL), `READ ONLY`, fetch only needed columns, filter in the cursor query rather than in the loop, and keep per-row work minimal.
- **Close promptly.** An open cursor can hold locks and prevent version-store/undo cleanup; avoid holding it across user interaction or slow app-side processing.
- Streaming large exports: use a server-side/named cursor with a sensible fetch size so the client doesn't buffer everything; keep it in a read-only transaction.
- Use ref cursors to return result sets from procedures instead of building temp tables, but make ownership explicit — who closes it.
- Measure: `SET STATISTICS IO/TIME`, `sys.dm_exec_cursors` (SQL Server), `v$open_cursor` / `v$sesstat` (Oracle), `pg_cursors` (PostgreSQL), and the number of statements executed (`sys.dm_exec_query_stats.execution_count`) to expose hidden per-row statements.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `DECLARE c CURSOR LOCAL FAST_FORWARD FOR …` | Cheapest T-SQL cursor | Unavoidable read-only loop |
| `FETCH NEXT/PRIOR/FIRST/LAST/ABSOLUTE/RELATIVE` | Positioned reads (scroll cursors) | Paging back/forward in a stable set |
| `@@FETCH_STATUS`, `@@CURSOR_ROWS` | Loop control / row count | Looping, sizing |
| `UPDATE … WHERE CURRENT OF c` | Modify the row under an updatable cursor | Rare; prefer keyed set-based update |
| `SYS_REFCURSOR` / `OPEN rc FOR …` | Cursor variable | Returning result sets, dynamic queries |
| `FOR r IN (SELECT …) LOOP` (PL/SQL) | Auto-managed explicit cursor | Simple PL/SQL loops (Oracle optimises with array fetch of 100) |
| `DECLARE … CURSOR WITH HOLD` (PG/DB2) | Cursor survives COMMIT | Long batch loops committing in chunks |
| `sys.dm_exec_cursors(0)` | List open cursors | Find leaks/long-lived cursors |
| JDBC `setFetchSize`, psycopg2 `cursor(name)` | Control rows per round trip | Streaming big results |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| **Cursor loop** — per-row dynamic SQL/DDL, per-row procedure/API call, complex row-wise state that can't be windowed | Any data transformation expressible as UPDATE/INSERT…SELECT, window functions, aggregates, recursive CTEs |
| **FAST_FORWARD / LOCAL** — read-only single pass | You need to scroll back or see concurrent changes |
| **STATIC** — need a stable snapshot and scrolling | Large result sets (tempdb cost) |
| **Client-side streaming reader** — stream to file/Kafka with limited memory | You'd hold a transaction open for hours |
| **Ref cursor** — return flexible result sets, hand off between routines | Callers can't be trusted to close it; plain `SELECT` result sets are enough |
| **WHILE loop on a key** — simple batching by key range | Gaps/duplicates in the key (loop counter must handle gaps) |

### 9. Common mistakes
- **Using a cursor to update/aggregate/pivot** → minutes instead of seconds, long locks → rewrite as set-based (see the companion lesson).
- **Forgetting `CLOSE`/`DEALLOCATE`** (or not using `LOCAL`) → leaked cursors, "cursor already exists" errors → always release, prefer `LOCAL`.
- **Default cursor options** (`DECLARE c CURSOR FOR …` = DYNAMIC-ish, updatable, global) → needless tempdb and locking → state `LOCAL FAST_FORWARD READ_ONLY`.
- **Putting a lookup `SELECT` inside the loop** → N extra queries → join in the cursor query.
- **MySQL `CONTINUE HANDLER FOR NOT FOUND`** swallowing a `NOT FOUND` from a nested `SELECT INTO` inside the loop → loop ends early (or never) → set a flag in a separate block or use a dedicated handler scope.
- **Opening an already open ref cursor / fetching after close** → runtime errors → check `%ISOPEN`, structure ownership.
- **SCROLL cursor "just in case"** → keeps a full snapshot in tempdb → forward-only unless needed.
- **Assuming a cursor sees later changes (or doesn't)** → wrong results → check cursor type/sensitivity (next lessons).

### 10. 🎯 Interview kit
**30-second answer:** "A cursor lets procedural code process a query result row by row: declare it, open it, fetch in a loop, close it. It's needed for per-row actions SQL can't express, but it's slow because each iteration has statement-level overhead, so I replace it with set-based SQL whenever possible and, when I must use one, make it local, forward-only, read-only. A ref cursor is a cursor held in a variable so it can be opened for different queries and passed to or returned from procedures."

**Structure a longer answer:** definition and lifecycle → why/when needed → cost model (per-row overhead, tempdb, locks) → types/options → alternatives → real example (per-database maintenance loop vs set-based UPDATE).

**Likely follow-ups:**
- *Implicit vs explicit cursor?* → Implicit is engine-managed per statement (`SQL%ROWCOUNT`); explicit is declared/controlled by you.
- *What happens if you never close it?* → resource and lock leaks until session end; in Oracle `ORA-01000`.
- *FORWARD_ONLY vs SCROLL?* → SCROLL needs a stable (static/keyset) result and more memory.
- *Keyset cursor behaviour?* → keys fixed at open: deletes appear as missing rows, updates to non-key columns visible, new rows not visible.
- *Cursor variable vs fixed cursor?* → query bound at open time; passable; weak typing.
- *Why are cursors valid only within a transaction?* → by default, they're closed at commit (non-holdable); `WITH HOLD` keeps them past commit.

**Signals interviewers look for:** you don't dismiss cursors absurdly but can state the cost, name the cheapest options, offer the set-based alternative first, and know lifecycle/cleanup.

### 11. Check yourself
- [ ] Write a T-SQL cursor from DECLARE to DEALLOCATE with correct loop control.
- [ ] List the three cursor characteristic categories and the cheapest combination.
- [ ] Explain how a ref cursor differs from an ordinary cursor and who must close it.
- [ ] Identify a cursor in a stored procedure and say whether it is justified.
- [ ] Describe what leaks if a cursor is opened but never closed.
<!-- explainer:end -->

## Implementations (tools)

- [Oracle](<../../../TOOLS/Oracle/_index.md>)
- [SQL Server](<../../../TOOLS/SQL_Server/_index.md>)

## Practice questions

- [Basic] [When does Oracle create an implicit cursor automatically?](<../../../PRACTICE/01_Databases_and_SQL/Database_Objects/Cursors/Q-10335_When_does_Oracle_create_an_implicit_cursor_automatically.md>)
- [Basic] [What is meant by row-by-row processing and how does a cursor enable it?](<../../../PRACTICE/01_Databases_and_SQL/Database_Objects/Cursors/Q-10479_What_is_meant_by_row-by-row_processing_and_how_does_a_cursor.md>)
- [Intermediate] [In SQL Server how does the @@FETCH_STATUS variable control a cursor WHILE loop?](<../../../PRACTICE/01_Databases_and_SQL/Database_Objects/Cursors/Q-10446_In_SQL_Server_how_does_the_FETCH_STATUS_variable_control_a_c.md>)
- [Intermediate] [How would you handle the situation where a cursor query returns no rows at all?](<../../../PRACTICE/01_Databases_and_SQL/Database_Objects/Cursors/Q-10480_How_would_you_handle_the_situation_where_a_cursor_query_retu.md>)
- [Intermediate] [Where should you place the CLOSE cursor statement so it runs even when an exception occurs?](<../../../PRACTICE/01_Databases_and_SQL/Database_Objects/Cursors/Q-10521_Where_should_you_place_the_CLOSE_cursor_statement_so_it_runs.md>)
- [Deep] [What is the difference between a strong REF CURSOR and a weak REF CURSOR in PL/SQL?](<../../../PRACTICE/01_Databases_and_SQL/Database_Objects/Cursors/Q-10475_What_is_the_difference_between_a_strong_REF_CURSOR_and_a_wea.md>)
- [Deep] [When a JDBC ResultSet streams rows from the database how is that similar to using a cursor?](<../../../PRACTICE/01_Databases_and_SQL/Database_Objects/Cursors/Q-10513_When_a_JDBC_ResultSet_streams_rows_from_the_database_how_is_.md>)
- [Deep] [Explain why a cursor FOR loop in Oracle 12c+ implicitly uses an array fetch of 100 rows and when you would override that behavior.](<../../../PRACTICE/01_Databases_and_SQL/Database_Objects/Cursors/Q-10353_Explain_why_a_cursor_FOR_loop_in_Oracle_12c_implicitly_uses_.md>)

## Related concepts

- [Stored Procedure Performance and Design](<../Stored_Procedures/Stored_Procedure_Performance_and_Design.md>) *(Stored Procedures)*
- [Dynamic SQL](<../../Database_Internals/DBMS_Concepts/Dynamic_SQL.md>) *(DBMS Concepts)*
- [PL-SQL in Production](<../../../TOOLS/Oracle/PL-SQL/PL-SQL_in_Production.md>) *(PL-SQL)*

## References

- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 8: Query Optimization and Performance Tuning › Overusing Cursors
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 16: Best Practices for T-SQL Development and Database Design › Cursors for Row-by-Row Processing
- *SQL Server 2025 Query Performance Tuning (7th ed.)* — Grant Fritchey — FULL_BOOK_NEEDS_MANUAL_SPLIT (cont.)
- *SQL Server 2025 Query Performance Tuning (7th ed.)* — Grant Fritchey — FULL_BOOK_NEEDS_MANUAL_SPLIT (cont.) › Chapter 18 Row-By-Row Processing From Cursors And Other Causes
- *100 SQL Server Mistakes and How to Avoid Them* — Peter A. Carter — T-SQL development › 5.7 #20 Using cursors
- *100 SQL Server Mistakes and How to Avoid Them* — Peter A. Carter — Instance and database management › 9.10 #52 Using cursors for administrative purposes
- *Beginning T-SQL* — Kathi Kellenberger, Lee Everest — Temporary Tables and Table Variables › Using a Cursor
- *T-SQL Fundamentals (4th ed.)* — Itzik Ben-Gan — Programmable objects
- *SQL Server Tutorial* (YouTube series) — Part 63: Cursors in sql server ([watch](https://www.youtube.com/watch?v=INw_KGjyfDw))

