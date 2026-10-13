---
id: "01_Databases_and_SQL/Transactions_and_Concurrency/Isolation_Levels/Isolation_for_Shared_Subsystems"
level: 5
type: "deep-dive"
topic: "01_Databases_and_SQL/Transactions_and_Concurrency/Isolation_Levels"
tools: ["AWS"]
---

# Isolation for Shared Subsystems

**L5 Expert** · [↑ Isolation Levels](<../_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** When many services share a database or a platform component, isolation is a policy decision per invariant: put each rule where it can be enforced cheaply (constraint, atomic statement, single writer), document the level each transaction needs, and keep hot shared counters out of the primary database.

### 1. The big picture
At architect level the question changes from "which isolation level does this query use?" to "who owns each invariant, and what happens when forty services and three database engines touch it?" A shared subsystem (quota service, rate limiter, ledger, transaction library, schema migration tooling) concentrates contention: if its isolation or locking is wrong, its slowness becomes everyone's outage.

Three themes follow. (1) Pick the *mechanism* by the invariant: a declarative constraint, an atomic conditional statement, a lock, a queue/single-writer, or a higher isolation level, in roughly that order of preference. (2) Make isolation choices explicit and portable, so a policy survives an engine change (Oracle to PostgreSQL) and a hybrid on-prem/cloud period. (3) Recognize that connection pools, lock hold time and isolation jointly set your real concurrency ceiling.

### 2. Core concepts and vocabulary
- **Invariant** — a business rule that must always hold (quota not exceeded, one active subscription).
- **Enforcement tier** — database (constraint/isolation), application (locks/queues), or architecture (single-writer partitioning).
- **Hot row / hot counter** — a single row updated by very many transactions.
- **Quota check race** — `SELECT COUNT(*)` then `INSERT`; two sessions both see room.
- **Predicate / range lock** — protects the set of rows matching a condition (SERIALIZABLE).
- **Single-writer principle** — route all writes for a key through one actor/partition.
- **Pool ceiling** — effective concurrency ≈ pool size, bounded by lock hold time (Little's law).
- **Idempotent consumer** — at-least-once messages applied once.
- **Online DDL** — schema change running concurrently with DML.
- **Isolation governance standard** — written policy: default level, when to raise, required retry behavior.

### 3. How it works, step by step
Deciding where an invariant lives:

1. Write the invariant as a sentence ("a user may have at most 5 active API keys").
2. Can a **constraint** express it? (unique, check, FK, filtered unique index, exclusion constraint). If yes, use it: it holds under any isolation level and any code path.
3. Can a **single atomic statement** guard it? (`UPDATE ... WHERE used < limit`; `INSERT ... SELECT ... WHERE NOT EXISTS` with a unique index). Check rows affected.
4. Otherwise take an explicit **lock** on a stable anchor row (the user row `UPDLOCK`) so concurrent checkers serialize; this is cheaper and more predictable than raising the whole transaction to SERIALIZABLE.
5. If contention is extreme, move to **architecture**: partition by key and let one worker own it, or a queue; counters aggregated asynchronously.
6. Only then consider raising isolation (SERIALIZABLE/SSI) with a retry wrapper.

Reasoning about isolation across services:

1. A request fanning out to five services has no global isolation; the user gets, at best, a set of independently consistent reads.
2. Make the contract explicit: which entities must be mutually consistent? Co-locate them or read from one source of truth (snapshot read, version token passed along).
3. Pass an entity version or timestamp through the request so downstream reads are "at least as fresh as" it.

Pool and isolation sizing:

1. Estimate transactions per second and mean hold time → required concurrent connections (≈ rate × hold time).
2. Stricter isolation or lock waits raise hold time and therefore needed concurrency; a pool smaller than that queues requests.
3. A pool larger than the database can run (cores, I/O) increases contention; size to the database, not to the clients, and use a proxy/pooler for many services.

### 4. Worked example
Per-user quota, three variants.

```sql
-- Wrong: both transactions see 4 keys and both insert (RC or RR in many engines)
SELECT COUNT(*) FROM ApiKey WHERE UserID = @u AND Active = 1;   -- 4
INSERT ApiKey(UserID, Active) VALUES (@u, 1);

-- Right 1: serialize on the anchor row, short transaction
BEGIN TRAN;
SELECT 1 FROM AppUser WITH (UPDLOCK, ROWLOCK) WHERE UserID = @u;
IF (SELECT COUNT(*) FROM ApiKey WHERE UserID = @u AND Active = 1) < 5
    INSERT ApiKey(UserID, Active) VALUES (@u, 1);
ELSE THROW 50004, 'Quota exceeded', 1;
COMMIT;

-- Right 2: atomic counter owned by the anchor row
UPDATE AppUser SET ActiveKeys = ActiveKeys + 1
 WHERE UserID = @u AND ActiveKeys < 5;
-- rows affected = 0 → quota exceeded; then insert the key in the same transaction
```
Why SERIALIZABLE is the blunt option here: it would also work (range locks on the predicate) but locks every scan, produces deadlocks and retries, and depends on an index covering `UserID`.

A hot "likes" counter at web scale is different: any pessimistic level serializes all writers on one row. Use sharded counters (N rows per counter, sum on read), an append-only event table aggregated periodically, or a cache/stream processor that flushes batches.

### 5. Under the hood
- With READ COMMITTED a `COUNT(*)` takes S locks briefly and releases them; a concurrent insert that commits afterward changes the count the other transaction already used. REPEATABLE READ locks existing rows only, so phantoms remain; SERIALIZABLE adds range locks; snapshot-based levels do not stop write skew without extra locking or SSI.
- PostgreSQL REPEATABLE READ is snapshot isolation and holds an old snapshot, so long transactions delay vacuum (autovacuum bloat); a fleet of RR transactions needs monitoring of `xmin` horizon and tuned autovacuum.
- Online DDL needs brief metadata locks; a long transaction holding a lock can make the DDL queue, which then blocks all new DML behind it (lock queueing). Use lock timeouts and run DDL where no long transactions exist.
- Serverless databases can change connection scaling (cold starts, proxy-based pooling, transaction-mode pooling that breaks session-level state and advisory locks), so session-scoped assumptions need re-checking.
- Migrating engines changes semantics behind the same names (Oracle READ COMMITTED vs SQL Server RC without RCSI, MySQL RR with gap locks vs PostgreSQL RR snapshot), so a policy should be expressed in *invariants and required guarantees*, not level names.

### 6. Performance and design implications
- Isolation policy should state a default (READ COMMITTED or RCSI), the criteria to raise, and the mandatory retry behavior.
- Keep long human-in-the-loop work out of database transactions: model as workflow states with short transactions and optimistic checks.
- At-least-once consumers: processed-id table with unique key in the same transaction as the effect, so redelivery under READ COMMITTED never double-applies.
- Rate limiters and quotas shared by all tenants belong outside the primary OLTP database (in-memory/Redis, token buckets, local limiter with periodic sync) so their contention cannot become the platform bottleneck; persist for audit asynchronously.
- Test invariants with concurrency: chaos-style tests that fire parallel writers at the chosen level before any cutover; assert the invariant, not just absence of errors.
- A regulatory immutable-history requirement plus low-latency serving suggests an append-only ledger (consistent, auditable) with derived read models.
- Transaction abstraction libraries should expose isolation and retry as explicit parameters with safe defaults rather than hiding them.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| Unique / filtered unique / exclusion constraint | Declarative invariant | First choice |
| `UPDATE ... WHERE counter < limit` | Atomic guard | Quotas, stock |
| `WITH (UPDLOCK, ROWLOCK)` / `SELECT ... FOR UPDATE` | Lock anchor row | Check-then-write |
| `pg_advisory_xact_lock` | Application-defined lock | Serialize per key in PostgreSQL |
| `READ_COMMITTED_SNAPSHOT` / MVCC | Reader-writer non-blocking | Default for OLTP |
| `lock_timeout`, `SET LOCK_TIMEOUT` | Bound waits | DDL and migrations |
| Pooler (PgBouncer, RDS Proxy) | Share connections | Many services, serverless |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| Constraint-enforced invariant | Rule spans systems or needs complex predicates |
| Atomic conditional update | Hot single row at huge scale |
| Row lock on anchor | Anchor row becomes a bottleneck |
| SERIALIZABLE/SSI | Retries unacceptable or contention high |
| External counter/limiter | Strict global exactness required |
| Single-writer partition | Re-partitioning cost, ordering requirements |
| Common platform library | It hides isolation and retry semantics |

### 9. Common mistakes
- Quota check with `COUNT(*)` then insert → exceeded under concurrency → lock an anchor, use atomic counter, or constraint.
- Reaching for SERIALIZABLE everywhere → deadlocks, retries, throughput collapse → targeted enforcement.
- Hot counter on one row with pessimistic locks → serialization → shard or aggregate asynchronously.
- Long human-in-the-loop transaction → locks and snapshot bloat → workflow states.
- Policy written in engine-specific level names → surprises on migration → write in guarantees and test.
- Schema change run beside long transactions → DDL lock queue stalls the table → timeouts, off-peak, small steps.
- Pool sized to client count, not database capacity → thrash → size by rate × hold time, use a pooler.

### 10. 🎯 Interview kit
**30-second answer:** I place each invariant at the cheapest effective tier: a constraint first, then an atomic conditional statement, then a lock on an anchor row, then architectural single-writer or queue, and only then higher isolation with retries. For shared counters and rate limiters I keep them out of the primary database.

**Structure a longer answer:** clarify → core mechanism → trade-offs → close with a real scenario.
State the invariant and its contention profile, walk the tier ladder, then cover pooling, DDL, migration and governance.

**Likely follow-ups:**
- *Isolation and `COUNT(*)` quota?* → RC races; lock the user row or atomic counter; SERIALIZABLE works but is blunt.
- *Request fanning out across services?* → no global isolation; pass versions, read from owners, document guarantee.
- *Pool sizing?* → rate × hold time, bounded by database capacity; isolation lengthens hold time.
- *Engine migration policy?* → define invariants and required guarantees, test with concurrent chaos.
- *Hot counter?* → sharded counters / batch aggregation; pessimistic levels are the wrong tool.
- *At-least-once consumer?* → processed-id unique key in same transaction.

**Signals interviewers look for:** invariant-first reasoning, awareness of operational side effects (pools, vacuum, DDL), and refusal to default to the strongest level.

### 11. Check yourself
- [ ] Order the enforcement tiers and justify the order.
- [ ] Fix a COUNT-then-INSERT quota race three ways.
- [ ] Compute a pool size from rate and hold time.
- [ ] Explain why DDL can stall a hot table.
- [ ] Write an isolation policy in invariant terms, not level names.
<!-- explainer:end -->

## Implementations (tools)

- [AWS](<../../../../TOOLS/AWS/_index.md>)

## Practice questions

- [Intermediate] [How does the isolation level affect the behaviour of SELECT COUNT(*) used to enforce a per-user quota? Where would you place the lock or constraint?](<../../../../PRACTICE/01_Databases_and_SQL/Transactions_and_Concurrency/Isolation_Levels/Q-30213_How_does_the_isolation_level_affect_the_behaviour_of_SELECT_.md>)
- [Deep] [Explain how you'd design an isolation strategy for a system that must satisfy both a hard regulatory audit (immutable consistent history) and low-latency serving.](<../../../../PRACTICE/01_Databases_and_SQL/Transactions_and_Concurrency/Isolation_Levels/Q-30109_Explain_how_you_d_design_an_isolation_strategy_for_a_system_.md>)
- [Deep] [How do you design isolation for a system where the same row is a hot contended counter (likes/views) at web scale? Why is any pessimistic level the wrong tool here?](<../../../../PRACTICE/01_Databases_and_SQL/Transactions_and_Concurrency/Isolation_Levels/Q-30134_How_do_you_design_isolation_for_a_system_where_the_same_row_.md>)
- [Deep] [How do you decide isolation for a system where the same data is written by both a real-time API and a nightly bulk ETL that MERGEs millions of rows? How do you keep them from anomaly interference?](<../../../../PRACTICE/01_Databases_and_SQL/Transactions_and_Concurrency/Isolation_Levels/Q-30155_How_do_you_decide_isolation_for_a_system_where_the_same_data.md>)
- [Deep] [How do isolation levels shape your choice between a queue-in-the-database pattern and a dedicated broker? Where does SKIP LOCKED stop scaling and force the broker decision?](<../../../../PRACTICE/01_Databases_and_SQL/Transactions_and_Concurrency/Isolation_Levels/Q-30167_How_do_isolation_levels_shape_your_choice_between_a_queue-in.md>)
- [Expert] [You're standardizing isolation policy across 400 microservices on a shared Aurora fleet. How do you set a defensible default and let teams opt up or down without silent guarantee drift?](<../../../../PRACTICE/01_Databases_and_SQL/Transactions_and_Concurrency/Isolation_Levels/Q-30083_You_re_standardizing_isolation_policy_across_400_microservic.md>)
- [Expert] [How do you architect isolation for a multi-tenant SaaS where enterprise tenants demand Serializable SLAs but self-serve tenants share the same cluster at RC?](<../../../../PRACTICE/01_Databases_and_SQL/Transactions_and_Concurrency/Isolation_Levels/Q-30107_How_do_you_architect_isolation_for_a_multi-tenant_SaaS_where.md>)
- [Expert] [Explain how you'd design a formal or semi-formal spec (TLA+/Alloy) of a critical transaction to prove the chosen isolation level actually prevents the anomaly you care about.](<../../../../PRACTICE/01_Databases_and_SQL/Transactions_and_Concurrency/Isolation_Levels/Q-30135_Explain_how_you_d_design_a_formal_or_semi-formal_spec_TLA_Al.md>)

## Related concepts

- [Range Locks and Retries](<../../Locking/Range_Locks_and_Retries.md>) *(Locking)*
- [Isolation Anomalies](<../../../SQL_Language/SQL_Fundamentals/Isolation_Anomalies.md>) *(SQL Fundamentals)*
- [Isolation Levels in Transactions](<../../ACID_Transactions/Isolation_Levels_in_Transactions.md>) *(ACID Transactions)*

## References

- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 10: Transactions, Concurrency, and Isolation Levels › Isolation Levels and Their Trade-offs
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 10: Transactions, Concurrency, and Isolation Levels › Importance of Isolation Levels
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 10: Transactions, Concurrency, and Isolation Levels › Comparing Isolation Levels
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 10: Transactions, Concurrency, and Isolation Levels › Choosing the Right Isolation Level
- *Designing Data-Intensive Applications* — Martin Kleppmann — Chapter 7. Transactions › Isolation
- *Designing Data-Intensive Applications* — Martin Kleppmann — J › L
- *Database Internals* — Alex Petrov — Distributed Transactions › Distributed Transactions with Percolator
- *Data Governance with Unity Catalog on Databricks* — Kiran Sreekumar, Karthik Subbarao — Unity Catalog Under the Hood › Isolation using Unity Catalog Lakeguard in standard or shared access mode
- *Data Governance with Unity Catalog on Databricks* — Kiran Sreekumar, Karthik Subbarao — Unity Catalog Under the Hood › Data Isolation at Catalog and Schema Level
- *Data Governance with Unity Catalog on Databricks* — Kiran Sreekumar, Karthik Subbarao — Unity Catalog and Compute › Unity Catalog Lakeguard for standard cluster

