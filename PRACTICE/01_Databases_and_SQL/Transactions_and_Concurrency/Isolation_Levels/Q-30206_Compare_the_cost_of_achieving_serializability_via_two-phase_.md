---
type: "practice"
difficulty: "Expert"
---

# Compare the cost of achieving serializability via two-phase locking versus via SSI in terms of blocking, abort rate, and read-only workloads.

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-1, SDE-1/2, SDE-2/3 · [↑ Serializable Performance](<../../../../01_Databases_and_SQL/Transactions_and_Concurrency/Isolation_Levels/Setting_Isolation_Levels/Serializable_Performance.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Serializability is a key property that ensures database transactions are executed as if they were run one at a time, even though multiple transactions can be interleaved. In this comparison, we will evaluate two-phase locking (2PL) and serialization isolation (SSI) in terms of blocking, abort rate, and their impact on read-only workloads.

#### Triage
The question is essentially asking us to compare the performance characteristics of serializable transactions under 2PL and SSI. Specifically, it seeks insight into how these isolation levels affect blocking patterns, transaction abort rates, and the behavior of read-only queries in a shared database environment.

Key Assumptions:

We are comparing 2PL with SSI.
The focus is on serializability to prevent phantom reads and ensure repeatable reads.
Read-only workloads do not modify data, which affects how locks are acquired and held.

#### First-principles Solution

Blocking Patterns:

In 2PL, transactions acquire locks at the start of execution and release them when they complete or abort. This can lead to blocking if one transaction waits for another to release a lock.
SSI allows transactions to use read-write locks, where readers can coexist with writers but only up to a certain point. Once a writer attempts to write, all existing readers must be rolled back and reacquire the data.

Abort Rate:

2PL may experience more aborts due to conflicts between transactions. For example, if two transactions need different locks on the same resource, one transaction will block until it can acquire its required locks.
SSI generally has a lower abort rate because readers do not conflict with writers; however, when a writer is present, any ongoing read transactions that depend on the written data must be reacquired.

Read-Only Workloads:

Read-only workloads are less affected by 2PL since they typically hold no locks for writing and only block during the initial lock acquisition phase.
SSI can still impact read-only workloads if a writer is present, as these transactions must be rolled back and reacquired.

#### Closing Summary
In summary, while both isolation levels ensure serializability, 2PL tends to have higher blocking due to strict lock management. Conversely, SSI offers better concurrency for read-only operations but may introduce more aborts during data conflicts. The choice between the two depends on the specific workload and performance requirements.

#### Common Mistakes/Follow-ups

Overlooking the impact of read-only transactions in 2PL.
Underestimating the overhead introduced by frequent rollbacks in SSI.
Not considering the potential for deadlocks in 2PL environments.

**Code:**

```sql
-- Example scenario in SQL
BEGIN TRANSACTION;

SELECT * FROM orders WHERE order_id = 1; -- Read-only operation

UPDATE orders SET status = 'shipped' WHERE order_id = 1; -- Write operation, locks the row
```

