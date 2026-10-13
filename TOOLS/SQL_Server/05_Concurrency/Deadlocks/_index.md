---
id: "TOOLS/SQL_Server/05_Concurrency/Deadlocks"
level: 4
type: "curriculum"
curriculum: "sql_server"
---

# Deadlocks

**L4 Advanced** · [↑ Concurrency, Locking and Blocking](<../_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** A deadlock is a cycle of sessions each waiting for a lock another one holds. SQL Server's deadlock monitor detects the cycle, rolls back the cheapest session (error 1205), and you prevent repeats by reading the deadlock graph and fixing access order, indexes, transaction length or isolation.

### 1. The big picture
Blocking (previous lesson) is a *chain*: it ends when the head blocker finishes. A deadlock is a *cycle*: A waits for B, B waits for A, and neither can finish. Without help they would wait forever, so SQL Server breaks the cycle by killing one participant. Deadlocks are a normal hazard of locking. A few per day may be acceptable if the application retries. Frequent ones point to design problems: inconsistent access order, missing indexes, long transactions, or overly strict isolation.

### 2. Core concepts and vocabulary
- **Deadlock**: circular wait among two or more sessions on lock (or other) resources.
- **Deadlock monitor (lock monitor)**: background thread that searches for cycles every 5 seconds by default. When it finds deadlocks, it checks more often (down to ~100 ms).
- **Victim**: the session chosen to be rolled back. It receives **error 1205** ("Transaction (Process ID n) was deadlocked … and has been chosen as the deadlock victim. Rerun the transaction.").
- **Victim choice**: lowest `DEADLOCK_PRIORITY` first. On a tie, the transaction that is cheapest to roll back (least log used).
- **Deadlock graph**: XML (`xml_deadlock_report`) listing the processes, the resources, who owns what, and who waits for what.
- **Owner / waiter list**: in the graph, each resource shows which process holds it (owner, with mode) and which one wants it (waiter, with requested mode).
- **Cycle types**: *cycle (order) deadlock*, *conversion deadlock*, *key lookup (bookmark) deadlock*, *parallelism (intra-query) deadlock*, *range-lock deadlock under SERIALIZABLE*.

### 3. How it works, step by step
**Detection and resolution**
1. Sessions wait on locks as usual.
2. The deadlock monitor builds a wait-for graph of waiting tasks and the resources they need.
3. If it finds a cycle, it picks a victim (priority, then rollback cost).
4. The victim's current transaction is **rolled back**, its locks are released, and it gets error 1205. The other sessions continue.
5. The event is recorded in the `system_health` Extended Events session (always running) as `xml_deadlock_report`.

**The common patterns**

| Pattern | How it happens | Typical fix |
|---|---|---|
| **Access-order (cycle)** | Tx1 updates A then B; Tx2 updates B then A | access tables/rows in the same order everywhere |
| **Conversion** | two sessions hold S on a row (REPEATABLE READ / HOLDLOCK), both try to convert to X | read with `UPDLOCK`; use a single atomic UPDATE |
| **Key lookup** | a reader seeks a nonclustered index (S) then looks up the clustered key; a writer updates the clustered row (X) then the nonclustered index | covering index (no lookup); RCSI for the reader |
| **Range (SERIALIZABLE)** | two "check then insert" transactions hold RangeS-S on the same gap, both want RangeI-N | `UPDLOCK, HOLDLOCK` on the check; MERGE carefully; READ COMMITTED |
| **Parallelism** | threads of one parallel query wait on each other's exchange buffers | usually resolved by the engine; fix the plan, lower MAXDOP for that query |
| **Foreign key / cascade / trigger** | hidden reads of parent/child tables in unexpected order | index FK columns; review trigger order |

### 4. Worked example
```sql
-- Classic access-order deadlock
CREATE TABLE dbo.A (id int PRIMARY KEY, v int); INSERT dbo.A VALUES (1, 0);
CREATE TABLE dbo.B (id int PRIMARY KEY, v int); INSERT dbo.B VALUES (1, 0);

-- Session 1
BEGIN TRAN; UPDATE dbo.A SET v = 1 WHERE id = 1;      -- X on A.1
-- Session 2
BEGIN TRAN; UPDATE dbo.B SET v = 1 WHERE id = 1;      -- X on B.1
-- Session 1
UPDATE dbo.B SET v = 2 WHERE id = 1;                  -- waits for session 2
-- Session 2
UPDATE dbo.A SET v = 2 WHERE id = 1;                  -- cycle → one session gets error 1205

-- Read the graph from system_health
SELECT xed.value('@timestamp','datetime2') AS utc_time,
       xed.query('.') AS deadlock_xml
FROM (SELECT CAST(target_data AS xml) AS td
      FROM sys.dm_xe_session_targets AS t
      JOIN sys.dm_xe_sessions AS s ON s.address = t.event_session_address
      WHERE s.name = 'system_health' AND t.target_name = 'ring_buffer') AS x
CROSS APPLY td.nodes('//RingBufferTarget/event[@name="xml_deadlock_report"]') AS q(xed)
ORDER BY utc_time DESC;
-- (or read the event_file target: sys.fn_xe_file_target_read_file('system_health*.xel', NULL, NULL, NULL))
```
**Reading the graph** (open the XML in SSMS as a `.xdl` file to see the picture):
1. `<victim-list>` → which process was killed.
2. `<process-list>` → for each process: `spid`, `isolationlevel`, `trancount`, `lasttranstarted`, the `inputbuf` (batch) and `executionStack` (the exact statement and procedure line).
3. `<resource-list>` → each resource (`keylock`, `pagelock`, `objectlock`, `ridlock`…) with `objectname`, `indexname`, an **owner** (`mode="X"`) and a **waiter** (`requestType="wait" mode="U"`).
4. Draw the arrows: process 1 owns resource R1 and waits on R2; process 2 owns R2 and waits on R1. Now you know the order problem and which statements cause it.

The fix here: always update A before B in both code paths. Even better, make the transactions shorter.

**Application side: retry**
```sql
DECLARE @retry int = 3;
WHILE @retry > 0
BEGIN
  BEGIN TRY
    BEGIN TRAN;
      -- work
    COMMIT; BREAK;
  END TRY
  BEGIN CATCH
    IF XACT_STATE() <> 0 ROLLBACK;
    IF ERROR_NUMBER() = 1205 AND @retry > 1 BEGIN SET @retry -= 1; WAITFOR DELAY '00:00:00.1'; CONTINUE; END;
    THROW;
  END CATCH
END;
```

### 5. Under the hood
- **Resources beyond locks.** The monitor also detects cycles involving worker threads, memory grants (`RESOURCE_SEMAPHORE`), parallel exchange ports and MARS sessions. Graphs may show `exchangeEvent` or `threadpool` resources.
- **Intra-query parallelism deadlocks** appear with `e_waitPipeGetRow`/`e_waitPipeNewRow`. The engine usually resolves them by spilling exchange buffers to tempdb. They're reported, but often no user error is raised.
- **Detection cost.** Searching is cheap, but the 5-second interval means a deadlocked session can wait up to ~5 s before resolution. That's part of the user-visible latency.
- **`DEADLOCK_PRIORITY`** accepts LOW (-5), NORMAL (0), HIGH (5) or −10…10. Use LOW for background jobs, so user transactions win.
- **Optimized locking** (2025/Azure) releases row locks right after each row is modified (keeping only a transaction-ID lock), and evaluates predicates on the latest committed version before locking. That removes many update/update deadlock patterns.
- **Trace flags 1204/1222** write deadlock info to the error log. They're legacy; `system_health` already captures every graph.

### 6. Performance and design implications
- **Consistent object order** across all procedures (e.g. parent then child, lower id first).
- **Short transactions:** fewer and shorter-held locks mean a lower chance of a cycle.
- **Indexes:** seeks lock fewer rows than scans. Covering indexes remove key-lookup deadlocks.
- **RCSI** removes reader/writer deadlocks entirely (readers take no S locks).
- **Avoid SERIALIZABLE/REPEATABLE READ** unless needed. Use `UPDLOCK` to serialize read-then-write.
- **Always add retry logic** for 1205 in the application or the procedure. Deadlocks can never be fully eliminated.
- Track the **Number of Deadlocks/sec** counter and the count of `xml_deadlock_report` events per day as a health metric.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `system_health` XE session (`xml_deadlock_report`) | always-on capture of deadlock graphs | first place to look |
| Custom XE session on `sqlserver.xml_deadlock_report` to `event_file` | longer retention | frequent deadlocks |
| `SET DEADLOCK_PRIORITY LOW` | make this session the preferred victim | batch jobs, reports |
| `SET LOCK_TIMEOUT` | stop waiting after N ms (error 1222) | fail fast |
| `WITH (UPDLOCK)` / `WITH (UPDLOCK, HOLDLOCK)` | serialize read-then-write | prevent conversion/range deadlocks |
| `ALTER DATABASE … SET READ_COMMITTED_SNAPSHOT ON` | readers stop taking S locks | reader/writer deadlocks |
| Perfmon *SQLServer:Locks — Number of Deadlocks/sec* | deadlock rate | monitoring |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| Fix access order: two or more code paths touch the same tables | third-party code you can't change (use indexes/RCSI) |
| Covering index: key-lookup deadlocks | write-heavy table where another index costs too much |
| RCSI: reader/writer deadlocks | writer/writer cycles (no help) |
| `UPDLOCK`: read-then-update patterns | held for long → more blocking |
| Retry logic: always, as a safety net | as the *only* fix for hundreds per hour |

### 9. Common mistakes
- Treating 1205 as a fatal error → users see failures → retry the whole transaction (not just the statement).
- Retrying inside the same failed transaction → error 3930 / doomed transaction → roll back, then retry from the start.
- Using `NOLOCK` to "fix" deadlocks → wrong data → RCSI, indexes, ordering.
- Reading only the victim's statement → missing the other side's role → analyse every process and resource in the graph.
- Turning on trace flags for capture → `system_health` already has the graphs (check the ring buffer *and* event files).

### 10. 🎯 Interview kit
**30-second answer:** A deadlock is a cycle where each session waits on a lock another one holds. SQL Server's lock monitor checks every five seconds, picks the victim with the lowest deadlock priority or the lowest rollback cost, rolls it back, and raises error 1205. I get the deadlock graph from the `system_health` Extended Events session and identify the processes, statements, resources, owners and waiters. Then I fix the pattern: consistent access order, covering indexes for key-lookup deadlocks, `UPDLOCK` for read-then-update, RCSI for reader/writer cases, shorter transactions. The application always needs retry logic for 1205.
**Structure a longer answer:** definition vs blocking → detection and victim selection → how to capture and read the graph → common patterns with fixes → prevention and retry → a scenario.
**Likely follow-ups:** *How is the victim chosen?* → DEADLOCK_PRIORITY, then least log to roll back. · *What's a conversion deadlock?* → two S holders both want X; U locks prevent it. · *Can a single query deadlock itself?* → yes, intra-query parallelism. · *How often does detection run?* → every 5 s by default, faster after deadlocks are found.
**Signals interviewers look for:** reading graphs (owner/waiter), naming patterns, preferring design fixes, mentioning retries and `system_health`.

### 11. Check yourself
- [ ] Reproduce the A/B access-order deadlock and capture its graph from `system_health`.
- [ ] Label owners and waiters in the graph and draw the cycle.
- [ ] Reproduce a conversion deadlock with REPEATABLE READ and fix it with `UPDLOCK`.
- [ ] Explain the key-lookup deadlock and why a covering index fixes it.
- [ ] Write a retry wrapper for 1205 that rolls back properly.
<!-- explainer:end -->

## References

- *SQL Server 2025 Query Performance Tuning (7th ed.)* — Grant Fritchey — FULL_BOOK_NEEDS_MANUAL_SPLIT (cont.) › Chapter 16 Blocking And Blocked Processes
- *SQL Server 2025 Query Performance Tuning (7th ed.)* — Grant Fritchey — FULL_BOOK_NEEDS_MANUAL_SPLIT (cont.) › Chapter 17 Causes And Solutions For Deadlocks
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 10: Transactions, Concurrency, and Isolation Levels › Deadlock Detection and Resolution
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 10: Transactions, Concurrency, and Isolation Levels › Deadlock Graph Analysis
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 10: Transactions, Concurrency, and Isolation Levels › Deadlocks
- *Expert Performance Indexing in SQL Server* — Jason Strate, Grant Fritchey — Number of Deadlocks per Second
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 10: Transactions, Concurrency, and Isolation Levels › Preventing Deadlocks with Application Design
- *T-SQL Fundamentals (4th ed.)* — Itzik Ben-Gan — Transactions and concurrency
- *T-SQL Fundamentals (4th ed.)* — Itzik Ben-Gan — SQL Graph (cont.)

