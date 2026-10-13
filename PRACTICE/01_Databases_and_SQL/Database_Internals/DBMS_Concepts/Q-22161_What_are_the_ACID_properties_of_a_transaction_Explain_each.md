---
type: "practice"
difficulty: "Basic"
---

# What are the ACID properties of a transaction? Explain each.

**Difficulty:** Basic · **Levels:** Fresher, SDE-1 · [↑ Distributed Transactions and 2PC](<../../../../01_Databases_and_SQL/Database_Internals/DBMS_Concepts/Distributed_Transactions_and_2PC.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

#### Triage

The question asks me to name and explain the four ACID properties of a database transaction. It's tagged "basic concept," so the interviewer wants a clear, correct, foundational answer — the right definitions, one crisp real-world example (payments companies like Razorpay, Stripe, PhonePe, Zerodha dominate the tag list, so a money-transfer example fits perfectly), and evidence that I understand why each property matters. I'll define a transaction first, then walk through A-C-I-D one at a time with the money example, and close with a one-line summary of each.

What is a transaction? A transaction is a single logical unit of work made up of one or more operations (reads/writes) that the database treats as one indivisible step. The canonical example: transferring ₹100 from account A to account B is two operations — debit A by ₹100, credit B by ₹100 — but it must behave as one. ACID is the set of guarantees that makes that safe.

A — Atomicity ("all or nothing")

Atomicity means a transaction either completes entirely or has no effect at all — there is no partial state left behind. If the debit of A succeeds but the system crashes before crediting B, atomicity guarantees the debit is rolled back, so no money vanishes. The database provides COMMIT (make all changes permanent) and ROLLBACK (undo everything since the transaction began). Practically, this is implemented with an undo log: if anything fails, the DB reverses the partial work.

Why it matters: without atomicity you'd lose or duplicate money on any crash mid-transfer.

C — Consistency ("valid state to valid state")

#### Cons
istency means a transaction moves the database from one valid state to another valid state, never violating the defined rules — constraints (primary/foreign keys, CHECK, UNIQUE, NOT NULL) and application invariants. In the transfer, an invariant might be "total money in the system is conserved" and "no balance goes negative." If crediting B would break a constraint (say B doesn't exist, violating a foreign key), the whole transaction is rejected. Note: the database enforces declared constraints; the application is responsible for defining the invariants correctly. Consistency is really a joint responsibility of the DB rules and the transaction logic.

Why it matters: it guarantees the data is always logically correct — you never see a balance below zero or an orphaned row.

I — Isolation ("concurrent transactions don't step on each other")

Isolation means that when many transactions run concurrently, each behaves as if it were running alone — the intermediate, uncommitted state of one transaction isn't visible to others. If two people transfer money out of account A at the same time, isolation prevents them from both reading the same starting balance and overdrawing it (a race condition). Databases offer isolation levels that trade strictness for performance: READ UNCOMMITTED, READ COMMITTED, REPEATABLE READ, and SERIALIZABLE (strongest — full illusion of one-at-a-time execution). Weaker levels allow anomalies like dirty reads, non-repeatable reads, and phantom reads; you pick a level per the correctness the workload needs. It's implemented with locking or MVCC (multi-version concurrency control, e.g. in Postgres).

Why it matters: it prevents concurrency bugs like double-spending and lost updates.

D — Durability ("committed = permanent")

Durability means once a transaction is committed, its changes survive anything afterward — power loss, crash, restart. The moment the DB tells the client "committed," that data is safely persisted, typically by writing to a write-ahead log (WAL) flushed to durable storage (and often replicated) before acknowledging the commit. After a crash, the DB replays the log on restart to recover committed work.

Why it matters: a user who saw "payment successful" must never find the payment gone after a server reboot.

Summary
Atomicity — all-or-nothing; partial work is rolled back on failure.

#### Cons
istency — every transaction leaves the DB in a valid state, respecting all constraints and invariants.
Isolation — concurrent transactions don't interfere; each sees a clean, serial-like view.
Durability — once committed, changes are permanent and survive crashes.

What a strong answer emphasizes: tie each property to a concrete failure it prevents (crash mid-transfer → Atomicity; negative balance → Consistency; double-spend race → Isolation; reboot loses a committed payment → Durability). Common follow-up probes: "Which property is relaxed in eventually-consistent NoSQL systems?" (often C/I via BASE), "Name the isolation-level anomalies," and "How is durability actually achieved?" (WAL flushed before commit acknowledgement).

**Comparison:**

```
Property	Guarantee	Prevents	Typical mechanism
Atomicity	All-or-nothing	Partial updates after a crash	Undo log, COMMIT/ROLLBACK
Consistency	Valid state → valid state	Constraint/invariant violations (e.g. negative balance)	Constraints + transaction logic
Isolation	Concurrent txns don't interfere	Dirty/non-repeatable/phantom reads, lost updates, double-spend	Locking or MVCC; isolation levels
Durability	Committed data is permanent	Losing committed work on crash/reboot	Write-ahead log flushed before ack
```

**Code:**

```sql
-- A money transfer as one atomic, consistent, isolated, durable transaction.
BEGIN;                                              -- start transaction

  UPDATE accounts SET balance = balance - 100
   WHERE id = 'A' AND balance >= 100;               -- Consistency: no negative balance

  -- If the above changed 0 rows (insufficient funds), the app issues ROLLBACK instead:
  -- ROLLBACK;   -- Atomicity: no partial effect

  UPDATE accounts SET balance = balance + 100
   WHERE id = 'B';

COMMIT;   -- Durability: changes now permanent (WAL flushed); Atomicity: both updates applied together
```

**Example input / output:**

```text
Initial: A.balance = 500, B.balance = 200

Case 1 — success:
  BEGIN; debit A 100; credit B 100; COMMIT;
  -> A = 400, B = 300   (atomic: both applied; durable after COMMIT)

Case 2 — crash after debit, before credit:
  BEGIN; debit A 100; <CRASH>
  -> on restart, uncommitted work is undone
  -> A = 500, B = 200   (atomicity: no money lost)

Case 3 — insufficient funds (A has 50):
  UPDATE ... WHERE balance >= 100  --> 0 rows -> ROLLBACK
  -> A = 50, B = 200    (consistency: balance never goes negative)
```

