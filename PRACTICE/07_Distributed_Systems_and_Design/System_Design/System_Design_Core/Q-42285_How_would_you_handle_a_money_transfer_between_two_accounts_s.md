---
type: "practice"
difficulty: "Deep"
---

# How would you handle a money transfer between two accounts safely?

**Difficulty:** Deep · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Payment Reconciliation](<../../../../07_Distributed_Systems_and_Design/System_Design/System_Design_Core/Payment_Reconciliation.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

#### Triage

The question — "How would you handle a money transfer between two accounts safely?" — is the classic fintech correctness probe. On the surface it looks like "debit A, credit B", but the interviewer is testing whether I understand: atomicity (both legs succeed or neither does), consistency under concurrency (two transfers hitting the same account at once must not corrupt balances), idempotency (a retried request must not double-charge), durability (a committed transfer survives a crash), and auditability (every money movement is traceable). For companies like Razorpay, PhonePe, Paytm, CRED, Zerodha, and Stripe this is the single most important correctness property they own.

Clarifying questions I would raise up front:

Are the two accounts in the same database/service (intra-bank ledger) or in different systems (cross-bank, or A on our ledger and B at a payment network like UPI/NPCI)? The single-DB case is solvable with a local transaction; the distributed case needs a saga/2-phase pattern.
Is this synchronous (user waits) or can it be asynchronous with eventual settlement?
What are the currency, rounding, and minimum-balance/overdraft rules?
What throughput and latency SLAs? (Razorpay-scale = tens of thousands of transfers/sec.)

Plan: start with the canonical single-database double-entry ledger and a local ACID transaction; layer on idempotency; then discuss concurrency control; then extend to the distributed case with sagas/outbox; finish with failure modes and audit.

First principles: model money as a double-entry ledger, not a balance field

The most common junior mistake is storing a mutable balance column and doing UPDATE accounts SET balance = balance - 100 WHERE id = A. That works but throws away history and makes reconciliation impossible. The correct model is an append-only ledger of entries plus a derived (and optionally cached) balance.

A transfer of ₹100 from A to B is two ledger entries that must sum to zero: -100 on A, +100 on B, sharing one transfer_id. This is double-entry bookkeeping. Its invariant — the sum of all entries for an account equals its balance, and the sum across the whole ledger for a balanced transfer is zero — is what auditors and reconciliation jobs rely on.

Atomicity: one local ACID transaction

If both accounts live in the same relational database, the whole thing is a single transaction. Postgres/MySQL give me ACID for free:

CODE
BEGIN;
  -- lock and read balances
  -- check A has funds (and B/A are active, not frozen)
  INSERT entry (transfer_id, account=A, amount=-100);
  INSERT entry (transfer_id, account=B, amount=+100);
  UPDATE cached balances;
COMMIT;

Either both inserts + the balance updates commit, or the whole thing rolls back. There is never a state where A is debited but B is not credited. This is the single most important safety property, and it is why I strongly prefer a relational DB here.

Idempotency: survive retries and network timeouts

The client (or an upstream gateway) will time out and retry. Without protection, a retry double-transfers. The fix: the caller supplies an idempotency key (e.g., a UUID). I store it in a transfers table with a unique constraint on the key. On a retry:

If a row with that key already exists and is SUCCESS, return the stored result (do not re-execute).
If it exists and is PENDING/FAILED, resume or return the recorded outcome.
The unique constraint means even a race between two concurrent retries can only insert once; the loser gets a constraint violation and reads back the winner's result.

This turns "at-least-once delivery" into "exactly-once effect", which is the property users actually care about.

Concurrency: prevent lost updates and negative balances

Two transfers debiting A simultaneously can both read balance ₹500, both approve a ₹400 debit, and drive A to −₹300. Two defenses:

Pessimistic locking — SELECT ... FOR UPDATE on account A (and B) inside the transaction. To avoid deadlocks when A→B and B→A run at once, always lock accounts in a canonical order (e.g., ascending account id). Simple, correct, and standard in ledgers.
Optimistic locking — carry a version column; the balance update is ... WHERE version = :read_version; a mismatch means someone else changed it, so retry. Better under low contention.

For a hot account (a merchant settlement account) pessimistic ordered locking is usually the pragmatic choice. The balance check (balance - amount >= 0, or >= -overdraft_limit) happens while the row is locked, so it cannot be raced.

Distributed case: A and B in different systems

If B is at another bank/on the UPI network, one local transaction is impossible. I use a saga with a state machine and the transactional outbox pattern:

Local transaction: create transfer row in state PENDING, debit A into a hold/reserved sub-account, and write an outbox event — all atomically.
A relay reads the outbox and calls the external network idempotently.
On network SUCCESS, move PENDING → SETTLED and release the credit; on FAILURE, run the compensating action — reverse the hold and refund A — and move to FAILED.
If the network response is lost, a reconciliation job queries transfer status by our idempotency key and drives the state machine forward. Money is never "lost"; it is always in exactly one bucket (A, in-flight/reserved, or B).

This trades global ACID for eventual consistency with guaranteed convergence — the only realistic option across trust boundaries.

Durability, audit, and observability
Durability comes from the DB's WAL/redo log (commit means fsynced) plus replication; a committed transfer survives a node crash.
Every state change is an immutable ledger/audit row (who, when, amount, key, from/to, resulting state) — required for RBI/PCI compliance and dispute resolution.
A daily reconciliation job re-derives balances from ledger entries and compares against cached balances and the external network's statements; any drift pages on-call.
Metrics: pending-transfer age, compensation rate, idempotency-hit rate.
What a strong answer emphasizes

Double-entry ledger + a single ACID transaction for the intra-DB case; idempotency key with a unique constraint for exactly-once effect; ordered locking (or optimistic versioning) to stop lost updates and negative balances; and a saga + outbox + reconciliation for the cross-system case. The through-line is: money never disappears; it is always in exactly one accounted-for place, and every retry is safe.

#### Common mistakes
to avoid: mutable balance without a ledger; no idempotency (double charges); unordered locking (deadlocks); doing external network calls inside a DB transaction (holds locks across the network); and trusting distributed 2PC at scale instead of sagas. Common follow-ups: "how do you prevent deadlocks?" (ordered locking), "what if the credit succeeds but the debit is later reversed?" (compensation + reconciliation), and "how do you scale a hot account?" (sharded sub-ledgers / netting).

**Comparison:**

```
Concern	Risk if ignored	Mechanism used
Atomicity	A debited, B not credited	Single ACID transaction (both legs or neither)
Idempotency	Double charge on retry	Idempotency key + UNIQUE constraint
Lost update / negative balance	Money created/destroyed	SELECT ... FOR UPDATE in ordered lock, or optimistic version
Deadlock	Stuck transfers under contention	Lock accounts in canonical (ascending id) order
Durability	Committed transfer lost on crash	WAL/redo + synchronous replication
Cross-system settlement	Partial/lost transfer across banks	Saga + outbox + compensation + reconciliation
Auditability	Cannot resolve disputes / comply	Append-only double-entry ledger
Approach	Best when	Consistency	Cost
Local ACID txn	Both accounts in one DB	Strong	Lowest, simplest
Saga + outbox	Accounts in different services/banks	Eventual (guaranteed convergence)	Higher; needs compensation + reconciliation
2PC / XA	Small, low-throughput distributed set	Strong	Poor scaling, coordinator is SPOF
```

**Code:**

```

-- Schema: transfers (idempotency + state) and ledger entries (double-entry, append-only) CREATE TABLE transfers ( id BIGSERIAL PRIMARY KEY, idempotency_key UUID NOT NULL UNIQUE, -- exactly-once effect from_account BIGINT NOT NULL, to_account BIGINT NOT NULL, amount_paise BIGINT NOT NULL CHECK (amount_paise > 0), status TEXT NOT NULL DEFAULT 'PENDING', -- PENDING|SUCCESS|FAILED created_at TIMESTAMPTZ NOT NULL DEFAULT now() );

CREATE TABLE ledger_entries ( id BIGSERIAL PRIMARY KEY, transfer_id BIGINT NOT NULL REFERENCES transfers(id), account_id BIGINT NOT NULL, amount_paise BIGINT NOT NULL, -- negative = debit, positive = credit created_at TIMESTAMPTZ NOT NULL DEFAULT now() );

CREATE TABLE accounts ( id BIGINT PRIMARY KEY, balance_paise BIGINT NOT NULL DEFAULT 0, version BIGINT NOT NULL DEFAULT 0 -- for optimistic locking );

CODE
Copy

```java
// Intra-DB transfer: single ACID transaction, ordered pessimistic locks, idempotent.
@Transactional
public TransferResult transfer(UUID key, long from, long to, long amount) {
    // 1. Idempotency: unique key -> retry returns the recorded outcome
    Optional<Transfer> existing = transfers.findByKey(key);
    if (existing.isPresent()) return existing.get().toResult();

    Transfer t = transfers.insert(key, from, to, amount, "PENDING"); // unique key guards races

    // 2. Deadlock-safe: always lock in canonical (ascending id) order
    long first = Math.min(from, to), second = Math.max(from, to);
    Account a1 = accounts.selectForUpdate(first);   // SELECT ... FOR UPDATE
    Account a2 = accounts.selectForUpdate(second);
    Account src = (from == first) ? a1 : a2;
    Account dst = (to   == first) ? a1 : a2;

    // 3. Business checks while rows are locked
    if (src.balancePaise() < amount) {
        transfers.updateStatus(t.id(), "FAILED");
        throw new InsufficientFundsException();
    }

    // 4. Double-entry: two entries summing to zero, plus cached balances
    ledger.insert(t.id(), from, -amount);
    ledger.insert(t.id(), to,   +amount);
    accounts.addBalance(from, -amount);
    accounts.addBalance(to,   +amount);
    transfers.updateStatus(t.id(), "SUCCESS");

    return new TransferResult(t.id(), "SUCCESS"); // COMMIT via @Transactional
}
```

**Example input / output:**

```text
Scenario 1 — happy path
  A(1001) balance = 50000 paise, B(2002) = 0
  POST transfer {key=K1, from=1001, to=2002, amount=10000}
  -> 200 { transfer_id: 501, status: SUCCESS }
  Ledger: (501, 1001, -10000), (501, 2002, +10000)   (sums to 0)
  A = 40000, B = 10000

Scenario 2 — retry (network timeout, client resends SAME key)
  POST transfer {key=K1, ...}          # same idempotency key
  -> 200 { transfer_id: 501, status: SUCCESS }   # replayed, NOT re-executed
  A = 40000, B = 10000                  # unchanged — no double debit

Scenario 3 — insufficient funds
  A = 40000; POST {key=K2, from=1001, to=2002, amount=99999}
  -> 402 { status: FAILED, reason: INSUFFICIENT_FUNDS }
  A = 40000 (no ledger entries written; transaction rolled back)

Scenario 4 — concurrent debits on A (T1: 30000, T2: 20000), A=40000
  Both grab SELECT ... FOR UPDATE on 1001; one wins the lock first.
  T1 debits 30000 -> A=10000, commits; T2 then sees A=10000 < 20000
  -> T2 FAILED. No negative balance, no lost update.
```

**Data:**

```json
// Idempotent transfer request the client sends (key generated once, reused on retry)
{
  "idempotency_key": "3f1c9a7e-9b21-4c0a-8f7a-1e2d3c4b5a6f",
  "from_account": 1001,
  "to_account": 2002,
  "amount_paise": 10000,
  "currency": "INR"
}
```

**Flow:**

1. CrossSystem [If B is external]
2. yes
3. no
4. no
5. yes
6. success
7. failure/timeout
8. Request: transferkey,from,to,amount
9. Idempotency key seen?
10. Return recorded result
11. BEGIN txn
12. Lock accounts in ascending-idorder FOR UPDATE
13. Source balance >= amount?
14. status=FAILED, ROLLBACK
15. Insert -amount and +amountledger entries
16. Update cached balances,status=SUCCESS
17. COMMIT
18. Return SUCCESS
19. Write outbox event
20. Relay calls networkidempotently
21. PENDING -> SETTLED
22. Compensate: reverse hold,refund A -> FAILED
23. Reconciliation job resolvesunknowns

