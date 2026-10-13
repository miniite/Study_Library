---
id: "TOOLS/PostgreSQL/PostgreSQL_Platform/Deadlocks_and_Lock_Ordering"
level: 4
type: "concept"
topic: "TOOLS/PostgreSQL/PostgreSQL_Platform"
tools: ["PostgreSQL"]
---

# Deadlocks and Lock Ordering

**L4 Advanced** · [↑ PostgreSQL Platform](<_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** A deadlock is a cycle of transactions each waiting on a lock another holds; PostgreSQL detects it after `deadlock_timeout` and aborts one victim — you prevent them by always locking resources in the same order and keeping transactions short.

### 1. The big picture
PostgreSQL's MVCC means readers never block writers, but **writers still block writers**, and explicit locks (`SELECT … FOR UPDATE`, DDL, advisory locks) can block more. Concurrency bugs surface as three symptoms: *waiting* (a lock queue), *lost updates* (two read-modify-write cycles), and *deadlocks* (a cycle). This lesson covers how to see who blocks whom, how to design access order, and the standard patterns: `FOR UPDATE SKIP LOCKED` queues, optimistic locking, atomic `UPDATE … SET n = n + 1`, advisory locks, and `lock_timeout` for migrations.

### 2. Core concepts and vocabulary
- **Row-level lock** — taken by `UPDATE`, `DELETE`, `SELECT … FOR UPDATE/SHARE`; stored in the tuple header, not in shared memory.
- **Table-level lock modes** — `ACCESS SHARE` (SELECT) … `ACCESS EXCLUSIVE` (most DDL); they conflict per a fixed matrix.
- **Lock queue** — waiters are served in order; a waiting `ACCESS EXCLUSIVE` request blocks all later `SELECT`s behind it.
- **Deadlock** — circular wait; one transaction gets `ERROR: deadlock detected` (SQLSTATE `40P01`).
- **`deadlock_timeout`** — wait time (default 1 s) before the server checks for a cycle.
- **`lock_timeout` / `statement_timeout` / `idle_in_transaction_session_timeout`** — bounds on waiting, running, and idling inside a transaction.
- **Advisory lock** — application-defined lock on an arbitrary number, session- or transaction-scoped.
- **Lost update** — two transactions read the same value, both write, one overwrite disappears.
- **Optimistic locking** — a `version` column checked at write time instead of holding a lock.

### 3. How it works, step by step
**A classic deadlock**
1. T1: `UPDATE accounts SET … WHERE id = 1;` (holds row 1)
2. T2: `UPDATE accounts SET … WHERE id = 2;` (holds row 2)
3. T1: `UPDATE … WHERE id = 2;` → waits for T2.
4. T2: `UPDATE … WHERE id = 1;` → waits for T1. Cycle.
5. After `deadlock_timeout` one backend runs the detector, finds the cycle, and cancels one transaction (not necessarily the one that detected it). The survivor proceeds; the app must **retry** the victim.

**Prevention:** every code path acquires locks in one global order (e.g., ascending primary key), including inside one statement: `SELECT … FROM accounts WHERE id IN (1,2) ORDER BY id FOR UPDATE`.

**Finding the blocker**
1. `pg_stat_activity` shows state, wait event, query.
2. `pg_blocking_pids(pid)` returns who blocks a session.
3. `pg_locks` shows granted/waiting locks per relation/row/transaction.

### 4. Worked example
```sql
-- who is blocking whom?
SELECT a.pid, a.state, a.wait_event_type, a.wait_event,
       pg_blocking_pids(a.pid) AS blocked_by,
       now() - a.xact_start    AS xact_age,
       left(a.query, 80)       AS query
FROM pg_stat_activity a
WHERE cardinality(pg_blocking_pids(a.pid)) > 0;

-- job queue: two workers never claim the same row
WITH next AS (
  SELECT id FROM jobs
  WHERE status = 'queued'
  ORDER BY id
  FOR UPDATE SKIP LOCKED
  LIMIT 1)
UPDATE jobs SET status = 'running', started_at = now()
FROM next WHERE jobs.id = next.id
RETURNING jobs.*;

-- migration that fails fast instead of queuing the whole app
SET lock_timeout = '3s';
ALTER TABLE orders ADD COLUMN note text;

-- atomic counter, no lost update
UPDATE counters SET n = n + 1 WHERE name = 'visits';

-- optimistic locking
UPDATE docs SET body = $1, version = version + 1
WHERE id = $2 AND version = $3;     -- 0 rows -> conflict, return 409
```
Reading the output: if `blocked_by` lists a pid in state `idle in transaction`, the culprit is an application that opened a transaction and left it (often around an external HTTP call). Decide: wait if it's short; `pg_cancel_backend(pid)` to cancel its query; `pg_terminate_backend(pid)` to kill the connection (rolls back).

### 5. Under the hood   *(optional at this level)*
- Row locks live in the tuple (`xmax` + lock bits); to wait on a row, a backend actually waits on the *transaction id* of the holder via a heavyweight lock on that xid — hence `transactionid` wait events.
- The lock manager keeps a hash table of heavyweight locks; the deadlock checker builds a wait-for graph and looks for a cycle that includes the checking backend.
- `FOR UPDATE SKIP LOCKED` skips rows locked by others, giving a queue with no contention but **no fairness/ordering guarantee**; `NOWAIT` raises an error instead of waiting.
- An advisory lock taken at session scope survives `COMMIT` and, with a connection pool, stays attached to the pooled connection — use `pg_advisory_xact_lock` (transaction scope) instead.
- Foreign keys take `FOR KEY SHARE` locks on parent rows, which is why inserts into children can block/deadlock with parent updates.

### 6. Performance and design implications
- Long transactions hold locks and pin the vacuum horizon; never do network calls inside a transaction. Fetch, commit, call, then open a short transaction to write.
- Set `idle_in_transaction_session_timeout` (e.g. 30 s) to cap leaks.
- Keep transactions touching rows in sorted key order; batch multi-row updates with `ORDER BY` in a subselect.
- `SERIALIZABLE` isolation is correct but produces serialization failures (`40001`) that you **must** retry; `READ COMMITTED` plus explicit row locks is often the pragmatic default.
- Batch inserts with multi-row `VALUES` or `COPY`; batching trades latency for throughput.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `SELECT … FOR UPDATE [NOWAIT \| SKIP LOCKED]` | lock rows | read-modify-write, queues |
| `pg_blocking_pids(pid)` | list blockers | incident triage |
| `pg_cancel_backend` / `pg_terminate_backend` | cancel query / kill session | stuck blockers |
| `SET lock_timeout` | give up waiting | migrations |
| `log_lock_waits = on` | log waits > `deadlock_timeout` | find chronic contention |
| `pg_advisory_xact_lock(key)` | app-level mutex | serialize a job or migration |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| `UPDATE … SET n = n + 1` | you read the value into the app first |
| Optimistic locking (version column) | high conflict rates → many retries |
| `FOR UPDATE` pessimistic | holding it across user think-time |
| `SKIP LOCKED` queue | strict FIFO needed |
| Advisory lock | session-level locks with connection pooling |

### 9. Common mistakes
- Locking rows in different orders → deadlocks → global ordering.
- No retry on `40P01`/`40001` → user-visible 500s → wrap in a retry loop with backoff.
- `SELECT` then `UPDATE` for counters → lost updates → single atomic `UPDATE`.
- `ALTER TABLE` without `lock_timeout` → queues all traffic behind it → set `lock_timeout` and retry.
- Transaction left open across an HTTP call → blocks others, bloat → shorten.
- Relying on insertion order without `ORDER BY` → nondeterministic, especially with parallel plans → always specify order.

### 10. 🎯 Interview kit
**30-second answer:** "A deadlock is a circular lock wait; Postgres detects it after `deadlock_timeout` and aborts one transaction. Prevent by taking locks in a consistent order, keeping transactions short, and retrying on `40P01`."
**Structure a longer answer:** MVCC (what blocks what) → how to see waits (`pg_locks`, `pg_blocking_pids`) → prevention → patterns (SKIP LOCKED, optimistic) → timeouts.
**Likely follow-ups:** *How do you debug a blocked migration?* → blockers, `lock_timeout`. *Optimistic vs pessimistic?* → conflict rate. *Why is SERIALIZABLE not free?* → aborts, retries, predicate locks. *What happens if a client drops mid-transaction?* → server rolls back when it notices the closed connection.
**Signals interviewers look for:** retries, lock ordering, `idle in transaction` awareness, no locks held across I/O.

### 11. Check yourself
- [ ] Draw a two-transaction deadlock and fix it by ordering.
- [ ] Write a worker-queue query that is safe with 20 concurrent workers.
- [ ] Find and terminate the session blocking an `ALTER TABLE`.
- [ ] Explain why a session advisory lock and a pooler don't mix.
<!-- explainer:end -->

## Implementations (tools)

- [PostgreSQL](<../_index.md>)

## Practice questions

- [Intermediate] [Your CI runs migrations against a shared schema and they intermittently deadlock during CREATE INDEX; how would you serialize DDL and set lock_timeout so a migration fails fast instead of blocking the whole app?](<../../../PRACTICE/TOOLS/PostgreSQL/PostgreSQL_Platform/Q-36249_Your_CI_runs_migrations_against_a_shared_schema_and_they_int.md>)
- [Intermediate] [How would you find the exact statement holding a lock that's blocking a migration, using pg_locks joined to pg_stat_activity, and then decide between waiting, canceling, or killing the blocker?](<../../../PRACTICE/TOOLS/PostgreSQL/PostgreSQL_Platform/Q-35802_How_would_you_find_the_exact_statement_holding_a_lock_that_s.md>)
- [Deep] [Two concurrent SELECT ... FOR UPDATE statements on overlapping rows in different orders deadlock; how would you enforce a deterministic locking order via ORDER BY in the SELECT that acquires locks?](<../../../PRACTICE/TOOLS/PostgreSQL/PostgreSQL_Platform/Q-36289_Two_concurrent_SELECT_FOR_UPDATE_statements_on_overlapping_r.md>)
- [Deep] [What is the difference between READ COMMITTED and REPEATABLE READ behavior for a query run twice in the same transaction?](<../../../PRACTICE/TOOLS/PostgreSQL/PostgreSQL_Platform/Q-35865_What_is_the_difference_between_READ_COMMITTED_and_REPEATABLE.md>)
- [Deep] [What is the difference between optimistic and pessimistic locking?](<../../../PRACTICE/TOOLS/PostgreSQL/PostgreSQL_Platform/Q-36207_What_is_the_difference_between_optimistic_and_pessimistic_lo.md>)
- [Deep] [Your app fires LISTEN/NOTIFY for real-time updates but notifications get dropped when clients reconnect; explain the delivery semantics and when you'd move to a durable queue.](<../../../PRACTICE/TOOLS/PostgreSQL/PostgreSQL_Platform/Q-35990_Your_app_fires_LISTEN_NOTIFY_for_real-time_updates_but_notif.md>)
- [Deep] [You must guarantee at-most-once processing of a webhook while the underlying insert is retried by the network layer; design the dedup key, the unique constraint, and the ON CONFLICT behavior that makes a retried delivery a no-op.](<../../../PRACTICE/TOOLS/PostgreSQL/PostgreSQL_Platform/Q-36060_You_must_guarantee_at-most-once_processing_of_a_webhook_whil.md>)
- [Expert] [How does SELECT ... FOR UPDATE SKIP LOCKED help build a simple job queue in PostgreSQL?](<../../../PRACTICE/TOOLS/PostgreSQL/PostgreSQL_Platform/Q-35868_How_does_SELECT_FOR_UPDATE_SKIP_LOCKED_help_build_a_simple_j.md>)

## Related concepts

- [Lock Granularity](<../../../01_Databases_and_SQL/Database_Internals/DBMS_Concepts/Lock_Granularity.md>) *(DBMS Concepts)*
- [Lock Holding and Optimistic Concurrency](<../../../01_Databases_and_SQL/Transactions_and_Concurrency/ACID_Transactions/Lock_Holding_and_Optimistic_Concurrency.md>) *(ACID Transactions)*
- [SKIP LOCKED Job Queues](<../../../01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/SKIP_LOCKED_Job_Queues.md>) *(SQL Fundamentals)*

## References

- *Designing Data-Intensive Applications* — Martin Kleppmann — Preventing Lost Updates › Automatically detecting lost updates

