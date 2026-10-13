---
type: "practice"
difficulty: "Deep"
---

# What is a deadlock in a database and how can you prevent it?

**Difficulty:** Deep · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Isolation Anomalies](<../../../../01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Isolation_Anomalies.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

#### Triage

What is being asked: Explain what a database deadlock is, why it happens, and the concrete techniques to prevent (and handle) it.

What the interviewer is probing: Do you understand lock acquisition and the four Coffman conditions well enough to reason about a real deadlock trace? Can you go beyond "the DB will detect it" and give design fixes (consistent lock ordering, shorter transactions, right isolation, retry-on-40001) that you'd actually apply in a payments/orders codebase? Uber, Amazon, and every fintech (Razorpay, PhonePe, CRED, Zerodha, Stripe) ask this because deadlocks show up under load on hot rows — wallets, inventory, seat maps — and the fix is architectural, not magic.

#### Assumptions
/ clarifying questions: I'll assume a locking-based OLTP engine (InnoDB, PostgreSQL, SQL Server). Clarify: is the deadlock between explicit FOR UPDATE/UPDATE row locks, or gap/next-key locks from range scans (InnoDB REPEATABLE READ inserts)? And is the app doing multi-statement transactions where the lock order varies by code path? Those two questions almost always locate the cause.

Plan: Define deadlock via a two-transaction cycle → the Coffman conditions that make it possible → how engines detect it (wait-for graph, victim, timeout) → the prevention toolkit ranked by impact (consistent ordering, atomic/short transactions, lower lock footprint, retry) → summary + common causes.

What a deadlock is

A deadlock is a cycle of transactions each holding a lock the next one needs, so none can proceed. The simplest case is two transactions grabbing two rows in opposite order:

CODE
T1: locks row A  (UPDATE A)        T2: locks row B  (UPDATE B)
T1: wants row B  --> waits         T2: wants row A  --> waits

Now T1 waits for B (held by T2) and T2 waits for A (held by T1): a wait-for cycle. Left alone, both would wait forever. This differs from a livelock (transactions keep retrying and interfering, never progressing) and from ordinary lock contention/blocking (one transaction simply waits for another and does eventually get the lock — no cycle).

Why deadlocks happen — the four Coffman conditions

A deadlock is possible only when all four hold simultaneously:

Mutual exclusion — a lock (e.g., an exclusive row lock) can be held by only one transaction.
Hold and wait — a transaction holds locks while requesting more.
No preemption — a held lock can't be forcibly taken; the holder must release it.
Circular wait — a closed chain T1→T2→…→T1 where each waits on the next.

Prevention works by breaking at least one of these (usually circular wait or hold-and-wait), because the database can't relax mutual exclusion or preemption for correctness.

How databases handle deadlocks (detection)

Engines don't prevent deadlocks by default — they detect and resolve them so the system doesn't hang:

Wait-for graph detection (InnoDB, PostgreSQL, SQL Server): the engine maintains a graph of "who waits for whom." When it finds a cycle, it picks a victim (usually the transaction that has done the least work / holds the fewest rows / is cheapest to roll back), aborts and rolls it back, and returns a deadlock error. InnoDB: error 1213 ER_LOCK_DEADLOCK; PostgreSQL: SQLSTATE 40P01; SQL Server: error 1205.
Timeout-based (some engines / as a fallback): if a lock wait exceeds innodb_lock_wait_timeout, the waiter is aborted. Cruder — it can't tell a true deadlock from slow contention.

Key consequence for the app: deadlocks are a normal, expected runtime event under concurrency. Any transaction that touches contended rows must be wrapped in retry logic, because the engine may choose it as the victim.

How to prevent deadlocks (design toolkit, ranked)

1. Consistent lock ordering (breaks circular wait — the #1 fix). Always acquire locks/rows in the same global order across all code paths — e.g., always lock accounts by ascending id. In a transfer from A to B, lock min(A,B) first, then max(A,B). If every transaction agrees on order, no cycle can form. Most real production deadlocks are two code paths locking the same two rows in opposite order.

2. Keep transactions short and touch fewer rows (breaks hold-and-wait). Do all reads/computation before opening the transaction; inside the transaction only do the minimal lock-and-write, then commit fast. Long transactions hold locks longer and widen the deadlock window. Never wait on user input, network calls, or slow work while holding locks.

3. Prefer atomic single-statement updates over read-then-write. UPDATE accounts SET balance = balance - 100 WHERE id = 42 AND balance >= 100 takes and releases the lock in one shot and needs no explicit SELECT ... FOR UPDATE, shrinking the lock hold and eliminating one order-of-locking hazard.

4. Acquire all needed locks up front (breaks hold-and-wait). Take every lock the transaction needs at the start (e.g., a single SELECT ... FOR UPDATE over all rows, in canonical order), rather than incrementally as you go. Some engines/DBs support LOCK TABLE/SELECT ... FOR UPDATE ... NOWAIT to fail fast instead of waiting.

5. Lower the lock footprint via indexing and isolation. Missing indexes force full/range scans that lock far more rows than needed — a huge source of surprise deadlocks. Add the index so the write locks exactly one row. On InnoDB, REPEATABLE READ takes gap/next-key locks that deadlock on inserts into scanned ranges; dropping to READ COMMITTED reduces gap locking. Coarser locks (table locks) also deadlock less on rows but kill concurrency — a trade-off, not a default.

6. Reduce concurrency on hot rows / use different concurrency control. For extreme hotspots (one wallet, one inventory counter), sharded counters, queue-based serialization of writes to that key, or optimistic concurrency (version column: read version, UPDATE ... WHERE version = v, retry on 0 rows) avoid holding pessimistic locks entirely — no lock, no deadlock, just retries.

7. Retry on deadlock (mandatory safety net). Because detection aborts a victim, wrap the transaction in a bounded retry loop with small randomized backoff. This turns a deadlock from an error into a brief latency blip.

A concrete example — money transfer done right

The classic transfer deadlock (transfer(A→B) locks A then B; a concurrent transfer(B→A) locks B then A) is eliminated by canonical ordering: both always lock the lower id first. See code artifact. Add a retry wrapper and you have a production-safe transfer.

Summary / what a strong answer emphasizes

Define a deadlock as a circular wait where each transaction holds a lock the next needs; explain the Coffman conditions and that databases detect (wait-for graph) and abort a victim rather than prevent by default. Then give the ranked prevention toolkit — consistent lock ordering first, then short/atomic transactions, up-front locking, right indexes + isolation to shrink lock footprint, optimistic concurrency for hotspots, and retry-on-deadlock as the non-negotiable safety net. Common causes to name: opposite lock ordering across code paths; long transactions holding locks during slow work; missing indexes causing over-locking scans; InnoDB gap locks on range inserts. Common mistakes in weak answers: saying "increase the lock timeout" (masks, doesn't fix), "raise isolation to SERIALIZABLE" (usually increases lock conflicts), or forgetting that the app must retry the aborted victim.

**Comparison:**

```
Technique	Coffman condition broken	Effect	Cost / caveat
Consistent lock ordering (e.g. by id)	Circular wait	Eliminates the classic 2-row cycle	Must be enforced in every code path
Short + atomic transactions	Hold-and-wait	Smaller lock window	Move slow work outside the txn
Acquire all locks up front	Hold-and-wait	No incremental grabbing	Can reduce concurrency
Proper indexing	(reduces lock scope)	Locks 1 row, not a scan range	Requires the right index
READ COMMITTED vs InnoDB RR	(fewer gap locks)	Fewer phantom/gap deadlocks	Weaker isolation guarantees
Optimistic concurrency (version col)	Mutual exclusion (no lock held)	No pessimistic lock at all	App must retry on conflict
Retry on deadlock error	— (recovery, not prevention)	Turns abort into a latency blip	Bound attempts + backoff
```

**Code:**

```

-- DEADLOCK-PRONE: order of locking depends on direction of transfer -- transfer(A -> B): locks A then B ; transfer(B -> A): locks B then A => cycle possible BEGIN; SELECT balance FROM accounts WHERE id = :from FOR UPDATE; -- lock 1 SELECT balance FROM accounts WHERE id = :to FOR UPDATE; -- lock 2 (order varies!) UPDATE accounts SET balance = balance - :amt WHERE id = :from; UPDATE accounts SET balance = balance + :amt WHERE id = :to; COMMIT;

CODE
Copy

```sql
-- DEADLOCK-SAFE: always lock the lower id first (canonical ordering breaks circular wait)
BEGIN;
  SELECT balance FROM accounts
   WHERE id IN (:from, :to)
   ORDER BY id                 -- deterministic global order
   FOR UPDATE;
  UPDATE accounts SET balance = balance - :amt WHERE id = :from AND balance >= :amt;
  UPDATE accounts SET balance = balance + :amt WHERE id = :to;
COMMIT;
PYTHON
Copy
# Retry-on-deadlock wrapper (mandatory safety net)
import random, time
DEADLOCK_STATES = {"40P01", "1213", "1205"}  # PG / MySQL / SQL Server

def run_with_retry(txn_fn, max_attempts=5):
    for attempt in range(max_attempts):
        try:
            return txn_fn()                 # opens txn, does work, commits
        except DatabaseError as e:
            if str(e.code) in DEADLOCK_STATES and attempt < max_attempts - 1:
                time.sleep((2 ** attempt) * 0.02 + random.random() * 0.02)  # backoff + jitter
                continue
            raise
SQL
Copy
-- Optimistic alternative for a hot row (no pessimistic lock => no deadlock, just retry)
UPDATE accounts SET balance = balance - :amt, version = version + 1
 WHERE id = :id AND version = :expected_version AND balance >= :amt;
-- rowcount 0 => someone else won; reload and retry in app code
```

**Example input / output:**

```

Deadlock interleaving that the engine detects and aborts:

CODE
Copy
time  T1                              T2
 t1   UPDATE accounts SET .. id=1     -                          (T1 holds lock on row 1)
 t2   -                               UPDATE accounts SET .. id=2 (T2 holds lock on row 2)
 t3   UPDATE .. id=2  -> WAITS        -                          (T1 waits for row 2)
 t4   -                               UPDATE .. id=1 -> WAITS     (T2 waits for row 1)
 --   wait-for graph has cycle T1->T2->T1  => engine picks a victim
 t5   ERROR 1213 (deadlock);          T2 commits successfully
      T1 is rolled back  -> app RETRIES T1, which now succeeds
```

**Flow:**

1. waits for row B
2. waits for row A
3. engine detects cycle
4. yes
5. T1 holds row A
6. T2 holds row B
7. Wait-for graphcycle?
8. Pick victim -> ROLLBACKreturn deadlock error
9. App retries victimwith backoff

