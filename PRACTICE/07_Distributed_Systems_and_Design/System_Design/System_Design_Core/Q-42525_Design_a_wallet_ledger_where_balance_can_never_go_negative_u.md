---
type: "practice"
difficulty: "Deep"
---

# Design a wallet ledger where balance can never go negative under concurrency. Compare SELECT FOR UPDATE, a conditional UPDATE ... WHERE balance >= amount, and event-sourced ledgers on throughput and auditability.

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Payment Reconciliation](<../../../../07_Distributed_Systems_and_Design/System_Design/System_Design_Core/Payment_Reconciliation.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To design a system where a wallet's balance can never go negative under concurrent operations, we need to compare three approaches: SELECT FOR UPDATE, conditional UPDATE ... WHERE balance >= amount, and an event-sourced ledger. Each method has its own trade-offs in terms of throughput and auditability.

#### Triage

The question is asking us to evaluate the concurrency control mechanisms for ensuring a wallet's balance never goes negative, considering both transactional integrity and performance aspects. We need to understand how each approach handles concurrent updates, potential race conditions, and their impacts on system throughput and auditability.

#### First-Principles Solution
1. SELECT FOR UPDATE

Concept: SELECT FOR UPDATE is a locking mechanism that locks the row(s) being read until the transaction ends.

Process:

A client reads the current balance using SELECT FOR UPDATE.
The database locks the row to prevent other transactions from modifying it.
If the balance is sufficient, the client updates the balance with a new value.
The transaction commits, and the lock is released.

#### Pros

Ensures data integrity by preventing concurrent modifications.
Simplifies logic since you only need to perform the read and update within a single transaction.

#### Cons

Can lead to deadlocks if not managed properly.
May reduce throughput due to the overhead of acquiring and releasing locks.
Limited auditability as operations are tightly coupled with the current state.
2. Conditional UPDATE ... WHERE balance >= amount

Concept: This approach uses a conditional UPDATE statement that checks the balance against the transaction amount before performing the update.

Process:

A client tries to update the balance by first checking if the current balance is greater than or equal to the requested amount.
If true, the balance is updated; otherwise, an error or retry mechanism handles it.

#### Pros

Reduces lock contention since only one transaction needs to acquire a lock.
Improved throughput compared to SELECT FOR UPDATE due to less locking overhead.
Can handle failed updates more gracefully with retries.

#### Cons

May lead to inconsistent states if concurrent transactions fail due to race conditions (e.g., two transactions reading the same balance and both failing).
Requires careful retry logic to avoid infinite loops or unnecessary rollbacks.
3. Event-Sourced Ledger

Concept: Event sourcing records all changes to a wallet's balance as separate events, which can be replayed to reconstruct its current state.

Process:

Each transaction is recorded as an event (e.g., Deposit, Withdrawal).
To update the balance, we apply these events in order of occurrence.
A read model or snapshot can provide a consistent view of the wallet's balance.

#### Pros

High auditability since every transaction is logged and can be reviewed.
Scalable and maintainable as state changes are recorded independently of concurrent operations.
Can handle complex business rules by replaying events.

#### Cons

Higher write latency due to the need to record each event.
More complex implementation required for maintaining consistency during read operations.

#### Closing Summary

Each approach has its strengths and weaknesses, making them suitable in different contexts. SELECT FOR UPDATE is straightforward but can limit throughput. The conditional UPDATE ... WHERE balance >= amount offers better performance with careful management of race conditions. An event-sourced ledger provides high auditability at the cost of write latency.

#### Common Mistakes/Follow-Ups
Mistake: Overlooking the potential for deadlocks in SELECT FOR UPDATE.

#### Follow-Up
Discussing how to manage retries and avoid infinite loops in the conditional update approach.

#### Follow-Up
Exploring how event sourcing can be integrated with a caching layer to balance write latency and read performance.

**Comparison:**

```
Wallet Table	Event Table
id	wallet_id
balance	event_type
	amount
```

**Code:**

```sql
-- Example SQL for SELECT FOR UPDATE
BEGIN TRANSACTION;
SELECT * FROM wallet WHERE id = ? FOR UPDATE;
UPDATE wallet SET balance = balance - ? WHERE id = ? AND balance >= ?;
COMMIT;

-- Example SQL for conditional UPDATE ... WHERE
BEGIN TRANSACTION;
IF (SELECT balance FROM wallet WHERE id = ?) >= ? THEN
    UPDATE wallet SET balance = balance - ? WHERE id = ?;
ELSE
    ROLLBACK; -- Handle error or retry logic here.
END IF;
COMMIT;

-- Event Sourcing Example in pseudo-code
class WalletEvent {
  event_type: string,
  amount: number,
}

def handle_withdrawal(event):
  # Apply event to wallet state, ensuring balance does not go negative

def replay_events(wallet_id):
  for event in get_events_for_wallet(wallet_id):
    handle_event(event)
```

**Example input / output:**

```

SELECT FOR UPDATE Example:

SQL
Copy
BEGIN TRANSACTION;
-- Client reads current balance with SELECT FOR UPDATE lock.
SELECT * FROM wallet WHERE id = 1 FOR UPDATE;

-- Client checks if balance is sufficient and updates accordingly.
UPDATE wallet SET balance = balance - 200 WHERE id = 1 AND balance >= 200; -- Lock held until commit.

COMMIT;

Conditional Update Example:

SQL
Copy
BEGIN TRANSACTION;
-- Client reads current balance without locking.
SELECT * FROM wallet WHERE id = 1;

IF (SELECT balance FROM wallet WHERE id = 1) >= 200 THEN
    UPDATE wallet SET balance = balance - 200 WHERE id = 1;
ELSE
    ROLLBACK; -- Handle error or retry logic here.
END IF;

COMMIT;

Event Sourced Example:

SQL
Copy
BEGIN TRANSACTION;
-- Client records a withdrawal event.
INSERT INTO wallet_events (wallet_id, event_type, amount) VALUES (1, 'withdrawal', -200);

-- Client updates the balance by replaying events.
REPLAY_EVENTS(1); -- This method applies all relevant events to the current state.

COMMIT;
```

**Data:**

```json
{
  "wallet": {
    "id": 1,
    "balance": 500
  },
  "event": {
    "type": "withdrawal",
    "amount": -200
  }
}
```

**Flow:**

1. Yes
2. No
3. Sufficient
4. Insufficient
5. Start Transaction
6. Wallet Balance Sufficient?
7. Update Wallet Balance
8. Rollback / Retry Logic
9. Commit Transaction
10. Handle Error or Retry Logic
11. Start Transaction
12. Check Conditional Update
13. Update Wallet Balance
14. Rollback
15. Commit Transaction
16. Handle Error or Retry Logic
17. Event Sourced Ledger
18. Record Withdrawal Event
19. Replay Events to Update State
20. Commit Transaction

