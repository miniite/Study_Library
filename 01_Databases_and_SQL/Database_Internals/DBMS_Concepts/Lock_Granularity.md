---
id: "01_Databases_and_SQL/Database_Internals/DBMS_Concepts/Lock_Granularity"
level: 4
type: "concept"
topic: "01_Databases_and_SQL/Database_Internals/DBMS_Concepts"
tools: ["MySQL", "PostgreSQL"]
---

# Lock Granularity

**L4 Advanced** · [↑ DBMS Concepts](<_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** Lock granularity is the size of the thing a transaction locks — row, page, key range or table — and it is a trade between concurrency (fine locks) and bookkeeping cost (coarse locks), managed by the engine through intent locks and escalation.

### 1. The big picture
Locks are how a database enforces isolation between concurrent transactions: a reader or writer announces "I am using this resource" and others must wait or be compatible. The question is *what resource*. Locking the whole table is cheap to track and trivially safe, but serialises everyone. Locking individual rows allows many transactions to work on the same table at once, but every lock is memory and CPU, and a statement touching a million rows would need a million locks.

Engines therefore support a **hierarchy** of lock sizes and choose automatically. This lesson uses SQL Server terminology (resource types `RID`, `KEY`, `PAGE`, `OBJECT`), with notes on PostgreSQL and InnoDB. It connects upward to transactions and isolation levels (which decide *how long* locks are held) and downward to blocking and deadlock diagnosis.

### 2. Core concepts and vocabulary
- **Lock granularity** — the unit locked: row/key, page, table (object), partition, database.
- **Lock mode** — what kind of access is claimed: **Shared (S)** for reads, **Exclusive (X)** for writes, **Update (U)** to read-with-intent-to-modify (avoids a classic deadlock), plus schema locks.
- **Intent locks (IS, IX, SIX)** — placed on a *higher* level (table, page) to signal that a lower-level lock exists beneath. They do not block by themselves; they let the engine check table-level conflicts without scanning every row lock.
- **Schema locks** — `Sch-S` (stability: held by any query that uses the table) and `Sch-M` (modification: taken by `ALTER TABLE`; blocks everything).
- **Key lock** — a lock on an index entry; the "row lock" you get on a clustered index (or on a nonclustered index entry).
- **RID lock** — row identifier lock, used on a heap (a table with no clustered index).
- **Range lock (key-range / gap / next-key)** — locks the gaps between index keys to prevent phantoms.
- **Lock escalation** — converting many fine-grained locks into one coarse lock to save memory.
- **Blocking** — one session waits for a lock another holds; **deadlock** — a cycle of waits the engine breaks by killing a victim.

### 3. How it works, step by step
1. A statement starts; the optimiser's plan determines how many rows it will touch, and the storage engine requests locks as it reads/writes.
2. For a single-row `UPDATE … WHERE id = 5` using an index seek:
   1. Acquire **IX** on the table (OBJECT).
   2. Acquire **IX** on the page containing the row.
   3. Acquire **U**, converted to **X**, on the row's KEY (or RID on a heap).
3. A second transaction updating a *different* row of the same table also takes IX on the table and page — IX is compatible with IX — and its own X on its own row. They run in parallel.
4. A third transaction issues a table scan with a shared table lock (or `ALTER TABLE`): it needs S/Sch-M on the table, sees the IX held, and waits. That is why intent locks exist: one cheap check at the table level, no scan of row locks.
5. If a single statement accumulates too many locks (SQL Server: roughly **5,000 locks on one object** in one statement, or memory pressure), the engine **escalates**: it tries to take a table (or partition) lock and releases the row/page locks. If other sessions hold conflicting locks the attempt fails and is retried after further locks accumulate.
6. Locks are released at commit/rollback under most isolation levels (S locks earlier under `READ COMMITTED`: released as soon as the row has been read).

Compatibility sketch:

| Held \ Requested | IS | IX | S | U | X |
|---|---|---|---|---|---|
| IS | ✔ | ✔ | ✔ | ✔ | ✘ |
| IX | ✔ | ✔ | ✘ | ✘ | ✘ |
| S | ✔ | ✘ | ✔ | ✔ | ✘ |
| U | ✔ | ✘ | ✔ | ✘ | ✘ |
| X | ✘ | ✘ | ✘ | ✘ | ✘ |

### 4. Worked example
```sql
CREATE TABLE dbo.LockTest (id INT PRIMARY KEY, v INT);
INSERT dbo.LockTest VALUES (1,10),(2,20),(3,30);

-- Session A
BEGIN TRAN;
DELETE dbo.LockTest WHERE id = 1;       -- leave open

-- Session B (another window): inspect locks held by A
SELECT resource_type, resource_description, request_mode, request_status
FROM   sys.dm_tran_locks
WHERE  resource_database_id = DB_ID() AND request_session_id <> @@SPID;
```
Typical output: an `OBJECT` lock in mode `IX`, a `PAGE` lock in `IX`, and a `KEY` (or `RID` on a heap) lock in `X`. `resource_description` for a RID reads `FileID:PageID:Slot`, e.g. `1:2480:0`. Now run `SELECT * FROM dbo.LockTest WHERE id = 2` in session B: it succeeds (different key), whereas `SELECT * FROM dbo.LockTest WHERE id = 1` blocks. Resolve names with `OBJECT_NAME()` on the associated entity id. Finish with `ROLLBACK` in A.

To see escalation, update ~10,000 rows in one statement without a supporting index and re-query `sys.dm_tran_locks`: you will see a single `OBJECT` lock in `X` instead of thousands of key locks.

### 5. Under the hood
- **Lock manager structures.** Each lockable resource hashes to a lock-hash-table entry holding a queue of granted and waiting requests. Every lock costs on the order of ~100 bytes of memory — the reason escalation exists.
- **Update locks** exist because two sessions that both take S on a row and then try to convert to X deadlock each other. U is compatible with S but not with another U, serialising the would-be writers up front.
- **Escalation thresholds** in SQL Server: ~5,000 locks held by a single statement on a single object triggers an attempt; table-level setting `LOCK_ESCALATION = TABLE | AUTO | DISABLE` (`AUTO` allows partition-level on partitioned tables). Escalation is always to table/partition, never row→page.
- **Optimistic row versioning** (`READ_COMMITTED_SNAPSHOT`, `SNAPSHOT`) removes most S locks: readers read a version instead, so only writer-writer conflicts remain.
- **Other engines.** PostgreSQL has no lock escalation: row locks are stored in the tuple header (not in a lock table), so row-level locking is unbounded; table locks have eight modes. InnoDB uses row-level locks on index records and, under REPEATABLE READ, **gap and next-key locks** to prevent phantoms — which can lock more than the rows a query returns and cause surprising deadlocks.

### 6. Performance and design implications
- **No granularity is "best".** Row locks give concurrency; coarser locks give throughput for bulk operations. A single-row delete with a seek should take a row lock; a full-table maintenance statement is better off with a table lock.
- **Missing indexes inflate locks.** A scan to find one row may lock (or at least touch under S) many rows; with RC isolation it holds and releases, but updates scanning take U on every row examined.
- **Long transactions hold locks long.** Keep transactions short, do not wait for user input inside them.
- **Foreign key checks.** Cascading deletes without an index on the child key scan the child table and lock widely.
- **Measure:** `sys.dm_tran_locks`, `sys.dm_os_wait_stats` (`LCK_M_*` waits), blocked process report, `sys.dm_exec_requests.blocking_session_id`; Postgres: `pg_locks` joined with `pg_stat_activity`.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `sys.dm_tran_locks` | Lists granted/waiting locks with resource and mode | Diagnosing blocking |
| `WITH (ROWLOCK / PAGLOCK / TABLOCK)` | Hint the granularity | Rare; bulk loads (`TABLOCK`) |
| `WITH (UPDLOCK)` | Takes U locks on read | Read-then-update patterns |
| `ALTER TABLE … SET (LOCK_ESCALATION = …)` | Controls escalation | Partitioned tables, contention from escalation |
| `SET LOCK_TIMEOUT n` | Fail instead of waiting forever | Fail-fast services |
| `READ_COMMITTED_SNAPSHOT ON` | Readers use versions, not S locks | Reader/writer blocking |
| `SELECT … FOR UPDATE` (Postgres/MySQL) | Row-level pessimistic lock | Preventing lost updates |
| `pg_locks` / `pg_stat_activity` | Postgres lock and session inspection | Finding the blocker |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| Row-level locks (default) for OLTP point access | Statement touches most of the table — bookkeeping overhead |
| Table lock hint for bulk load / maintenance window | Online system; blocks all readers/writers |
| `LOCK_ESCALATION = DISABLE` to prevent surprise table locks | Memory pressure — thousands of locks, lock-manager memory growth |
| Row versioning to remove reader blocking | tempdb/version-store growth, long-running transactions |
| Pessimistic `FOR UPDATE` on high-conflict rows | Low contention — optimistic version column is cheaper |

### 9. Common mistakes
- Blaming "the database is locking" without looking at the mode and resource → wrong fix → query `sys.dm_tran_locks` and find the head blocker.
- Updating many rows in one huge statement → escalation to a table X lock → batch in chunks (e.g. 2,000–4,000 rows).
- Missing index on the filter or foreign-key column → scans lock far more → add the index.
- Using `NOLOCK` as a performance fix → dirty/missing/duplicated rows → use row versioning.
- Holding a transaction open across user interaction or remote calls → long blocking chains → keep transactions short.
- Assuming intent locks conflict with row locks → confusion reading `IX` entries → they only conflict with table-level S/X.

### 10. 🎯 Interview kit
**30-second answer:** Lock granularity is the size of the resource a transaction locks: row, page or table. Fine-grained locks give higher concurrency but cost more memory to track; coarse locks are cheap but block more. Engines use intent locks to make hierarchical locking cheap and escalate from many row locks to a table lock when a statement accumulates too many.

**Structure a longer answer:** clarify (which engine?) → hierarchy and intent locks → escalation and its trade-off → how to observe (DMVs) → a scenario (batch update causing a blocking storm, fixed by batching + index).

**Likely follow-ups:**
- *Why intent locks?* → To detect table-level conflicts without scanning every row lock.
- *What is escalation and how do you control it?* → Row/page → table when ~5,000 locks; control by batching, indexing, table option.
- *Row-level vs table-level locking and concurrency?* → Row allows parallel writers on different rows; table serialises writers.
- *Gap/next-key locks in InnoDB?* → Lock index ranges under REPEATABLE READ to block phantoms; can cause unexpected deadlocks.

**Signals interviewers look for:** you tie granularity to isolation level and indexing, mention intent locks and escalation, and know how to observe locks rather than guess.

### 11. Check yourself
- [ ] Draw the lock hierarchy a single-row update takes and name each mode.
- [ ] Explain why two sessions can update different rows of one table concurrently but `ALTER TABLE` blocks.
- [ ] Describe what triggers escalation and two ways to avoid it.
- [ ] Reproduce a block with two sessions and identify the blocker from `sys.dm_tran_locks`.
- [ ] Explain why Postgres has no lock escalation.
<!-- explainer:end -->

## Implementations (tools)

- [MySQL](<../../../TOOLS/MySQL/_index.md>)
- [PostgreSQL](<../../../TOOLS/PostgreSQL/_index.md>)

## Practice questions

- [Basic] [What is a foreign key and what does referential integrity mean?](<../../../PRACTICE/01_Databases_and_SQL/Database_Internals/DBMS_Concepts/Q-22152_What_is_a_foreign_key_and_what_does_referential_integrity_me.md>)
- [Intermediate] [What is the difference between ON DELETE CASCADE and ON DELETE SET NULL for a foreign key?](<../../../PRACTICE/01_Databases_and_SQL/Database_Internals/DBMS_Concepts/Q-22236_What_is_the_difference_between_ON_DELETE_CASCADE_and_ON_DELE.md>)
- [Intermediate] [What are the OLD and NEW references inside a trigger and when is each available?](<../../../PRACTICE/01_Databases_and_SQL/Database_Internals/DBMS_Concepts/Q-22416_What_are_the_OLD_and_NEW_references_inside_a_trigger_and_whe.md>)
- [Deep] [What is the READ COMMITTED default in PostgreSQL and how does it differ from MySQL's REPEATABLE READ default?](<../../../PRACTICE/01_Databases_and_SQL/Database_Internals/DBMS_Concepts/Q-22307_What_is_the_READ_COMMITTED_default_in_PostgreSQL_and_how_doe.md>)
- [Deep] [How do you use pg_stat_activity or v$session to find the query and lock currently blocking a stuck transaction?](<../../../PRACTICE/01_Databases_and_SQL/Database_Internals/DBMS_Concepts/Q-22502_How_do_you_use_pg_stat_activity_or_v_session_to_find_the_que.md>)
- [Expert] [How would you implement a rate limiter or a job queue using only a relational database?](<../../../PRACTICE/01_Databases_and_SQL/Database_Internals/DBMS_Concepts/Q-22247_How_would_you_implement_a_rate_limiter_or_a_job_queue_using_.md>)
- [Expert] [What is a mutating-table error in an Oracle trigger and how do compound triggers or deferred logic get around it?](<../../../PRACTICE/01_Databases_and_SQL/Database_Internals/DBMS_Concepts/Q-22457_What_is_a_mutating-table_error_in_an_Oracle_trigger_and_how_.md>)
- [Expert] [What is a gap lock and next-key lock in InnoDB and how do they prevent phantoms under REPEATABLE READ while sometimes causing surprising deadlocks?](<../../../PRACTICE/01_Databases_and_SQL/Database_Internals/DBMS_Concepts/Q-22508_What_is_a_gap_lock_and_next-key_lock_in_InnoDB_and_how_do_th.md>)

## Related concepts

- [Row Locks Two-phase Locking and Lock Footprint](<../../Transactions_and_Concurrency/Locking/Row_Locks_Two-phase_Locking_and_Lock_Footprint.md>) *(Locking)*
- [Lock Holding and Optimistic Concurrency](<../../Transactions_and_Concurrency/ACID_Transactions/Lock_Holding_and_Optimistic_Concurrency.md>) *(ACID Transactions)*
- [Deadlocks and Lock Ordering](<../../../TOOLS/PostgreSQL/PostgreSQL_Platform/Deadlocks_and_Lock_Ordering.md>) *(PostgreSQL Platform)*

## References

- *Database Reliability Engineering* — Laine Campbell, Charity Majors — Release Management › Locking of objects
- *Database Reliability Engineering* — Laine Campbell, Charity Majors — Release Management › Pattern: locking operations
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 10: Transactions, Concurrency, and Isolation Levels › Lock Modes
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 10: Transactions, Concurrency, and Isolation Levels › Lock Hints
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 10: Transactions, Concurrency, and Isolation Levels › Schema and Metadata Locks
- *SQL Server 2025 Query Performance Tuning (7th ed.)* — Grant Fritchey — FULL_BOOK_NEEDS_MANUAL_SPLIT (cont.) › Chapter 16 Blocking And Blocked Processes
- *Designing Data-Intensive Applications* — Martin Kleppmann — Two-Phase Locking (2PL)

