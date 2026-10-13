---
type: "practice"
difficulty: "Expert"
---

# You need cross-database consistency between Postgres (orders) and a separate ledger service. Compare 2PC, sagas, and the transactional outbox in terms of the isolation guarantees each preserves.

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ From Serializable to Sagas](<../../../../01_Databases_and_SQL/Transactions_and_Concurrency/Isolation_Levels/Setting_Isolation_Levels/From_Serializable_to_Sagas.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When dealing with cross-database consistency between PostgreSQL (used for orders) and a separate ledger service, ensuring that transactions are coordinated correctly is crucial. We need to compare two-phase commit (2PC), sagas, and the transactional outbox in terms of their isolation guarantees.

TL;DR
Two-Phase Commit (2PC): Ensures atomicity but can lead to deadlocks and is complex.
Sagas: Flexible for complex workflows but can suffer from linearization points, leading to potential inconsistencies.
Transactional Outbox: Simple and effective for certain use cases, but it relies on the success of subsequent events.

#### Triaging the Question

The question asks us to compare 2PC, sagas, and the transactional outbox in terms of their isolation guarantees. Specifically, we need to understand how each approach handles consistency between databases and services. The key concepts here are:

Isolation Levels: Different levels like Read Uncommitted (RU), Read Committed (RC), Repeatable Read (RR), and Serializable.
Transactionality: Ensuring that all parts of a transaction commit or roll back atomically.

#### First Principles Solution
1. Two-Phase Commit (2PC)

Core Idea: 2PC is a protocol used to ensure atomicity in distributed transactions, where both the local database and the remote ledger service must agree on the state change before it can be committed.

Isolation Guarantees:

Serializable: In 2PC, if both parties commit successfully, the transaction behaves as if no other transactions interfered with its execution.
Potential Issues:
Deadlocks: Both parties might wait indefinitely for each other to complete their part of the transaction.
Complexity and Overhead: Managing distributed transactions can introduce significant complexity and performance overhead.

Worked Example: Suppose we have a command transfer(amount, from_order_id, to_ledger_id) in PostgreSQL. This triggers a 2PC operation where both the database and the ledger service must acknowledge before proceeding.

2. Sagas

Core Idea: Sagas are a pattern that breaks down complex transactions into multiple steps, each of which is its own transaction. If any step fails, the saga rolls back all previous steps to maintain consistency.

Isolation Guarantees:

Serializable: Each individual step in the saga can provide serializability locally.
Potential Issues:
Linearization Points: There are points where a failure could leave inconsistent states. For example, if the first step commits but the second fails, you might have an out-of-sync state.
Compensating Actions: Sagas require implementing compensating actions to reverse side effects of failed steps.

Worked Example: Consider transferring funds from one account to another in PostgreSQL and updating a ledger. The saga would involve:

Deduct amount from the source order in PostgreSQL.
If successful, send an event to the ledger service.
Ledger service updates its state if it succeeds.
If any step fails, all previous steps are rolled back.
3. Transactional Outbox

Core Idea: The transactional outbox pattern involves writing events to a queue or log after committing the local transaction in the database. The application then processes these events asynchronously to update other services.

Isolation Guarantees:

Read Committed (RC): Events are only processed once the transaction is committed.
Potential Issues:
Latency and Consistency: There can be a delay before the outbox event reaches its target, potentially leading to inconsistent states until all downstream services have updated.

Worked Example: In PostgreSQL, after committing a transfer command, an event is written to an outbox. The application then sends this event to the ledger service, which processes it asynchronously.

#### Closing Summary

Each of these approaches has trade-offs in terms of complexity and isolation guarantees:

2PC: Ensures strong consistency but at the cost of complexity and potential deadlocks.
Sagas: More flexible for complex workflows but introduces linearization points that can cause inconsistencies if not managed carefully.
Transactional Outbox: Simple to implement, especially when you have well-defined events, but it relies on eventual consistency.

#### Common Mistakes/Follow-Ups
Mistake 1: Overlooking the need for compensating actions in sagas. This can lead to inconsistent states if not implemented correctly.

#### Follow-Up
1: Discussing how to handle retries and dead-letter queues when using transactional outbox.
Mistake 2: Assuming that 2PC always provides serializability without considering the potential for deadlocks.

#### Follow-Up
2: Exploring alternative protocols like distributed two-phase locking (DTPL) or consensus algorithms like Raft for managing cross-database transactions.

**Comparison:**

| Command          | Phase 1 (Local) | Phase 2 (Remote) |
|------------------|----------------|-----------------|
| Transfer Amount  | Lock Order      | Update Ledger   |
|                  |                 |                 |
| On Success       | Commit          | Acknowledge     |
|                  | Rollback        | Fail            |

**Code:**

```sql
-- Example of a PostgreSQL transfer command using 2PC
BEGIN;
UPDATE orders SET balance = balance - amount WHERE id = from_order_id FOR UPDATE; -- Phase 1: Locking
SELECT ledger_service.transfer(amount, to_ledger_id); -- Phase 2: Communication with ledger service

-- On success commit, on failure rollback
COMMIT;
```

**Example input / output:**

```text
Input:
- Transfer command with amount, from_order_id, to_ledger_id

Output:
- If successful, both PostgreSQL and ledger service updated.
- If failed at any step, all changes are rolled back.
```

**Data:**

```json
{
    "transfer": {
        "amount": 50,
        "from_order_id": "123",
        "to_ledger_id": "456"
    }
}
```

**Flow:**

1. No
2. Yes
3. 2PC
4. Saga
5. Outbox
6. Start
7. Is transaction needed?
8. Process locally
9. Use 2PC, Saga or Outbox?
10. Lock resources ->Communicate with remoteservice -> Commit/rollback onsuccess/failure
11. Decompose into steps ->Execute each step in order ->Compensating actions if failure
12. Commit local transaction ->Write event to outbox ->Process asynchronously

