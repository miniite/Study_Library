---
id: "07_Distributed_Systems_and_Design/Distributed_Systems/03_Transactions_and_Consistency/ACID_and_Isolation"
level: 4
type: "curriculum"
curriculum: "data_engineering"
---

# ACID and Weak Isolation Levels

**L4 Advanced** · [↑ Transactions and Consistency](<../_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** A transaction bundles several reads and writes into one unit with four promises (ACID), and the *isolation* promise is not all-or-nothing — databases sell a ladder of weaker levels that trade correctness guarantees for speed.

### 1. The big picture
Almost every application needs to change several pieces of data together: debit one account and credit another, insert an order and decrement stock, store an email and bump an unread counter. Without help from the database you would have to reason about crashes halfway through, and about other clients reading or writing the same rows at the same moment. A **transaction** is the database's answer: you group the operations, and the database promises they behave as one logical unit.

The promises are summarised as **ACID** — Atomicity, Consistency, Isolation, Durability — a term coined in 1983 by Härder and Reuter. The word is more marketing than specification today: two databases that both say "ACID compliant" can behave very differently, mostly because *isolation* is implemented in many strengths. This lesson sets up the vocabulary; its child lessons go deep on snapshot isolation/MVCC and on write skew and phantoms, and the sibling topic *Serializability* covers the strongest level.

### 2. Core concepts and vocabulary
- **Transaction** — a group of reads and writes that either all take effect (*commit*) or none do (*abort*/*rollback*).
- **Atomicity** — if a fault happens partway (crash, network cut, full disk, constraint violation), everything the transaction wrote so far is discarded. It is *not* about concurrency; "abortability" would be a better name. Its payoff: an aborted transaction can be retried safely, with no half-applied changes and no duplicates.
- **Consistency** — application-level invariants ("credits and debits balance") stay true. This one is really the *application's* job; the database only helps via constraints. Atomicity, isolation and durability are database properties, consistency is not.
- **Isolation** — concurrently running transactions don't step on each other. In the strict textbook sense it means *serializability*: the result is as if they ran one after another.
- **Durability** — once commit returns, the data survives crashes. On one node: written to non-volatile storage, usually through a write-ahead log. In a replicated system: copied to some number of nodes before commit is acknowledged.
- **BASE** — "Basically Available, Soft state, Eventual consistency"; effectively just "not ACID".
- **Dirty read** — reading another transaction's uncommitted write.
- **Dirty write** — overwriting another transaction's uncommitted write.
- **Non-repeatable read (read skew)** — reading the same row twice in one transaction and getting different committed values.
- **Phantom** — a repeated search query returns a different *set* of rows because someone inserted or deleted matches.
- **Lost update / write skew** — two concurrent read-modify-write cycles where one overwrites or invalidates the other (detail in the child lessons).

### 3. How it works, step by step
Atomicity and durability are usually delivered by the same machinery — a log:

1. The transaction's changes are first appended to a **write-ahead log** (WAL) on durable storage.
2. At commit, the commit record is flushed (`fsync`) to disk; only then does the client see "committed". In a replicated database the commit may also wait for replicas.
3. After a crash, recovery replays committed log records (redo) and discards those of transactions that never committed (undo). That is atomicity *and* durability.

Isolation is the hard one. The database can't cheaply run everything serially, so it offers levels. Each level is defined by which anomalies it still allows:

| Level | Dirty reads | Non-repeatable reads | Phantoms | Write skew |
|---|---|---|---|---|
| Read uncommitted | possible | possible | possible | possible |
| Read committed | prevented | possible | possible | possible |
| Repeatable read / snapshot isolation | prevented | prevented | prevented for reads (SI) | possible |
| Serializable | prevented | prevented | prevented | prevented |

(The SQL-standard "repeatable read" is defined loosely and different engines map it to different behaviours — PostgreSQL's is snapshot isolation, MySQL/InnoDB's differs. Always check your engine's documentation.)

**Read committed** gives two guarantees: you only read committed data (no dirty reads) and you only overwrite committed data (no dirty writes). Dirty reads are bad because a reader could see some of a transaction's writes but not others (the unread email without the updated counter), or see data later rolled back. Dirty writes are bad because two concurrent transactions updating the same rows could interleave and leave a mixed result (Alice gets the car, Bob gets the invoice).

**Serializability** is the strongest: transactions may run concurrently, but the outcome must equal *some* serial order. It requires coordination between conflicting transactions, which is why it costs throughput.

### 4. Worked example
```sql
-- Session A                                  -- Session B
BEGIN;                                        
UPDATE accounts SET balance = balance - 100
  WHERE id = 1;                               -- not committed yet
                                              BEGIN;
                                              SELECT balance FROM accounts WHERE id = 1;
                                              -- Read committed: still the OLD balance (no dirty read)
UPDATE accounts SET balance = balance + 100
  WHERE id = 2;
COMMIT;
                                              SELECT balance FROM accounts WHERE id = 1;
                                              -- Read committed: NEW balance -> non-repeatable read
                                              -- Snapshot isolation: still the OLD balance
                                              COMMIT;
```
Under read committed, session B's two identical `SELECT`s return different values — legal, and exactly what snapshot isolation fixes. To see which level you are on in PostgreSQL: `SHOW transaction_isolation;`. To change it for one transaction: `BEGIN ISOLATION LEVEL REPEATABLE READ;`.

### 5. Under the hood
- **Durability is not absolute.** Disks and SSDs can lie about `fsync` on power loss, firmware has bugs, and silent corruption can spread into replicas and backups. Memory-only commits are lost by correlated failures (a whole rack or a crash bug triggered by one input). Asynchronous replication can lose the most recent writes at failover. Real durability is layered: WAL + replication + tested backups.
- **How read committed is usually built:** row-level write locks held until commit prevent dirty writes. Dirty reads are prevented not by read locks (a long writer would block all readers) but by remembering, per row, both the old committed value and the new uncommitted one; readers get the old one until commit.
- **How stronger levels are built:** multi-version storage (MVCC) for snapshots; two-phase locking, serial execution or serializable snapshot isolation for full serializability (see the sibling lessons).

### 6. Performance and design implications
- Stronger isolation = more blocking or more aborts. Read committed is the default in many OLTP databases precisely because it has the best latency/throughput.
- Retries are mandatory: serialization failures and deadlocks are *normal* signals at higher levels, and the application must catch and retry the whole transaction.
- Keep transactions short: long transactions hold locks (or old snapshots) longer and amplify every cost above.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `BEGIN` / `COMMIT` / `ROLLBACK` | Delimit a transaction | Any multi-statement change |
| `SET TRANSACTION ISOLATION LEVEL …` | Choose the level for the next/current transaction | When an invariant needs more than the default |
| `SELECT … FOR UPDATE` | Take row locks on what you read | Read-modify-write on existing rows |
| `SHOW transaction_isolation` (PostgreSQL) | Display current level | Debugging anomalies |
| `synchronous_commit` (PostgreSQL) | Wait for WAL flush/replica before acking | Trading durability for latency |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| Read committed: independent short statements, reports that tolerate small drift | Multi-step logic whose correctness depends on seeing a stable state |
| Snapshot / repeatable read: consistent long reads, backups, analytics | Read-then-write logic that checks an invariant over several rows (write skew) |
| Serializable: invariants span rows and you can't enumerate every race | Very high contention and no retry logic — aborts pile up |
| Relaxed "BASE" systems: scale and availability matter more than cross-row invariants | You'd have to reinvent atomicity and constraints in application code |

### 9. Common mistakes
- Assuming "ACID" on a product page means serializable → subtle race conditions in production → read the engine's actual default level and anomalies.
- Not retrying aborted transactions → user-visible errors or lost work → wrap transactions in a retry loop with backoff.
- Treating "consistency" as something the database gives for free → invalid states that constraints never described → encode invariants as constraints or serializable transactions.
- Believing `COMMIT` means data is on several machines → failover loses writes → check sync vs async replication settings.

### 10. 🎯 Interview kit
**30-second answer:** ACID is atomic all-or-nothing writes, application-defined consistency, isolation between concurrent transactions, and durability after commit. Isolation is a spectrum — read committed, snapshot isolation, serializable — each allowing fewer anomalies at higher cost.

**Structure a longer answer:** clarify which letter → core mechanism (WAL for A and D, locks/versions for I) → trade-offs between levels → close with a concrete anomaly (two doctors going off call) and which level prevents it.

**Likely follow-ups:**
- *Does atomicity mean "no one sees half a transaction"?* → No, that's isolation; atomicity is about abort-and-undo on failure.
- *Who is responsible for consistency?* → The application; the DB only enforces declared constraints.
- *What does read committed not protect against?* → Non-repeatable reads, lost updates, write skew, phantoms.
- *Is durability guaranteed forever?* → No; it's a probability game across disk, replicas, and backups.

**Signals interviewers look for:** you separate atomicity from isolation, name specific anomalies, know that defaults are weak, and mention retry on abort.

### 11. Check yourself
- [ ] Define each ACID letter in one sentence and say which are database properties vs. application properties.
- [ ] List the anomalies prevented by read committed and the ones it still allows.
- [ ] Explain why dirty reads are normally avoided with old-value copies instead of read locks.
- [ ] Give an example where "committed" data can still be lost despite durability.
<!-- explainer:end -->

## In this section (study in order)

1. [Snapshot Isolation and MVCC](<Snapshot_Isolation_MVCC/_index.md>) (L5 Expert)
2. [Write Skew and Phantoms](<Write_Skew_and_Phantoms/_index.md>) (L5 Expert)

## References

- *Database Internals* — Alex Petrov — Transaction Processing and Recovery › Isolation Levels
- *Fundamentals of Data Engineering* — Joe Reis, Matt Housley — Data Generation in Source Systems › ACID
- *Designing Data-Intensive Applications* — Martin Kleppmann — Weak Isolation Levels › Read Committed
- *Designing Data-Intensive Applications* — Martin Kleppmann — Chapter 7. Transactions › Atomicity
- *Designing Data-Intensive Applications* — Martin Kleppmann — Chapter 7. Transactions › Durability
- *Designing Data-Intensive Applications* — Martin Kleppmann — Chapter 7. Transactions › The Meaning of ACID

