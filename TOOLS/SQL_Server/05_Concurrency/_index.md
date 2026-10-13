---
id: "TOOLS/SQL_Server/05_Concurrency"
level: 3
type: "curriculum"
curriculum: "sql_server"
---

# Concurrency, Locking and Blocking

**L3 Practitioner** · [↑ SQL Server](<../_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** Concurrency control lets many sessions read and change the same data at once without corrupting it. SQL Server does this with locks (pessimistic) and row versions (optimistic), and the price is blocking, deadlocks and tempdb/version-store overhead, which you manage through transaction design and isolation levels.

### 1. The big picture
A database with one user needs no concurrency control. With hundreds of sessions, two things must hold at the same time: **consistency** (each transaction sees and leaves data in a valid state, the ACID "I": isolation) and **throughput** (sessions shouldn't wait on each other more than necessary). These goals pull against each other. Stronger isolation means more waiting, and weaker isolation means more anomalies.

SQL Server offers two families of mechanisms:
- **Pessimistic (locking)**: before reading or writing a resource, a session takes a *lock*. Incompatible requests wait. This is the default (`READ COMMITTED` with locking).
- **Optimistic (row versioning)**: writers keep the previous version of changed rows in the **version store** (tempdb, or the database itself with Accelerated Database Recovery). Readers get a consistent snapshot without taking shared locks (`READ_COMMITTED_SNAPSHOT`, `SNAPSHOT`).

Below the transaction level, **latches** protect in-memory structures (pages, buffers) for microseconds. They aren't part of isolation, but they cause their own contention.

This block's child lessons go deep on each piece:

| Child lesson | Question it answers |
|---|---|
| *Lock Modes and Granularity* | what gets locked, in which mode, and what is compatible |
| *Lock Escalation* | when thousands of row locks become one table lock |
| *Isolation and Row Versioning* | which anomalies each level allows; RCSI vs SNAPSHOT |
| *Blocking Analysis* | how to find the head blocker and why it holds locks |
| *Deadlocks* | cycles of waiting, the deadlock monitor, graphs and fixes |
| *Latches* | PAGELATCH / PAGEIOLATCH contention and hot pages |
| *Optimized Locking* | TID locking and lock-after-qualification (2025 / Azure) |

### 2. Core concepts and vocabulary
- **Transaction**: a unit of work that commits or rolls back as a whole. Locks taken for writes are held until it ends.
- **Lock**: a logical reservation on a resource (row/key, page, table, …) in a **mode** (S, U, X, IS, IX, Sch-S, Sch-M, …).
- **Blocking**: session B waits because session A holds an incompatible lock. This is *normal* and short-lived, and becomes a problem when it lasts.
- **Head blocker**: the session at the start of a blocking chain. It is usually not waiting on anything itself, often an open transaction that is idle or slow.
- **Deadlock**: a cycle (A waits for B, B waits for A). Neither can proceed, so SQL Server kills one (*the victim*, error 1205).
- **Isolation level**: how much a transaction is protected from other transactions' changes: Read Uncommitted, Read Committed (locking or snapshot), Repeatable Read, Serializable, Snapshot.
- **Concurrency anomalies**: dirty reads, non-repeatable reads, phantoms, lost updates.
- **Version store**: old row versions kept for snapshot readers and online operations.
- **Latch**: a lightweight, short-term synchronization object on in-memory structures. It isn't held for the transaction.

### 3. How it works, step by step
A typical update under default `READ COMMITTED`:
1. `BEGIN TRAN; UPDATE Orders SET Status = 'Shipped' WHERE OrderID = 42;`
2. The engine takes **intent locks** top-down: `IX` on the table, `IX` on the page. This announces "I'll lock something inside."
3. It finds the row with an **update (U)** lock while searching, then converts it to **exclusive (X)** on the key it changes.
4. Another session runs `SELECT ... WHERE OrderID = 42`. Under locking read committed it requests **S** on that key, which is incompatible with X, so **it waits** (`LCK_M_S`).
5. The first session commits. Its X lock is released, and the reader gets S, reads the committed value, and releases S right after reading the row.
6. If the database had `READ_COMMITTED_SNAPSHOT ON`, step 4 would not wait. The reader would get the last committed version from the version store.
7. If the first session also wanted a row the reader held, and vice versa, the **deadlock monitor** (running every ~5 seconds, more often when deadlocks are frequent) detects the cycle and rolls back the cheaper victim.

### 4. Worked example
```sql
-- Session 1
BEGIN TRAN;
UPDATE Sales.SalesOrderHeader SET Comment = 'hold' WHERE SalesOrderID = 43659;
-- (do not commit yet)

-- Session 2
SELECT Comment FROM Sales.SalesOrderHeader WHERE SalesOrderID = 43659;  -- waits

-- Session 3: who is blocking whom?
SELECT r.session_id, r.blocking_session_id, r.wait_type, r.wait_time, t.text
FROM sys.dm_exec_requests AS r
CROSS APPLY sys.dm_exec_sql_text(r.sql_handle) AS t
WHERE r.blocking_session_id <> 0;

SELECT request_session_id, resource_type, request_mode, request_status
FROM sys.dm_tran_locks
WHERE resource_database_id = DB_ID();
-- Session 1: X on KEY (GRANT), IX on PAGE and OBJECT
-- Session 2: S on the same KEY (WAIT)

-- Session 1
COMMIT;   -- session 2 returns immediately
```
Repeat after `ALTER DATABASE AdventureWorks SET READ_COMMITTED_SNAPSHOT ON WITH ROLLBACK IMMEDIATE;`. Session 2 now returns the *old* committed value at once, without waiting.

### 5. Under the hood   *(overview; details in the child lessons)*
- **Lock manager.** Locks live in memory in a hash table of lock resources. Each costs about 96 bytes (more for granted lists). Memory pressure and very large lock counts trigger **escalation** (about 5,000 locks on one object per statement).
- **Lock hierarchy.** Database → table (object) → partition (HoBT, if enabled) → page → row/key. Intent locks on higher levels make table-level checks cheap.
- **Key-range locks** under `SERIALIZABLE` lock gaps between keys to prevent phantoms.
- **Row versioning** adds a 14-byte version tag to modified rows and copies prior versions to the version store. Long-running snapshot transactions keep versions alive and grow tempdb.
- **Lock timeouts.** By default sessions wait forever (`LOCK_TIMEOUT -1`). Applications usually impose their own command timeouts, which cancel the statement but **don't roll back an open transaction**. That's a classic source of orphaned locks.

### 6. Performance and design implications
- **Short transactions** are the biggest single lever. Never wait for user input, network calls or file I/O inside a transaction.
- **Good indexes reduce locking.** A scan to find one row locks (and blocks on) many rows; a seek locks only the target.
- **Access objects in a consistent order** across code paths to avoid deadlocks.
- **RCSI** removes most reader/writer blocking for OLTP, at the cost of tempdb version-store use and slightly more write overhead.
- **Monitor** wait stats (`LCK_M_*`), the *Lock Waits/sec* and *Number of Deadlocks/sec* counters, the blocked process report (`blocked process threshold`), and the `system_health` session (which captures deadlocks automatically).

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `SET TRANSACTION ISOLATION LEVEL ...` | sets the session's isolation level | per-session needs |
| `ALTER DATABASE ... SET READ_COMMITTED_SNAPSHOT ON` | makes READ COMMITTED use row versions | reduce reader/writer blocking |
| `ALTER DATABASE ... SET ALLOW_SNAPSHOT_ISOLATION ON` | permits `SNAPSHOT` level | consistent multi-statement reads |
| `SET LOCK_TIMEOUT ms` | max time to wait for a lock | fail fast instead of hanging |
| `SET DEADLOCK_PRIORITY LOW/HIGH/n` | influences victim choice | protect important work |
| `sys.dm_tran_locks`, `sys.dm_exec_requests`, `sys.dm_os_waiting_tasks` | current locks and waits | live diagnosis |
| `sp_configure 'blocked process threshold (s)'` + XE | reports blocking longer than *n* seconds | ongoing monitoring |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| Locking READ COMMITTED: default, low tempdb budget, code that relies on blocking semantics | heavy read/write mix with reporting queries |
| RCSI: OLTP with readers blocked by writers | code that assumes a read sees in-flight state, or uses reads to "reserve" rows |
| SNAPSHOT: reports needing one consistent view across statements | write-heavy transactions (update conflicts, error 3960) |
| SERIALIZABLE: strict invariants (no phantoms) | high concurrency; range locks cause blocking and deadlocks |
| `NOLOCK` / READ UNCOMMITTED: almost never | anything that needs correct results (dirty, missing, duplicate rows) |

### 9. Common mistakes
- Using `NOLOCK` everywhere "for speed" → wrong results and even error 601 → use RCSI instead.
- Long transactions that span user interaction → long blocking chains → keep transactions short and server-side.
- Killing the blocked session instead of the head blocker → nothing improves → trace `blocking_session_id` up to the root.
- Application command timeouts without `XACT_ABORT ON` → open transactions keep their locks → `SET XACT_ABORT ON` and proper error handling.
- Turning on RCSI without sizing tempdb → version store growth → monitor `sys.dm_tran_version_store_space_usage`.

### 10. 🎯 Interview kit
**30-second answer:** SQL Server controls concurrency with locks and with row versioning. Locks are taken at row/key, page or table level in modes like shared, update and exclusive, with intent locks on parents. Incompatible requests block until the holder's transaction ends. Isolation levels decide how long shared locks are held and whether readers use versions instead. RCSI and SNAPSHOT let readers see committed versions from the version store, so readers and writers stop blocking each other. Problems show up as blocking chains, deadlocks, escalation and latch contention. The main fixes are short transactions, good indexes, consistent access order, and choosing the right isolation level.
**Structure a longer answer:** goals (consistency vs throughput) → pessimistic vs optimistic → lock modes and hierarchy → isolation levels and anomalies → failure modes (blocking, deadlocks, escalation) → diagnosis tools → design fixes → a scenario.
**Likely follow-ups:** *Difference between blocking and deadlock?* → blocking resolves when the holder finishes; a deadlock is a cycle that never resolves without a victim. · *Why not just use NOLOCK?* → dirty reads, skipped/duplicate rows during page splits. · *What does RCSI cost?* → tempdb version store, 14 bytes per row, longer version chains for long readers.
**Signals interviewers look for:** you separate locks from latches, know RCSI vs SNAPSHOT, find head blockers, and design transactions to be short.

### 11. Check yourself
- [ ] Create blocking between two sessions and find it with `sys.dm_exec_requests` and `sys.dm_tran_locks`.
- [ ] Explain intent locks and why they exist.
- [ ] List the four anomalies and which isolation levels prevent each.
- [ ] Turn on RCSI in a test database and show that the reader no longer waits.
- [ ] Explain why a command timeout can leave locks held.
<!-- explainer:end -->

## In this section (study in order)

1. [Lock Modes and Granularity](<Lock_Modes_and_Granularity/_index.md>) (L4 Advanced)
2. [Lock Escalation](<Lock_Escalation/_index.md>) (L4 Advanced)
3. [Latches and Latch Contention](<Latches/_index.md>) (L4 Advanced)
4. [Blocking Analysis](<Blocking_Analysis/_index.md>) (L4 Advanced)
5. [Deadlocks](<Deadlocks/_index.md>) (L4 Advanced)
6. [Isolation Levels and Row Versioning](<Isolation_and_Row_Versioning/_index.md>) (L4 Advanced)
7. [Optimized Locking (2025)](<Optimized_Locking/_index.md>) (L4 Advanced)

## References

- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 10: Transactions, Concurrency, and Isolation Levels › Deadlocks and Blocking
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 16: Best Practices for T-SQL Development and Database Design › Concurrency and Locking
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 10: Transactions, Concurrency, and Isolation Levels › Key Takeaways
- *T-SQL Fundamentals (4th ed.)* — Itzik Ben-Gan — Transactions and concurrency
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 16: Best Practices for T-SQL Development and Database Design › Deadlocks and Blocking
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 10: Transactions, Concurrency, and Isolation Levels › Combining Locks with Isolation Levels
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 10: Transactions, Concurrency, and Isolation Levels › Combining Isolation Levels and Locking Strategies
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 10: Transactions, Concurrency, and Isolation Levels › Best Practices
- *SQL Server 2025 Query Performance Tuning (7th ed.)* — Grant Fritchey — FULL_BOOK_NEEDS_MANUAL_SPLIT (cont.) › Chapter 16 Blocking And Blocked Processes
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 10: Transactions, Concurrency, and Isolation Levels › Blocking in SQL Server

