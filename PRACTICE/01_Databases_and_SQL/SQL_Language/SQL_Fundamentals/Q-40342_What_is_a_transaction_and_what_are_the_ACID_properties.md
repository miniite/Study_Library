---
type: "practice"
difficulty: "Intermediate"
---

# What is a transaction and what are the ACID properties?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Isolation Anomalies](<../../../../01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Isolation_Anomalies.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

#### Triage

What's really being asked: Define a database transaction and explain the four ACID properties — Atomicity, Consistency, Isolation, Durability — well enough that the interviewer trusts you'd reason correctly about correctness in a payments/e-commerce system. The profile is backend/fullstack and the tag list is heavy on fintech (Razorpay, PhonePe, Paytm, Zerodha, Stripe), so the subtext is: "Do you understand why we wrap money movement in transactions and what could go wrong without each property?"

Interviewer is probing for: Not a textbook recitation, but practitioner understanding — can you tie each ACID letter to a concrete failure it prevents, do you know isolation levels and their anomalies, and do you understand the difference between "consistency" the ACID property and consistency in distributed systems (CAP)?

#### Assumptions
/ clarifying questions: I'll assume a single relational database (Postgres/MySQL/Oracle). I'd clarify whether they mean single-node ACID or distributed transactions (2PC / sagas), because durability and isolation get much harder across services — I'll mention that at the end.

Plan: Define a transaction and the BEGIN/COMMIT/ROLLBACK lifecycle → define each ACID property with a money-transfer example and the failure it prevents → go one level deeper on Isolation (levels + anomalies) since that's where real interviews go → note how databases actually implement these (WAL, locks, MVCC) → summarize and mention distributed caveats.

What a transaction is

A transaction is a group of one or more SQL statements executed as a single logical unit of work — it either happens completely or not at all. You mark its boundaries explicitly:

BEGIN (or START TRANSACTION) — open it.
... one or more INSERT/UPDATE/DELETE/SELECT ...
COMMIT — make all changes permanent and visible to others.
ROLLBACK — discard all changes, as if nothing happened.

The canonical example: transfer ₹500 from account A to account B. That's two writes — debit A, credit B. If the system crashes after the debit but before the credit, money vanishes. Wrapping both writes in one transaction guarantees the pair is all-or-nothing.

ACID is the set of four guarantees a compliant transactional database makes about that unit of work.

A — Atomicity ("all or nothing")

Every statement in the transaction succeeds, or the database is rolled back to the state before it started. There is no partial application. If the credit to B fails (constraint violation, crash, deadlock), the debit to A is undone automatically.

Failure it prevents: money disappearing or being double-created because only some of the writes landed.
How it's implemented: an undo log / write-ahead log (WAL). On ROLLBACK or crash recovery, the engine reverses uncommitted changes.
C — Consistency ("valid state to valid state")

A transaction moves the database from one valid state to another valid state, respecting all defined rules — primary keys, foreign keys, unique/CHECK constraints, triggers, and application invariants. If a transaction would violate an invariant (e.g. a CHECK (balance >= 0) that would go negative), it's rejected and rolled back.

Failure it prevents: leaving the database in a state that breaks its own rules (orphaned rows, negative balances, broken references).
Nuance to state explicitly: ACID-consistency is about integrity constraints, and it's partly the application's job to define the right invariants. It is not the same as "consistency" in CAP/distributed systems (which is about all nodes seeing the same latest value). Naming this difference signals seniority.
I — Isolation ("concurrent transactions don't corrupt each other")

Concurrent transactions execute as if they were serialized — the intermediate, uncommitted state of one transaction is not visible to another in a way that produces wrong results. Without isolation you get classic anomalies:

Dirty read: T2 reads a row T1 wrote but hasn't committed; T1 rolls back → T2 read data that never existed.
Non-repeatable read: T1 reads a row twice and gets different values because T2 committed an update in between.
Phantom read: T1 runs the same WHERE twice and a new row appears because T2 inserted a matching row.
Lost update: two transactions read a balance, both add to it, second overwrites the first's change.

SQL defines isolation levels trading correctness for concurrency: Read Uncommitted → Read Committed → Repeatable Read → Serializable. Higher = fewer anomalies but more locking/aborts and lower throughput. Postgres defaults to Read Committed; a bank transfer often wants Repeatable Read or Serializable, or explicit SELECT ... FOR UPDATE row locks. See the table.

Failure it prevents: race conditions like two people withdrawing the last ₹100 simultaneously.
How it's implemented: locking (2-phase locking) and/or MVCC (multi-version concurrency control — readers see a snapshot, writers create new versions).
D — Durability ("committed = permanent")

Once COMMIT returns success, the change survives crashes, power loss, and restarts. It's on stable storage.

Failure it prevents: telling a user "payment successful" and then losing it when the server reboots.
How it's implemented: the WAL is fsync'd to disk before COMMIT acknowledges (write-ahead logging); replication and backups extend durability across machines.
How this plays out in a real system

For the ₹500 transfer, the four properties together guarantee: both writes happen or neither does (A), the total money and non-negative-balance rules hold (C), a concurrent transfer can't read a half-done state or double-spend (I), and once we tell the user it's done it stays done (D).

