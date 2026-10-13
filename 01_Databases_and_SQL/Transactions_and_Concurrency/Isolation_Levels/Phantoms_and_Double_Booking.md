---
id: "01_Databases_and_SQL/Transactions_and_Concurrency/Isolation_Levels/Phantoms_and_Double_Booking"
level: 4
type: "concept"
topic: "01_Databases_and_SQL/Transactions_and_Concurrency/Isolation_Levels"
tools: ["MySQL", "Oracle"]
---

# Phantoms and Double Booking

**L4 Advanced** · [↑ Isolation Levels](<_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** Isolation levels are defined by which **read anomalies** they allow: dirty reads, non-repeatable reads and phantoms. Phantoms (new rows appearing in a range you already checked) are what cause double bookings.

### 1. What it is
The SQL standard defines levels by the anomalies they **prevent**:

| Anomaly | What you see | Example |
|---|---|---|
| **Dirty read** | Data another transaction hasn't committed (and may roll back) | A report shows an order that is then cancelled |
| **Non-repeatable read** | You read the same **row** twice and get different values | The price changes between two reads in one transaction |
| **Phantom** | You run the same **range query** twice and get different **rows** | `COUNT(*) WHERE seat=12 AND show=7` returns 0, then 1 |
| **Read skew** | Related rows read at different moments don't agree | An account balance and its ledger sum differ mid-transfer |

**Double booking** is a phantom in practice. Two requests both check "is seat 12 free?" (no rows), and both insert a booking. Row locks can't help, because the row being fought over **doesn't exist yet**.

### 2. Why it matters
- Most applications run at READ COMMITTED (the default in SQL Server, PostgreSQL and Oracle), which allows non-repeatable reads, phantoms and read skew.
- Check-then-insert logic (bookings, unique usernames, quotas) is the classic source of phantom bugs.
- ETL jobs and reports at READ COMMITTED can produce totals that never existed at any single moment (read skew).

### 3. How it works
| Level | Dirty | Non-repeatable | Phantom | Mechanism (typical) |
|---|---|---|---|---|
| READ UNCOMMITTED | ✔ allowed | ✔ | ✔ | No shared locks on read (SQL Server `NOLOCK`) |
| READ COMMITTED | ✖ | ✔ | ✔ | Short read locks, or a statement-level snapshot (RCSI, PostgreSQL, Oracle) |
| REPEATABLE READ | ✖ | ✖ | ✔ in the standard\* | Read locks held to the end (SQL Server), or a transaction snapshot |
| SNAPSHOT | ✖ | ✖ | ✖ for reads | Transaction-level row versions (still allows write skew) |
| SERIALIZABLE | ✖ | ✖ | ✖ | **Key-range locks** (SQL Server, MySQL next-key locks) or SSI (PostgreSQL) |

\*In PostgreSQL and MySQL, REPEATABLE READ uses a snapshot, so *reads* don't see phantoms. But a check-then-insert can still double book, because the snapshot doesn't block the other insert.

Range protection works by locking the **gap**. SERIALIZABLE in SQL Server takes key-range locks on the index range your WHERE clause scanned, and InnoDB takes next-key locks. This only works if an **index** covers the predicate. Without one, the engine locks much more (up to the whole table).

### 4. Key details to remember
- Dirty reads are impossible at READ COMMITTED and above, in every major engine.
- Oracle effectively offers only READ COMMITTED and its "SERIALIZABLE", which is snapshot isolation. It has no dirty reads at all.
- `NOLOCK` / READ UNCOMMITTED can also return **duplicated or missing rows** during page splits, not just uncommitted ones.
- Long snapshot transactions in PostgreSQL stop VACUUM from removing old row versions, which causes table bloat.

### 5. Trade-offs and fixes for double booking
| Fix | How | Cost |
|---|---|---|
| **Unique constraint** (best) | `UNIQUE (show_id, seat_no)`, so the second insert fails | Simple; handle the duplicate-key error |
| Exclusion constraint (PostgreSQL) | Prevents overlapping time ranges | PostgreSQL only |
| Lock an anchor row | `SELECT … FROM shows WHERE id=7 FOR UPDATE`, then check and insert | Serializes bookings per show |
| SERIALIZABLE + retry | Range locks or SSI catch the conflict | Blocking or aborts; needs a retry loop |
| Pre-created seat rows | `UPDATE seats SET booked_by=? WHERE seat=12 AND booked_by IS NULL` | Turns a phantom into a single-row atomic update |

### 6. Common mistakes
- "Our ORM uses READ COMMITTED, so we're consistent" → phantoms and read skew are still possible → add constraints or lock anchor rows.
- Using `NOLOCK` everywhere for speed → you get wrong, duplicated or missing rows → use RCSI or snapshot for non-blocking reads.
- Raising everything to SERIALIZABLE after an incident → p99 latency spikes and deadlocks → protect only the critical check-then-act path.
- Range protection on an unindexed column → huge locks → index the predicate column.

### 7. 🎯 Interview kit
**30-second answer:** "Isolation levels are defined by the anomalies they allow. READ COMMITTED stops dirty reads but allows non-repeatable reads and phantoms. A phantom is a new row appearing in a range you already checked, and it's exactly what causes double bookings: two transactions see 'no booking', and both insert. Row locks can't help because the row doesn't exist yet. I'd fix it with a unique constraint, or by locking a parent row, and use SERIALIZABLE with retries only on that path."

**Structure a longer answer:**
1. Clarify: which engine and default level? Is it check-then-insert or update-existing? What's the booking volume?
2. Core idea: name the anomaly table, then explain why phantoms need **range** (gap) locking rather than row locks.
3. Trade-offs: constraint vs anchor lock vs SERIALIZABLE (blocking, aborts, retries).
4. Close: "Let the database enforce the invariant declaratively wherever possible."

**Likely follow-ups:**
- *Why can't REPEATABLE READ stop a double booking?* → It protects rows you **read**. It doesn't stop **new** rows being inserted into the range (gap locks are only taken at SERIALIZABLE).
- *How do you diagnose latency after moving to SERIALIZABLE?* → Look at lock waits and range locks on unindexed predicates, deadlock graphs and retry rates.
- *How should an ETL read a table that's being updated?* → Use a snapshot (transaction-level) read, or read from a consistent export, not plain READ COMMITTED.
- *Postmortem: double charge during a promo. Corrective action?* → Add an idempotency key with a unique constraint, write an anomaly test under concurrency, and set a policy for check-then-act code.

**Signals interviewers look for:** using the anomaly vocabulary precisely, explaining *why* phantoms need range locks, and preferring constraints over isolation hacks.

### 8. Check yourself
- [ ] Fill in the anomaly-by-level table from memory.
- [ ] Explain why a unique constraint fixes double booking regardless of isolation level.
- [ ] Describe what SQL Server SERIALIZABLE locks for `WHERE seat_no BETWEEN 10 AND 20`.

*See the References section of this note for the books this lesson draws on.*
<!-- explainer:end -->

## Implementations (tools)

- [MySQL](<../../../TOOLS/MySQL/_index.md>)
- [Oracle](<../../../TOOLS/Oracle/_index.md>)

## Practice questions

- [Basic] [What is a dirty read and why can't it happen at READ COMMITTED or above? Give a scenario where it would corrupt a financial report.](<../../../PRACTICE/01_Databases_and_SQL/Transactions_and_Concurrency/Isolation_Levels/Q-30125_What_is_a_dirty_read_and_why_can_t_it_happen_at_READ_COMMITT.md>)
- [Intermediate] [How would you explain to a backend engineer why 'our ORM defaults to READ COMMITTED' matters for their money-transfer code?](<../../../PRACTICE/01_Databases_and_SQL/Transactions_and_Concurrency/Isolation_Levels/Q-30223_How_would_you_explain_to_a_backend_engineer_why_our_ORM_defa.md>)
- [Intermediate] [A nightly ETL under READ COMMITTED reads a dimension table while it's being updated, producing a mix of old and new rows. How do you make the read set consistent?](<../../../PRACTICE/01_Databases_and_SQL/Transactions_and_Concurrency/Isolation_Levels/Q-30234_A_nightly_ETL_under_READ_COMMITTED_reads_a_dimension_table_w.md>)
- [Intermediate] [What does 'skewed read' look like when reading two related tables under READ COMMITTED, and how do you make a multi-table read atomic in effect?](<../../../PRACTICE/01_Databases_and_SQL/Transactions_and_Concurrency/Isolation_Levels/Q-30262_What_does_skewed_read_look_like_when_reading_two_related_tab.md>)
- [Deep] [A team reports intermittent stale reads from a read replica. Is that an isolation-level problem or a replication-lag problem? How do you distinguish them?](<../../../PRACTICE/01_Databases_and_SQL/Transactions_and_Concurrency/Isolation_Levels/Q-30197_A_team_reports_intermittent_stale_reads_from_a_read_replica_.md>)
- [Deep] [How do phantom reads specifically affect aggregate constraints like 'no more than N rows matching a predicate'? Why can't row locks alone stop them?](<../../../PRACTICE/01_Databases_and_SQL/Transactions_and_Concurrency/Isolation_Levels/Q-30217_How_do_phantom_reads_specifically_affect_aggregate_constrain.md>)
- [Deep] [How does MVCC let readers avoid blocking writers? Where does the old row version live in PostgreSQL versus Oracle?](<../../../PRACTICE/01_Databases_and_SQL/Transactions_and_Concurrency/Isolation_Levels/Q-30258_How_does_MVCC_let_readers_avoid_blocking_writers_Where_does_.md>)
- [Expert] [How does the choice of isolation level affect index behaviour under concurrent inserts — specifically, why do gap locks appear only for some index types?](<../../../PRACTICE/01_Databases_and_SQL/Transactions_and_Concurrency/Isolation_Levels/Q-30233_How_does_the_choice_of_isolation_level_affect_index_behaviou.md>)

## Related concepts

- [Range Locks and Retries](<../Locking/Range_Locks_and_Retries.md>) *(Locking)*
- [Isolation Levels in Transactions](<../ACID_Transactions/Isolation_Levels_in_Transactions.md>) *(ACID Transactions)*

## References

- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 10: Transactions, Concurrency, and Isolation Levels › The Phenomena of Concurrency
- *Database Reliability Engineering* — Laine Campbell, Charity Majors — Datastore Field Guide › Isolation
- *Designing Data-Intensive Applications* — Martin Kleppmann — Weak Isolation Levels › Implementing read committed
- *Designing Data-Intensive Applications* — Martin Kleppmann — Snapshot Isolation and Repeatable Read
- *Designing Data-Intensive Applications* — Martin Kleppmann — Snapshot Isolation and Repeatable Read › Visibility rules for observing a consistent snapshot
- *Designing Data-Intensive Applications* — Martin Kleppmann — Serializable Snapshot Isolation (SSI) › Detecting writes that affect prior reads
- *100 SQL Server Mistakes and How to Avoid Them* — Peter A. Carter — Optimization › 10.11 #64 Using unnecessarily strong isolation levels
- *Think Distributed Systems* — Dominik Tornow — Transactions › 5.3 The model of transactions
- *Think Distributed Systems* — Dominik Tornow — Transactions › 5.3.4 Application-level abort
- *SQL Server Tutorial* (YouTube series) — Part 76: Read committed snapshot isolation level in sql server ([watch](https://www.youtube.com/watch?v=S-z3uuhdUew))

