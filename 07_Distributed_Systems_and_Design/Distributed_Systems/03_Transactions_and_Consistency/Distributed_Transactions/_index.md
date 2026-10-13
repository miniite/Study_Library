---
id: "07_Distributed_Systems_and_Design/Distributed_Systems/03_Transactions_and_Consistency/Distributed_Transactions"
level: 4
type: "curriculum"
curriculum: "data_engineering"
---

# Distributed Transactions and Two-Phase Commit

**L4 Advanced** · [↑ Transactions and Consistency](<../_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** To make a transaction that spans several nodes commit on all of them or none, systems use an atomic commit protocol — most commonly two-phase commit (2PC), which is simple and cheap but *blocks* if the coordinator dies at the wrong moment.

### 1. The big picture
On one node, atomic commit is easy because a single device decides the outcome: the transaction commits at the instant the commit record reaches the write-ahead log on disk. Before that moment a crash aborts it; after it, a crash can't undo it.

Once a transaction touches several partitions, or a secondary index lives on a different node than the data, you can't just send "commit" to every node independently: some would succeed and others fail (a constraint violation, a crash, a lost packet), leaving the data half-applied. Nodes can't "un-commit" either, because commit is irrevocable once visible to others. You need an **atomic commitment** protocol: all participants reach the *same* outcome, and one "no" vote vetoes the commit. This is different from consensus: participants only vote on a proposed transaction, they can't propose alternatives, and a single no-vote means abort.

### 2. Core concepts and vocabulary
- **Atomic commitment** — an agreement problem: commit on all participants or abort on all.
- **Coordinator (transaction manager)** — the process that drives the protocol and holds the decision log.
- **Cohort / participant** — a node (usually a partition) that does part of the transaction and votes.
- **Prepare / propose phase** — coordinator asks "can you commit?"; a *yes* is a binding promise.
- **Commit / abort phase** — coordinator announces the decision.
- **Point of no return** — when the coordinator durably writes its decision; the outcome is then fixed.
- **In-doubt (uncertain) state** — a participant voted yes but hasn't heard the decision; it must wait, holding locks.
- **Blocking protocol** — participants can't make progress without the coordinator.
- **Three-phase commit (3PC)** — adds a pre-commit step and timeouts to avoid blocking, under synchronous-network assumptions.
- **XA** — the standard API for heterogeneous distributed transactions.
- **Heuristic decision** — an operator or participant unilaterally commits/aborts an in-doubt transaction, risking inconsistency.

### 3. How it works, step by step
**Two-phase commit**
1. The application starts a global transaction and does reads/writes on several nodes; the coordinator obtains a globally unique transaction ID.
2. When the app asks to commit, the coordinator sends **prepare** to every participant.
3. Each participant checks it can definitely commit (data is durable in its log, constraints hold, no conflict) and replies yes/no. By answering *yes* it **surrenders the right to abort**.
4. The coordinator collects all votes. All yes → **commit**; any no or timeout → **abort**.
5. The coordinator **writes the decision to its own durable log**. This is the commit point.
6. It sends commit/abort to all participants, retrying forever if one is unreachable.
7. Participants apply the decision, release locks, and acknowledge.

**Failure cases**
| Failure | Outcome |
|---|---|
| Participant crashes before voting | Coordinator times out → abort |
| Participant crashes after voting yes | On recovery it asks the coordinator for the decision, and applies it |
| Coordinator crashes before logging decision | Participants stuck in doubt until it recovers (or abort via recovery protocol) |
| Coordinator crashes after logging decision | Decision is safe; delivered after recovery |

**Three-phase commit** inserts a *pre-commit* round: propose and vote → coordinator broadcasts "prepare to commit" and collects acks → commit. Once all participants are pre-committed they all know everyone voted yes, so on coordinator failure they can safely commit by timeout; if the coordinator fails before pre-commit completes, they abort by timeout. It needs bounded network delay and bounded process pauses, which real networks don't give — so it's rarely used.

### 4. Worked example
Transferring $100 between accounts on two shards:
```
Coordinator C, shards A (debit), B (credit)
1  C -> A, B : PREPARE tx-77
2  A : writes redo/undo to log, holds row lock on acct 1, replies YES
3  B : writes log, holds lock on acct 2, replies YES
4  C : writes "COMMIT tx-77" to its log      <- point of no return
5  C -> A, B : COMMIT tx-77
6  A, B : apply, release locks, ACK
```
Same flow in PostgreSQL's SQL interface for a prepared transaction:
```sql
-- on shard A
BEGIN; UPDATE acct SET bal = bal - 100 WHERE id = 1;
PREPARE TRANSACTION 'tx-77';          -- durable, locks still held
-- ... coordinator gathers votes ...
COMMIT PREPARED 'tx-77';              -- or ROLLBACK PREPARED 'tx-77'

SELECT * FROM pg_prepared_xacts;      -- find orphaned in-doubt transactions
```
If the coordinator crashes after step 3 and never recovers, `tx-77` stays in `pg_prepared_xacts` holding its row locks, blocking other writers until someone resolves it.

### 5. Under the hood
- **Why the coordinator's log matters:** it is the single source of truth. A participant that voted yes cannot decide alone, because other participants may already have committed. Replicating commit decisions is safe because they are unanimous by construction; a backup coordinator or peer logs can supply the answer.
- **Locks during doubt:** in-doubt participants keep exclusive locks, so a stuck coordinator can freeze hot rows indefinitely. Recovery often requires an administrator to commit/abort manually, sacrificing atomicity.
- **Replicated coordinator:** to avoid being a single point of failure, the coordinator's decision log can itself be replicated with consensus (Raft/Paxos), as in Spanner (2PC across Paxos groups) — consensus makes the *coordinator* fault-tolerant.
- **Atomic commit ≠ consensus, but related:** atomic commit requires *unanimity* on a fixed proposal; consensus lets any proposal win by majority. Consensus-based commit protocols exist (e.g. Paxos Commit) that tolerate coordinator failure.
- **Not Byzantine tolerant:** a node that lies about its vote breaks unanimity.
- **Homogeneous vs heterogeneous:** inside one database product, engines can optimise the protocol. Across different systems (database + message broker via XA), the lowest common denominator applies and the coordinator is usually an application-library process whose crash leaves orphan transactions.

### 6. Performance and design implications
- Overhead: extra network round trips and forced log writes per commit. Reports show MySQL distributed transactions being 10× slower than single-node ones.
- Availability: *every* participant must be up for commit; availability of the transaction is the product of node availabilities.
- Prefer designs where a transaction touches one partition (co-locate related data) and use **sagas / idempotent compensations / outbox patterns** for cross-service workflows that don't need true atomicity.
- Monitor in-doubt transactions, lock wait times, and coordinator recovery time.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `PREPARE TRANSACTION` / `COMMIT PREPARED` (PostgreSQL) | Manual 2PC participant primitives | External coordinator |
| `max_prepared_transactions` (PostgreSQL) | Enables prepared transactions | Must be > 0 for 2PC |
| `XA START/END/PREPARE/COMMIT` (MySQL) | XA transactions | Cross-engine atomicity |
| `XA RECOVER` (MySQL) | List in-doubt XA txns | After crashes |
| `pg_prepared_xacts` | View orphaned prepared txns | Lock pile-up diagnosis |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| 2PC: need true atomicity across a few nodes, fast LAN, reliable coordinator | WAN latency, many participants, strict availability goals |
| Replicated coordinator (consensus) | You can't afford the added complexity |
| 3PC: only in theory/learning — synchronous assumptions | Real networks with unbounded delay |
| Sagas/outbox: microservices, long workflows | You need isolation, not just eventual atomicity |

### 9. Common mistakes
- Assuming a prepared transaction can time out safely → inconsistent outcome → never abort unilaterally after voting yes.
- Forgetting orphan in-doubt transactions → locks held for days → alert on `pg_prepared_xacts` age.
- Using XA across systems with an unreliable app-server coordinator → orphans after crash → persist coordinator log, test recovery.
- Retrying a failed commit without idempotency → duplicate effects → use transaction IDs.

### 10. 🎯 Interview kit
**30-second answer:** 2PC has a prepare phase where each participant votes and promises it can commit, then a commit phase where the coordinator, after durably logging its decision, tells everyone. It guarantees atomicity but blocks if the coordinator fails after participants voted yes.

**Structure a longer answer:** why single-node commit doesn't generalise → the two phases and two points of no return (yes votes; coordinator log) → failure cases → blocking & remedies (replicated coordinator, 3PC) → real alternatives (sagas).

**Likely follow-ups:**
- *Why is 2PC blocking?* → In-doubt participants can't know the decision nor safely abort.
- *How does 3PC avoid blocking?* → Pre-commit state plus timeouts; assumes synchronous network, so impractical.
- *2PC vs Paxos/Raft?* → 2PC needs everyone; consensus needs a majority; they complement each other.
- *What does Spanner do?* → 2PC between Paxos-replicated groups.

**Signals interviewers look for:** you identify the in-doubt window, mention locks held while waiting, and know practical alternatives.

### 11. Check yourself
- [ ] Draw the 2PC message flow and mark both points of no return.
- [ ] Explain why a participant that voted yes cannot unilaterally abort.
- [ ] Describe what happens when the coordinator fails at each step.
- [ ] Contrast atomic commit with consensus.
- [ ] Name two ways to avoid distributed transactions.
<!-- explainer:end -->

## References

- *Designing Data-Intensive Applications* — Martin Kleppmann — Distributed Transactions and Consensus › Atomic Commit and Two-Phase Commit (2PC)
- *Designing Data-Intensive Applications* — Martin Kleppmann — Don't confuse 2PC and 2PL › A system of promises
- *Designing Data-Intensive Applications* — Martin Kleppmann — W › X
- *Designing Data-Intensive Applications* — Martin Kleppmann — Distributed Transactions and Consensus › Introduction to two-phase commit
- *Database Internals* — Alex Petrov — Distributed Transactions › Three-Phase Commit
- *Database Internals* — Alex Petrov — Distributed Transactions › Two-Phase Commit
- *Designing Data-Intensive Applications* — Martin Kleppmann — Don't confuse 2PC and 2PL › Limitations of distributed transactions
- *Database Internals* — Alex Petrov — Distributed Transactions › Coordinator Failures in 2PC
- *Database Internals* — Alex Petrov — Distributed Transactions › Making Operations Appear Atomic
- *Designing Data-Intensive Applications* — Martin Kleppmann — Distributed Transactions and Consensus › From single-node to distributed atomic commit

