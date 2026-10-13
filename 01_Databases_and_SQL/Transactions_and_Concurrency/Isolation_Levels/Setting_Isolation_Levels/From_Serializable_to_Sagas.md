---
id: "01_Databases_and_SQL/Transactions_and_Concurrency/Isolation_Levels/From_Serializable_to_Sagas"
level: 5
type: "deep-dive"
topic: "01_Databases_and_SQL/Transactions_and_Concurrency/Isolation_Levels"
tools: []
---

# From Serializable to Sagas

**L5 Expert** · [↑ Isolation Levels](<../_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** Serializable isolation protects invariants inside one database transaction; once the work spans shards or services, you must replace it with explicit coordination (2PC, sagas, outbox, idempotency) and accept weaker, named guarantees.

### 1. The big picture
Inside a single database, a SERIALIZABLE transaction lets you write checkout code as if nothing else were running: check stock, reserve it, create the order, charge, commit. The engine guarantees the outcome equals some serial order, so an invariant like "never oversell" holds.

Scale breaks the premise. Data splits across shards, the payment and inventory systems become separate services with separate databases, and no single transaction spans them. Per-shard serializability does not give global serializability. The industry's answers trade guarantees for availability and independence: two-phase commit (atomic but blocking), sagas (sequence of local transactions with compensations; eventually consistent, not isolated), and the transactional outbox (atomically write state and the message to send). This lesson shows how to evolve a design along that path without losing the invariant.

### 2. Core concepts and vocabulary
- **Serializable** — outcome equals some serial execution of the transactions.
- **Write skew** — transactions read overlapping data and write disjoint rows, violating an invariant that each individually respected.
- **Two-phase commit (2PC)** — coordinator asks all participants to *prepare*, then *commit*; atomic but blocks if the coordinator fails.
- **Saga** — a chain of local transactions; if one fails, run compensating transactions for the completed steps.
- **Compensating transaction** — a business-level undo (refund, release reservation), not a rollback.
- **Orchestration vs choreography** — a central coordinator drives the saga vs services react to each other's events.
- **Transactional outbox** — write the event to an `Outbox` table in the same local transaction as the state change; a relay publishes it.
- **Idempotent consumer** — handles duplicate messages without double effects.
- **Semantic lock / reservation** — a pending state ("reserved") that stands in for a lock across steps.
- **Anomalies in sagas** — lack of isolation: dirty reads of pending state, lost updates between steps.

### 3. How it works, step by step
Evolving a monolithic serializable checkout into a saga while preserving "no oversell":

1. **Identify the invariant** and the single place that can enforce it: `available >= 0` per product, owned by the Inventory service.
2. **Decompose** checkout into local transactions: (1) Order service creates order in `PENDING`; (2) Inventory reserves stock; (3) Payment charges; (4) Order marks `CONFIRMED`.
3. **Make the invariant local**: Inventory does `UPDATE ... SET reserved = reserved + n WHERE available - reserved >= n`, an atomic conditional update; oversell is impossible regardless of the global ordering.
4. **Define compensations**: payment fails → Inventory releases reservation → Order `CANCELLED`. Each step must be idempotent and each compensation must always be possible.
5. **Publish via outbox**: each service writes its state and outgoing event in one local transaction; a relay delivers at-least-once.
6. **Consume idempotently**: record processed message ids with a unique constraint.
7. **Expose pending states** in the product/UI (order "processing") since intermediate states are visible.
8. **Add a timeout/reaper** that cancels stuck sagas and releases reservations.

Choosing a cross-database strategy:

| Approach | Atomicity | Isolation | Cost / risk |
|---|---|---|---|
| Single DB transaction | Full | Chosen level | Couples services; not scalable |
| 2PC / XA | Atomic commit | Locks held across prepare | Blocking, coordinator failure, latency, vendor support |
| Saga | Eventual, via compensation | None (visible intermediate states) | Design effort for compensations, anomalies |
| Outbox + idempotent consumer | At-least-once, effectively once | Per-service | Needs relay, dedupe store |

### 4. Worked example
Inventory reservation as the local invariant owner:

```sql
-- Inventory service, step 2 of the saga
CREATE TABLE Stock (
    ProductID int PRIMARY KEY,
    OnHand    int NOT NULL,
    Reserved  int NOT NULL DEFAULT 0,
    CHECK (Reserved <= OnHand)
);
CREATE TABLE Reservation (
    SagaID    uniqueidentifier PRIMARY KEY,   -- idempotency
    ProductID int NOT NULL, Qty int NOT NULL, State varchar(10) NOT NULL
);
CREATE TABLE Outbox (
    MsgID uniqueidentifier PRIMARY KEY DEFAULT NEWID(),
    Topic varchar(50), Payload nvarchar(max), SentAt datetime2 NULL
);

BEGIN TRAN;
    INSERT Reservation(SagaID, ProductID, Qty, State) VALUES (@Saga, @P, @Q, 'HELD');
    UPDATE Stock SET Reserved = Reserved + @Q
     WHERE ProductID = @P AND OnHand - Reserved >= @Q;
    IF @@ROWCOUNT = 0 THROW 50003, 'Insufficient stock', 1;
    INSERT Outbox(Topic, Payload) VALUES ('stock.reserved', @json);
COMMIT;
```
A replay of the same saga fails on `Reservation` primary key, so the step is idempotent. The `CHECK` plus conditional update makes oversell impossible even if the global flow is eventually consistent. The relay later reads `Outbox WHERE SentAt IS NULL`, publishes, and marks sent.

### 5. Under the hood
- **Why per-shard serializable fails globally**: each shard orders its own transactions, but a transaction touching two shards can be ordered T1<T2 on one and T2<T1 on the other; the union has a cycle. Global serializability needs a distributed concurrency protocol (2PC with locking, timestamps/TrueTime, Percolator-style snapshot with commit timestamps).
- **2PC mechanics**: prepare makes each participant durably promise it *can* commit and hold locks; the coordinator logs the decision; participants then commit or abort. If the coordinator dies between phases, participants stay in-doubt holding locks until recovery, which is why it is avoided across organizational or high-latency boundaries.
- **Snapshot-based distributed systems** (Percolator/TiDB-style) give snapshot isolation using a timestamp oracle and a primary lock per transaction; they permit write skew unless extended.
- **Sagas lack isolation (the "I" in ACID)**: another saga can read a reserved-but-not-confirmed state or see an intermediate result. Countermeasures: semantic locks (explicit pending states), commutative updates, re-reading values, versioning, pivot transactions (the point of no return).
- **Observability for anomalies**: log saga id, step, isolation level and invariant outcome; reconcile periodically (stock counts vs orders) to attribute KPI regressions (oversells) to specific anomalies.

### 6. Performance and design implications
- A saga turns synchronous latency into asynchronous completion; plan user experience around pending states.
- Compensations may fail or be impossible (an email sent); design steps so the irreversible one is last (the pivot).
- Outbox relays add small delay; ensure ordering where needed (per aggregate key partitioning).
- Hot rows (a flash-sale SKU) serialize in any model; shard the counter or queue requests per SKU.
- Reconciliation jobs are part of the system, not an afterthought.
- Vendor claims such as "serializable at NoSQL scale": ask which anomalies remain, how cross-partition transactions commit, behavior under partition, and clock assumptions.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `SET TRANSACTION ISOLATION LEVEL SERIALIZABLE` | Strong isolation in one database | Local invariants |
| Conditional `UPDATE ... WHERE avail >= n` | Local atomic guard | Inventory, quotas, balances |
| Outbox table + relay / CDC | Reliable event publication | Any cross-service step |
| Unique constraint on message/saga id | Idempotent consumer | Replays and duplicates |
| MS DTC / XA | 2PC coordinator | Rare, controlled environments |
| Saga orchestrator (workflow engine) | Drives steps and compensations | Complex flows |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| One serializable transaction | Data or services will split; contention high |
| 2PC | Participants unreliable, cross-region, long-running |
| Orchestrated saga | Few steps, need clear visibility and timeouts |
| Choreographed saga | Many independent consumers; harder to trace |
| Reservation (semantic lock) | Reservations can leak; need expiry |
| Reconciliation + alerting | Treated as optional |

### 9. Common mistakes
- Assuming serializable per shard gives global correctness → write skew across shards → move the invariant into one owner or add global coordination.
- Compensations that are not idempotent or can fail → stuck sagas → make each retryable, with manual escalation.
- Dual write (DB then message broker) → lost or phantom events → outbox or CDC.
- Treating saga intermediate states as final → customers see "paid" before inventory → expose and document pending states.
- No timeouts → reservations hang forever → reaper job.
- Choosing 2PC then calling an external payment API in it → blocking locks held over network → avoid mixing.

### 10. 🎯 Interview kit
**30-second answer:** Serializable gives a single database the illusion of serial execution; across services you cannot keep it, so you keep each invariant local with an atomic conditional update, sequence local transactions as a saga with compensations, publish events with an outbox, and make consumers idempotent. You trade isolation for availability and must design for visible intermediate states.

**Structure a longer answer:** clarify → core mechanism → trade-offs → close with a real scenario.
State the invariant, show where it can be enforced locally, then the saga steps, compensations, outbox, and the anomalies you accept.

**Likely follow-ups:**
- *2PC vs saga vs outbox?* → atomic-blocking vs eventual-compensating vs reliable messaging.
- *Write skew across shards?* → each shard's serial order differs; need a single owner or global protocol.
- *Vendor "serializable at NoSQL scale"?* → ask about cross-partition commit, clocks, anomalies, failure behavior.
- *How to prove oversell came from isolation anomaly?* → correlate logs by saga id, step, isolation level; reconciliation.
- *Cross-database Postgres and ledger?* → outbox from Postgres, idempotent ledger consumer, reconciliation.

**Signals interviewers look for:** you localize the invariant first, name what guarantees are lost, and include idempotency and reconciliation.

### 11. Check yourself
- [ ] Decompose a checkout into saga steps with compensations.
- [ ] Explain why 2PC blocks and when it is acceptable.
- [ ] Show an outbox write and an idempotent consume.
- [ ] Describe write skew across shards with an example.
- [ ] List three saga isolation countermeasures.
<!-- explainer:end -->

## Practice questions

- [Deep] [How would you set up observability to attribute a business KPI regression (e.g. oversells) back to an isolation-level anomaly rather than a code bug across a large service graph?](<../../../../PRACTICE/01_Databases_and_SQL/Transactions_and_Concurrency/Isolation_Levels/Q-30096_How_would_you_set_up_observability_to_attribute_a_business_K.md>)
- [Expert] [How would you evolve a monolith's single-transaction Serializable checkout into a saga across services while preserving the same no-oversell invariant? What replaces isolation?](<../../../../PRACTICE/01_Databases_and_SQL/Transactions_and_Concurrency/Isolation_Levels/Q-30104_How_would_you_evolve_a_monolith_s_single-transaction_Seriali.md>)
- [Expert] [A vendor claims their NewSQL database gives 'serializable at NoSQL scale.' As the architect signing the contract, what questions expose whether that's true or marketing?](<../../../../PRACTICE/01_Databases_and_SQL/Transactions_and_Concurrency/Isolation_Levels/Q-30106_A_vendor_claims_their_NewSQL_database_gives_serializable_at_.md>)
- [Expert] [How does write skew manifest in a distributed inventory system split across shards, and why can per-shard Serializable still permit the global anomaly? What's the architectural fix?](<../../../../PRACTICE/01_Databases_and_SQL/Transactions_and_Concurrency/Isolation_Levels/Q-30117_How_does_write_skew_manifest_in_a_distributed_inventory_syst.md>)
- [Expert] [You need cross-database consistency between Postgres (orders) and a separate ledger service. Compare 2PC, sagas, and the transactional outbox in terms of the isolation guarantees each preserves.](<../../../../PRACTICE/01_Databases_and_SQL/Transactions_and_Concurrency/Isolation_Levels/Q-30138_You_need_cross-database_consistency_between_Postgres_orders_.md>)

## Related concepts

- [Multi-tenant Migrations](<../../../../07_Distributed_Systems_and_Design/System_Design/System_Design_Core/Read-through_and_Write-behind_Caching/Multi-tenant_Migrations.md>) *(System Design Core)*
- [Distributed Counters and Consistency](<../../../../07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Distributed_Joins_in_MPP_Systems/Distributed_Counters_and_Consistency.md>) *(Distributed Systems Core)*
- [Isolation Anomalies](<../../../SQL_Language/SQL_Fundamentals/Isolation_Anomalies.md>) *(SQL Fundamentals)*

