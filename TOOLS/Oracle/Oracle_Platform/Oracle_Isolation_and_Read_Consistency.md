---
id: "TOOLS/Oracle/Oracle_Platform/Oracle_Isolation_and_Read_Consistency"
level: 4
type: "concept"
topic: "TOOLS/Oracle/Oracle_Platform"
tools: ["Oracle"]
---

# Oracle Isolation and Read Consistency

**L4 Advanced** · [↑ Oracle Platform](<_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** Oracle gives every statement a consistent snapshot using undo (multiversion read consistency), so readers never block writers and writers never block readers; only writers block writers, through row-level (TX) locks.

### 1. The big picture
A transaction is a unit of work that either commits as a whole or is rolled back. The harder question is what *other* sessions see while it is running. Many databases answer with read locks, so reading can wait on writing. Oracle answers with **multiversion concurrency control**: when a query starts, it is assigned a point in time (an SCN, system change number), and whenever it meets a block that changed after that time it rebuilds the older version from **undo** data. Result: queries are consistent as of one instant, with no read locks.

This lesson covers transactions and ACID in Oracle, `COMMIT`/`ROLLBACK`/`SAVEPOINT`, the two isolation levels Oracle really supports (plus read-only), undo and read consistency, locks (TX/TM), autonomous transactions, DDL's implicit commit, the diagnostic views, and logical vs. physical I/O.

### 2. Core concepts and vocabulary
- **Transaction** — starts at the first DML (or `SET TRANSACTION`), ends with `COMMIT`, `ROLLBACK`, or DDL/session end (implicit commit/rollback).
- **ACID** — Atomicity (all or nothing, via undo), Consistency (constraints hold), Isolation (concurrent transactions don't see each other's uncommitted work), Durability (committed redo is flushed to the online redo log).
- **SCN** — monotonically increasing logical clock; every commit gets one; queries read "as of" an SCN.
- **Undo (rollback) segments** — store before-images of changed data; used for rollback, read consistency, and flashback.
- **Redo log** — change vectors written sequentially; basis of crash recovery and durability; `LGWR` flushes it on commit.
- **Read consistency** — statement-level (default) or transaction-level (`SERIALIZABLE`/`READ ONLY`).
- **TX lock** — transaction (row) enqueue; a session holds it while it has uncommitted changes; **TM lock** — table-level DML enqueue protecting the table definition from DDL.
- **Savepoint** — named point inside a transaction to roll back to.
- **Autonomous transaction** — PL/SQL block with its own independent transaction (`PRAGMA AUTONOMOUS_TRANSACTION`).
- **ORA-01555 "snapshot too old"** — undo needed to rebuild a consistent read was overwritten.

### 3. How it works, step by step
**What a query does**
1. Statement starts; Oracle notes the current SCN as the query SCN.
2. For each block read, if the block's SCN/ITL shows a change newer than the query SCN or by an uncommitted transaction, Oracle clones the block in the buffer cache and applies undo records until it matches the query SCN.
3. The query returns results consistent as of the start; changes committed during the query are invisible.

**What an update does**
1. Locate rows, lock each row (the lock is a flag in the row header pointing to the transaction's slot in the block's ITL — no lock manager list per row).
2. Write undo (before image) and redo (change) records.
3. Modify the block in the buffer cache.
4. If another session wants the same row, it waits on the TX enqueue until commit/rollback.
5. `COMMIT`: assign a commit SCN, write a commit record to the redo buffer, `LGWR` flushes it (the session waits in `log file sync`), locks are released lazily ("delayed block cleanout").

**Isolation levels**

| Level | Oracle support | Behaviour |
|---|---|---|
| READ UNCOMMITTED | Not offered | Never see dirty data |
| READ COMMITTED | **Default** | Statement-level consistency; non-repeatable reads and phantoms possible between statements |
| REPEATABLE READ | Not offered | — |
| SERIALIZABLE | Supported | Transaction-level snapshot; fails with `ORA-08177` if a row you want to change was changed after your snapshot |
| READ ONLY | Oracle extension | Transaction-level snapshot, no DML allowed |

Differences from the standard: Oracle never allows dirty reads; SERIALIZABLE is really *snapshot isolation* (write skew is possible, so it is not truly serializable).

**Savepoints:** `SAVEPOINT a; … ROLLBACK TO a;` undoes work after `a` but keeps the transaction open and earlier locks.

### 4. Worked example
```sql
-- Session 1
UPDATE accounts SET balance = balance - 100 WHERE id = 1;   -- TX lock on row id=1

-- Session 2 (READ COMMITTED)
SELECT balance FROM accounts WHERE id = 1;       -- not blocked: sees the old balance via undo
UPDATE accounts SET balance = balance + 50 WHERE id = 1;    -- blocks: waits for session 1

-- Session 1
COMMIT;                                          -- session 2's UPDATE now proceeds
```
Finding the blocker:
```sql
SELECT s.sid, s.serial#, s.username, s.blocking_session, s.event, s.seconds_in_wait
FROM   v$session s
WHERE  s.blocking_session IS NOT NULL;

-- or locks directly: a waiter has request > 0
SELECT sid, type, id1, id2, lmode, request, block
FROM   v$lock
WHERE  type IN ('TX','TM') AND (block = 1 OR request > 0);
```
Wait event `enq: TX - row lock contention` is a row waiting for another transaction; `enq: TM - contention` is usually an unindexed foreign key (a child table with no index on the FK column takes a table lock on the child when the parent is updated or deleted).

Savepoint inside a batch:
```sql
BEGIN
  FOR r IN (SELECT id FROM staging) LOOP
    SAVEPOINT row_start;
    BEGIN
      process_row(r.id);
    EXCEPTION WHEN OTHERS THEN
      ROLLBACK TO row_start;           -- only this row is undone
      log_error(r.id, SQLERRM);        -- logger is an AUTONOMOUS TRANSACTION so the log survives
    END;
  END LOOP;
  COMMIT;
END;
```
The logger procedure declares `PRAGMA AUTONOMOUS_TRANSACTION` and commits its own insert, so error log rows persist even if the main transaction is rolled back later. That is the legitimate use of autonomous transactions: logging/auditing. Using them to "work around" a mutating table or to commit part of business logic breaks atomicity.

### 5. Under the hood
- **Undo and read consistency.** `UNDO_RETENTION` (seconds) is the target for keeping *committed* undo; with `RETENTION GUARANTEE` Oracle never overwrites it (at the risk of failing DML when the undo tablespace fills). Long-running queries on busy tables raise ORA-01555 when undo is recycled too early; fixes: bigger undo tablespace/retention, shorter queries, fewer commits across a fetch loop (committing inside a loop that fetches from a cursor on the same table is the classic trigger), `Flashback`-friendly sizing.
- **ITL (interested transaction list).** Each block header has slots (`INITRANS`, `MAXTRANS`) that record transactions touching rows in the block; lack of free ITL slots produces `enq: TX - allocate ITL entry` waits on hot blocks (fix with a larger `INITRANS` or lower `PCTFREE` pressure).
- **Row lock = bit in the row header**, so there is no lock escalation in Oracle: locking a million rows takes no more lock memory than one; a table lock is never "promoted" from many row locks.
- **Delayed block cleanout.** After a commit the data blocks are not necessarily cleaned; the next reader cleans them (and may generate redo during a `SELECT`).
- **Redo and commit.** `COMMIT` waits for `LGWR` to write the redo; `COMMIT WRITE NOWAIT BATCH` trades durability for speed. "log file sync" waits reflect commit latency (slow storage for redo, or too many commits).
- **Deadlock.** Two sessions each holding a row the other wants. Oracle detects it quickly, rolls back **the statement** of one session (not the whole transaction) with `ORA-00060`, and writes a trace file with the graph; the application must retry/rollback.
- **DDL.** Oracle issues an implicit `COMMIT` before and after any DDL (`CREATE`, `ALTER`, `TRUNCATE`, `DROP`). DDL is not transactional (except certain online operations and edition-based redefinition), so a migration script cannot be rolled back as a unit: design migrations as small idempotent steps.
- **Logical vs. physical reads.** A *logical read* (consistent get/db block get) is a buffer-cache access, costing CPU and latches/mutexes; a *physical read* goes to disk (`db file sequential read` for single-block, `db file scattered read` for multiblock, `direct path read` for large scans bypassing the cache). Reducing logical I/O matters even when all hits are in cache because each consistent get costs CPU and contention scales with it.

### 6. Performance and design implications
- Commit when the business transaction ends: committing per row inflates `log file sync` and breaks atomicity; one huge commit risks undo exhaustion and long rollback. Typical compromise: commit every few thousand rows *or* better, set-based DML.
- Index foreign keys to avoid TM contention and full scans.
- Keep transactions short; no user think-time while holding locks. Use `SELECT … FOR UPDATE NOWAIT | WAIT n | SKIP LOCKED` to control blocking (queue processing uses `SKIP LOCKED`).
- Diagnose with AWR (hourly snapshots, history), ASH (`V$ACTIVE_SESSION_HISTORY`, per-second samples, best for "what happened at 10:03?"), `V$SESSION_WAIT`, `V$TRANSACTION` (undo blocks used), `V$SESSTAT` (`consistent gets`, `physical reads`).
- `SERIALIZABLE` is useful for consistent multi-statement reports and for detecting lost updates, but prepare to retry on `ORA-08177`.
- Connection pools: always close cursors/statements and end transactions before returning connections to the pool (leaks show as `ORA-01000: maximum open cursors exceeded` or sessions "idle in transaction" holding locks).

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `COMMIT` / `ROLLBACK` / `SAVEPOINT x` / `ROLLBACK TO x` | Transaction control | Always, explicitly |
| `SET TRANSACTION ISOLATION LEVEL SERIALIZABLE` / `READ ONLY` | Transaction-level consistency | Reports spanning statements |
| `SELECT … FOR UPDATE [NOWAIT|WAIT n|SKIP LOCKED]` | Take row locks on read | Pessimistic update, queues |
| `LOCK TABLE t IN … MODE` | Explicit table lock | Rare, maintenance |
| `V$SESSION`, `V$LOCK`, `DBA_WAITERS`, `DBA_BLOCKERS` | See blockers | Hung transactions |
| `V$TRANSACTION` | Active transactions, undo used | Long rollback estimation |
| `UNDO_RETENTION`, undo tablespace size | Control read-consistency window | ORA-01555 |
| `PRAGMA AUTONOMOUS_TRANSACTION` | Independent commit scope | Audit/log |
| `ALTER SYSTEM KILL SESSION 'sid,serial#'` | Terminate blocker | Last resort |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| READ COMMITTED (default) | Multi-statement logic assumes the data does not change between statements |
| SERIALIZABLE | High-contention hot rows (retries) or you believe it prevents write skew (it does not) |
| READ ONLY | You need DML in the same transaction |
| Optimistic concurrency (version column) | Rows are contended heavily; pessimistic `FOR UPDATE` may be simpler |
| Autonomous transaction for logging | Business operations needing atomicity with the caller |
| Frequent commits in batch loops | Fetching across commits from a cursor over changing data (ORA-01555) |

### 9. Common mistakes
- Believing readers block writers as in lock-based systems → misdesigned retries → understand MVCC.
- Commit inside a fetch loop → `ORA-01555` and broken atomicity → fetch into a collection first (`BULK COLLECT … LIMIT`) or restructure.
- Forgetting that DDL commits → half-applied migrations → script each step idempotently.
- Unindexed FKs → TM locks and deadlocks → index them.
- Treating `SERIALIZABLE` as truly serializable → write skew → use constraints/locking.
- Long transactions with idle clients → blocked sessions → timeouts, `idle_time` profile limits.
- Killing sessions without checking rollback time → long "KILLED" state → look at `V$TRANSACTION.USED_UBLK`.

### 10. 🎯 Interview kit
**30-second answer:** "Oracle uses multiversion read consistency: each query sees data as of its start SCN by rebuilding old versions from undo, so readers don't block writers and vice versa. Writers lock only the rows they change. The default level is READ COMMITTED; SERIALIZABLE gives transaction-level snapshots and READ ONLY forbids DML. Uncommitted data is never visible, so no dirty reads."

**Structure a longer answer:** ACID refresher → how a SELECT builds a consistent image from undo → how UPDATE locks a row and what a waiter waits on → isolation levels vs. the SQL standard → monitoring blocking and ORA-01555 → practical advice (short transactions, index FKs).

**Likely follow-ups:**
- *Row vs. table lock; does Oracle escalate?* → TX row-level, TM table-level for DDL protection; no escalation.
- *What is `ORA-01555` and how do you fix it?* → undo overwritten; size undo, avoid cross-commit fetches, tune the query.
- *Autonomous transaction?* → independent commit scope; logging use-case.
- *Is DDL transactional?* → no, implicit commits.
- *How to find a blocking session?* → `V$SESSION.BLOCKING_SESSION`, `V$LOCK`, `DBA_BLOCKERS`.
- *`log file sync`?* → commit waits for LGWR; reduce commits, faster redo devices, batch.
- *How does Oracle handle deadlock?* → detects, rolls back one statement, writes trace.

**Signals interviewers look for:** you explain MVCC in terms of undo and SCN, you know there is no lock escalation, and you can name concrete diagnostic views.

### 11. Check yourself
- [ ] Explain how a long query still sees a consistent snapshot while others commit.
- [ ] State which isolation levels Oracle supports and how they differ from the standard.
- [ ] Reproduce a row-lock wait with two sessions and find the blocker.
- [ ] Explain why a missing FK index can block unrelated DML.
- [ ] Describe what happens during `COMMIT` and what `log file sync` measures.
- [ ] Explain logical vs. physical reads and what `direct path read` means.
<!-- explainer:end -->

## Implementations (tools)

- [Oracle](<../_index.md>)

## Practice questions

- [Basic] [What is a transaction in Oracle and what are the ACID properties?](<../../../PRACTICE/TOOLS/Oracle/Oracle_Platform/Q-35254_What_is_a_transaction_in_Oracle_and_what_are_the_ACID_proper.md>)
- [Intermediate] [What is a SAVEPOINT in Oracle and how is it used within a transaction?](<../../../PRACTICE/TOOLS/Oracle/Oracle_Platform/Q-35256_What_is_a_SAVEPOINT_in_Oracle_and_how_is_it_used_within_a_tr.md>)
- [Deep] [What are the different isolation levels Oracle supports and how does Oracle differ from the SQL standard on this?](<../../../PRACTICE/TOOLS/Oracle/Oracle_Platform/Q-35134_What_are_the_different_isolation_levels_Oracle_supports_and_.md>)
- [Deep] [What is the role of undo segments in Oracle and how do they enable read consistency and rollback?](<../../../PRACTICE/TOOLS/Oracle/Oracle_Platform/Q-35136_What_is_the_role_of_undo_segments_in_Oracle_and_how_do_they_.md>)
- [Deep] [What is a row-level lock versus a table-level lock in Oracle and when does Oracle escalate locking?](<../../../PRACTICE/TOOLS/Oracle/Oracle_Platform/Q-35142_What_is_a_row-level_lock_versus_a_table-level_lock_in_Oracle.md>)
- [Deep] [What is an autonomous transaction in Oracle PL/SQL and what is a legitimate use case such as logging inside a failing transaction?](<../../../PRACTICE/TOOLS/Oracle/Oracle_Platform/Q-35146_What_is_an_autonomous_transaction_in_Oracle_PL_SQL_and_what_.md>)
- [Deep] [What is the difference between DDL that is transactional and DDL that implicitly commits in Oracle and why does that matter for migrations?](<../../../PRACTICE/TOOLS/Oracle/Oracle_Platform/Q-35173_What_is_the_difference_between_DDL_that_is_transactional_and.md>)
- [Deep] [What is the difference between logical reads and physical reads in Oracle and why can reducing logical reads matter even when they hit the buffer cache?](<../../../PRACTICE/TOOLS/Oracle/Oracle_Platform/Q-35209_What_is_the_difference_between_logical_reads_and_physical_re.md>)

## Related concepts

- [PL-SQL in Production](<../PL-SQL/PL-SQL_in_Production.md>) *(PL-SQL)*
- [Transaction Control and Prepared Transactions](<../../../01_Databases_and_SQL/SQL_Language/SQL_Command_Types/Transaction_Control_and_Prepared_Transactions.md>) *(SQL Command Types)*
- [Lock Holding and Optimistic Concurrency](<../../../01_Databases_and_SQL/Transactions_and_Concurrency/ACID_Transactions/Lock_Holding_and_Optimistic_Concurrency.md>) *(ACID Transactions)*

