---
id: "TOOLS/SQL_Server/SQL_Server_Platform/Read_Committed_Locking_Anomalies"
level: 4
type: "concept"
topic: "TOOLS/SQL_Server/SQL_Server_Platform"
tools: ["SQL_Server"]
---

# Read Committed Locking Anomalies

**L4 Advanced** · [↑ SQL Server Platform](<_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** Under the default `READ COMMITTED` isolation level SQL Server never shows you uncommitted data, but because shared locks are released as soon as each row is read, a statement can still read rows twice, skip rows, or see different values on re-reads.

### 1. The big picture
Isolation levels trade consistency for concurrency. SQL Server's default is **READ COMMITTED** using *locking* (RCSI is a separate, opt-in database setting). It prevents **dirty reads** by making readers take shared (S) locks that conflict with writers' exclusive (X) locks — but it holds those S locks only for the instant needed to read the row (or page), not until the transaction ends. Everything that goes wrong follows from that short duration.

This topic sits between locking fundamentals (lock modes, compatibility, escalation) and the higher isolation levels / row versioning (REPEATABLE READ, SERIALIZABLE, SNAPSHOT, RCSI).

### 2. Core concepts and vocabulary
- **Dirty read** — reading another transaction's uncommitted change.
- **Non-repeatable read** — re-reading a row gives a different value because someone updated it between reads.
- **Phantom** — re-running a predicate returns new/missing rows.
- **Lost update** — two read-modify-write cycles overwrite each other.
- **S / U / X locks** — shared, update, exclusive; **IS/IX/SIX** intent locks at higher levels.
- **Lock duration** — how long a lock is kept (statement vs transaction).
- **Allocation-order scan** — scan following IAM pages instead of index order; can miss/duplicate rows if pages move.
- **RCSI** — `READ_COMMITTED_SNAPSHOT`: READ COMMITTED using row versions rather than S locks.
- **Blocking vs deadlock** — waiting vs circular waiting.

### 3. How it works, step by step
Reader at READ COMMITTED (locking):
1. Request an S lock on the row/page being read (IS on table and page first).
2. If a writer holds X → wait (block) until the writer commits or rolls back.
3. Read the row, then **release the S lock immediately** (not at transaction end).
4. Move to the next row/page.

Writer: takes U (during search) → X on modification, held until commit.

| Anomaly | Possible at READ COMMITTED? | Why |
|---|---|---|
| Dirty read | No | S lock blocks on X |
| Non-repeatable read | **Yes** | S released after read |
| Phantom | **Yes** | no range locks |
| Lost update | **Yes** (if read then write separately) | no lock held between |
| Missed/duplicate rows during a scan | **Yes** | data moves behind/ahead of the scan |

### 4. Worked example
Non-repeatable read:
```sql
-- Session 1
BEGIN TRAN;
SELECT Balance FROM dbo.Account WHERE ID = 1;   -- 100, S lock released immediately
-- Session 2
UPDATE dbo.Account SET Balance = 50 WHERE ID = 1;  -- commits fine
-- Session 1
SELECT Balance FROM dbo.Account WHERE ID = 1;   -- 50 → different value in same transaction
COMMIT;
```
Lost update:
```sql
-- both sessions: SELECT @b = Balance ...; (100)
-- S1: UPDATE ... SET Balance = @b - 10;  -- 90
-- S2: UPDATE ... SET Balance = @b - 20;  -- 80, S1's change overwritten
```
Fix: do it atomically (`UPDATE … SET Balance = Balance - 10`), or read with `WITH (UPDLOCK, ROWLOCK)`, or use an optimistic `rowversion` check.

Inspect locks live:
```sql
SELECT request_session_id, resource_type, request_mode, request_status
FROM sys.dm_tran_locks WHERE resource_database_id = DB_ID();
```
`request_status = WAIT` shows who is blocked; `sys.dm_os_waiting_tasks` shows `blocking_session_id` and wait type `LCK_M_S`.

### 5. Under the hood
- The lock manager keeps hash tables of lock resources (keys hashed as `(index_id, key hash)`); intent locks signal finer locks below so table-level requests avoid scanning all rows.
- **Scan anomalies:** a scan over a clustered index in key order can read a row, then a concurrent update moves it ahead of the scan position (key change or page split) → read twice; moved behind → missed. A scan in allocation order (`NOLOCK`/large tables) is even more vulnerable to page splits.
- `READ UNCOMMITTED`/`NOLOCK` also allows dirty reads and can return the same row twice or skip it entirely.
- **RCSI** changes the picture: readers get the last committed version from the version store in `tempdb` (or PVR in ADR databases), so they neither block nor are blocked, and statement-level consistency is a point-in-time snapshot — but non-repeatable reads across statements and lost updates remain.
- Lock escalation: ≥ 5,000 locks on one object in a statement attempts escalation to a table lock, enlarging blocking.
- Deadlock detector wakes every 5 s (faster when deadlocks are frequent) and kills the cheapest-to-rollback victim (error 1205).

### 6. Performance and design implications
- Long-running reads under locking block writers; long writers block readers → throughput collapse and timeouts.
- RCSI cuts reader/writer blocking at the cost of `tempdb` version store and 14 bytes per modified row.
- Keep transactions short; access objects in a consistent order; add covering indexes to shorten lock hold and scope.
- Measure: `sys.dm_exec_requests.blocking_session_id`, wait stats (`LCK_M_*`), Extended Events `xml_deadlock_report`, Query Store wait stats, `sys.dm_tran_version_store_space_usage`.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `SET TRANSACTION ISOLATION LEVEL …` | session isolation | per-workload |
| `ALTER DATABASE … SET READ_COMMITTED_SNAPSHOT ON` | RCSI | OLTP reader/writer blocking |
| `ALLOW_SNAPSHOT_ISOLATION ON` | enables SNAPSHOT | consistent reports |
| `WITH (UPDLOCK, HOLDLOCK, ROWLOCK, READPAST)` | lock hints | read-modify-write, queues |
| `sys.dm_tran_locks`, `dm_os_waiting_tasks` | diagnose blocking | live troubleshooting |
| `sp_whoisactive` (community) | current activity | triage |
| `SET LOCK_TIMEOUT` | max wait | avoid indefinite blocking |
| `SET DEADLOCK_PRIORITY` | victim choice | batch vs OLTP |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| Default locking RC for short OLTP transactions | Reports run on the same tables as heavy writes |
| RCSI for reader/writer concurrency | Code relies on blocking for serialization (queues, "check then insert") |
| SNAPSHOT for consistent multi-statement reads | tempdb is undersized |
| REPEATABLE READ / SERIALIZABLE for strict invariants | High contention; deadlocks rise |

### 9. Common mistakes
- Assuming a transaction at RC gives repeatable reads → inconsistent logic → use REPEATABLE READ/SNAPSHOT or lock hints.
- "Check then insert" without `UPDLOCK, HOLDLOCK` → duplicates → hint or unique constraint.
- `NOLOCK` everywhere → dirty/duplicate/missing rows → RCSI instead.
- Turning on RCSI blindly → code that depended on blocking now races → test.
- Holding transactions open across user think time → blocking chains → short transactions.
- Retry-less deadlock handling → intermittent failures → retry on 1205.

### 10. 🎯 Interview kit
**30-second answer:** Locking READ COMMITTED stops dirty reads by taking S locks, but releases them right after reading, so non-repeatable reads, phantoms, lost updates and scan anomalies (missed/duplicate rows) can occur. RCSI uses row versions to avoid blocking but doesn't fix those across statements.

**Structure a longer answer:** clarify → lock duration → anomaly table → demo → fixes (levels, hints, versioning) → trade-offs.

**Likely follow-ups:** *RC vs RCSI?* → locks vs versions. *Why does NOLOCK return duplicates?* → allocation scans + page splits. *Prevent lost update?* → atomic update, UPDLOCK, rowversion. *What does SERIALIZABLE add?* → key-range locks vs phantoms.

**Signals interviewers look for:** knows lock *duration* is the crux, reads DMVs, understands versioning costs.

### 11. Check yourself
- [ ] Reproduce a non-repeatable read with two sessions.
- [ ] Explain why NOLOCK can duplicate rows.
- [ ] Show blocking via `sys.dm_tran_locks`.
- [ ] State what RCSI changes and what it doesn't.
<!-- explainer:end -->

## Implementations (tools)

- [SQL Server](<../_index.md>)

## Practice questions

- [Basic] [What is a transaction in SQL Server and what are the ACID properties?](<../../../PRACTICE/TOOLS/SQL_Server/SQL_Server_Platform/Q-41491_What_is_a_transaction_in_SQL_Server_and_what_are_the_ACID_pr.md>)
- [Deep] [What is the default transaction isolation level in SQL Server and what does it mean?](<../../../PRACTICE/TOOLS/SQL_Server/SQL_Server_Platform/Q-41494_What_is_the_default_transaction_isolation_level_in_SQL_Serve.md>)
- [Deep] [What do PAGEIOLATCH, LCK_M_X, and CXPACKET waits typically indicate, and how would you act on each?](<../../../PRACTICE/TOOLS/SQL_Server/SQL_Server_Platform/Q-41357_What_do_PAGEIOLATCH_LCK_M_X_and_CXPACKET_waits_typically_ind.md>)
- [Deep] [Explain why NOLOCK can return duplicate or missing rows even on committed data, and why it is not a safe default for reporting.](<../../../PRACTICE/TOOLS/SQL_Server/SQL_Server_Platform/Q-41372_Explain_why_NOLOCK_can_return_duplicate_or_missing_rows_even.md>)
- [Deep] [Explain how the transaction log works internally (VLFs, log truncation, checkpoints), and what causes a log file to grow uncontrollably.](<../../../PRACTICE/TOOLS/SQL_Server/SQL_Server_Platform/Q-41395_Explain_how_the_transaction_log_works_internally_VLFs_log_tr.md>)
- [Deep] [Explain how retry logic for deadlock victims (error 1205) should be implemented at the application or procedure level, and what makes an operation safe to retry.](<../../../PRACTICE/TOOLS/SQL_Server/SQL_Server_Platform/Q-41439_Explain_how_retry_logic_for_deadlock_victims_error_1205_shou.md>)
- [Expert] [You are seeing write-write conflicts under SNAPSHOT isolation (error 3960). Explain the cause and how you would design around it.](<../../../PRACTICE/TOOLS/SQL_Server/SQL_Server_Platform/Q-41369_You_are_seeing_write-write_conflicts_under_SNAPSHOT_isolatio.md>)
- [Expert] [Explain how SQL Server's read-committed default (locking) can cause a query to read the same row twice or skip rows during an index scan while data moves.](<../../../PRACTICE/TOOLS/SQL_Server/SQL_Server_Platform/Q-41449_Explain_how_SQL_Server_s_read-committed_default_locking_can_.md>)

## Related concepts

- [MERGE Pitfalls and Error Handling](<../T-SQL/MERGE_Pitfalls_and_Error_Handling.md>) *(T-SQL)*
- [Lock Granularity](<../../../01_Databases_and_SQL/Database_Internals/DBMS_Concepts/Lock_Granularity.md>) *(DBMS Concepts)*
- [SKIP LOCKED Job Queues](<../../../01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/SKIP_LOCKED_Job_Queues.md>) *(SQL Fundamentals)*

## References

- *100 SQL Server Mistakes and How to Avoid Them* — Peter A. Carter — Error handling, testing, source control, and deployment › 7.1 #26 Writing code that doesn't handle errors
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 10: Transactions, Concurrency, and Isolation Levels › Common Violations Prevented by SQL Server
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 10: Transactions, Concurrency, and Isolation Levels › @@TRANCOUNT
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 10: Transactions, Concurrency, and Isolation Levels › Lock Types and Behaviors
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 10: Transactions, Concurrency, and Isolation Levels › Lock Compatibility
- *T-SQL Fundamentals (4th ed.)* — Itzik Ben-Gan — Transactions and concurrency
- *Beginning T-SQL* — Kathi Kellenberger, Lee Everest — Managing Transactions › Writing an Explicit Transaction
- *Beginning T-SQL* — Kathi Kellenberger, Lee Everest — Managing Transactions › Thinking About Performance
- *SQL Server 2025 Query Performance Tuning (7th ed.)* — Grant Fritchey — FULL_BOOK_NEEDS_MANUAL_SPLIT (cont.) › Chapter 1 Query Performance Tuning
- *SQL Server 2025 Query Performance Tuning (7th ed.)* — Grant Fritchey — FULL_BOOK_NEEDS_MANUAL_SPLIT (cont.) › Chapter 16 Blocking And Blocked Processes
- *Designing Data-Intensive Applications* — Martin Kleppmann — Two-Phase Locking (2PL) › Implementation of two-phase locking
- *SQL Server Tutorial* (YouTube series) — Part 70: sql server concurrent transactions ([watch](https://www.youtube.com/watch?v=TWv2jpmxaf8))
- *SQL Server Tutorial* (YouTube series) — Part 71: sql server dirty read example ([watch](https://www.youtube.com/watch?v=5ZEchu2WnD4))
- *SQL Server Tutorial* (YouTube series) — Part 72: sql server lost update problem ([watch](https://www.youtube.com/watch?v=jD0c4X0tSc8))
- *SQL Server Tutorial* (YouTube series) — Part 80: Logging deadlocks in sql server ([watch](https://www.youtube.com/watch?v=MygimSTXOFk))

