---
id: "01_Databases_and_SQL/Transactions_and_Concurrency/Deadlocks/Deadlock_Causes_and_Diagnosis"
level: 4
type: "concept"
topic: "01_Databases_and_SQL/Transactions_and_Concurrency/Deadlocks"
tools: ["SQL_Server"]
---

# Deadlock Causes and Diagnosis

**L4 Advanced** · [↑ Deadlocks](<_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** A deadlock is a cycle of sessions each holding a lock another needs; the engine detects it, kills one victim, and your job is to find the access pattern that creates the cycle and remove it.

### 1. The big picture
Blocking is normal: session 2 waits because session 1 holds a lock. A deadlock is blocking with a loop: session 1 holds A and wants B, session 2 holds B and wants A. Neither can ever proceed, so the engine's deadlock monitor picks one session as the **victim**, rolls it back and returns an error (SQL Server error 1205; "deadlock detected" in PostgreSQL and MySQL), letting the other finish.

Deadlocks are fundamentally a performance and design problem made visible: they occur when transactions are long, touch objects in different orders, or use access paths that take locks in unexpected sequences. They cannot be eliminated entirely in lock-based systems, so you both reduce their frequency and handle the remaining ones with retries.

### 2. Core concepts and vocabulary
- **Deadlock** — cyclic wait among two or more transactions.
- **Waits-for graph** — nodes are transactions, edges mean "waits for"; a cycle is a deadlock.
- **Deadlock monitor** — background task (every ~5 s in SQL Server, faster when deadlocks occur) searching the graph.
- **Victim** — the transaction chosen to roll back; by default the cheapest to undo, influenced by `DEADLOCK_PRIORITY`.
- **Lock conversion deadlock** — two sessions hold S locks and both try to upgrade to X.
- **Resource ordering** — acquiring locks in a globally consistent order.
- **Deadlock graph / XML** — the engine's report of resources, locks, statements and victim.
- **Extended Events `xml_deadlock_report`** — the standard capture mechanism.
- **Correlation id** — an identifier tying a deadlock to an application request.
- **Advisory lock** — application-defined lock (PostgreSQL) used to serialize critical sections.

### 3. How it works, step by step
How a deadlock forms and resolves:

1. T1 updates row in `Customers` (X lock) and, later, wants a row in `Orders`.
2. T2 updates a row in `Orders` (X lock) and wants the same `Customers` row.
3. T1 waits for T2's lock; T2 waits for T1's lock. Neither releases locks until its transaction ends.
4. The monitor builds the waits-for graph, finds the cycle.
5. It chooses the victim: lowest `DEADLOCK_PRIORITY`, otherwise the transaction with least log used (cheapest rollback).
6. The victim is rolled back (locks released), the survivor proceeds, the victim's client gets error 1205.

Common patterns:

| Pattern | Example | Typical fix |
|---|---|---|
| Opposite object order | A: Customers→Orders, B: Orders→Customers | Same order everywhere |
| Update–update opposite rows | Transfer A→B and B→A | Order by key (lowest id first) |
| Select–update via different indexes | Reader holds S on nonclustered index, then needs clustered key lock; writer holds clustered X, needs index key | Covering index; RCSI; shorter reads |
| Lock conversion | Two sessions read under REPEATABLE READ then update same row | `UPDLOCK` on read |
| Insert–insert with unique/range locks | Concurrent inserts under SERIALIZABLE | Lower isolation or ordering |
| Long transaction with external waits | Transaction open across a network call | Move work out of transaction |
| Batch vs OLTP | Nightly job scans/updates while OLTP runs | Batch in small chunks, off-hours, versioning |

Diagnosis steps:

1. Capture: Extended Events session on `xml_deadlock_report` (or `system_health`, which already records them), trace flag 1222 for the error log.
2. Read the graph: find the two (or more) **statements**, the **resources** (key, page, object, index name), the **modes** (S, X, U, IX) and which was chosen victim.
3. Identify why each statement holds its first lock (isolation level, scan vs seek, trigger, foreign key checks).
4. Fix the root cause, then confirm the deadlock count falls.

### 4. Worked example
Reproduce an order-of-access deadlock:

```sql
-- Session 1                                -- Session 2
BEGIN TRAN;                                 BEGIN TRAN;
UPDATE Customers SET Name = 'A'             UPDATE Orders SET Status = 'X'
 WHERE CustomerID = 1;                       WHERE OrderID = 10;
-- now run session 2's first statement, then:
UPDATE Orders SET Status = 'Y'              UPDATE Customers SET Name = 'B'
 WHERE OrderID = 10;   -- waits               WHERE CustomerID = 1;  -- deadlock
```
One session gets: `Msg 1205 ... Transaction was deadlocked on lock resources with another process and has been chosen as the deadlock victim.`

Reading the graph from `system_health`:
```sql
SELECT CAST(event_data AS xml) AS deadlock
FROM sys.fn_xe_file_target_read_file('system_health*.xel', NULL, NULL, NULL)
WHERE object_name = 'xml_deadlock_report';
```
In the XML look at `<victim-list>`, `<process-list>` (each `<inputbuf>` shows the statement), and `<resource-list>` (`keylock`, `objectname`, `indexname`, `mode`). If a `keylock` appears on a nonclustered index and the other on the clustered index, you have the two-index select/update pattern.

### 5. Under the hood
- Deadlock detection is a graph cycle search. SQL Server's monitor wakes at an interval that shrinks (down to 100 ms) when deadlocks are found, so detection delay is short, but sessions wait until then.
- Victim selection weights `DEADLOCK_PRIORITY` (-10 to 10, LOW/NORMAL/HIGH) first, then log bytes written, so a mid-way big transaction is rarely chosen against a small one.
- Textbook strategies: timeouts (simple but abort healthy transactions), conservative 2PL (acquire all locks up front; limits concurrency), **detection** (what SQL Server does) and **prevention** by timestamp ordering (wait-die / wound-wait in some systems).
- The resource ordering trick works because a cycle requires someone to acquire locks in the reverse order; a total order on resources makes cycles impossible for the ordered acquisitions.
- ORMs hide boundaries: an implicit transaction around `SaveChanges` plus lazy loading can acquire locks in an order that differs per code path, creating deadlocks that "do not appear" in the code.

### 6. Performance and design implications
- Deadlock frequency scales with transaction duration and the number of locks held; shorten both first.
- Proper indexing reduces locks (seeks lock few rows; scans lock many and can escalate); covering indexes eliminate the second index lookup that causes select/update cycles.
- RCSI removes reader locks, killing a large class of deadlocks (those involving S locks).
- Foreign keys without supporting indexes cause scans of child tables on parent changes, locking widely.
- Each deadlock costs a rollback plus a retry; monitor counts with `Deadlocks/sec` perf counter.
- Setting `DEADLOCK_PRIORITY LOW` on batch jobs makes them the victim, protecting OLTP.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| Extended Events `xml_deadlock_report` / `system_health` | Captures deadlock graphs | Always on in production |
| `DBCC TRACEON (1222, -1)` | Writes deadlock info to error log | Legacy/quick capture |
| `SET DEADLOCK_PRIORITY LOW|NORMAL|HIGH|n` | Influences victim choice | Batch vs OLTP |
| `SET LOCK_TIMEOUT` | Fails a wait after n ms | Avoid long waits |
| `WITH (UPDLOCK)` | Take U on read | Prevent conversion deadlocks |
| `READ_COMMITTED_SNAPSHOT ON` | Readers don't take S locks | Remove reader-involved deadlocks |
| `sp_whoisactive`, `sys.dm_tran_locks` | Live lock/blocking view | During incidents |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| Consistent access order | Order cannot be known up front (dynamic) → retry |
| RCSI | Readers need to wait for writers; `tempdb` limited |
| Covering index | Index maintenance cost on write-heavy table |
| `DEADLOCK_PRIORITY` | Overusing HIGH makes important victims anyway |
| Smaller batches | Atomicity required across the whole batch |
| Advisory/application locks | You can't reach consistent ordering otherwise; adds a second locking layer |

### 9. Common mistakes
- Treating deadlocks as random bad luck → recurring errors → capture graphs and fix the pattern.
- Not retrying 1205 → user-facing failures → wrap and retry the whole transaction.
- Fixing by `NOLOCK` everywhere → wrong reads, deadlocks between writers remain → target the cycle.
- Holding transactions over remote calls or user think-time → wide lock windows → reduce scope.
- Assuming a deadlock between "SELECT and UPDATE" cannot happen → it can via two indexes → inspect the graph resources.
- Raising `DEADLOCK_PRIORITY` of everything → no effect → reserve for what must win.
- Nightly job deadlocking with OLTP → run in small keyed batches, lower priority, versioning, or move off-hours.

### 10. 🎯 Interview kit
**30-second answer:** A deadlock is a circular wait: each transaction holds a lock the other needs. The engine's monitor detects the cycle, rolls back the cheapest victim and returns error 1205. I fix them by capturing the deadlock graph, finding the statements and resources, then ordering access consistently, shortening transactions, adding covering indexes or enabling RCSI, and retrying the victim.

**Structure a longer answer:** clarify → core mechanism → trade-offs → close with a real scenario.
Define blocking vs deadlock, walk the formation and victim choice, list patterns, then the diagnosis workflow and prevention.

**Likely follow-ups:**
- *How is the victim chosen?* → lowest `DEADLOCK_PRIORITY`, else least log used.
- *Nightly job vs OLTP?* → read the graph, chunk the job, lower its priority, align access order.
- *Why can a select deadlock with an update?* → different index access paths take locks in opposite order.
- *ORM hidden deadlocks?* → implicit transactions and lazy loading change lock order; log SQL, correlation ids.
- *Capture in MySQL/InnoDB?* → `SHOW ENGINE INNODB STATUS` shows only the last; use `innodb_print_all_deadlocks`.
- *PostgreSQL advisory locks?* → serialize a critical section by key; remember to release and keep ordering.

**Signals interviewers look for:** a diagnosis workflow (graph first), awareness of isolation and indexes as causes, and retry as part of the solution.

### 11. Check yourself
- [ ] Reproduce a two-table deadlock and read its graph.
- [ ] Name five deadlock patterns and a fix for each.
- [ ] Explain how the victim is chosen and how to influence it.
- [ ] Say how RCSI and covering indexes reduce deadlocks.
- [ ] Describe how you would tie a deadlock back to a user request.
<!-- explainer:end -->

## Implementations (tools)

- [SQL Server](<../../../TOOLS/SQL_Server/_index.md>)

## Practice questions

- [Basic] [How would you explain to a junior developer why holding a transaction open across a network call or user think-time is a deadlock hazard?](<../../../PRACTICE/01_Databases_and_SQL/Transactions_and_Concurrency/Deadlocks/Q-23135_How_would_you_explain_to_a_junior_developer_why_holding_a_tr.md>)
- [Intermediate] [How does SQL Server's deadlock monitor decide which session becomes the victim and how can you influence that with DEADLOCK_PRIORITY?](<../../../PRACTICE/01_Databases_and_SQL/Transactions_and_Concurrency/Deadlocks/Q-23081_How_does_SQL_Server_s_deadlock_monitor_decide_which_session_.md>)
- [Intermediate] [Describe how you'd instrument code to attach a correlation id so every deadlock log ties back to a specific request and user action.](<../../../PRACTICE/01_Databases_and_SQL/Transactions_and_Concurrency/Deadlocks/Q-23120_Describe_how_you_d_instrument_code_to_attach_a_correlation_i.md>)
- [Deep] [Why is the InnoDB status output insufficient for high-frequency deadlocks and what do you set up instead for full capture?](<../../../PRACTICE/01_Databases_and_SQL/Transactions_and_Concurrency/Deadlocks/Q-23058_Why_is_the_InnoDB_status_output_insufficient_for_high-freque.md>)
- [Deep] [Explain how an ORM's implicit transaction boundaries and lazy loading can create deadlocks that are invisible in the application code.](<../../../PRACTICE/01_Databases_and_SQL/Transactions_and_Concurrency/Deadlocks/Q-23077_Explain_how_an_ORM_s_implicit_transaction_boundaries_and_laz.md>)
- [Deep] [A batch job and an OLTP transaction deadlock every night at the same time; how would you diagnose the root cause and redesign to eliminate it?](<../../../PRACTICE/01_Databases_and_SQL/Transactions_and_Concurrency/Deadlocks/Q-23092_A_batch_job_and_an_OLTP_transaction_deadlock_every_night_at_.md>)
- [Deep] [How would you use application-level advisory locks in PostgreSQL to prevent a deadlock, and what are their pitfalls?](<../../../PRACTICE/01_Databases_and_SQL/Transactions_and_Concurrency/Deadlocks/Q-23113_How_would_you_use_application-level_advisory_locks_in_Postgr.md>)
- [Deep] [How do you approach a deadlock where the two statements are actually a SELECT and an UPDATE on the same table via different indexes?](<../../../PRACTICE/01_Databases_and_SQL/Transactions_and_Concurrency/Deadlocks/Q-23129_How_do_you_approach_a_deadlock_where_the_two_statements_are_.md>)

## Related concepts

- [Row Locks Two-phase Locking and Lock Footprint](<../Locking/Row_Locks_Two-phase_Locking_and_Lock_Footprint.md>) *(Locking)*
- [SKIP LOCKED Job Queues](<../../SQL_Language/SQL_Fundamentals/SKIP_LOCKED_Job_Queues.md>) *(SQL Fundamentals)*
- [Lock Holding and Optimistic Concurrency](<../ACID_Transactions/Lock_Holding_and_Optimistic_Concurrency.md>) *(ACID Transactions)*

## References

- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 10: Transactions, Concurrency, and Isolation Levels › Deadlocks
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 10: Transactions, Concurrency, and Isolation Levels › Monitoring Locks
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 10: Transactions, Concurrency, and Isolation Levels › Deadlocks and Blocking
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 10: Transactions, Concurrency, and Isolation Levels › Deadlocks in SQL Server
- *SQL Server 2025 Query Performance Tuning (7th ed.)* — Grant Fritchey — FULL_BOOK_NEEDS_MANUAL_SPLIT (cont.) › Chapter 15 Reduce Query Resource Use
- *SQL Server 2025 Query Performance Tuning (7th ed.)* — Grant Fritchey — FULL_BOOK_NEEDS_MANUAL_SPLIT (cont.) › Chapter 16 Blocking And Blocked Processes
- *SQL Server 2025 Query Performance Tuning (7th ed.)* — Grant Fritchey — FULL_BOOK_NEEDS_MANUAL_SPLIT (cont.) › Chapter 17 Causes And Solutions For Deadlocks
- *Database Internals* — Alex Petrov — Transaction Processing and Recovery › Deadlocks
- *Expert Performance Indexing in SQL Server* — Jason Strate, Grant Fritchey — Number of Deadlocks per Second
- *T-SQL Fundamentals (4th ed.)* — Itzik Ben-Gan — Transactions and concurrency
- *SQL Server Tutorial* (YouTube series) — Part 79: SQL Server deadlock victim selection ([watch](https://www.youtube.com/watch?v=GYADk9TtS88))
- *SQL Server Tutorial* (YouTube series) — Part 81: SQL Server deadlock analysis and prevention ([watch](https://www.youtube.com/watch?v=5uTLRZEbBmU))
- *SQL Server Tutorial* (YouTube series) — Part 82: Capturing deadlocks in sql profiler ([watch](https://www.youtube.com/watch?v=gFxI8VcAj4Y))
- *SQL Server Tutorial* (YouTube series) — Part 84: Handling deadlocks in ado net ([watch](https://www.youtube.com/watch?v=D7PvlUbUQpM))

