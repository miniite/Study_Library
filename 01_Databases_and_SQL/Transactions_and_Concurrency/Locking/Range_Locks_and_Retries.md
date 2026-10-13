---
id: "01_Databases_and_SQL/Transactions_and_Concurrency/Locking/Range_Locks_and_Retries"
level: 4
type: "concept"
topic: "01_Databases_and_SQL/Transactions_and_Concurrency/Locking"
tools: ["MySQL"]
---

# Range Locks and Retries

**L4 Advanced** · [↑ Locking](<_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** Range (key-range, gap, next-key) locks stop other transactions from inserting rows into a range you have read, which prevents phantoms but invites blocking and deadlocks — so code that uses them must be built to retry.

### 1. The big picture
Row locks protect rows that *exist*. But a query like `WHERE status = 'open'` also depends on rows that do *not yet* exist: if another transaction inserts a new open row, your next read of the same predicate returns something different (a **phantom**). To prevent phantoms a database has to lock not just rows but the *gaps between index keys* — the "range".

Locks that are held longer (Repeatable Read, Serializable) and cover more (ranges) mean more waiting, and the possibility of **deadlock**: two transactions each holding something the other wants. Deadlocks and serialization failures are not bugs to eliminate entirely; they're a normal outcome of concurrency under strong isolation. The robust pattern is: keep transactions short, take locks in a consistent order, **and retry** the victim.

This lesson covers (a) why a Repeatable Read read-modify-write deadlocks, (b) how each engine implements range locks, and (c) how to write a retry loop that is correct.

### 2. Core concepts and vocabulary
- **Shared (S) lock** — taken to read; many holders allowed.
- **Exclusive (X) lock** — taken to write; sole holder.
- **Update (U) lock** — "I read now and will probably write"; prevents the S→X conversion deadlock.
- **Phantom read** — new rows matching a predicate appear between two reads in a transaction.
- **Key-range lock (SQL Server)** — lock on an index key *and* the gap before it, named like `RangeS-S`, `RangeS-U`, `RangeI-N`, `RangeX-X`.
- **Gap lock / next-key lock (InnoDB)** — gap lock covers the space between index records; next-key = record lock + gap lock before it.
- **Predicate lock (SIREAD)** — PostgreSQL's non-blocking marker for what a Serializable transaction read.
- **Lock timeout / `NOWAIT`** — bounds how long to wait for a lock.
- **Deadlock victim** — the transaction the engine rolls back to break a cycle.
- **Transient error** — failure expected to disappear on retry (deadlock, serialization failure, lock timeout, dropped connection).

### 3. How it works, step by step
**Repeatable Read read-modify-write deadlock**
1. T1 reads a row under Repeatable Read: takes S and *keeps it* until commit.
2. T2 does the same read: also takes S (compatible).
3. T1 tries to `UPDATE`: needs X, blocked by T2's S.
4. T2 tries to `UPDATE`: needs X, blocked by T1's S. → cycle → deadlock.
5. The engine picks a victim (SQL Server: cheapest to roll back, error 1205), rolls it back, the other continues.

**Key-range locks on a Serializable range scan (SQL Server)**
1. `SELECT … WHERE Price BETWEEN 10 AND 20` uses an index on Price.
2. For each key found, take `RangeS-S` (lock key + gap before it).
3. Also lock the next key after the last qualifying one, to protect the gap at the end.
4. A concurrent `INSERT` of Price = 15 needs `RangeI-N` (insert-intent on the gap) → blocked until the reader commits.
5. If no index supports the predicate, the whole table (or its entire scanned set) is range-locked.

**Retry**
1. Start a transaction.
2. Do the work.
3. If a *transient* error occurs: roll back (the transaction is already dead for deadlocks), wait with exponential backoff + jitter, repeat from step 1 with a limit.
4. If non-transient (constraint violation, syntax): fail immediately.

### 4. Worked example
**Lost update, Repeatable Read, and the deadlock fix** (modelled on a price-normalisation scenario):

```sql
CREATE TABLE dbo.MyProduct (ProductID int PRIMARY KEY, Price money);
INSERT dbo.MyProduct VALUES (1, 15.0), (2, 22.0), (3, 9.99);

-- Connection 1 (REPEATABLE READ)
SET TRANSACTION ISOLATION LEVEL REPEATABLE READ;
BEGIN TRAN;
DECLARE @p money;
SELECT @p = Price FROM dbo.MyProduct WHERE ProductID = 1;  -- holds S on key 1
WAITFOR DELAY '00:00:10';
IF @p > 10 UPDATE dbo.MyProduct SET Price = Price - 10 WHERE ProductID = 1;  -- needs X
COMMIT;

-- Connection 2, started during the delay
BEGIN TRAN;
UPDATE dbo.MyProduct SET Price = Price * 0.6;   -- needs X on key 1: BLOCKED by C1's S
COMMIT;
```

C2 waits for C1; if C1's `UPDATE` then needs something C2 already holds, you get a deadlock. Lower the isolation level and you'd allow a lost update instead. The simple fix is to read with intent to update:

```sql
SELECT @p = Price FROM dbo.MyProduct WITH (UPDLOCK) WHERE ProductID = 1;
```

`UPDLOCK` takes a U lock: compatible with S readers, but only one U at a time, so the second writer waits *before* holding anything and the cycle cannot form.

**Retry wrapper (T-SQL):**

```sql
DECLARE @try int = 0, @max int = 5, @done bit = 0;
WHILE @done = 0 AND @try < @max
BEGIN
  BEGIN TRY
    BEGIN TRAN;
      -- critical work, idempotent, no external side effects
    COMMIT;
    SET @done = 1;
  END TRY
  BEGIN CATCH
    IF XACT_STATE() <> 0 ROLLBACK;
    IF ERROR_NUMBER() IN (1205, 1222, 3960)       -- deadlock, lock timeout, snapshot conflict
    BEGIN
      SET @try += 1;
      WAITFOR DELAY '00:00:00.100';               -- backoff (use app side for jitter)
    END
    ELSE THROW;
  END CATCH
END
IF @done = 0 THROW 50001, 'Retries exhausted', 1;
```

### 5. Under the hood
- **Lock compatibility** is a matrix: S–S ok; S–X, X–X conflict; U–S ok but U–U conflict; intent locks (IS/IX) live at higher levels so a table lock request can be decided without scanning rows.
- **SQL Server ranges** exist only at Serializable (or `HOLDLOCK`). A Repeatable Read scan locks only existing keys, so it still has phantoms — range protection is the difference between RR and Serializable.
- **InnoDB** at Repeatable Read uses next-key locks for locking reads and `UPDATE/DELETE`, so it prevents most phantoms there; unique-index equality lookups use only record locks. Gap locks never conflict with each other, only with inserts — which is why two sessions can hold the same gap and then deadlock on insert.
- **PostgreSQL** doesn't lock ranges; Serializable records predicate reads and aborts one transaction at commit when it sees a dangerous dependency, so it fails with `40001` instead of blocking.
- **Deadlock detection** is a wait-for-graph search run periodically (SQL Server about every 5 seconds, sooner under contention; InnoDB immediately on lock wait; PostgreSQL after `deadlock_timeout`, default 1 s). It selects a victim by cost/priority (`SET DEADLOCK_PRIORITY`).
- **Lock waits without deadlock** end only by commit, rollback, or timeout; SQL Server's default `LOCK_TIMEOUT` is -1 (wait forever), PostgreSQL's `lock_timeout` default 0 (disabled), InnoDB `innodb_lock_wait_timeout` 50 s.

### 6. Performance and design implications
- Make predicates **index-supported**; otherwise ranges become table-wide.
- Hold range locks as briefly as possible: do the validation read and the write in the same short transaction, no user think-time.
- Always touch tables/rows in a consistent order (e.g. parent then child, ascending key).
- Retries must be *bounded* and *jittered*; unbounded immediate retry turns a hot spot into a thundering herd.
- Measure: SQL Server `xml_deadlock_report` extended event, `sys.dm_tran_locks` (`RangeS-S`…), `sys.dm_os_wait_stats` for `LCK_M_RS_*` / `LCK_M_RI_*`; InnoDB `SHOW ENGINE INNODB STATUS` (LATEST DETECTED DEADLOCK); PostgreSQL `log_lock_waits`.
- Consider alternatives that need no ranges: a unique constraint to enforce "no duplicate", or `INSERT … ON CONFLICT` / `MERGE` with a key, instead of "check then insert".

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `SET TRANSACTION ISOLATION LEVEL REPEATABLE READ / SERIALIZABLE` | Holds S locks / adds range locks | Need stable reads |
| `WITH (UPDLOCK, HOLDLOCK)` | U lock + range protection for one statement | "Check then insert" in SQL Server |
| `SET LOCK_TIMEOUT 2000` | Fail after 2 s (error 1222) | UI actions |
| `SELECT … FOR UPDATE NOWAIT / SKIP LOCKED` | Fail immediately / skip locked rows | Work queues, user-facing actions |
| `SET DEADLOCK_PRIORITY LOW` | Be the preferred victim | Batch jobs |
| `deadlock_timeout`, `innodb_deadlock_detect` | Detector tuning | Diagnostics |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| `UPDLOCK` / `FOR UPDATE` on a known row | Locking large ranges "just in case" |
| `NOWAIT` for interactive actions | Background jobs where waiting is fine |
| Serializable range protection for a rare critical check | Highly contended insert streams into the same key range |
| Retry with backoff | Operations with non-idempotent side effects inside the transaction |

### 9. Common mistakes
- Using Repeatable Read for read-then-update → S→X deadlocks → read with `UPDLOCK`/`FOR UPDATE` or use an atomic `UPDATE … SET x = x - 10 WHERE x > 10`.
- Retrying without rolling back/reopening the transaction → "transaction aborted" errors → treat each attempt as a fresh transaction.
- Retrying all errors → masks real bugs → retry only a known transient list.
- Ranges without an index → table-wide blocking → add an index on the predicate.
- Doing HTTP calls or user prompts inside a locked transaction → hours-long lock holds → gather input first, then transact.

### 10. 🎯 Interview kit
**30-second answer:** Range locks lock the gaps between index keys so no one can insert into a range you read, preventing phantoms at Serializable. They increase blocking and deadlocks, so the design must keep transactions short, use consistent access order and indexes, and retry transient failures with backoff.

**Structure a longer answer:** phantom example → how the engine locks (key-range / gap / predicate) → what that does to concurrency → deadlock example and fix → retry design.

**Likely follow-ups:**
- *Why does Repeatable Read still allow phantoms in SQL Server?* → it locks existing rows only.
- *How do you avoid S→X deadlocks?* → U/`FOR UPDATE` locks or atomic update.
- *NOWAIT vs lock timeout for a UI?* → fail fast with a friendly retry vs bounded wait; choose by user experience.
- *How do you ensure retries are safe?* → idempotent work, side effects after commit, retry caps.

**Signals interviewers look for:** you can name the lock modes, tie them to isolation levels, and describe a retry policy including backoff and idempotency.

### 11. Check yourself
- [ ] Walk through the S→X deadlock between two Repeatable Read transactions.
- [ ] Say what `RangeI-N` is and what blocks it.
- [ ] Explain why an index changes how much a Serializable query locks.
- [ ] List four error codes you'd treat as transient and one you wouldn't.
- [ ] Write a retry loop with bounded attempts and jittered backoff.
<!-- explainer:end -->

## Implementations (tools)

- [MySQL](<../../../TOOLS/MySQL/_index.md>)

## Practice questions

- [Basic] [What are the four ANSI SQL isolation levels and how do they relate to locking?](<../../../PRACTICE/01_Databases_and_SQL/Transactions_and_Concurrency/Locking/Q-31187_What_are_the_four_ANSI_SQL_isolation_levels_and_how_do_they_.md>)
- [Intermediate] [How does the SERIALIZABLE isolation level use locking to prevent anomalies?](<../../../PRACTICE/01_Databases_and_SQL/Transactions_and_Concurrency/Locking/Q-31209_How_does_the_SERIALIZABLE_isolation_level_use_locking_to_pre.md>)
- [Intermediate] [What is the REPEATABLE READ isolation level and what does it guarantee?](<../../../PRACTICE/01_Databases_and_SQL/Transactions_and_Concurrency/Locking/Q-31338_What_is_the_REPEATABLE_READ_isolation_level_and_what_does_it.md>)
- [Intermediate] [How would you route reads to primary versus replica when a transaction just wrote data and must not read stale rows?](<../../../PRACTICE/01_Databases_and_SQL/Transactions_and_Concurrency/Locking/Q-31216_How_would_you_route_reads_to_primary_versus_replica_when_a_t.md>)
- [Deep] [What is a write skew anomaly in simple terms?](<../../../PRACTICE/01_Databases_and_SQL/Transactions_and_Concurrency/Locking/Q-31343_What_is_a_write_skew_anomaly_in_simple_terms.md>)
- [Deep] [A read replica shows stale data that breaks a read-after-write flow. How does replication lag interact with your concurrency assumptions and how do you fix it?](<../../../PRACTICE/01_Databases_and_SQL/Transactions_and_Concurrency/Locking/Q-31215_A_read_replica_shows_stale_data_that_breaks_a_read-after-wri.md>)
- [Deep] [Explain the phenomenon where increasing an isolation level fixes a data bug but introduces frequent serialization retries, and how you balance the two.](<../../../PRACTICE/01_Databases_and_SQL/Transactions_and_Concurrency/Locking/Q-31268_Explain_the_phenomenon_where_increasing_an_isolation_level_f.md>)
- [Expert] [A write skew bug lets two on-call engineers both go off-duty. Show how you would reproduce it and which isolation change or explicit lock fixes it.](<../../../PRACTICE/01_Databases_and_SQL/Transactions_and_Concurrency/Locking/Q-31193_A_write_skew_bug_lets_two_on-call_engineers_both_go_off-duty.md>)

## Related concepts

- [Isolation Levels in Transactions](<../ACID_Transactions/Isolation_Levels_in_Transactions.md>) *(ACID Transactions)*
- [Phantoms and Double Booking](<../Isolation_Levels/Phantoms_and_Double_Booking.md>) *(Isolation Levels)*
- [Setting Isolation Levels](<../Isolation_Levels/Setting_Isolation_Levels/_index.md>) *(Isolation Levels)*
- [Isolation Anomalies](<../../SQL_Language/SQL_Fundamentals/Isolation_Anomalies.md>) *(SQL Fundamentals)*

## References

- *T-SQL Fundamentals (4th ed.)* — Itzik Ben-Gan — Transactions and concurrency
- *SQL Server 2025 Query Performance Tuning (7th ed.)* — Grant Fritchey — FULL_BOOK_NEEDS_MANUAL_SPLIT (cont.) › Chapter 16 Blocking And Blocked Processes
- *Designing Data-Intensive Applications* — Martin Kleppmann — Write Skew and Phantoms
- *Designing Data-Intensive Applications* — Martin Kleppmann — Write Skew and Phantoms › Phantoms causing write skew
- *Designing Data-Intensive Applications* — Martin Kleppmann — Two-Phase Locking (2PL) › Predicate locks
- *Designing Data-Intensive Applications* — Martin Kleppmann — Observing Derived State › Reads are events too
- *100 SQL Server Mistakes and How to Avoid Them* — Peter A. Carter — Optimization › 10.10 #63 Using Read Uncommitted
- *SQL Server Tutorial* (YouTube series) — Part 73: Non repeatable read example in sql server ([watch](https://www.youtube.com/watch?v=d5QNpsezNTs))
- *SQL Server Tutorial* (YouTube series) — Part 74: Phantom reads example in sql server ([watch](https://www.youtube.com/watch?v=_UQ9Pu2W7Zg))
- *SQL Server Tutorial* (YouTube series) — Part 75: Snapshot isolation level in sql server ([watch](https://www.youtube.com/watch?v=9NVu17LjPSA))

