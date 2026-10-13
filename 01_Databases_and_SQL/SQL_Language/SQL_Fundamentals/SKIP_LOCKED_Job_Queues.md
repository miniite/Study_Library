---
id: "01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/SKIP_LOCKED_Job_Queues"
level: 4
type: "concept"
topic: "01_Databases_and_SQL/SQL_Language/SQL_Fundamentals"
tools: []
---

# SKIP LOCKED Job Queues

**L4 Advanced** · [↑ SQL Fundamentals](<_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** `SELECT … FOR UPDATE SKIP LOCKED` (or `READPAST` in SQL Server) lets many workers claim different rows from the same table at once without blocking each other, which turns an ordinary table into a safe job queue.

### 1. The big picture
You have a table of work: emails to send, files to process, remediation actions to run. Several worker processes must each take *different* rows, process them, and mark them done, even when workers crash or restart. Without help, two workers `SELECT` the same "next pending" row and both do the work, or they serialize on a lock and the queue runs at one worker's speed.

Row locks are the building block: a transaction that locks a row prevents others from locking or changing it. The default behaviour for a second transaction that wants the same row is to **wait**. `SKIP LOCKED` changes that to "pretend that row isn't there and give me the next one". Each worker therefore gets a disjoint set of rows with no coordination service.

This sits on top of concepts from concurrency control (row locks, lost updates, deadlocks). It is a classic example of *using the lock manager as a work distributor*.

### 2. Core concepts and vocabulary
- **Row lock** — a lock on one row held until the transaction ends. `FOR UPDATE` takes an exclusive row lock on every row the `SELECT` returns.
- **Lost update** — two transactions read a value, both write based on the stale read, one write is lost.
- **Pessimistic locking** — lock first, then work. **Optimistic** — work, then check a version at write time.
- **`SKIP LOCKED`** — skip rows already locked by other transactions instead of waiting (PostgreSQL 9.5+, MySQL 8.0+, Oracle).
- **`NOWAIT`** — raise an error instead of waiting.
- **`READPAST`** — SQL Server's table hint with the same effect as `SKIP LOCKED`.
- **Claim** — atomically marking a row as taken by a worker.
- **Visibility timeout / lease** — a deadline after which a claimed-but-unfinished job is considered abandoned and eligible again.
- **Dead letter** — a job moved aside after too many failed attempts.
- **Thundering herd** — many idle workers waking at once and hammering the table.

### 3. How it works, step by step
There are two designs.

**Design 1: hold the transaction open while working (simple, risky).**
1. `BEGIN`.
2. `SELECT id, payload FROM jobs WHERE status='pending' ORDER BY id LIMIT 1 FOR UPDATE SKIP LOCKED;`
3. Do the work.
4. `UPDATE jobs SET status='done' WHERE id = :id;`
5. `COMMIT`.

If the worker crashes, the connection drops and the lock vanishes, so the row is automatically available again. But a long transaction holds a connection, delays vacuum/undo cleanup, and cannot be used for work that takes minutes.

**Design 2: claim with a short transaction, then work outside it (preferred).**
1. In one short transaction, atomically flip `status` from `pending` to `running`, stamp `locked_by` and `locked_until`, and return the rows.
2. Commit immediately; no lock is held while working.
3. Process the job.
4. In another statement, set `status='done'` (or schedule a retry with `attempts+1`).
5. A reaper query periodically resets jobs whose `locked_until` has passed.

### 4. Worked example
```sql
CREATE TABLE jobs (
  id           bigserial PRIMARY KEY,
  status       text        NOT NULL DEFAULT 'pending',   -- pending | running | done | failed
  run_after    timestamptz NOT NULL DEFAULT now(),
  attempts     int         NOT NULL DEFAULT 0,
  locked_by    text,
  locked_until timestamptz,
  payload      jsonb       NOT NULL
);

-- Small partial index: only pending rows, in claim order
CREATE INDEX jobs_claim_idx ON jobs (run_after, id) WHERE status = 'pending';

-- Claim up to 10 jobs for 5 minutes (PostgreSQL)
WITH next AS (
  SELECT id
  FROM   jobs
  WHERE  status = 'pending'
    AND  run_after <= now()
  ORDER  BY run_after, id
  LIMIT  10
  FOR UPDATE SKIP LOCKED
)
UPDATE jobs j
SET    status       = 'running',
       attempts     = j.attempts + 1,
       locked_by    = 'worker-7',
       locked_until = now() + interval '5 minutes'
FROM   next
WHERE  j.id = next.id
RETURNING j.id, j.payload;
```

What happens: the inner `SELECT` locks up to 10 qualifying rows, skipping any that another worker's claim currently holds. The outer `UPDATE` changes them in the same statement and `RETURNING` hands them to the worker. Two workers running this simultaneously receive disjoint sets.

Reaper:

```sql
UPDATE jobs
SET    status = CASE WHEN attempts >= 5 THEN 'failed' ELSE 'pending' END,
       locked_by = NULL, locked_until = NULL,
       run_after = now() + (interval '30 seconds' * attempts)   -- backoff
WHERE  status = 'running' AND locked_until < now();
```

SQL Server version of the claim:

```sql
;WITH next AS (
  SELECT TOP (10) *
  FROM   dbo.jobs WITH (ROWLOCK, UPDLOCK, READPAST)
  WHERE  status = 'pending' AND run_after <= SYSUTCDATETIME()
  ORDER  BY run_after, id
)
UPDATE next
SET    status = 'running', attempts = attempts + 1,
       locked_by = 'worker-7', locked_until = DATEADD(minute, 5, SYSUTCDATETIME())
OUTPUT inserted.id, inserted.payload;
```

To see contention, SQL Server exposes `row_lock_count`, `row_lock_wait_count`, `row_lock_wait_in_ms` per index in `sys.dm_db_index_operational_stats`; with `READPAST` the wait counters for the queue index should stay near zero.

### 5. Under the hood
- In PostgreSQL the row lock is stored in the tuple header (`xmax` plus lock-mode bits) rather than a separate lock table, so locking a million rows doesn't blow up lock memory, but it dirties pages (hence write amplification and WAL).
- `SKIP LOCKED` tests each candidate tuple's lock status as the scan proceeds. Because it skips rows, `LIMIT 10` returns the next 10 *unlocked* rows. The result set is non-deterministic and therefore **not a consistent view** of the table. That is perfect for queues, wrong for reports.
- `ORDER BY` plus `FOR UPDATE` plus `LIMIT` locks rows in index order, so concurrent claimers meet in the same order: no deadlocks between claimers. Without a consistent order, two sessions locking sets of rows in different orders can deadlock.
- Dead tuples: every claim and completion is an update, which creates a new row version. A high-churn queue table needs aggressive autovacuum settings (or partitioning/`TRUNCATE` of old partitions); otherwise the partial index fills with dead entries and claims slow down.
- In SQL Server, `READPAST` only skips *row/page* locks and works under read committed; with `UPDLOCK` the scan takes update locks on the rows it keeps.

### 6. Performance and design implications
- Throughput is bounded by index maintenance and WAL, typically thousands to tens of thousands of claims per second on one table. Past that, use a broker (Kafka, SQS, RabbitMQ).
- Keep the partial index on `pending` rows only so it stays small when most jobs are done.
- Delete or archive finished jobs on a schedule; a queue table should be small.
- Claim in **batches** to amortize round trips, but not so big that one worker hoards work and one crash delays many jobs.
- Make processing **idempotent** (idempotency key, upsert) because "at least once" is what you get when a worker dies after doing the work but before marking done.
- Use `LISTEN/NOTIFY` (PostgreSQL) or a small poll with jitter to avoid thundering-herd polling.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `FOR UPDATE SKIP LOCKED` | Lock rows, skip ones already locked | Queue claims |
| `FOR UPDATE NOWAIT` | Fail immediately if locked | Fail-fast operations |
| `FOR NO KEY UPDATE` | Weaker lock that doesn't block FK checks (PostgreSQL) | Updating non-key columns |
| `WITH (ROWLOCK, UPDLOCK, READPAST)` | SQL Server equivalent | Queue claims on SQL Server |
| `lock_timeout` / `SET LOCK_TIMEOUT` | Max wait for a lock | Bound waits for non-skipping paths |
| `pg_locks`, `sys.dm_tran_locks` | See current locks | Debug stuck jobs |
| `LISTEN`/`NOTIFY` | Wake workers on insert | Reduce polling |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| Moderate volume, you already run the DB, need transactional enqueue (job inserted in the same transaction as business data) | Very high throughput, fan-out to many consumers, replay needs (use a log/broker) |
| Design 2 (claim then work) for long jobs | Design 1 only for jobs shorter than a few seconds |
| Strict ordering not required | You need global FIFO: skipping locked rows reorders processing |
| Single table with status column | Hundreds of job types with different SLAs — separate tables/queues |

### 9. Common mistakes
- Using `SELECT … FOR UPDATE` without `SKIP LOCKED` → workers serialize behind one another → add `SKIP LOCKED`.
- Holding the claim transaction open while calling external APIs → connection and vacuum pressure, long locks → claim short, work outside.
- No lease/timeout → a crashed worker leaves jobs `running` forever → `locked_until` + reaper.
- No attempt cap → a poison job retries forever → `attempts` limit and dead-letter status.
- Not idempotent → duplicate emails/charges after a retry → idempotency key on the side effect.
- Forgetting `ORDER BY` → arbitrary lock order → deadlocks and starvation.
- Using it for reporting → silently missing rows that happened to be locked.

### 10. 🎯 Interview kit
**30-second answer:** Put jobs in a table with a status. Workers run a claim statement `SELECT … WHERE status='pending' ORDER BY … LIMIT n FOR UPDATE SKIP LOCKED` wrapped in an `UPDATE … RETURNING` that sets running and a lease. Each worker gets disjoint rows without waiting; a reaper recovers expired leases and a retry cap dead-letters poison jobs.

**Structure a longer answer:** the problem (duplicates or serialization) → what SKIP LOCKED changes → claim statement → failure modes (crash, timeout, poison, herd) → idempotency → when to move to a broker.

**Likely follow-ups:**
- *Why not just `UPDATE … WHERE id = (SELECT … LIMIT 1)`?* → two sessions can pick the same row; the second blocks and then (in some isolation levels) updates it again or returns zero rows.
- *How do you avoid deadlocks?* → consistent order, short transactions.
- *What does skip-locked do to ordering?* → breaks strict FIFO.
- *Exactly-once?* → not from the queue; achieve effectively-once via idempotent consumers.

**Signals interviewers look for:** lease/visibility timeout, attempts and dead-letter, short transactions, idempotent handlers, honest limits vs a broker.

### 11. Check yourself
- [ ] Write the claim statement for your engine and explain each clause.
- [ ] Describe what happens when a worker dies after claiming but before finishing.
- [ ] Explain why `LIMIT` with `SKIP LOCKED` gives a non-deterministic result.
- [ ] Choose an index for the claim query and justify a partial index.
- [ ] State two reasons you would outgrow a table-based queue.
<!-- explainer:end -->

## Practice questions

- [Basic] [What is a trace in observability, and how would you count the number of spans per trace_id from a spans table?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Q-40970_What_is_a_trace_in_observability_and_how_would_you_count_the.md>)
- [Deep] [Your app deadlocks every few minutes between two transactions. Walk through reading the deadlock graph, identifying the lock-ordering conflict, and the code or index change you'd ship.](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Q-39887_Your_app_deadlocks_every_few_minutes_between_two_transaction.md>)
- [Deep] [What is the difference between optimistic and pessimistic locking?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Q-40398_What_is_the_difference_between_optimistic_and_pessimistic_lo.md>)
- [Deep] [How do watermarks in a streaming SQL engine decide when a window closes, and what happens to a late event that arrives after the watermark and the allowed lateness both pass?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Q-40101_How_do_watermarks_in_a_streaming_SQL_engine_decide_when_a_wi.md>)
- [Deep] [Explain why SELECT ... FOR UPDATE inside a loop over rows can deadlock when different sessions lock rows in different orders, and how ordering the SELECT (e.g. ORDER BY id) plus SKIP LOCKED avoids it.](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Q-40454_Explain_why_SELECT_FOR_UPDATE_inside_a_loop_over_rows_can_de.md>)
- [Deep] [Explain how you'd enforce a remediation rate-limit in SQL (no more than N restarts of the same service per hour) using a windowed running count, to prevent an automation loop from hammering a flapping service.](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Q-41085_Explain_how_you_d_enforce_a_remediation_rate-limit_in_SQL_no.md>)
- [Expert] [How does SELECT ... FOR UPDATE work, and how would you use it to safely decrement inventory?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Q-40257_How_does_SELECT_FOR_UPDATE_work_and_how_would_you_use_it_to_.md>)
- [Expert] [Walk me through diagnosing a lock wait chain where one long-running analytical query blocks hundreds of short OLTP writes. What views reveal the blocking tree and how do you break it safely?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Q-40326_Walk_me_through_diagnosing_a_lock_wait_chain_where_one_long-.md>)

## Related concepts

- [MERGE Pitfalls and Error Handling](<../../../TOOLS/SQL_Server/T-SQL/MERGE_Pitfalls_and_Error_Handling.md>) *(T-SQL)*
- [Read Committed Locking Anomalies](<../../../TOOLS/SQL_Server/SQL_Server_Platform/Read_Committed_Locking_Anomalies.md>) *(SQL Server Platform)*
- [Lock Holding and Optimistic Concurrency](<../../Transactions_and_Concurrency/ACID_Transactions/Lock_Holding_and_Optimistic_Concurrency.md>) *(ACID Transactions)*

## References

- *Expert Performance Indexing in SQL Server* — Jason Strate, Grant Fritchey — Row Lock
- *Expert Performance Indexing in SQL Server* — Jason Strate, Grant Fritchey — Page Lock
- *SQL Server 2025 Query Performance Tuning (7th ed.)* — Grant Fritchey — FULL_BOOK_NEEDS_MANUAL_SPLIT (cont.) › Chapter 16 Blocking And Blocked Processes
- *Analytics Optimization with Columnstore Indexes in Microsoft SQL Server* — Edward Pollack — Columnstore metadata › Rowgroup Operational Statistics
- *Think Distributed Systems* — Dominik Tornow — Consistency › 9.2.1 Queue and stack
- *Database Internals* — Alex Petrov — I

