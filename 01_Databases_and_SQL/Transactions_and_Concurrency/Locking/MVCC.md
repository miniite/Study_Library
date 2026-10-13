---
id: "01_Databases_and_SQL/Transactions_and_Concurrency/Locking/MVCC"
level: 4
type: "concept"
topic: "01_Databases_and_SQL/Transactions_and_Concurrency/Locking"
tools: []
---

# MVCC

**L4 Advanced** · [↑ Locking](<_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** MVCC keeps several versions of each row, tagged with transaction IDs or timestamps, so readers see a consistent older version while writers create new ones — readers and writers stop blocking each other, at the price of storing and cleaning up old versions.

### 1. The big picture
With classic locking, a reader needs a shared lock and a writer needs an exclusive lock, so they conflict: a long `SUM` blocks updates, or updates block the report. **Multiversion concurrency control (MVCC)** removes that conflict by never overwriting data in place from the reader's point of view. An update creates a *new version*; transactions that began earlier keep reading the *old* version until they finish. The last committed version is the "current" one.

Nearly every modern engine uses MVCC for at least some isolation levels: PostgreSQL, MySQL InnoDB, Oracle, SQL Server (READ_COMMITTED_SNAPSHOT and SNAPSHOT), CockroachDB, and even file-based lakehouse formats such as Delta Lake, where every change writes new files and the log says which are live.

MVCC is not one algorithm. It is a family: version ordering by transaction ID or timestamp, plus a rule for which version each transaction may see, plus a cleanup process. Most of the operational pain — bloat, undo growth, "snapshot too old", long-transaction trouble — comes from the cleanup side.

### 2. Core concepts and vocabulary
- **Version** — one stored state of a row, with metadata about who created it and who (if anyone) superseded it.
- **Transaction ID (xid) / timestamp / SCN / LSN** — monotonically increasing identifier that orders transactions.
- **Snapshot / read view** — the set of transactions considered committed from a reader's perspective, fixed either per statement or per transaction.
- **Visible version** — the newest version created by a transaction that is committed in your snapshot and not deleted by one that is.
- **Committed vs uncommitted version** — only one uncommitted version of a row exists at a time (writers still lock each other).
- **Undo / rollback segment** — where Oracle and InnoDB keep the *before-images* from which old versions are rebuilt.
- **In-heap versioning** — PostgreSQL stores every version as a separate tuple in the table itself.
- **Version store** — SQL Server's tempdb (or persisted in the user database with Accelerated Database Recovery) area for old versions.
- **Garbage collection** — VACUUM (PostgreSQL), purge (InnoDB), undo retention (Oracle), version store cleanup (SQL Server).
- **First-committer-wins / first-updater-wins** — the rule for two writers on the same row under snapshot isolation.

### 3. How it works, step by step
1. **Begin.** The transaction receives an ID, and (at the right moment) a snapshot of which transactions are committed. Under Read Committed the snapshot is taken **per statement**; under Repeatable Read/Snapshot it is taken **once per transaction**.
2. **Read.** For each row, the engine walks the version chain and returns the newest version that the snapshot can see. No shared lock is needed.
3. **Write.** An `UPDATE` finds the visible version, takes a row-level lock/marks it, and writes a new version tagged with the writer's ID. The old version is marked as superseded by that ID. `DELETE` just marks the current version superseded.
4. **Conflict check.** If another transaction already changed that row after your snapshot, either you wait for it and then fail (snapshot isolation: "could not serialize access", SQL Server error 3960), or at Read Committed you re-read the latest committed version and continue.
5. **Commit.** The writer's ID flips to committed; new snapshots now see its versions. Older snapshots do not.
6. **Cleanup.** Once no live snapshot can ever need an old version, the engine reclaims it.

### 4. Worked example
Two sessions on PostgreSQL showing the hidden columns that make MVCC tick:

```sql
CREATE TABLE acct(id int primary key, bal int);
INSERT INTO acct VALUES (1, 100);

-- Session A
BEGIN ISOLATION LEVEL REPEATABLE READ;
SELECT xmin, xmax, bal FROM acct;          -- xmin=741, xmax=0, bal=100

-- Session B
UPDATE acct SET bal = 50 WHERE id = 1;     -- autocommit

-- Session A again
SELECT xmin, xmax, bal FROM acct;          -- still 741, 0, 100  (old version!)
COMMIT;
SELECT xmin, xmax, bal FROM acct;          -- now 742, 0, 50
```

What happened: the update did not touch the old tuple's data. It stamped `xmax = 742` on the old tuple and inserted a new tuple with `xmin = 742`. Session A's snapshot said "742 is not committed as far as I'm concerned", so the old tuple stayed visible and the new one invisible. After A's commit a new snapshot sees 742 as committed.

In InnoDB the same experiment would show the row updated *in place* in the clustered index, with the old value rebuilt from the undo log through the row's roll pointer.

### 5. Under the hood
**Where old versions live (and why it matters):**

| Engine | Location of old versions | Cleanup | Typical symptom |
|---|---|---|---|
| PostgreSQL | Other tuples in the same heap page | VACUUM / autovacuum, HOT pruning | Table & index bloat, wraparound risk |
| MySQL InnoDB | Undo log, linked from the clustered record | Purge thread | Huge history list length |
| Oracle | Undo tablespace | Automatic; `UNDO_RETENTION` | ORA-01555 snapshot too old |
| SQL Server | tempdb version store (PVS in-database with ADR) | Background cleanup after oldest snapshot ends | tempdb growth |
| Delta Lake | Parquet files not yet removed | `VACUUM` with retention | Storage growth, time travel window |

**Visibility rule (PostgreSQL flavour):** a tuple is visible if `xmin` is committed and "before" the snapshot (not in the in-progress list, below `xmax` of the snapshot) *and* either `xmax` is unset, aborted, or not yet visible. Hint bits cache commit status in the tuple to avoid repeated lookups of the commit log.

**HOT updates.** PostgreSQL can skip updating indexes when the new version fits on the same page and no indexed column changes (heap-only tuple); this reduces bloat dramatically.

**Locking and MVCC coexist.** Readers don't block, but writer–writer conflicts still need locks. `SELECT … FOR UPDATE` deliberately goes to the *latest* version and locks it, bypassing snapshot visibility.

**Time ordering.** MVCC can be implemented over two-phase locking, timestamp ordering, or optimistic validation; "MVCC" tells you about version retention, not about the isolation level (snapshot isolation vs serializable).

### 6. Performance and design implications
- Reads scale because they don't take shared locks; this is why reporting on an MVCC engine rarely blocks OLTP.
- **Long transactions are the enemy.** One idle-in-transaction session or a nightly report holds back the oldest-snapshot horizon, so *nothing* newer can be cleaned: undo grows, bloat builds, `tempdb` fills. Watch `pg_stat_activity.xact_start`, InnoDB `history list length`, and SQL Server `sys.dm_tran_active_snapshot_database_transactions`.
- Update-heavy workloads write more: a new version, new index entries (unless HOT), and later cleanup I/O.
- Under Read Committed with statement-level snapshots, a multi-statement transaction can see different data per statement; if you need one consistent view across statements, use a transaction-level snapshot (Repeatable Read / SNAPSHOT).
- `COUNT(*)` on MVCC engines is not free: each row's visibility must be checked, so PostgreSQL cannot answer from a counter.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `SELECT xmin, xmax, ctid FROM t` (PostgreSQL) | Show version metadata | Teaching, debugging visibility |
| `VACUUM (VERBOSE, ANALYZE)`, `pg_stat_user_tables.n_dead_tup` | Reclaim/inspect dead tuples | Bloat investigation |
| `idle_in_transaction_session_timeout` | Kill sessions that hold snapshots idly | Protect cleanup |
| `ALTER DATABASE … SET READ_COMMITTED_SNAPSHOT ON` (SQL Server) | Use versions for RC readers | Remove reader/writer blocking |
| `SHOW ENGINE INNODB STATUS` → *History list length* | Undo backlog | MySQL long transactions |
| `UNDO_RETENTION`, `V$UNDOSTAT` (Oracle) | Undo sizing | Avoiding ORA-01555 |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| Mixed read/write workloads, reporting on live data | Extremely write-heavy, tiny rows where version churn dominates |
| You need repeatable snapshots without locks | Transactions routinely run for hours |
| Optimistic concurrency fits (rare write conflicts) | Hot-row updates; first-committer-wins aborts will be frequent |
| Time travel / audit from old versions (lakehouse) | You must guarantee old data is purged immediately (compliance) |

### 9. Common mistakes
- Assuming updated rows are overwritten → bloat surprises on PostgreSQL → tune autovacuum, avoid unnecessary updates, use HOT-friendly fillfactor.
- Leaving transactions open (idle in transaction, forgotten `BEGIN`) → version store/undo explode → timeouts, short transactions.
- Believing MVCC removes all locking → still have write–write locks and deadlocks → design access order and keep writers short.
- Mixing driver autocommit with a multi-step read-modify-write → each statement is its own transaction, snapshot and commit → explicit `BEGIN … COMMIT`, or an atomic `UPDATE … SET x = x + 1`.
- Assuming snapshot reads see the latest data → a stale-but-consistent view → use `FOR UPDATE` or the highest level when "latest" is required.

### 10. 🎯 Interview kit
**30-second answer:** MVCC keeps multiple versions of a row stamped with transaction IDs. A reader uses a snapshot to pick the newest version visible to it, so reads don't block writes and writes don't block reads; old versions are cleaned once no snapshot needs them.

**Structure a longer answer:** problem with pure locking → versions + snapshot visibility → where versions live in this engine → write–write conflicts → cleanup cost and long-transaction pitfalls.

**Likely follow-ups:**
- *What happens to the old version on update?* → retained (heap tuple / undo / version store) until no snapshot needs it.
- *Why is a SELECT non-blocking under MVCC?* → it reads from a snapshot, takes no S lock.
- *Contrast Oracle's undo with PostgreSQL's heap versions.* → undo reconstructs old rows (snapshot too old risk); heap versions need VACUUM (bloat risk).
- *First-committer-wins vs a pessimistic lock?* → one aborts at commit/update time; the other waits up front.

**Signals interviewers look for:** you tie MVCC to snapshot isolation, know the cleanup story, and mention long transactions unprompted.

### 11. Check yourself
- [ ] Explain in four steps what `UPDATE` does to a row in an MVCC engine.
- [ ] Say why a forgotten open transaction makes a table grow.
- [ ] Give the difference between statement-level and transaction-level snapshots with an example.
- [ ] Name the cleanup mechanism and failure symptom for PostgreSQL, InnoDB, Oracle and SQL Server.
- [ ] Explain why `SELECT … FOR UPDATE` doesn't use the snapshot version.
<!-- explainer:end -->

## Practice questions

- [Basic] [What is concurrency control in a database and why is it needed?](<../../../PRACTICE/01_Databases_and_SQL/Transactions_and_Concurrency/Locking/Q-31386_What_is_concurrency_control_in_a_database_and_why_is_it_need.md>)
- [Deep] [In MVCC, what happens to the old version of a row when it is updated?](<../../../PRACTICE/01_Databases_and_SQL/Transactions_and_Concurrency/Locking/Q-31304_In_MVCC_what_happens_to_the_old_version_of_a_row_when_it_is_.md>)
- [Deep] [What is the difference between locking-based and MVCC-based concurrency control?](<../../../PRACTICE/01_Databases_and_SQL/Transactions_and_Concurrency/Locking/Q-31305_What_is_the_difference_between_locking-based_and_MVCC-based_.md>)
- [Deep] [Why is a SELECT statement usually non-blocking under MVCC databases?](<../../../PRACTICE/01_Databases_and_SQL/Transactions_and_Concurrency/Locking/Q-31356_Why_is_a_SELECT_statement_usually_non-blocking_under_MVCC_da.md>)
- [Deep] [What is a consistent read (snapshot read) in MySQL InnoDB?](<../../../PRACTICE/01_Databases_and_SQL/Transactions_and_Concurrency/Locking/Q-31357_What_is_a_consistent_read_snapshot_read_in_MySQL_InnoDB.md>)
- [Deep] [Why can an ALTER TABLE block ongoing reads and writes on that table?](<../../../PRACTICE/01_Databases_and_SQL/Transactions_and_Concurrency/Locking/Q-31368_Why_can_an_ALTER_TABLE_block_ongoing_reads_and_writes_on_tha.md>)
- [Deep] [What is an online DDL operation and why does it help with locking during schema changes?](<../../../PRACTICE/01_Databases_and_SQL/Transactions_and_Concurrency/Locking/Q-31369_What_is_an_online_DDL_operation_and_why_does_it_help_with_lo.md>)
- [Deep] [Your MVCC database's undo or old row versions are ballooning during a nightly report. Explain the connection between long transactions, snapshots, and version retention.](<../../../PRACTICE/01_Databases_and_SQL/Transactions_and_Concurrency/Locking/Q-31204_Your_MVCC_database_s_undo_or_old_row_versions_are_ballooning.md>)

## Related concepts

- [Lock Holding and Optimistic Concurrency](<../ACID_Transactions/Lock_Holding_and_Optimistic_Concurrency.md>) *(ACID Transactions)*
- [Lock Granularity](<../../Database_Internals/DBMS_Concepts/Lock_Granularity.md>) *(DBMS Concepts)*
- [Transaction Control and Prepared Transactions](<../../SQL_Language/SQL_Command_Types/Transaction_Control_and_Prepared_Transactions.md>) *(SQL Command Types)*

## References

- *Designing Data-Intensive Applications* — Martin Kleppmann — Preventing Lost Updates › Atomic write operations
- *Designing Data-Intensive Applications* — Martin Kleppmann — Serializable Snapshot Isolation (SSI) › Detecting stale MVCC reads
- *Database Internals* — Alex Petrov — Transaction Processing and Recovery › Concurrency Control
- *Database Internals* — Alex Petrov — Transaction Processing and Recovery › Multiversion Concurrency Control
- *Delta Lake: The Definitive Guide* — Denny Lee et al. — Introduction to the Delta Lake Lakehouse Format › Multiversion Concurrency Control (MVCC) File and Data Observations

