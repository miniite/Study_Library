---
type: "practice"
difficulty: "Deep"
---

# What is a deadlock in a database and how can it be prevented or resolved?

**Difficulty:** Deep · **Levels:** Fresher, SDE-1 · [↑ Distributed Transactions and 2PC](<../../../../01_Databases_and_SQL/Database_Internals/DBMS_Concepts/Distributed_Transactions_and_2PC.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

#### Triage

The question asks me to define a database deadlock, then explain how it can be prevented and how it is resolved. At "deep understanding" level for backend/fullstack SDEs at Amazon, Microsoft, Uber, and fintech firms (Razorpay, CRED, Zerodha, Stripe), the interviewer is probing whether I understand the four Coffman conditions that create a deadlock, how a real engine detects one (wait-for graph, victim selection, ERROR 1213 / SQLSTATE 40P01), the difference between the database resolving it (kill a victim) versus the application preventing it (lock ordering, retries), and whether I can diagnose one in production. This is a classic OLTP-at-scale question because deadlocks appear precisely under high concurrency on shared rows (a hot inventory SKU, a popular account, a counter).

#### Assumptions
/ clarifications a strong candidate raises: (1) By "deadlock" I mean a lock-wait cycle among transactions, not a livelock or a long lock-wait timeout (those are different). (2) The database can only resolve a deadlock (by aborting a victim); it cannot silently make both succeed — so the application must retry. (3) Prevention is mostly an application-design responsibility (lock ordering, short transactions); the engine provides detection and victim selection.

Plan: define deadlock via the Coffman conditions with a concrete two-transaction example, then split the answer into (a) how databases detect + resolve it and (b) how engineers prevent it, then note diagnosis and give a summary.

What a deadlock is

A deadlock is a state where two or more transactions are each waiting for a lock the other holds, forming a cycle, so none can ever proceed. The classic minimal case:

CODE
T1: locks row A, then wants row B
T2: locks row B, then wants row A
-> T1 waits for T2 (holds B); T2 waits for T1 (holds A) -> cycle, stuck forever

Four conditions (Coffman) must all hold for a deadlock:

Mutual exclusion — a lock is held exclusively (a row write-lock).
Hold and wait — a transaction holds locks while requesting more.
No preemption — a lock can't be forcibly taken; it's released only by the holder.
Circular wait — a cycle exists in the "who waits for whom" (wait-for) graph.

Break any one of the four and deadlock becomes impossible — that's the theoretical basis for every prevention strategy below.

How the database DETECTS and RESOLVES a deadlock

Because "no preemption" plus a cycle means the transactions will wait forever, the engine must intervene. Two engine strategies:

Deadlock detection (most engines: InnoDB, PostgreSQL, SQL Server). The engine maintains a wait-for graph; when a lock request would create a cycle it detects it (immediately in InnoDB, or after a deadlock_timeout, default 1s, in PostgreSQL). It then picks a victim and rolls that transaction back, releasing its locks so the others proceed. The victim's client receives an error:
MySQL/InnoDB: ERROR 1213 (40001): Deadlock found when trying to get lock; try restarting transaction
PostgreSQL: deadlock detected (SQLSTATE 40P01)
SQL Server: error 1205. Victim selection favors killing the "cheaper" transaction to roll back — typically the one with the least work done / fewest rows modified / least undo (InnoDB uses transaction weight; SQL Server uses a DEADLOCK_PRIORITY + log volume).
Deadlock avoidance via timeouts. Simpler engines just time out a lock wait (innodb_lock_wait_timeout, default 50s) and roll back the waiter. This resolves the stall but is coarser than true cycle detection.

Key takeaway: resolution is the database's job (kill a victim), but the application must catch the deadlock error and retry the whole transaction — a deadlock error is transient, not a bug in the data.

How to PREVENT deadlocks (application design — the important part)

Each strategy breaks one Coffman condition:

#### Cons
istent lock ordering (breaks circular wait) — the #1 fix. Always acquire locks/rows in a global canonical order (e.g. ascending primary key). If both T1 and T2 lock account A before account B, they can never form the A↔B cycle. In a money transfer, sort the two account IDs and lock the lower id first. This is the single most effective, most common real-world remedy.
Acquire all locks up front / lock the parent row (breaks hold-and-wait). Grab every needed lock in one shot (e.g. one SELECT ... FOR UPDATE over all needed rows ordered), or serialize on a single coarse lock. Reduces concurrency but eliminates hold-and-wait.
Keep transactions short & touch fewer rows (shrinks the window). Do validation/computation before opening the transaction; don't hold locks across network calls or user think-time. Shorter locked spans → far fewer cycles.
Lower isolation where safe / use optimistic concurrency (reduces locking). Use an atomic conditional UPDATE ... WHERE version = ? (optimistic locking) so no long lock is held; on conflict, retry. This trades locks for retries.
Timeout + retry with backoff (breaks no-preemption, at app level). Set a sane lock_wait_timeout, and wrap deadlock-prone transactions in a retry loop with jittered exponential backoff so retried transactions don't re-collide.
Add the right indexes. Missing indexes cause full scans that lock many/extra rows (and, in InnoDB, gap locks), dramatically widening the surface for deadlocks. A good index that locks only the target rows often makes deadlocks disappear.
Diagnosing deadlocks in production
MySQL/InnoDB: SHOW ENGINE INNODB STATUS shows the latest deadlock (both transactions, the locks each held/waited on, the chosen victim); enable innodb_print_all_deadlocks to log every one.
PostgreSQL: deadlocks are logged with both queries and the involved PIDs; pg_locks/pg_stat_activity show live lock waits.
SQL Server: deadlock graph via Extended Events / the system_health session. Read the graph, find the two lock-acquisition orders that oppose each other, and impose a consistent order or add the missing index.
Summary — what a strong answer emphasizes

A deadlock is a cyclic lock-wait where transactions each hold what the other needs (all four Coffman conditions hold). The database resolves it by detecting a cycle in the wait-for graph and rolling back a victim (InnoDB 1213 / PG 40P01 / SQL Server 1205), choosing the cheapest to undo. But real robustness comes from the application: consistent lock ordering (breaks circular wait, the top fix), short transactions, acquiring locks up front, optimistic concurrency, right indexing, and always a catch-the-deadlock-error retry with backoff. The senior framing is: a single deadlock is normal under contention and self-heals; a rising deadlock rate is a design smell pointing to inconsistent lock order or a missing index.

#### Common mistakes
confusing a deadlock (cycle, auto-resolved by the engine) with a lock-wait timeout (single long waiter) or a livelock; saying "the database prevents deadlocks" (it resolves, doesn't prevent — prevention is design); forgetting the mandatory retry loop; ignoring that missing indexes and gap locks cause deadlocks. Follow-up probes: "How does InnoDB pick the victim?"; "Write the money-transfer that can't deadlock" (lock lower id first); "Why does adding an index fix a deadlock?"; "Difference between deadlock and livelock/timeout?"

**Comparison:**

```
Coffman condition	Meaning	Prevention that breaks it
Mutual exclusion	lock held exclusively	(hard to remove) optimistic concurrency / MVCC reads
Hold and wait	hold locks while asking for more	acquire all locks up front
No preemption	can't force-release a lock	lock timeout + retry (app-level preemption)
Circular wait	cycle in wait-for graph	consistent lock ordering (top fix)
Concept	Deadlock	Lock-wait timeout	Livelock
Cause	cyclic wait, 2+ txns	one txn waits too long on a busy lock	txns keep retrying and re-colliding
Resolution	engine kills a victim	waiter aborted after timeout	needs backoff/jitter to break symmetry
Error	1213 / 40P01 / 1205	lock wait timeout exceeded	none (just no progress)
App action	retry (transient)	retry / investigate contention	add randomized backoff
```

**Code:**

```

-- DEADLOCK-PRONE: opposite lock orders -- T1 BEGIN; UPDATE acct SET bal=bal-100 WHERE id=1; -- locks row 1 UPDATE acct SET bal=bal+100 WHERE id=2; -- wants row 2 -- T2 (concurrent) BEGIN; UPDATE acct SET bal=bal-50 WHERE id=2; -- locks row 2 UPDATE acct SET bal=bal+50 WHERE id=1; -- wants row 1 -> CYCLE -> deadlock

CODE
Copy

```sql
-- PREVENTED: consistent lock ordering (always lock the lower id first)
-- Application ensures src,dst are ordered before locking.
BEGIN;
  SELECT bal FROM acct WHERE id IN (1,2) ORDER BY id FOR UPDATE;  -- locks 1 then 2, always
  UPDATE acct SET bal=bal-100 WHERE id=1;
  UPDATE acct SET bal=bal+100 WHERE id=2;
COMMIT;
PYTHON
Copy
# Catch-and-retry: a deadlock error is transient, retry the whole transaction.
import time, random, psycopg2

def transfer(conn, a, b, amount):
    lo, hi = sorted((a, b))                 # canonical lock order -> no circular wait
    for attempt in range(5):
        try:
            cur = conn.cursor()
            cur.execute("SELECT id FROM acct WHERE id IN (%s,%s) ORDER BY id FOR UPDATE",(lo,hi))
            cur.execute("UPDATE acct SET bal = bal - %s WHERE id = %s", (amount, a))
            cur.execute("UPDATE acct SET bal = bal + %s WHERE id = %s", (amount, b))
            conn.commit()
            return
        except psycopg2.errors.DeadlockDetected:   # SQLSTATE 40P01 (MySQL: 1213)
            conn.rollback()
            time.sleep((0.05 * 2**attempt) + random.uniform(0, 0.05))  # backoff + jitter
    raise RuntimeError("transfer failed after retries")
```

**Example input / output:**

```text
DEADLOCK TIMELINE
 t0  T1: UPDATE acct id=1   -> acquires X-lock on row 1
 t1  T2: UPDATE acct id=2   -> acquires X-lock on row 2
 t2  T1: UPDATE acct id=2   -> BLOCKS (row 2 held by T2)
 t3  T2: UPDATE acct id=1   -> BLOCKS (row 1 held by T1)  => wait-for cycle
 t4  engine detects cycle, rolls back T2 (victim)
     T2 client: ERROR 1213 / 40P01 "Deadlock detected"
     T1 proceeds and commits
 t5  application catches error, RETRIES T2 -> succeeds

Wait-for graph at t3:
     T1 --waits for--> T2
     T2 --waits for--> T1        (cycle => deadlock)
```

**Flow:**

1. T2
2. DB
3. T1
4. T2
5. DB
6. T1
7. wait-for graph has a cycle T1->T2->T1
8. application retries transaction
9. lock row 1 (X)
10. lock row 2 (X)
11. request row 2 (blocked, held by T2)
12. request row 1 (blocked, held by T1)
13. ROLLBACK (victim) + deadlock error 40P01
14. proceeds, COMMIT

