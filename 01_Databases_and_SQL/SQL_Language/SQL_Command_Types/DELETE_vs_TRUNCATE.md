---
id: "01_Databases_and_SQL/SQL_Language/SQL_Command_Types/DELETE_vs_TRUNCATE"
level: 4
type: "concept"
topic: "01_Databases_and_SQL/SQL_Language/SQL_Command_Types"
tools: []
---

# DELETE vs TRUNCATE

**L4 Advanced** · [↑ SQL Command Types](<_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** `DELETE` removes chosen rows one by one with full logging (triggers fire, identity continues, `WHERE` allowed); `TRUNCATE` deallocates whole pages/partitions with minimal logging (no `WHERE`, identity reset, blocked by foreign keys) — and in SQL Server both are transactional and can be rolled back.

### 1. The big picture
Emptying a table sounds trivial, yet the two commands behave very differently in speed, logging, locks, triggers, identity values and permissions. `DELETE` is DML: it works row by row, so it can filter and it fires triggers. `TRUNCATE TABLE` is a DDL-class operation: it drops the table's data pages (or index allocation units) in one go, which is why it finishes in seconds on millions of rows where `DELETE` can take minutes and bloat the transaction log. Choosing between them — and a third option, dropping a partition — is a standard data-retention and ETL-reload decision.

### 2. Core concepts and vocabulary
- **`DELETE`** — DML; removes rows matching an optional `WHERE`; each row is logged.
- **`TRUNCATE TABLE`** — removes all rows by deallocating pages; no `WHERE`; keeps structure, indexes, constraints.
- **Fully logged** vs **minimally logged** — `DELETE` writes a log record per row (plus index entries); `TRUNCATE` logs only page/extent deallocations.
- **Identity reset** — `TRUNCATE` resets the identity to its seed; `DELETE` does not, even without a filter.
- **Ghost records** — rows deleted by `DELETE` are marked as ghosts and cleaned up later by a background task; space is not instantly reusable in a heap without a table lock.
- **Partition truncation** — `TRUNCATE TABLE t WITH (PARTITIONS (1, 3-5))` (SQL Server 2016+).
- **Referencing foreign key** — blocks `TRUNCATE` on the *referenced* table, even if the referencing table is empty or the constraint is disabled.
- **`ALTER` permission** — required for `TRUNCATE`; `DELETE` needs only `DELETE` permission.
- **Minimal logging is not non-transactional** — both can be rolled back inside a transaction.

### 3. How it works, step by step
**DELETE FROM t WHERE p**
1. Takes row/page locks (escalating to a table lock after ~5,000 locks) and `IX` on the table.
2. For each qualifying row: write a log record with the full row image (needed for undo/redo), remove index entries, mark as ghost.
3. Fires `AFTER DELETE` triggers once per statement; checks and cascades foreign keys.
4. Commit; the ghost-cleanup task reclaims space.

**TRUNCATE TABLE t**
1. Requires `ALTER` permission and no referencing foreign keys, indexed views or replication.
2. Takes a schema-modification (`Sch-M`) lock on the table.
3. Deallocates the table's allocation units (IAM chains), logging only the deallocation.
4. Resets the identity seed; resets statistics' row counts.
5. No `DELETE` triggers fire; the operation is part of the transaction and can be rolled back.

### 4. Worked example
```sql
CREATE TABLE audit_log (id INT IDENTITY(1,1) PRIMARY KEY, msg VARCHAR(100), created DATE);
-- ... 10 million rows loaded ...

-- (a) Remove old rows, keep recent: must use DELETE; do it in batches to limit log growth
WHILE 1 = 1
BEGIN
    DELETE TOP (50000) FROM audit_log WHERE created < '2024-01-01';
    IF @@ROWCOUNT = 0 BREAK;
END;

-- (b) Remove everything and restart identity: TRUNCATE
TRUNCATE TABLE audit_log;               -- seconds; next id = 1

-- (c) Same with DELETE: identity continues from the last value
DELETE FROM audit_log;                  -- slow, fully logged
DBCC CHECKIDENT('audit_log', RESEED, 0);   -- only if you really want id=1 again

-- (d) Retention on a partitioned table: truncate one partition
TRUNCATE TABLE Fact.Sale WITH (PARTITIONS (1));        -- oldest year only
-- or several: WITH (PARTITIONS (1, 2, 3)) / (1 TO 4)  (the form depends on the version)

-- (e) Truncate is transactional
BEGIN TRAN; TRUNCATE TABLE audit_log; SELECT COUNT(*) FROM audit_log;  -- 0
ROLLBACK;   SELECT COUNT(*) FROM audit_log;  -- rows are back
-- (f) FK blocks it
TRUNCATE TABLE Orders;  -- Msg 4712: Cannot truncate table because it is being referenced by a FOREIGN KEY constraint.
```
What to observe: `SET STATISTICS TIME ON` and `sys.dm_tran_database_transactions` (log bytes) show a huge difference between (a)/(c) and (b); `sys.dm_db_partition_stats` shows page counts dropping to zero after (b).

### 5. Under the hood   *(optional at L4)*
- **Log volume**: `DELETE` logs the whole row (and every index row) so undo is possible; log size can reach the size of the data. `TRUNCATE` logs page deallocations (`LOP_...` allocation records) only; under full recovery it is still fully recoverable.
- **Rollback**: since the deallocation log records can be undone, rollback re-links the pages. The popular claim "TRUNCATE can't be rolled back" is a myth in SQL Server (it is true in some other engines, e.g. Oracle where DDL auto-commits, and MySQL).
- **Heaps and `DELETE`**: without a table lock (`TABLOCK`), pages emptied by `DELETE` on a heap may stay allocated; `TRUNCATE`, or `DELETE … WITH (TABLOCK)`, releases them.
- **PostgreSQL**: `DELETE` leaves dead tuples reclaimed by `VACUUM` (space isn't returned to the OS except by `VACUUM FULL`), whereas `TRUNCATE` swaps in new empty files — instant space return, but it takes an `ACCESS EXCLUSIVE` lock and is MVCC-unsafe for concurrent snapshot readers.
- **Cascade**: `TRUNCATE … CASCADE` exists in PostgreSQL; in SQL Server the way is to drop and re-create the FKs, or `DELETE` in dependency order.

### 6. Performance and design implications
- Full emptying: `TRUNCATE` ≫ `DELETE`; partial: batch `DELETE` (5–50k rows) to keep locks small and the log bounded; check log backups/space.
- Large `DELETE` can escalate to a table lock and block readers; use batches or `READ_COMMITTED_SNAPSHOT`.
- Rebuilding a staging table each load: `TRUNCATE` + bulk insert is the standard, minimally logged pattern.
- Partition-based retention (switch out or truncate a partition) beats row deletes on large tables.
- After massive `DELETE`: rebuild/reorganize indexes, update statistics, consider shrinking only as a last resort.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `DELETE … WHERE` | Row-level removal, triggers fire | Partial deletes |
| `DELETE TOP (n)` in a loop | Batched deletion | Large retention jobs |
| `TRUNCATE TABLE t` | Empty the table, reseed identity | Staging, reload |
| `TRUNCATE … WITH (PARTITIONS …)` | Empty specific partitions | Retention windows |
| `DBCC CHECKIDENT(t, RESEED, n)` | Manually reset identity | After `DELETE` when needed |
| `ALTER TABLE … SWITCH PARTITION` | Metadata-only data removal | Sliding window |
| `OUTPUT deleted.*` | Capture deleted rows | Archiving while deleting |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| `TRUNCATE` for full reload/reset | Triggers or auditing on delete must run; FKs reference the table |
| `DELETE` with `WHERE` | Millions of rows in one go (log growth, locks) |
| Batched `DELETE` | You need atomicity of the whole purge |
| Partition truncate/switch | Table isn't partitioned or the partition key doesn't match the retention rule |
| `DROP TABLE` | You also want the structure gone |

### 9. Common mistakes
- Running `TRUNCATE` on the wrong database/connection → data gone (still recoverable only via transaction/backup) → verify the target, restrict `ALTER` permission, use transactions in ad hoc work.
- Believing `TRUNCATE` cannot be rolled back and avoiding it in transactions → needless complexity.
- Expecting identity to restart after `DELETE` → gaps/continuation → `RESEED` deliberately.
- One giant `DELETE` → log fills the disk, blocking → batch, back up the log.
- Expecting `DELETE` triggers or cascades from `TRUNCATE` → missing audit/cleanup.

### 10. 🎯 Interview kit
**30-second answer:** "`DELETE` is DML, row-by-row, fully logged, supports `WHERE`, fires triggers and doesn't reset identity. `TRUNCATE` deallocates pages, minimal logging, no `WHERE`, resets identity, needs `ALTER` and can't run if a foreign key references the table. In SQL Server both are transactional and rollbackable."

**Structure a longer answer:** semantics → logging/locking → identity/triggers/FKs → recoverability → engine differences (SQL Server vs PostgreSQL vs Oracle/MySQL) → practical retention strategies.

**Likely follow-ups:**
- *Can `TRUNCATE` have a `WHERE`?* → no; use `DELETE` or partition truncation.
- *Why can't I truncate a table referenced by an FK?* → the engine can't validate millions of references cheaply; drop the FKs, truncate, re-create (or delete in order).
- *How do you reclaim space after big deletes?* → batch deletes, rebuild indexes (SQL Server), `VACUUM`/`REINDEX` (PostgreSQL), or truncate/partition-drop.
- *Is `TRUNCATE` DDL or DML?* → classified as DDL; it still participates in transactions in SQL Server.

**Signals interviewers look for:** you mention logging, identity, triggers, FKs, permissions and rollback correctly (and call out the myth).

### 11. Check yourself
- [ ] Contrast the two on logging, triggers, identity, `WHERE`, FKs, permissions, rollback.
- [ ] Write a batched purge loop and say how you'd keep log growth under control.
- [ ] Explain how to purge the oldest year from a partitioned fact table instantly.
- [ ] Describe what happens to space after `DELETE` in SQL Server vs PostgreSQL.
<!-- explainer:end -->

## Practice questions

- [Basic] [Can you use a WHERE clause with TRUNCATE? Explain.](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Command_Types/Q-22989_Can_you_use_a_WHERE_clause_with_TRUNCATE_Explain.md>)
- [Intermediate] [Does TRUNCATE reset the AUTO_INCREMENT / identity counter? Explain.](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Command_Types/Q-22962_Does_TRUNCATE_reset_the_AUTO_INCREMENT_identity_counter_Expl.md>)
- [Intermediate] [What is the difference between DELETE and TRUNCATE?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Command_Types/Q-22992_What_is_the_difference_between_DELETE_and_TRUNCATE.md>)
- [Intermediate] [A batch DELETE of 50M rows is bloating your transaction log and holding locks. How would you restructure it into safe chunked deletes?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Command_Types/Q-22851_A_batch_DELETE_of_50M_rows_is_bloating_your_transaction_log_.md>)
- [Intermediate] [A TRUNCATE in a nightly job occasionally blocks reporting queries. Why does TRUNCATE take an exclusive lock and how do you schedule around it?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Command_Types/Q-22872_A_TRUNCATE_in_a_nightly_job_occasionally_blocks_reporting_qu.md>)
- [Intermediate] [How would you design a soft-delete pattern versus a hard DELETE, and what are the query, index, and compliance trade-offs?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Command_Types/Q-22882_How_would_you_design_a_soft-delete_pattern_versus_a_hard_DEL.md>)
- [Intermediate] [How would you choose between TRUNCATE with RESTART IDENTITY and DELETE + ALTER SEQUENCE when resetting a table for a test harness?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Command_Types/Q-22931_How_would_you_choose_between_TRUNCATE_with_RESTART_IDENTITY_.md>)
- [Deep] [How does DELETE affect triggers compared to TRUNCATE?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Command_Types/Q-22990_How_does_DELETE_affect_triggers_compared_to_TRUNCATE.md>)

## Related concepts

- [DELETE TRUNCATE and DROP](<../../../TOOLS/PostgreSQL/PostgreSQL_Platform/DELETE_TRUNCATE_and_DROP.md>) *(PostgreSQL Platform)*
- [SQL vs NoSQL](<../SQL_Fundamentals/SQL_vs_NoSQL.md>) *(SQL Fundamentals)*
- [Online Schema Changes](<../../../TOOLS/MySQL/MySQL_Platform/Online_Schema_Changes.md>) *(MySQL Platform)*

## References

- *T-SQL Fundamentals (4th ed.)* — Itzik Ben-Gan — Data ��difi�ati�n
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 1: Getting Started with T-SQL › TRUNCATE
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 1: Getting Started with T-SQL › DELETE
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 1: Getting Started with T-SQL › Quiz
- *Analytics Optimization with Columnstore Indexes in Microsoft SQL Server* — Edward Pollack — Delete anD UpDate OperatiOns UP ATE OPERATI › Delete Operations Delete Operations
- *Analytics Optimization with Columnstore Indexes in Microsoft SQL Server* — Edward Pollack — partitioning › Database Maintenance Database Maintenance
- *Azure Data Fundamentals* — Michael John Pena — Relational Data Concepts › Removing records with DELETE
- *100 SQL Server Mistakes and How to Avoid Them* — Peter A. Carter — T-SQL development › 5.8 #21 Deleting many rows in a single transaction
- *Fundamentals of Data Engineering* — Joe Reis, Matt Housley — Chapter 8. Queries, Modeling, and Transformation › Vacuum dead records
- *Fundamentals of Data Engineering* — Joe Reis, Matt Housley — Modeling Streaming Data › Delete

