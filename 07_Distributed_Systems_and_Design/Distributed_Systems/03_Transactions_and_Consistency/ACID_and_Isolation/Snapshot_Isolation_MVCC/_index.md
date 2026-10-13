---
id: "07_Distributed_Systems_and_Design/Distributed_Systems/03_Transactions_and_Consistency/ACID_and_Isolation/Snapshot_Isolation_MVCC"
level: 5
type: "curriculum"
curriculum: "data_engineering"
---

# Snapshot Isolation and MVCC

**L5 Expert** · [↑ ACID and Weak Isolation Levels](<../_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** Snapshot isolation (built on multi-version concurrency control) lets every transaction read from a frozen, consistent view of the database, so readers never block writers and writers never block readers.

### 1. The big picture
Read committed stops dirty reads, but a transaction can still see the database change *under its feet*. If you read two account balances while a transfer commits between your two reads, you see $500 and $400 — $100 has vanished (read skew). That's harmless for a page refresh but disastrous for **backups** (the inconsistency becomes permanent in the copy) and for **analytics or integrity checks** that scan lots of data over minutes.

Snapshot isolation (SI) fixes this: each transaction sees the database as of the moment it started, plus its own writes. To make that cheap, the engine keeps several committed versions of each row — **MVCC**. Marketing names vary: PostgreSQL, Oracle, SQL Server (snapshot level) and MySQL/InnoDB (as "repeatable read") all offer it. It's the isolation level underneath most OLTP workloads, and it is also where write skew and phantoms (next lesson) are born.

### 2. Core concepts and vocabulary
- **Snapshot** — the set of committed transactions visible to a reader; frozen at transaction (or statement) start.
- **MVCC** — keeping multiple versions of each object, tagged with the transaction IDs that created and deleted them.
- **Transaction ID (`txid`)** — monotonically increasing number assigned at transaction start.
- **`created_by` / `deleted_by`** — per-row-version markers: the txid that inserted it and (if any) the txid that deleted it.
- **Visibility rules** — the logic deciding which row versions a transaction may see.
- **Garbage collection / vacuum** — removing versions no running transaction can still see.
- **Lost update** — two read-modify-write cycles where the second write clobbers the first.
- **Read skew / non-repeatable read** — seeing two different committed states in one transaction.
- **Append-only (copy-on-write) B-tree** — an alternative where each write creates a new root, and each root is a snapshot (CouchDB, Datomic, LMDB).

### 3. How it works, step by step
Following the PostgreSQL-style design:

1. A starting transaction gets a unique, increasing `txid` and notes which transactions are currently in progress.
2. Every row version is tagged `created_by = txid of the inserter`. A delete does not remove the row; it sets `deleted_by = txid of the deleter`. An **update** is a delete plus an insert of a new version.
3. When the transaction reads, a row version is **visible** only if:
   - the creating transaction had committed before the reader's snapshot was taken, **and**
   - the row is not marked deleted, or the deleting transaction had not committed when the snapshot was taken.
   Writes by transactions that started later, or were still in progress, are ignored.
4. Writes take row locks, so two transactions writing the *same* row still serialise (this prevents dirty writes). But reads take no locks at all.
5. Periodically a background process removes versions that no live snapshot can see.

Read committed vs snapshot isolation inside an MVCC engine differ only in *when* the snapshot is taken: read committed takes a fresh snapshot **per statement**, snapshot isolation takes **one per transaction**.

### 4. Worked example
```sql
-- PostgreSQL
CREATE TABLE acct(id int PRIMARY KEY, bal int);
INSERT INTO acct VALUES (1,500),(2,500);

-- Session A (long report)                   -- Session B (transfer)
BEGIN ISOLATION LEVEL REPEATABLE READ;
SELECT sum(bal) FROM acct;      -- 1000
                                             BEGIN;
                                             UPDATE acct SET bal = bal-100 WHERE id=1;
                                             UPDATE acct SET bal = bal+100 WHERE id=2;
                                             COMMIT;
SELECT sum(bal) FROM acct;      -- still 1000
COMMIT;
SELECT sum(bal) FROM acct;      -- 1000 as well, but now a new snapshot

-- Peek at versions (PostgreSQL system columns)
SELECT xmin, xmax, * FROM acct;
```
`xmin` is the creating txid and `xmax` the deleting/locking txid — PostgreSQL's names for `created_by`/`deleted_by`. After B's update, row 1 exists as two versions: the old one (xmax = B's txid) and the new one (xmin = B's txid). Session A keeps seeing the old one until it ends.

Lost-update behaviour differs by engine. Two sessions both doing `SELECT bal … ; UPDATE … SET bal = <computed>`:
- PostgreSQL repeatable read, Oracle serializable, SQL Server snapshot: the second writer is **aborted** ("could not serialize access…") and must retry.
- MySQL/InnoDB repeatable read: **no detection** — the second write silently overwrites. Use `SELECT … FOR UPDATE` or `UPDATE … SET bal = bal - 100`.

### 5. Under the hood
**Indexes with versions.** One approach: the index points to *all* versions of a row, and the executor filters invisible ones by the visibility rules; when GC drops a version, its index entries go too. PostgreSQL optimises by avoiding index updates if a new version fits on the same page. The alternative is the append-only B-tree: pages are never overwritten; a write copies the modified leaf and every ancestor up to a new root. Each root is a consistent snapshot, so no txid filtering is needed — but you must compact and garbage-collect old pages.

**Why MVCC at the page level matters.** In a B-tree engine, deletes only drop cell offsets and leave bytes in place; old versions stay "addressable" for concurrent transactions until none can see them, producing fragmentation that vacuum/compaction repairs. "Ghost records" are tracked and freed once all transactions that might see them finish.

**Timestamp ordering vs locking.** MVCC can be combined with 2PL, timestamp ordering, or optimistic validation; its main modern role is implementing snapshot isolation.

**Distributed snapshots.** A single counter is fine for one node, but across partitions a global monotonic txid becomes a coordination bottleneck, and it must respect causality (if B read A's write, B's ID must exceed A's). Spanner uses synchronized clocks: TrueTime returns an interval [earliest, latest]; a read-write transaction deliberately *waits out* the uncertainty (~7 ms with GPS/atomic clocks) before committing, so non-overlapping intervals guarantee ordering.

**Detecting stale MVCC reads (for SSI).** If transaction T ignored a write by U because U was uncommitted at snapshot time, and U commits before T commits, T's premise may be outdated. Serializable snapshot isolation tracks this and aborts T at commit time — not at read time, because T may be read-only or U may yet abort.

### 6. Performance and design implications
- Readers/writers don't block each other; long analytic scans coexist with OLTP.
- Cost: storage bloat (dead versions), vacuum/GC load, and index pressure. Long-running transactions *pin* old versions and cause bloat — watch for idle-in-transaction sessions.
- Monitor: PostgreSQL `pg_stat_activity` (`xact_start`, `state`), `n_dead_tup` in `pg_stat_user_tables`; SQL Server tempdb version-store size.
- Use atomic updates (`SET bal = bal + 1`) instead of read-modify-write in the app wherever possible.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `BEGIN ISOLATION LEVEL REPEATABLE READ` (PG) | One snapshot per transaction | Consistent multi-query reports |
| `ALTER DATABASE … SET ALLOW_SNAPSHOT_ISOLATION ON` (SQL Server) | Enables SI | Row-versioning workloads |
| `READ_COMMITTED_SNAPSHOT ON` (SQL Server) | Read committed implemented via versions | Remove reader/writer blocking |
| `SELECT … FOR UPDATE` | Lock rows you read | Prevent lost updates in engines without detection |
| `VACUUM` / autovacuum (PG) | Reclaim dead versions | Tune when tables bloat |
| `SELECT xmin, xmax, …` | Inspect row versions | Teaching/debugging |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| Mixed OLTP + long reads/backups | Invariants span several rows and are checked-then-written (write skew) |
| Contention is moderate and retries are acceptable | You need the lock-based "wait, don't abort" behaviour |
| Append-only B-tree for simple, crash-safe snapshots | Write amplification from copying the root path is unacceptable |

### 9. Common mistakes
- Assuming snapshot isolation = serializable → write skew and phantom-based bugs → use serializable or explicit locks.
- Ignoring abort errors under SI → failed transfers → retry loop.
- Leaving a transaction open for hours → bloat, old snapshots → keep transactions short; set `idle_in_transaction_session_timeout`.
- Relying on lost-update detection on MySQL → silent overwrite → use `FOR UPDATE` or atomic updates.

### 10. 🎯 Interview kit
**30-second answer:** MVCC keeps multiple committed versions of each row tagged with the creating and deleting transaction IDs. Snapshot isolation gives each transaction one consistent snapshot, so readers don't block writers; it prevents dirty reads, non-repeatable reads and (for reads) phantoms, but not write skew.

**Structure a longer answer:** clarify the problem (read skew, backups) → `created_by`/`deleted_by` visibility rule → locks only for write-write conflicts → trade-offs (bloat, vacuum, write skew) → scenario (long report during transfers).

**Likely follow-ups:**
- *Read committed vs snapshot in MVCC?* → Snapshot per statement vs per transaction.
- *How does PostgreSQL handle updates?* → New row version; old marked deleted; vacuum later.
- *Does SI detect lost updates?* → In PG/Oracle/SQL Server yes (abort); in InnoDB no.
- *How do distributed DBs generate snapshots?* → Hybrid/TrueTime clocks or a timestamp oracle.

**Signals interviewers look for:** you can describe visibility rules, mention GC/vacuum cost, and know SI ≠ serializable.

### 11. Check yourself
- [ ] Write out the visibility rule for a row version given reader snapshot and creator/deleter commit status.
- [ ] Explain why an UPDATE is a delete plus insert under MVCC.
- [ ] Describe how a long-running transaction causes bloat.
- [ ] Compare MVCC-with-txid versus append-only B-tree snapshots.
<!-- explainer:end -->

## References

- *Designing Data-Intensive Applications* — Martin Kleppmann — Serializable Snapshot Isolation (SSI) › Detecting stale MVCC reads
- *Designing Data-Intensive Applications* — Martin Kleppmann — Snapshot Isolation and Repeatable Read › Implementing snapshot isolation
- *Designing Data-Intensive Applications* — Martin Kleppmann — Snapshot Isolation and Repeatable Read
- *Designing Data-Intensive Applications* — Martin Kleppmann — Preventing Lost Updates
- *Designing Data-Intensive Applications* — Martin Kleppmann — Clock Synchronization and Accuracy › Synchronized clocks for global snapshots
- *Database Internals* — Alex Petrov — Transaction Processing and Recovery › Multiversion Concurrency Control
- *Designing Data-Intensive Applications* — Martin Kleppmann — Snapshot Isolation and Repeatable Read › Indexes and snapshot isolation
- *Designing Data-Intensive Applications* — Martin Kleppmann — Preventing Lost Updates › Automatically detecting lost updates
- *Hadoop: The Definitive Guide* — Tom White — Administering Hadoop › Maintenance
- *Database Internals* — Alex Petrov — Implementing B-Trees › Fragmentation Caused by Updates and Deletes