#### Common mistakes
to avoid: confusing ACID-consistency with CAP-consistency; thinking SELECT needs no transaction (it can, for a consistent snapshot across multiple reads); assuming the default isolation level prevents lost updates (Read Committed does not — you need explicit locking, SERIALIZABLE, or an atomic UPDATE ... SET balance = balance - 500 WHERE balance >= 500); and holding transactions open too long (long transactions bloat MVCC, hold locks, and hurt throughput).

Distributed caveat (senior-level): across microservices/databases, true ACID is expensive — you either use two-phase commit (2PC, blocking, poor availability) or trade it for eventual consistency via the saga pattern (a sequence of local transactions with compensating actions) and the outbox pattern for reliable event publishing. That's the fintech reality behind the tag list.

Summary — what a strong answer emphasizes

A transaction is an all-or-nothing unit of work bounded by BEGIN/COMMIT/ROLLBACK. ACID = Atomicity (all or nothing, via undo/WAL), Consistency (valid → valid, constraints hold), Isolation (concurrent transactions don't corrupt each other; know the levels and anomalies), Durability (committed survives crashes, via fsync'd WAL). The differentiators are: mapping each letter to a concrete failure it prevents, going deep on isolation levels/anomalies, separating ACID-C from CAP-C, and knowing that distributed transactions force a trade-off (2PC vs sagas).

**Comparison:**

```
Property	Guarantee	Failure it prevents	Implemented via
Atomicity	All statements apply or none	Partial writes (money lost/created)	Undo log / WAL, ROLLBACK
Consistency	Valid state → valid state	Broken constraints, orphan/negative rows	Constraints, FKs, triggers
Isolation	Concurrent txns don't corrupt each other	Dirty/non-repeatable/phantom reads, lost updates	Locking (2PL) / MVCC
Durability	Committed data survives crashes	Losing an acknowledged commit	fsync'd WAL, replication
Isolation level	Dirty read	Non-repeatable read	Phantom read	Typical use
Read Uncommitted	Possible	Possible	Possible	Almost never
Read Committed	Prevented	Possible	Possible	Default (Postgres/Oracle)
Repeatable Read	Prevented	Prevented	Possible*	Financial reads
Serializable	Prevented	Prevented	Prevented	Strict correctness

*In PostgreSQL, Repeatable Read (snapshot) also blocks phantoms; the ANSI table allows them.
```

**Code:**

```

-- The money-transfer transaction: atomic, consistent, isolated, durable. BEGIN;

-- Debit A only if funds exist; the WHERE guard enforces the invariant atomically. UPDATE accounts SET balance = balance - 500 WHERE id = 'A' AND balance >= 500;

-- If exactly 0 rows were updated, A lacked funds -> abort the whole unit. -- (In app code: check affected-rows; if 0 -> ROLLBACK and return "insufficient funds".)

UPDATE accounts SET balance = balance + 500 WHERE id = 'B';

COMMIT; -- both writes become permanent together; on any error above, ROLLBACK

CODE
Copy

```sql
-- Isolation in action: prevent a lost update / double-spend by locking the row.
BEGIN;                                   -- e.g. under REPEATABLE READ
SELECT balance FROM accounts WHERE id = 'A' FOR UPDATE;  -- row lock until COMMIT
UPDATE accounts SET balance = balance - 500 WHERE id = 'A';
COMMIT;                                  -- lock released; a concurrent txn waited its turn
```

**Example input / output:**

```

Scenario 1 — successful transfer (all four properties hold):

CODE
Copy
Start:  A=500, B=100
BEGIN; UPDATE A -=500 (A=0); UPDATE B +=500 (B=600); COMMIT;
End:    A=0,   B=600           -- durable, atomic pair applied

Scenario 2 — Atomicity + Consistency on failure:

CODE
Copy
Start:  A=300, B=100
BEGIN; UPDATE A -=500 WHERE balance>=500;  -- 0 rows: guard fails
App sees affected_rows=0 -> ROLLBACK;
End:    A=300, B=100           -- unchanged; no partial debit, invariant intact

Scenario 3 — Isolation prevents a lost update:

CODE
Copy
T1: read A=500 ................. wants -500
T2: read A=500 ................. wants -500
Without isolation: both write A=0  -> ₹500 double-spent (WRONG)
With FOR UPDATE / SERIALIZABLE:  T2 blocks until T1 commits (A=0),
   then T2's guard balance>=500 fails -> T2 rolls back (CORRECT)
```

**Data:**

```sql
CREATE TABLE accounts (
    id      VARCHAR(10) PRIMARY KEY,
    balance NUMERIC(12,2) NOT NULL,
    CHECK (balance >= 0)      -- Consistency: DB rejects any txn that breaks this
);

INSERT INTO accounts VALUES ('A', 500.00), ('B', 100.00);
```

**Flow:**

1. Yes
2. No / error / crash
3. BEGIN transaction
4. Execute statements
5. All succeed &constraints hold?
6. COMMIT: WAL fsync'd ->durable & visible
7. ROLLBACK: undo log reversesall changes
8. DB back to pre-BEGIN state -Atomicity
9. Changes permanent -Durability

