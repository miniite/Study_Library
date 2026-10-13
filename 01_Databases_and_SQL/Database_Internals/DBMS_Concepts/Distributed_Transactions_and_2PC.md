---
id: "01_Databases_and_SQL/Database_Internals/DBMS_Concepts/Distributed_Transactions_and_2PC"
level: 4
type: "concept"
topic: "01_Databases_and_SQL/Database_Internals/DBMS_Concepts"
tools: ["MySQL"]
---

# Distributed Transactions and 2PC

**L4 Advanced** · [↑ DBMS Concepts](<_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** A distributed transaction makes changes on several nodes appear all-or-nothing; two-phase commit (2PC) achieves it with a *prepare/vote* phase and a *commit/abort* phase driven by a coordinator — correct, but blocking when the coordinator fails, which is why large systems prefer consensus protocols, sagas or idempotent messaging.

### 1. The big picture
A single-node transaction gets atomicity from one write-ahead log. Once a transaction spans two databases (or a database and a message broker), there is no single log: node A may commit while node B crashes. We need an **atomic commitment protocol**: all participants commit or all abort.

2PC is the classic answer (X/Open XA, MS DTC, PostgreSQL `PREPARE TRANSACTION`, JTA). It guarantees atomicity but has well-known costs — extra round trips, locks held across the network, and a *blocking* failure mode. Modern distributed databases usually do not run bare 2PC; they layer it over replicated consensus (Paxos/Raft) per shard, or avoid cross-shard transactions with sagas and the outbox pattern.

### 2. Core concepts and vocabulary
- **Atomic commitment** — every participant reaches the same outcome (commit or abort).
- **Coordinator (transaction manager)** — the node that drives the protocol and records the decision.
- **Participant (resource manager)** — a database/queue that executes part of the transaction and votes.
- **Prepare / vote phase** — participant makes its changes durable and promises it *can* commit; after voting yes it may not unilaterally abort.
- **In-doubt (uncertain) state** — a participant that voted yes and has not yet heard the decision; it must hold locks.
- **Commit point** — when the coordinator durably logs "commit"; the outcome is then fixed.
- **Blocking** — participants cannot decide alone if the coordinator is lost.
- **3PC** — adds a pre-commit phase to avoid blocking under assumptions (synchronous network); rarely used.
- **Saga** — sequence of local transactions with compensating actions; no global atomicity or isolation.
- **Outbox pattern** — write business change and message in one local transaction; relay publishes later.
- **Consensus (Paxos/Raft)** — replicated agreement on a value/log; makes a decision survive node failure.
- **Idempotency** — applying an operation twice has the same effect as once; essential for retries.
- **XA** — standard interface between a transaction manager and resource managers.

### 3. How it works, step by step
**Two-phase commit**
1. *Begin:* client sends work to coordinator; coordinator enlists participants P1…Pn (each runs local sub-transactions, holding locks).
2. *Phase 1 — prepare:* coordinator sends `PREPARE` to all.
3. Each participant checks it can commit, **forces its log** (prepare record + redo/undo info), and replies `YES` (now in-doubt, locks held) or `NO` (aborts locally).
4. *Decision:* if all voted yes, coordinator **forces a commit record** to its log (the commit point); any no or timeout → abort record.
5. *Phase 2:* coordinator sends `COMMIT`/`ABORT` to all.
6. Participants apply the decision, release locks, log completion, `ACK`.
7. Coordinator writes an end record and forgets the transaction.

**Failure behavior**

| Failure | Result |
|---|---|
| Participant fails before voting | Coordinator times out → abort |
| Participant fails after voting yes | On recovery it reads its log, asks the coordinator for the outcome |
| Coordinator fails before decision | Participants that voted yes are **blocked**, holding locks, until coordinator recovers |
| Coordinator fails after decision logged | Recovery re-sends the decision |
| Network partition | Participants in doubt wait |

**Message cost:** 2 round trips and 4 forced log writes per participant path — latency of the slowest participant dominates.

### 4. Worked example
```sql
-- PostgreSQL-style manual 2PC against two databases (illustrative)
-- On DB A:
BEGIN;
UPDATE accounts SET balance = balance - 100 WHERE id = 1;
PREPARE TRANSACTION 'transfer-42-A';
-- On DB B:
BEGIN;
UPDATE accounts SET balance = balance + 100 WHERE id = 9;
PREPARE TRANSACTION 'transfer-42-B';

-- Both prepared OK -> coordinator decides commit:
COMMIT PREPARED 'transfer-42-A';   -- on A
COMMIT PREPARED 'transfer-42-B';   -- on B
-- If either prepare failed: ROLLBACK PREPARED on the other.

-- Orphans after a crash show up here and hold locks until resolved:
SELECT gid, prepared, owner FROM pg_prepared_xacts;
```
In SQL Server the same is done by MSDTC: `BEGIN DISTRIBUTED TRANSACTION; … COMMIT;` across linked servers, or `TransactionScope` escalating to a distributed transaction in .NET.

**Alternative: saga with outbox**
```sql
-- Service A, one local transaction:
BEGIN TRAN;
  UPDATE Orders SET Status='Pending' WHERE OrderID=@id;
  INSERT Outbox(EventId, Type, Payload) VALUES (NEWID(),'OrderPlaced',@json);
COMMIT;
-- A relay publishes Outbox rows to the broker at-least-once;
-- consumers deduplicate by EventId (idempotent), and failures trigger compensations
-- (e.g. 'CancelOrder' event) instead of a global rollback.
```
What to look at: orphaned prepared transactions, long lock waits (`sys.dm_tran_locks`, `pg_locks`), DTC timeouts, outbox lag.

### 5. Under the hood
- **Why prepare must be durable.** A "yes" is a promise. If the participant crashed and forgot, it could not honor a later commit. Hence a forced log write before replying.
- **Why blocking is inherent.** FLP-style reasoning: with an asynchronous network and a crash-failure, no protocol can both always terminate and never violate safety. 2PC chooses safety (never inconsistent) over liveness (may wait).
- **Heuristic outcomes.** Operators can force an in-doubt participant to commit or abort (`heuristic decision`) to free locks — risking inconsistency if the guess is wrong; this is a last-resort recovery.
- **Presumed abort/commit optimizations.** Coordinators may omit logging/ACKs for the presumed outcome to save forced writes (Presumed Abort is common).
- **Consensus-backed commit.** Spanner/CockroachDB/YugabyteDB run 2PC *across* shards but make each participant (and the coordinator record) a Paxos/Raft group, so a failed leader is replaced and the in-doubt state is not stuck. Timestamps (TrueTime/HLC) order transactions; isolation is usually serializable or snapshot.
- **Lakehouse analog.** Table formats (Delta Lake) avoid distributed commit by making commit a single atomic operation (put-if-absent of a log file) with optimistic concurrency; concurrent writers detect conflicts and retry — serializable writes without 2PC.
- **Consistency vs isolation.** 2PC gives atomicity; isolation across nodes still depends on each node's locking/MVCC and ordering — global serializability needs more (timestamp ordering or global locks).

### 6. Performance and design implications
- Latency ≈ 2 × RTT to the slowest participant + log forces; throughput suffers from lock hold time → hot rows become bottlenecks.
- Availability = product of all participants' availability; the coordinator is a single point of failure unless replicated.
- Keep distributed transactions short, few participants, same region; avoid user think-time inside them.
- Prefer designing data so transactions are single-shard (partition by the transactional entity).
- When cross-service workflows are needed, use sagas/outbox with idempotent handlers and compensations; accept eventual consistency and design UX/reconciliation for it.
- Monitor in-doubt transactions, DTC/XA timeouts, retries, duplicate processing.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `BEGIN DISTRIBUTED TRANSACTION` (SQL Server) | Start MSDTC-coordinated tx | Linked-server writes |
| `SET XACT_ABORT ON` | Abort whole tx on error | Always with distributed tx |
| `PREPARE TRANSACTION` / `COMMIT PREPARED` (PG) | Manual 2PC | External coordinators |
| `max_prepared_transactions` (PG) | Enable prepared tx | XA workloads |
| XA / JTA `TransactionManager` | Standard 2PC API | Java EE, Spring |
| `sys.dm_tran_active_transactions`, `pg_prepared_xacts` | See in-doubt work | Troubleshooting |
| Outbox table + relay / CDC | Reliable messaging | Microservices |
| Idempotency keys | Dedupe retries | APIs and consumers |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| 2PC/XA | Few participants, same data center, strong atomicity required, legacy systems |
| Consensus-backed distributed SQL | You need global transactions with HA |
| Saga + outbox | Microservices, long-running workflows, tolerate eventual consistency |
| Single-database transaction | You can co-locate the data — always the simplest |
| Idempotent at-least-once messaging | Side effects cannot be made idempotent |

### 9. Common mistakes
- Distributed transactions over slow/unreliable links → timeouts and long locks → redesign or use sagas.
- Treating the coordinator as reliable → blocked participants → replicate it or use consensus.
- Retrying non-idempotent operations → duplicates → idempotency keys.
- Leaving prepared transactions orphaned → locks and bloat (PG vacuum can't clean) → monitor and resolve.
- Assuming 2PC gives isolation → anomalies persist → define isolation separately.
- Calling external HTTP APIs inside a DB transaction → can't be rolled back → outbox.

### 10. 🎯 Interview kit
**30-second answer:** 2PC makes a multi-node transaction atomic: the coordinator asks everyone to prepare and vote, and only if all say yes does it log commit and tell them to commit. It's safe but blocking — if the coordinator dies after participants voted yes, they hold locks until it recovers — so at scale we use consensus-replicated coordinators or sagas with idempotent steps.

**Structure a longer answer:** clarify (atomicity needed? latency? number of services?) → 2PC phases and logs → failure modes → alternatives (3PC, consensus, saga/outbox) → trade-offs and a scenario (money transfer vs order workflow).

**Likely follow-ups:**
- *What happens if the coordinator crashes after commit decision?* → recovery replays decision from log.
- *Why not 3PC?* → needs bounded delays/fail-stop assumptions; partitions break it.
- *2PC vs Paxos/Raft?* → atomic commit across partitions vs replication/agreement; used together.
- *Saga vs 2PC?* → no isolation, compensations, eventual consistency, higher availability.
- *Outbox?* → solves dual-write problem.

**Signals interviewers look for:** precise failure analysis, knowledge of durability points, and pragmatic alternatives.

### 11. Check yourself
- [ ] Walk through 2PC and mark every forced log write.
- [ ] Explain exactly where a participant becomes "in doubt" and why it must hold locks.
- [ ] List three ways to avoid distributed transactions.
- [ ] Design an outbox + idempotent consumer for an order-payment flow.
- [ ] Explain how consensus changes 2PC's availability.
<!-- explainer:end -->

## Implementations (tools)

- [MySQL](<../../../TOOLS/MySQL/_index.md>)

## Practice questions

- [Basic] [What are the ACID properties of a transaction? Explain each.](<../../../PRACTICE/01_Databases_and_SQL/Database_Internals/DBMS_Concepts/Q-22161_What_are_the_ACID_properties_of_a_transaction_Explain_each.md>)
- [Intermediate] [What is denormalization and when would you deliberately denormalize a schema?](<../../../PRACTICE/01_Databases_and_SQL/Database_Internals/DBMS_Concepts/Q-22160_What_is_denormalization_and_when_would_you_deliberately_deno.md>)
- [Intermediate] [What is caching in front of a database and what is the cache-aside pattern?](<../../../PRACTICE/01_Databases_and_SQL/Database_Internals/DBMS_Concepts/Q-22221_What_is_caching_in_front_of_a_database_and_what_is_the_cache.md>)
- [Intermediate] [What is autocommit mode and how does it affect whether each statement is its own transaction?](<../../../PRACTICE/01_Databases_and_SQL/Database_Internals/DBMS_Concepts/Q-22420_What_is_autocommit_mode_and_how_does_it_affect_whether_each_.md>)
- [Deep] [What is a deadlock in a database and how can it be prevented or resolved?](<../../../PRACTICE/01_Databases_and_SQL/Database_Internals/DBMS_Concepts/Q-22185_What_is_a_deadlock_in_a_database_and_how_can_it_be_prevented.md>)
- [Deep] [Explain the CAP theorem and how it applies to distributed databases.](<../../../PRACTICE/01_Databases_and_SQL/Database_Internals/DBMS_Concepts/Q-22219_Explain_the_CAP_theorem_and_how_it_applies_to_distributed_da.md>)
- [Deep] [What is a distributed transaction and what problem does two-phase commit try to solve?](<../../../PRACTICE/01_Databases_and_SQL/Database_Internals/DBMS_Concepts/Q-22311_What_is_a_distributed_transaction_and_what_problem_does_two-.md>)
- [Deep] [How do you retry a transaction that fails with a serialization or deadlock error, and where should that retry live in the application stack?](<../../../PRACTICE/01_Databases_and_SQL/Database_Internals/DBMS_Concepts/Q-22510_How_do_you_retry_a_transaction_that_fails_with_a_serializati.md>)

## Related concepts

- [Isolation Anomalies](<../../SQL_Language/SQL_Fundamentals/Isolation_Anomalies.md>) *(SQL Fundamentals)*
- [Read Your Writes with Replicas](<../../../06_Cloud_Computing/Cloud_Foundations/Cloud_Fundamentals/Read_Your_Writes_with_Replicas.md>) *(Cloud Fundamentals)*
- [Lock Holding and Optimistic Concurrency](<../../Transactions_and_Concurrency/ACID_Transactions/Lock_Holding_and_Optimistic_Concurrency.md>) *(ACID Transactions)*

## References

- *Delta Lake: The Definitive Guide* — Denny Lee et al. — Architecting Your Lakehouse › Transaction Support
- *Designing Data-Intensive Applications* — Martin Kleppmann — Encoding and Evolution › Terminology clash
- *Designing Data-Intensive Applications* — Martin Kleppmann — Chapter 7. Transactions
- *Designing Data-Intensive Applications* — Martin Kleppmann — Single-Object and Multi-Object Operations › The need for multi-object transactions
- *Designing Data-Intensive Applications* — Martin Kleppmann — System Model and Reality › Safety and liveness
- *Hadoop: The Definitive Guide* — Tom White — Flume › Transactions and Reliability
- *Database Internals* — Alex Petrov — Introduction and Overview › Shared State in a Distributed System
- *Database Internals* — Alex Petrov — Distributed Transactions › Two-Phase Commit
- *Data Engineering Design Patterns* — Bartosz Konieczny — Error Management Design Patterns › State Stores
- *Fundamentals of Data Engineering* — Joe Reis, Matt Housley — Data Generation in Source Systems › CRUD

