---
id: "07_Distributed_Systems_and_Design/Distributed_Systems/03_Transactions_and_Consistency/Serializability"
level: 4
type: "curriculum"
curriculum: "data_engineering"
---

# Serializability Techniques

**L4 Advanced** · [↑ Transactions and Consistency](<../_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** Serializability guarantees concurrent transactions produce the same result as *some* one-at-a-time order, and databases achieve it in three ways: literally running one at a time, two-phase locking, or optimistic validation such as serializable snapshot isolation.

### 1. The big picture
Weak isolation levels are hard to reason about, inconsistently implemented (the meaning of "repeatable read" varies), and concurrency bugs are nondeterministic, so they are very hard to test for. Researchers' answer since the 1970s has been simple: **use serializable isolation**. If your transactions are correct when run alone, they remain correct when run concurrently — the database prevents all race conditions, including write skew and phantoms.

The reason it is not the universal default is cost. Each implementation technique trades something: throughput, latency stability, or abort rate. This lesson compares the three families; the parent topic (ACID and isolation) covers the weaker levels.

### 2. Core concepts and vocabulary
- **Schedule** — the ordered list of database operations (reads, writes, commits, aborts) from a set of transactions.
- **Serial schedule** — transactions run completely one after another with no interleaving.
- **Serializable schedule** — equivalent to *some* serial schedule over the same transactions (with *n* transactions there are *n!* serial orders; any one is acceptable).
- **Pessimistic concurrency control** — block if something might go wrong (locks).
- **Optimistic concurrency control (OCC)** — proceed, validate at commit, abort on conflict.
- **Two-phase locking (2PL)** — acquire locks while executing, release only at commit/abort.
- **Shared / exclusive lock** — read lock (many holders) vs write lock (one holder).
- **Deadlock** — two transactions each wait for the other's lock; the DB aborts one.
- **Index-range (predicate) lock** — a lock covering a range of index values so phantoms can't be inserted.
- **Read set / write set** — objects a transaction read / wrote.
- **SSI** — serializable snapshot isolation: snapshot isolation plus conflict detection.

### 3. How it works, step by step
**A. Actual serial execution**
1. One thread executes transactions one at a time, so isolation is trivially serializable.
2. This became practical (~2007) because RAM is cheap enough to hold the working set, and OLTP transactions are short; long analytics run on snapshots outside the loop.
3. Transactions are submitted as stored procedures (no waiting on a user mid-transaction).
4. To scale, partition data so each transaction touches one partition, with one thread per partition. Cross-partition transactions need lock-step coordination and are far slower (VoltDB reported ~1,000 cross-partition writes/s).
Used by VoltDB/H-Store, Redis, Datomic.

**B. Two-phase locking (2PL)**
1. Reading an object requires a shared lock; writing requires an exclusive lock; read-then-write upgrades.
2. Locks are held until the transaction ends (phase 1 acquire, phase 2 release-all at the end).
3. Readers block writers *and* writers block readers.
4. Deadlocks are detected automatically; one transaction is aborted and must be retried.
5. To stop phantoms, index-range locks cover ranges that *could* receive new rows.
Used by MySQL/InnoDB and SQL Server serializable, and DB2 repeatable read.

**C. Serializable snapshot isolation (SSI)**
1. Run on snapshot isolation (readers don't block writers).
2. Track when a transaction read data that another transaction then changed (stale MVCC reads / writes affecting prior reads).
3. At commit, if a dangerous dependency exists, abort the transaction; the app retries.
Used by PostgreSQL serializable (since 9.1) and FoundationDB (distributed conflict detection).

**General optimistic pattern (from database internals):** read phase in a private workspace → validation phase compares read/write sets with concurrent transactions → write phase applies changes atomically. Validation can be *backward-oriented* (against already-committed transactions) or *forward-oriented* (against in-flight ones).

### 4. Worked example
```sql
-- PostgreSQL SSI: two sessions, doctors on call
CREATE TABLE doctors(name text PRIMARY KEY, on_call bool, shift int);
INSERT INTO doctors VALUES ('alice',true,1),('bob',true,1);

-- Session A                                       -- Session B
BEGIN ISOLATION LEVEL SERIALIZABLE;                BEGIN ISOLATION LEVEL SERIALIZABLE;
SELECT count(*) FROM doctors                       SELECT count(*) FROM doctors
 WHERE on_call AND shift=1;      -- 2               WHERE on_call AND shift=1;      -- 2
UPDATE doctors SET on_call=false                   UPDATE doctors SET on_call=false
 WHERE name='alice';                                WHERE name='bob';
COMMIT;      -- ok
                                                   COMMIT;
-- ERROR: could not serialize access due to read/write dependencies among transactions
-- SQLSTATE 40001 -> application retries; the retry sees count = 1 and refuses
```
Under 2PL (InnoDB serializable) the same race would show up as a lock wait or deadlock rather than a commit-time error. Both are correct; the application must handle the failure by retrying the entire transaction.

### 5. Under the hood
**Backward-oriented OCC validation** lets T2 commit if one of these holds against each already-committed T1: T1 committed before T2's read phase started; or T1 committed before T2's write phase and T1's write set doesn't intersect T2's read set; or T1's read phase finished before T2's and T2's write set doesn't intersect T1's read or write sets. Validation and write are a short critical section — only one transaction validates/commits at a time — acceptable because those phases are shorter than the read phase.

**SSI's two detections.** (1) *Stale reads*: T read a row version while writer U was uncommitted; if U commits before T does, T's premise is outdated. (2) *Writes affecting prior reads*: when T writes, the DB checks if another transaction recently read the affected data (using index-range markers, not blocking) and flags a conflict. Waiting until commit avoids needless aborts when T is read-only or U aborts. Tracking granularity is a trade-off: fine-grained tracking aborts less but costs bookkeeping; PostgreSQL uses a theory showing some reads of overwritten data are still serializable to cut false aborts.

**Pessimism spectrum.** Serial execution is pessimism to the extreme — an exclusive lock on a whole partition, offset by making each transaction very fast. 2PL is fine-grained pessimism; SSI is optimism on top of MVCC. Commutative operations (counter increments) reduce contention in OCC.

### 6. Performance and design implications
- **2PL:** unstable tail latencies — one slow or large transaction can stall everything; deadlocks far more frequent than at read committed, and each costs a full redo.
- **Serial:** throughput capped by one core per partition; cross-partition work is expensive; needs everything in memory and short stored procedures.
- **SSI:** no read/write blocking; predictable latency; abort rate rises with contention and long read-write transactions; read-only queries run lock-free.
- Always implement retries with jittered backoff; monitor serialization failure rate (`pg_stat_database.xact_rollback`, deadlock counters).

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `SET TRANSACTION ISOLATION LEVEL SERIALIZABLE` | Request full serializability | Cross-row invariants |
| `SELECT … FOR UPDATE / FOR SHARE` | Explicit pessimistic locks | Under weaker levels as a fallback |
| `SET TRANSACTION READ ONLY DEFERRABLE` (PG) | Safe snapshot, no SSI aborts | Long reports/backups |
| `deadlock_timeout` (PG) | When to run deadlock detection | Tuning lock-heavy workloads |
| `innodb_lock_wait_timeout` | Max wait for a row lock | Contention control |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| Serial execution: in-memory, short, partitionable OLTP | Transactions are interactive/long or cross many partitions |
| 2PL: high contention where waiting beats re-doing work | Latency SLOs at high percentiles; many long transactions |
| SSI: read-heavy, moderate contention, need predictable latency | Hot rows with long read-write transactions — abort storms |

### 9. Common mistakes
- Calling an "ACID" database serializable without checking → write skew in production → verify level and engine semantics.
- Not retrying 40001/deadlock errors → user-visible failures → generic retry wrapper.
- Long-running serializable transactions → aborts or lock queues → shrink them, move reads to a read-only deferrable snapshot.
- Choosing serial execution then using multi-partition transactions heavily → throughput collapse → redesign partition keys.

### 10. 🎯 Interview kit
**30-second answer:** Serializability means concurrent transactions behave as if executed in some serial order. You get it by actually running serially, by two-phase locking, or by optimistic techniques like serializable snapshot isolation, which detect conflicts at commit and abort.

**Structure a longer answer:** define → three families → performance profile of each → pick by workload → mention retries.

**Likely follow-ups:**
- *Serializable vs linearizable?* → Serializable is about multi-object transaction ordering, with no real-time guarantee; linearizability is a single-object recency guarantee.
- *Why is 2PL slow?* → Lock overhead plus reduced concurrency and frequent deadlocks.
- *What does SSI do differently from plain OCC?* → Reads come from a snapshot; it only adds conflict detection.
- *Why abort at commit rather than at read time in SSI?* → Avoid aborting read-only transactions or ones whose conflicting writer later aborts.

**Signals interviewers look for:** knowing the three techniques and their failure modes, awareness of retry requirement, clear separation from linearizability.

### 11. Check yourself
- [ ] Define serial vs serializable schedules and count serial orders for 3 transactions.
- [ ] Explain why 2PL is called "two-phase" and why it suffers deadlocks.
- [ ] Describe the three OCC phases and the backward validation rules.
- [ ] Say when serial execution beats 2PL, and when SSI beats both.
<!-- explainer:end -->

## References

- *Designing Data-Intensive Applications* — Martin Kleppmann — Serializable Snapshot Isolation (SSI) › Performance of serializable snapshot isolation
- *Designing Data-Intensive Applications* — Martin Kleppmann — Serializable Snapshot Isolation (SSI)
- *Designing Data-Intensive Applications* — Martin Kleppmann — S
- *Designing Data-Intensive Applications* — Martin Kleppmann — Serializability
- *Database Internals* — Alex Petrov — Transaction Processing and Recovery › Optimistic Concurrency Control
- *Designing Data-Intensive Applications* — Martin Kleppmann — Serializability › Partitioning
- *Database Internals* — Alex Petrov — Transaction Processing and Recovery › Serializability
- *Designing Data-Intensive Applications* — Martin Kleppmann — Two-Phase Locking (2PL) › Performance of two-phase locking
- *Designing Data-Intensive Applications* — Martin Kleppmann — Serializable Snapshot Isolation (SSI) › Pessimistic versus optimistic concurrency control
- *Designing Data-Intensive Applications* — Martin Kleppmann — Serializability › Actual Serial Execution

