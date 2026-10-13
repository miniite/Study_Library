---
id: "01_Databases_and_SQL/Database_Objects/Cursors/Ref_Cursor_Consistency_and_Limits"
level: 4
type: "concept"
topic: "01_Databases_and_SQL/Database_Objects/Cursors"
tools: ["Oracle"]
---

# Ref Cursor Consistency and Limits

**L4 Advanced** · [↑ Cursors](<_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** What a cursor *sees* — a frozen snapshot, live changes, or a mix — and what it *costs* (tempdb, locks, open transaction, open-cursor limits) is decided by its type, concurrency option, the isolation level and whether it survives COMMIT; ref cursors add ownership and lifetime concerns on top.

### 1. The big picture
Opening a cursor does not necessarily copy the data. Between OPEN and the last FETCH, other sessions may insert, update or delete rows the cursor has not reached yet. Databases therefore define **cursor sensitivity** (does it reflect those changes?), **concurrency** (can I update through it, and how is a conflict detected?) and **scope/lifetime** (until when does it exist?).

Two broad implementation families exist:
- **Snapshot/MVCC-based** (Oracle, PostgreSQL, SQL Server with RCSI/snapshot): the cursor sees data as of its open time (or statement start), so it is *insensitive* to later changes; the price is that old row versions must be retained (undo / version store / dead tuples) for as long as the cursor stays open.
- **Locking- or worktable-based** (SQL Server default): a STATIC cursor copies rows to tempdb, a KEYSET copies keys, a DYNAMIC cursor re-reads live data under locks — each trading consistency against cost.

This lesson explains those behaviours and the operational limits (leaks, transaction bloat, `ORA-01000`, tempdb pressure, orphaned cursors) that show up in production.

### 2. Core concepts and vocabulary
- **Sensitivity** — whether changes made after OPEN are visible: *insensitive/static*, *keyset-sensitive*, *sensitive/dynamic*.
- **Concurrency model (SQL Server)** — `READ_ONLY`, `SCROLL_LOCKS` (pessimistic: lock fetched row), `OPTIMISTIC` (no lock; verify at update using `ROWVERSION` or column values).
- **Holdable cursor (`WITH HOLD`)** — survives `COMMIT`; non-holdable cursors close at commit.
- **Updatable cursor** — `FOR UPDATE [OF cols]` + `UPDATE … WHERE CURRENT OF cursor`; limited to a single base table and listed columns.
- **Scrollable cursor** — supports `FETCH ABSOLUTE/RELATIVE/PRIOR`, which needs a stable ordered set.
- **Read-consistent snapshot** — the cursor's view as of its SCN/snapshot/transaction start.
- **Version store / undo / dead tuples** — retained old row versions that long-lived cursors force the engine to keep.
- **Open-cursor limit** — `OPEN_CURSORS` (Oracle `ORA-01000`), memory per session in others.
- **Cursor leak** — cursors opened but never closed (often in application code).
- **Session cursor cache** — Oracle's cache of closed cursors for reuse; makes `v$open_cursor` look "high" without a leak.
- **Orphaned cursor** — server cursor left after client disconnect/abort.

### 3. How it works, step by step
**What the cursor sees — by engine/isolation**
1. *Oracle / PostgreSQL READ COMMITTED:* the query result as of cursor open (Oracle: statement start; PostgreSQL: statement start for the portal). Later commits are invisible to it — even for rows not yet fetched. This differs from SQL Server's default.
2. *SQL Server, default READ COMMITTED (locking):* a **dynamic** cursor fetches the *current* committed row each time (sees new commits); a **static** cursor shows the OPEN-time copy; a **keyset** cursor shows the rows chosen at OPEN, with current values for non-key columns, "holes" for deleted rows, and no new inserts.
3. *SQL Server with `READ_COMMITTED_SNAPSHOT`/`SNAPSHOT`:* reads come from the version store; dynamic cursors are consistent per statement.
4. *SERIALIZABLE/REPEATABLE READ:* shared locks held until commit — the cursor blocks writers on every row it has touched (and ranges, for SERIALIZABLE).

| Cursor type (SQL Server) | Where data lives | Sees updates | Sees inserts | Cost |
|---|---|---|---|---|
| STATIC | tempdb worktable (snapshot) | no | no | up-front copy, tempdb I/O |
| KEYSET | tempdb keyset table + live rows | yes (non-key) | no | medium; deletes → fetch status −2 |
| DYNAMIC | live base tables | yes | yes | no snapshot but heaviest locking; no `ABSOLUTE` |
| FAST_FORWARD | engine-optimised read-only forward | per isolation | per isolation | cheapest |

**Updating through a cursor (`WHERE CURRENT OF`)**
1. FETCH positions the cursor on a row.
2. `UPDATE t SET … WHERE CURRENT OF c` targets that row's identity.
3. Under *optimistic* concurrency the engine re-checks the row (ROWVERSION or value comparison) and fails if someone else changed it; under *scroll locks* the row was already locked at FETCH; under `FOR UPDATE` in Oracle/PostgreSQL the rows are locked at open time (`FOR UPDATE`) and other writers wait.

### 4. Worked example
```sql
-- Session A (SQL Server, default READ COMMITTED)
DECLARE c CURSOR LOCAL KEYSET FOR
    SELECT AccountID, Balance FROM dbo.Acct ORDER BY AccountID;
OPEN c;
FETCH NEXT FROM c;          -- fetches account 1

-- Session B (meanwhile)
UPDATE dbo.Acct SET Balance = 999 WHERE AccountID = 2;   -- committed
DELETE FROM dbo.Acct WHERE AccountID = 3;
INSERT dbo.Acct VALUES (100, 50);

-- Session A continues
FETCH NEXT FROM c;   -- acct 2 with Balance = 999 (non-key column is live)
FETCH NEXT FROM c;   -- @@FETCH_STATUS = -2 : row 3 is "missing" (hole), NOT silently skipped
-- account 100 is never returned (keyset fixed at OPEN)
```
Compare: declare the same cursor `STATIC` and Session A sees the original values, original row 3, no row 100; declare it `DYNAMIC` and it sees 999, no row 3, and row 100. In PostgreSQL/Oracle, the same cursor behaves like STATIC (snapshot) regardless of what B does.

Detecting problems live:
```sql
-- SQL Server: who has cursors open, how old, how big
SELECT session_id, name, cursor_scope, status, is_open, fetch_status, reads, writes, creation_time
FROM sys.dm_exec_cursors(0);

-- Oracle: leak vs cache
SELECT sid, COUNT(*) open_cursors FROM v$open_cursor GROUP BY sid ORDER BY 2 DESC;
SELECT value FROM v$parameter WHERE name = 'open_cursors';

-- PostgreSQL
SELECT name, is_holdable, is_scrollable, creation_time FROM pg_cursors;
```

### 5. Under the hood
- **Snapshots need history.** Oracle satisfies a long-open cursor from undo; if the needed undo is overwritten you get `ORA-01555 snapshot too old`. PostgreSQL keeps old tuples (blocking `VACUUM` cleanup, causing bloat) for any transaction/cursor that is still open. SQL Server with row versioning keeps versions in the version store (tempdb or ADR) until the oldest snapshot ends. This is the "cursor holding a transaction open bloats undo/version store" interview point: it isn't the cursor's own memory, it's the retention horizon it pins.
- **Why scrollable cursors need stability.** `FETCH ABSOLUTE 500` must mean the same row twice. The engine guarantees this either by materialising a copy (STATIC worktable, PostgreSQL `SCROLL` cursors that materialise the result) or by keeping a keyset. A dynamic cursor can't support absolute positioning because the set changes under it. Hidden cost: the snapshot is built in tempdb *at OPEN* even if you fetch ten rows.
- **`WHERE CURRENT OF` restrictions.** The cursor query must be updatable: one base table (no `DISTINCT`, `GROUP BY`, aggregates, `UNION`, usually no joins); only columns in `FOR UPDATE OF` can be changed; in SQL Server a cursor over a table without a unique index is demoted to STATIC/read-only. It can deadlock with a concurrent set-based `UPDATE` on the same table because the cursor holds update locks in key order while the set statement acquires them in plan order.
- **Lifetime and scope.** Non-holdable cursors close at COMMIT/ROLLBACK (PL/pgSQL `COMMIT` inside a loop is allowed only in procedures and closes plain cursors unless `WITH HOLD`; a holdable cursor materialises its result at commit time, which can be expensive). T-SQL cursors are `LOCAL` (batch/procedure scope) or `GLOBAL` (connection scope — easy to leak and collide with "cursor already exists"). Closing a connection closes its cursors, but with **connection pooling** the physical session persists, so un-closed cursors and open transactions can survive logical close — a classic leak and a source of "inconsistent" data.
- **Ref cursors.** The server cursor lives in the session; passing it between routines passes a *handle*. Returned to a client via OUT parameter, the client owns fetching and closing. Mismatch (closing too early, never closing, or returning a cursor whose underlying transaction already ended) causes errors or leaks. A ref cursor over a snapshot is bounded by the same undo/retention horizon as any cursor.
- **Batch commits.** Committing every N rows inside a cursor loop causes: `ORA-01555`/`ORA-01002` (Oracle), loss of a non-holdable cursor (PostgreSQL), partial results after failure (non-atomic job), index/lock churn, and a restart problem. Safe pattern: iterate by key ranges (`WHERE id > @last ORDER BY id`, `TOP (n)`), commit per chunk, store a checkpoint so a failure at row 800,000 resumes there.
- **Leak vs cache (Oracle).** `v$open_cursor` includes cursors cached by the session cursor cache (`session_cached_cursors`) — closed from the application's view but kept for soft-parse reuse. Distinguish a leak by `cursor_type` (`OPEN` vs `SESSION CURSOR CACHED`) and by watching whether the count grows without bound for a given SQL text/session; `ORA-01000` means the *process* limit `open_cursors` was hit, usually from application code that doesn't close `ResultSet`/`Statement` (fix with try-with-resources / `using`).

### 6. Performance and design implications
- Prefer **forward-only, read-only, local** cursors and keep them open for the shortest possible time; don't interleave with slow application work or user think-time.
- If you only need a stable list of keys, copy them first (`SELECT id INTO #work`) and loop on the temp table; then the source cursor isn't pinning anything.
- Use row-versioning isolation (RCSI) so cursors don't take shared locks that block writers — but budget tempdb/undo.
- Make loops **restartable** (checkpoint table or status column) and **chunked** (key-range batches) rather than relying on cursor position.
- Avoid `DYNAMIC` cursors: heavy tempdb/worktable use and unstable positioning; switch to STATIC (if you need snapshot semantic) or FAST_FORWARD.
- In app code: close in `finally`/`using`; set statement/command timeouts; use idle-in-transaction timeouts (`idle_in_transaction_session_timeout` PG; `kill` orphan sessions in SQL Server) as a safety net.
- Monitor: open-cursor counts, longest open transactions (`sys.dm_tran_active_transactions`, `pg_stat_activity.xact_start`, `v$transaction`), version store size, undo retention, blocking chains.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `DECLARE … CURSOR STATIC/KEYSET/DYNAMIC/FAST_FORWARD` | Choose sensitivity/cost | Match semantics to need |
| `READ_ONLY / SCROLL_LOCKS / OPTIMISTIC` | Concurrency model | Updatable cursors only |
| `FOR UPDATE [OF cols]` + `WHERE CURRENT OF` | Updatable positioned update | Rare row-wise correction |
| `DECLARE … WITH HOLD` (PG) | Cursor survives commit | Chunked commit loops |
| `SET TRANSACTION ISOLATION LEVEL …`, `READ_COMMITTED_SNAPSHOT` | Visibility and locking | Control what cursor sees/blocks |
| `sys.dm_exec_cursors`, `v$open_cursor`, `pg_cursors` | List open cursors | Leak / orphan hunt |
| `open_cursors`, `session_cached_cursors` (Oracle) | Limits and cache | ORA-01000, tuning |
| `idle_in_transaction_session_timeout` (PG), `lock_timeout` | Auto-clean stuck sessions | Safety net |
| `undo_retention` / version store sizing | Retain history for long reads | Avoid snapshot-too-old |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| **STATIC / snapshot cursor** — need stable, repeatable scrolling or consistent report | Result is huge (tempdb copy) or must see fresh changes |
| **KEYSET** — large set, want updates visible but stable membership | Many deletes (holes) or you need new rows |
| **DYNAMIC** — must reflect all live changes | Performance matters; no absolute positioning; heavy locks |
| **FAST_FORWARD** — single pass, read-only | You must scroll or update through the cursor |
| **WITH HOLD / chunked commit** — long batch needing commits | Strict atomicity; very large held result materialised at commit |
| **Key-range batching** — restartable jobs, bounded transaction size | Keys aren't unique/ordered; gaps need care |

### 9. Common mistakes
- **Assuming a cursor sees committed changes made after OPEN (or doesn't)** → wrong report numbers → know sensitivity per engine/isolation.
- **Cursor open across user interaction/slow API calls** → long transaction, undo/version bloat, blocking → fetch, close, then process.
- **Commit inside `FOR UPDATE` cursor loop** → `ORA-01002`/closed cursor → chunk by key.
- **Never closing in exception paths** → leaked cursors, `ORA-01000`, pool poisoning → `finally`/`using`/try-with-resources.
- **`GLOBAL` cursors in shared code** → name collisions, leaks → `LOCAL`.
- **Diagnosing every high `v$open_cursor` as a leak** → ignoring the session cursor cache → check `cursor_type` and growth.
- **Using `WHERE CURRENT OF` on a join/aggregate** → non-updatable or deadlocks with set updates → update by key.
- **Scrollable cursor "to be safe"** → tempdb snapshot cost → forward-only.
- **Ignoring orphaned cursors after client crash** → locks held → idle-in-transaction/kill policy.

### 10. 🎯 Interview kit
**30-second answer:** "A cursor's behaviour depends on its sensitivity and the isolation level. In Oracle/PostgreSQL it reads a snapshot from when it opened, so later commits are invisible but old versions must be kept; in SQL Server a static cursor copies to tempdb, keyset fixes keys and sees updates, dynamic sees everything. A cursor held open pins undo/version history and may hold locks, so I keep them short-lived, forward-only, closed in `finally`, and chunk long jobs by key with checkpoints."

**Structure a longer answer:** define consistency needs → cursor type/isolation → cost (tempdb, undo, locks) → lifecycle and ownership (commit, holdable, ref cursor, pooling) → monitoring/limits → mitigation pattern (key-range batching, restartability).

**Likely follow-ups:**
- *Behaviour under READ COMMITTED if another transaction commits rows not yet fetched?* → Snapshot engines: not visible; SQL Server dynamic: visible; static: not.
- *Why valid only within the transaction?* → Non-holdable cursors are tied to the transaction's snapshot/locks and close at commit; `WITH HOLD` changes that.
- *Keyset cursor semantics?* → fixed keys, live non-key columns, deleted rows appear as holes, no new inserts.
- *When is a scrollable STATIC cursor justified?* → bounded results needing back-and-forth navigation or repeatable multi-pass over a stable set; cost is the tempdb copy at open.
- *How to tell I/O-bound vs CPU-bound?* → compare reads/waits vs CPU time per row (`sys.dm_exec_cursors` reads, wait stats, `STATISTICS IO/TIME`, profiling per iteration).
- *Runaway cursor blocking users — what do you do live?* → find head blocker, its cursor/SQL, kill session, then fix with chunking/isolation.
- *Cursor leak in .NET/Java pooled app?* → ensure close in `finally`, `using`; check pool; look at open cursors per session.

**Signals interviewers look for:** you tie semantics to isolation/MVCC, know the "holds history/locks" cost, handle lifecycle (commit, pooling, ownership), and propose restartable chunked patterns.

### 11. Check yourself
- [ ] Predict what each cursor type returns after another session updates, deletes and inserts rows.
- [ ] Explain why a long-open cursor can cause `ORA-01555` or table bloat.
- [ ] Rewrite a "commit every 500 rows" cursor as a restartable key-range loop.
- [ ] List the restrictions of `WHERE CURRENT OF` and how it can deadlock a set update.
- [ ] Describe how to tell a real cursor leak from the session cursor cache.
- [ ] State who must close a ref cursor returned from a procedure.
<!-- explainer:end -->

## Implementations (tools)

- [Oracle](<../../../TOOLS/Oracle/_index.md>)

## Practice questions

- [Intermediate] [What happens to database resources if a cursor is opened but never closed?](<../../../PRACTICE/01_Databases_and_SQL/Database_Objects/Cursors/Q-10512_What_happens_to_database_resources_if_a_cursor_is_opened_but.md>)
- [Deep] [What kind of lock can an updatable cursor hold on rows while it is open?](<../../../PRACTICE/01_Databases_and_SQL/Database_Objects/Cursors/Q-10462_What_kind_of_lock_can_an_updatable_cursor_hold_on_rows_while.md>)
- [Deep] [How can a cursor leak happen and how would you detect too many open cursors?](<../../../PRACTICE/01_Databases_and_SQL/Database_Objects/Cursors/Q-10524_How_can_a_cursor_leak_happen_and_how_would_you_detect_too_ma.md>)
- [Deep] [Walk through how you would add restartable checkpointing to a cursor-driven ETL so a failure at row 800k does not reprocess from zero.](<../../../PRACTICE/01_Databases_and_SQL/Database_Objects/Cursors/Q-10356_Walk_through_how_you_would_add_restartable_checkpointing_to_.md>)
- [Deep] [You have a cursor that must see rows inserted by its own transaction after it opened — which cursor sensitivity setting supports that and what is the cost?](<../../../PRACTICE/01_Databases_and_SQL/Database_Objects/Cursors/Q-10383_You_have_a_cursor_that_must_see_rows_inserted_by_its_own_tra.md>)
- [Deep] [Explain why fetching into a %ROWTYPE record from a cursor with a joined query can be brittle when the base table schema evolves.](<../../../PRACTICE/01_Databases_and_SQL/Database_Objects/Cursors/Q-10399_Explain_why_fetching_into_a_ROWTYPE_record_from_a_cursor_wit.md>)
- [Deep] [How does a cursor's FOR UPDATE NOWAIT versus FOR UPDATE WAIT n change behavior when another session holds the row locks?](<../../../PRACTICE/01_Databases_and_SQL/Database_Objects/Cursors/Q-10428_How_does_a_cursor_s_FOR_UPDATE_NOWAIT_versus_FOR_UPDATE_WAIT.md>)
- [Expert] [Your monitoring shows v$open_cursor climbing steadily over hours — how do you distinguish a genuine leak from Oracle's session cursor cache?](<../../../PRACTICE/01_Databases_and_SQL/Database_Objects/Cursors/Q-10378_Your_monitoring_shows_v_open_cursor_climbing_steadily_over_h.md>)

## Related concepts

- [Large Batch Updates](<../../SQL_Language/SQL_Fundamentals/Large_Batch_Updates.md>) *(SQL Fundamentals)*
- [SKIP LOCKED Job Queues](<../../SQL_Language/SQL_Fundamentals/SKIP_LOCKED_Job_Queues.md>) *(SQL Fundamentals)*
- [Lock Granularity](<../../Database_Internals/DBMS_Concepts/Lock_Granularity.md>) *(DBMS Concepts)*
- [Stored Procedure Performance and Design](<../Stored_Procedures/Stored_Procedure_Performance_and_Design.md>) *(Stored Procedures)*
- [PL-SQL in Production](<../../../TOOLS/Oracle/PL-SQL/PL-SQL_in_Production.md>) *(PL-SQL)*

## References

- *SQL Server 2025 Query Performance Tuning (7th ed.)* — Grant Fritchey — FULL_BOOK_NEEDS_MANUAL_SPLIT (cont.) › Chapter 18 Row-By-Row Processing From Cursors And Other Causes

