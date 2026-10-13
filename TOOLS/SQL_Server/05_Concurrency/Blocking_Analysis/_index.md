---
id: "TOOLS/SQL_Server/05_Concurrency/Blocking_Analysis"
level: 4
type: "curriculum"
curriculum: "sql_server"
---

# Blocking Analysis

**L4 Advanced** · [↑ Concurrency, Locking and Blocking](<../_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** Blocking analysis means finding the **head blocker** of a wait chain, learning *what it holds and why it's still holding it*, and fixing the cause (long transaction, missing index, wrong isolation, orphaned transaction), not just killing sessions.

### 1. The big picture
Blocking is a normal part of locking: a request waits briefly for an incompatible lock. It becomes a **problem** when waits are long or form chains: dozens of sessions stuck behind one. Users see timeouts and "the app is frozen," while CPU and I/O may look idle. This lesson gives you a repeatable process to diagnose live blocking and capture it historically. It relies on *Lock Modes and Granularity* and *Isolation and Row Versioning*, and leads to *Deadlocks*, where waits form a cycle instead of a chain.

### 2. Core concepts and vocabulary
- **Blocker / blocked (waiter)**: the session holding the lock / the one waiting for it.
- **Blocking chain**: A blocks B, B blocks C… The **head blocker** is the session that blocks others but isn't blocked itself.
- **`blocking_session_id`**: column in `sys.dm_exec_requests` / `sys.dm_os_waiting_tasks` naming who you are waiting on.
- **Lock wait types**: `LCK_M_<mode>`, e.g. `LCK_M_S` (wants shared), `LCK_M_U`, `LCK_M_X`, `LCK_M_IX`, `LCK_M_SCH_M`.
- **wait_resource**: what is being waited for, e.g. `KEY: 7:72057594047758336 (8194443284a0)` = database 7, HoBT id, key hash. Or `PAGE: 7:1:2345`, or `OBJECT: 7:245575913:0`.
- **Sleeping session with open transaction**: a session that isn't running anything (`status = sleeping`) but still has `open_transaction_count > 0`. That's the most common head blocker.
- **Blocked process report**: an event raised when a block exceeds the `blocked process threshold` (seconds). It contains XML for both blocker and blocked.
- **Lock Waits/sec, Average Wait Time (ms)**: performance counters under *SQLServer:Locks*.

### 3. How it works, step by step
**Live diagnosis (it's happening now)**
1. **Confirm it's blocking.** Look for requests with `blocking_session_id <> 0` and `LCK_M_*` wait types.
2. **Find the head blocker.** Walk `blocking_session_id` up the chain until you reach a session whose own `blocking_session_id` is 0.
3. **Look at the head blocker:**
   - Is it running (`sys.dm_exec_requests` has a row) or sleeping (only in `sys.dm_exec_sessions`)?
   - `open_transaction_count`, transaction start time (`sys.dm_tran_active_transactions` / `sys.dm_tran_session_transactions`)
   - Last statement: `sys.dm_exec_connections.most_recent_sql_handle` → `sys.dm_exec_sql_text`
   - `host_name`, `program_name`, `login_name`: which app or user is it?
4. **Look at the resource.** Decode `wait_resource` to a table/index (`sys.partitions` by `hobt_id`). Check what the blocker holds in `sys.dm_tran_locks`.
5. **Decide the immediate action.** Wait, ask the owner to commit, or `KILL <spid>` as a last resort. Killing rolls back its work, which can take as long as the work took.
6. **Find the root cause** (step 6 in the table below) so it doesn't recur.

**Typical root causes**

| Pattern seen | Root cause | Fix |
|---|---|---|
| Head blocker sleeping, open transaction | app opened a transaction and didn't commit (error path, user prompt, timeout without rollback) | `SET XACT_ABORT ON`, TRY/CATCH with rollback, no user interaction inside transactions |
| Head blocker running a long UPDATE/DELETE | big batch, maybe escalated to table lock | batch the work, run off-hours, index the WHERE |
| Many `LCK_M_S` readers behind writers | locking read committed | RCSI |
| Blocker scans a table to update a few rows | missing index; locks many rows | index the predicate |
| `LCK_M_SCH_M` / everyone behind DDL | index rebuild or ALTER waiting on a long query | `WAIT_AT_LOW_PRIORITY`, schedule DDL |
| SERIALIZABLE range locks | framework default isolation | set READ COMMITTED explicitly |

**Historical capture (it happened last night)**
1. Set `blocked process threshold` (e.g. 10–20 seconds).
2. Create an Extended Events session on `sqlserver.blocked_process_report`.
3. Read the XML: `<blocked-process>` and `<blocking-process>` show spid, statement, transaction start, isolation level, wait resource and input buffer.

### 4. Worked example
```sql
-- 1) Who is blocked, by whom, on what
SELECT r.session_id, r.blocking_session_id, r.wait_type, r.wait_time/1000 AS wait_s,
       r.wait_resource, DB_NAME(r.database_id) AS db, t.text AS running_sql
FROM sys.dm_exec_requests AS r
CROSS APPLY sys.dm_exec_sql_text(r.sql_handle) AS t
WHERE r.blocking_session_id <> 0;

-- 2) Head blockers: block others but are not blocked themselves
SELECT s.session_id, s.status, s.open_transaction_count, s.host_name, s.program_name,
       s.login_name, s.last_request_end_time, t.text AS last_sql
FROM sys.dm_exec_sessions AS s
JOIN sys.dm_exec_connections AS c ON c.session_id = s.session_id
CROSS APPLY sys.dm_exec_sql_text(c.most_recent_sql_handle) AS t
WHERE s.session_id IN (SELECT blocking_session_id FROM sys.dm_exec_requests WHERE blocking_session_id <> 0)
  AND s.session_id NOT IN (SELECT session_id FROM sys.dm_exec_requests WHERE blocking_session_id <> 0);
-- status = sleeping, open_transaction_count = 1, last_request_end_time 20 min ago
--   → an orphaned transaction from the app named in program_name

-- 3) Decode a KEY wait_resource 'KEY: 7:72057594047758336 (8194443284a0)'
SELECT OBJECT_NAME(p.object_id) AS tbl, i.name AS idx
FROM sys.partitions AS p JOIN sys.indexes AS i
  ON i.object_id = p.object_id AND i.index_id = p.index_id
WHERE p.hobt_id = 72057594047758336;

-- 4) Historical capture
EXEC sp_configure 'show advanced options', 1; RECONFIGURE;
EXEC sp_configure 'blocked process threshold (s)', 15; RECONFIGURE;
CREATE EVENT SESSION Blocking ON SERVER
ADD EVENT sqlserver.blocked_process_report
ADD TARGET package0.event_file (SET filename = N'Blocking');
ALTER EVENT SESSION Blocking ON SERVER STATE = START;
```
Community tools such as `sp_WhoIsActive` (with `@find_block_leaders = 1`) and `sp_BlitzWho` package the same queries.

### 5. Under the hood
- **Lock monitor** checks waits every few seconds. The blocked process report is produced by the same background thread, so reports appear at roughly threshold-sized intervals, repeated while the block lasts.
- **Waiting is FIFO-ish.** Lock requests queue in order. A new S request can't jump ahead of a waiting X (to avoid starving writers). So a single waiting DDL (Sch-M) can make *every* later query wait, even ones compatible with the current holder. That's the "everything froze behind ALTER INDEX" effect.
- **Application timeouts** cancel the current statement (an attention event) but don't roll back an open transaction unless `XACT_ABORT` is on. The connection goes back to the pool *with the transaction open*. Pooled connections are reset (`sp_reset_connection`) only when reused, so the locks can linger for a long time.
- **Implicit transactions** (`SET IMPLICIT_TRANSACTIONS ON`, the default in some drivers/JDBC modes) start a transaction on the first statement and wait for an explicit COMMIT. That causes orphaned locks from read-only code.

### 6. Performance and design implications
- Keep transactions short and server-side. Never span a transaction across user think time or remote calls.
- Use `SET XACT_ABORT ON` in procedures, so any error rolls back and releases locks.
- Good indexes reduce both the duration and the footprint of locks.
- Turn on RCSI to remove reader/writer blocking. Keep writer/writer blocking short.
- Monitor baseline numbers: `LCK_M_*` share of wait time, Lock Waits/sec, longest open transaction (`sys.dm_tran_database_transactions.database_transaction_begin_time`), and blocked process reports per day.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `sys.dm_exec_requests` (`blocking_session_id`, `wait_type`, `wait_resource`) | who waits on whom | first query during an incident |
| `sys.dm_os_waiting_tasks` | per-task waits (includes parallel threads) | complex chains, parallel queries |
| `sys.dm_tran_locks` | lock holders and waiters by resource | what exactly is held |
| `sys.dm_exec_sessions` (`open_transaction_count`, `status`) | find sleeping sessions with open transactions | orphaned transactions |
| `DBCC OPENTRAN` | oldest active transaction in a DB | quick check |
| `sp_configure 'blocked process threshold (s)'` + XE `blocked_process_report` | records blocks longer than N seconds | ongoing capture |
| `KILL <spid>` / `KILL <spid> WITH STATUSONLY` | terminates a session / shows rollback progress | last resort |
| `SET XACT_ABORT ON` | roll back the whole transaction on error/timeout | all stored procedures |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| `KILL` the head blocker: orphaned idle transaction, business approves | a large write is near completion (rollback can take as long) |
| RCSI: readers blocked by writers | writer/writer conflicts (no help) |
| Batch large writes: blocking during jobs | must be atomic (use partition switching or staging) |
| Low blocked process threshold (5 s): active investigation | permanent setting on busy systems (noise) |
| `LOCK_TIMEOUT`: fail fast in the app | apps that don't handle error 1222 |

### 9. Common mistakes
- Killing the blocked victims instead of the head blocker → nothing changes → walk the chain to the top.
- Killing a blocker in the middle of a huge update → hours of rollback → check progress first, consider waiting.
- Assuming the head blocker's *current* statement caused the locks → the locks may come from an earlier statement in the same transaction → check the transaction start time and what it holds.
- Ignoring sleeping sessions → orphaned transactions stay invisible → filter on `open_transaction_count > 0`.
- Fixing blocking with `NOLOCK` → wrong data → RCSI, indexes, short transactions.

### 10. 🎯 Interview kit
**30-second answer:** I check `sys.dm_exec_requests` for sessions with a non-zero `blocking_session_id`, walk the chain to the head blocker, and look at its status, open transaction count, last statement, host and program, and the locks it holds in `sys.dm_tran_locks`. Very often it's a sleeping session with an open transaction that the app forgot to commit after a timeout. Short term, I may kill it after checking the rollback cost. Long term, I fix the cause: `XACT_ABORT ON` and proper error handling, shorter transactions, indexes so updates touch fewer rows, batching, or RCSI for reader/writer blocking. For history, I use the blocked process report through Extended Events.
**Structure a longer answer:** symptoms → live DMV workflow → common root causes table → historical capture → prevention.
**Likely follow-ups:** *Why can a waiting index rebuild block SELECTs?* → lock queue order: later S requests wait behind the queued Sch-M. · *Why does a timeout leave locks?* → the statement is cancelled, the transaction isn't rolled back. · *How do you decode wait_resource?* → the HoBT id maps to `sys.partitions`.
**Signals interviewers look for:** head-blocker thinking, awareness of orphaned transactions and `XACT_ABORT`, a fixed process rather than guesswork.

### 11. Check yourself
- [ ] Create a three-session chain and find the head blocker with DMVs only.
- [ ] Reproduce an orphaned transaction (BEGIN TRAN + UPDATE, then idle) and spot it as `sleeping` with an open transaction.
- [ ] Decode a KEY `wait_resource` into table and index.
- [ ] Configure the blocked process report and read one event's XML.
- [ ] Explain how `SET XACT_ABORT ON` prevents lingering locks after a client timeout.
<!-- explainer:end -->

## References

- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 10: Transactions, Concurrency, and Isolation Levels › Detecting Blocking
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 10: Transactions, Concurrency, and Isolation Levels › Monitoring Locks
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 10: Transactions, Concurrency, and Isolation Levels › Detecting Blocking Chains
- *SQL Server 2025 Query Performance Tuning (7th ed.)* — Grant Fritchey — FULL_BOOK_NEEDS_MANUAL_SPLIT (cont.) › Chapter 16 Blocking And Blocked Processes
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 10: Transactions, Concurrency, and Isolation Levels › Capturing Deadlock Information
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 10: Transactions, Concurrency, and Isolation Levels › Monitoring Isolation Level Effects
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 10: Transactions, Concurrency, and Isolation Levels › Tools for Deadlock Troubleshooting
- *Expert Performance Indexing in SQL Server* — Jason Strate, Grant Fritchey — Lock Waits per Second

