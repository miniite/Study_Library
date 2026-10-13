---
id: "03_Big_Data_and_Streaming/02_Stream_Processing/Databases_and_Streams/Immutability_and_Derived_Views"
level: 5
type: "curriculum"
curriculum: "data_engineering"
---

# State Streams and Immutability

**L5 Expert** · [↑ Databases and Streams](<../_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** Current state is the integral of an immutable stream of events, and any number of read-optimized views can be derived from that one log — which makes systems easier to audit, debug, evolve and recover.

### 1. The big picture

Mutable state hides history: an `UPDATE` destroys the old value. But the mutable state you see is only a *cache* of a deeper truth — the sequence of changes that produced it. Accounting has always worked this way: a ledger is append-only, mistakes are corrected by a new entry, and the balance sheet is a view derived from the ledger.

Applying this to data systems gives a powerful mental model:

> **state** = integral of events over time; **event stream** = derivative of state.

If you keep the immutable log, you can derive as many views as you need — a table for transactions, a search index, a cache, a warehouse fact table — and rebuild any of them from scratch. The log is the system of record; everything else is derived, disposable data.

### 2. Core concepts and vocabulary

- **Immutable event** — a record that is never updated or deleted after being written.
- **Event log** — append-only ordered sequence of events.
- **State** — the result of applying all events to an initial value.
- **Derived view / read model** — a materialization of the log shaped for a specific query pattern.
- **Materialized view** — a stored, incrementally maintained query result.
- **Reprocessing** — recomputing a view from the log with new code.
- **Compensating event** — a new event that corrects an earlier mistake.
- **Lambda/Kappa architecture** — combining or unifying batch and stream paths over the same log.
- **CQRS** — separating the model you write to from the models you read from.
- **Excision / crypto-shredding** — ways of removing data from an otherwise immutable log.

### 3. How it works, step by step

1. **Record facts.** Every change is appended as an event (or captured via CDC).
2. **Derive a view.** A consumer reads events in order and updates a data structure optimized for a query (e.g., current balance per account, inverted index).
3. **Serve queries** from that view; writes still go to the log.
4. **Change requirements.** Need a new query pattern? Create a new view by replaying the log from the beginning with new code — without touching existing consumers.
5. **Fix bugs.** Wrong projection logic? Fix the code, drop the view, replay. Compare old and new views side by side, then switch traffic.
6. **Correct data errors.** Don't edit history; append a compensating event (e.g., a refund), keeping the audit trail.
7. **Archive.** Old log segments move to cheap storage; snapshots keep replay time bounded.

```text
          ┌── view A: current balances (OLTP lookup)
 event ───┼── view B: search index
  log     ├── view C: daily revenue (warehouse)
          └── view D: ML features
```

### 4. Worked example

A bank ledger as an immutable event table:

```sql
CREATE TABLE ledger (
  seq        bigint GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
  account_id int,
  amount     numeric(12,2),           -- + deposit / - withdrawal
  reason     text,
  at         timestamptz DEFAULT now()
);

-- A mistake: 50.00 was deposited by error. Do NOT delete; append a correction.
INSERT INTO ledger(account_id, amount, reason) VALUES (7, -50.00, 'correction of seq 1042');

-- Derived views:
CREATE MATERIALIZED VIEW balance AS
SELECT account_id, SUM(amount) AS balance FROM ledger GROUP BY account_id;

CREATE MATERIALIZED VIEW monthly_flow AS
SELECT account_id, date_trunc('month', at) AS m, SUM(amount) AS net
FROM ledger GROUP BY 1, 2;
```

Benefits seen here: the balance is always explainable (every entry is there), `monthly_flow` was added later without migrating data, and an audit asks "what was the balance on 1 March?" — answerable by summing entries up to that date, which a mutable `balance` column could not do.

### 5. Under the hood

- **Advantages of immutable events.**
  - *Auditability and debugging*: you can replay exactly what happened.
  - *Recoverability*: a buggy deployment that corrupts a mutable database is unrecoverable unless there is a backup; with an event log you fix the code and recompute.
  - *Richer information*: a cart item added then removed carries signal an overwrite would lose.
  - *Concurrency simplicity*: appends don't conflict like in-place updates.
- **Several views from one log** — separating the write format from read formats removes the schema-design compromise of normalization vs denormalization: write in the form that is easy to record; read in whatever form is fast. This is CQRS. Views are denormalized caches that can be rebuilt, so denormalization's usual update-anomaly danger is controlled by deriving them from the log.
- **Concurrency and low-level mechanics.** LSM-trees embody the same principle: immutable segments, buffered writes and background merges. Immutability simplifies concurrency (readers never see partial updates), at the cost of compaction work.
- **Limitations.**
  - *Storage growth*: workloads with high update rates produce an enormous log; compaction/snapshots are needed; some data are never worth keeping.
  - *Deletion*: privacy law and mistakes (leaked secrets) require true deletion; truly removing data from an immutable log means rewriting history (excision) or crypto-shredding. Even then copies may exist in backups and downstream views — "making it hard to retrieve" is often the realistic goal.
  - *Operational complexity*: more moving parts (log, consumers, rebuild pipelines).
  - *Read-your-writes lag*: views are asynchronously updated.
- **Fan-out semantics.** Because each view is a pure function of the log, adding a view cannot damage existing ones; failure in one consumer is isolated.
- **Reprocessing for evolution.** Gradual migration: run old and new views in parallel from the same log, shift traffic gradually, remove the old when comfortable — a rollback is always available.

### 6. Performance and design implications

- Rebuild time = log size / replay throughput; plan snapshots and parallel replay.
- Views trade storage for speed: precompute what is queried frequently.
- Compaction (by key) is valid when the latest value per key is the state; otherwise keep full history.
- Design events to be small, self-contained and versioned.
- Keep PII separated or encrypted per subject for later deletion.

### 7. Commands and settings you should know

| Command / setting | What it does | When to use |
|---|---|---|
| `CREATE MATERIALIZED VIEW` / `REFRESH ... CONCURRENTLY` | Stored derived result | Warehouse-style read models |
| Kafka `retention.ms=-1` / compaction | Keep full history or latest per key | Log as system of record |
| Consumer offset reset to earliest | Rebuild a view by replay | After logic change |
| Snapshots / checkpoints | Bound rebuild time | Long logs |
| Append-only table + no `UPDATE` grants | Enforce immutability | Ledgers, audit logs |
| Per-subject encryption keys | Crypto-shredding | GDPR erasure |

### 8. Trade-offs and decisions

| Choose this when | Avoid / watch out when |
|---|---|
| Auditability, evolution and many read patterns matter | Data is small, simple and rarely changes |
| You can tolerate asynchronous views | You require synchronous read-your-writes everywhere |
| Updates are low-moderate in volume | Extreme churn on a small set of keys with no value in history |
| Compliance requires history (finance) | Compliance requires hard deletion without a crypto plan |

### 9. Common mistakes

- **Editing history to fix errors** → breaks replay and audit → append compensating events.
- **Keeping views as the only copy** → cannot rebuild → keep the log as source of truth.
- **No plan for deletion** → compliance risk → separate PII or encrypt per subject.
- **Unbounded log without snapshots** → rebuilds take days → snapshot/archive.
- **Events too thin or too fat** → thin ones need lookups at replay, fat ones leak and bloat → balance facts and references.
- **Assuming views are consistent with each other at a moment** → each lags independently → design for eventual consistency.

### 10. 🎯 Interview kit

**30-second answer:** Treat the append-only event log as the source of truth and current state as a derived view. Because events are immutable you get an audit trail, can rebuild or add views by replaying, and recover from bugs by fixing code and recomputing. The costs are storage growth, handling deletion, and eventual consistency of views.

**Structure a longer answer:** state vs events (integral/derivative) → separation of write and read models → benefits (audit, recovery, evolution) → limits (storage, deletion, lag) → scenario (ledger or CQRS).

**Likely follow-ups:**
- *How do you correct a mistake?* → Append a compensating event.
- *How do you comply with GDPR?* → Crypto-shredding or keeping PII out of the log, plus rebuilding views.
- *What is CQRS and why useful?* → Separate write/read models, each optimized.
- *How does this relate to LSM-trees?* → Same principle at the storage layer.

**Signals interviewers look for:** sees the log as the primary data and views as caches, and also names the deletion and consistency limits.

### 11. Check yourself

- [ ] Explain "state is the integral of the event stream".
- [ ] Show how to add a new report to a ledger system without changing existing data.
- [ ] List three limits of immutability and mitigation for each.
- [ ] Describe how you would migrate to a new view with zero downtime.
<!-- explainer:end -->

## References

- *Designing Data-Intensive Applications* — Martin Kleppmann — Event Sourcing › State, Streams, and Immutability
- *Designing Data-Intensive Applications* — Martin Kleppmann — D
- *Designing Data-Intensive Applications* — Martin Kleppmann — Event Sourcing › Limitations of immutability
- *Designing Data-Intensive Applications* — Martin Kleppmann — Event Sourcing › Deriving several views from the same event log
- *Database Internals* — Alex Petrov — Introduction and Overview › Buffering, Immutability, and Ordering
- *Designing Data-Intensive Applications* — Martin Kleppmann — Event Sourcing › Advantages of immutable events
- *Database Internals* — Alex Petrov — Log-Structured Storage › Concurrency in LSM Trees
- *Hadoop: The Definitive Guide* — Tom White — ZooKeeper › Watch triggers
- *Learning Spark (2nd ed.)* — Jules S. Damji, Brooke Wenig, Tathagata Das, Denny Lee — Data Transformations › Stateless Transformations
- *Hadoop: The Definitive Guide* — Tom White — MapReduce Types and Formats › Warning

