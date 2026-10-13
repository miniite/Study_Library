---
id: "TOOLS/SQL_Server/06_T-SQL_Programming/Data_Modification"
level: 4
type: "curriculum"
curriculum: "sql_server"
---

# Data Modification and MERGE

**L4 Advanced** · [↑ T-SQL Programming](<../_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** T-SQL changes data with INSERT, UPDATE, DELETE and MERGE, plus extras: `SELECT INTO`, `TRUNCATE`, `OUTPUT` and sequences. Each statement is atomic. Making it correct means joining carefully and handling concurrency; making it fast means set-based statements, batching huge changes and knowing what gets logged.

### 1. The big picture
Reading data never changes the database. Modifying it does: it takes exclusive locks, writes the transaction log, maintains every index and fires triggers. That makes data modification the part of T-SQL where both correctness (no double updates, no lost rows, no race conditions) and operational impact (log growth, blocking, replica lag) matter most.

This lesson covers the statements, their T-SQL extensions (`UPDATE … FROM`, `DELETE … FROM`, `OUTPUT`, `TOP`), `MERGE` and its known pitfalls, the upsert pattern, and the common mistake of deleting or updating millions of rows in one statement.

### 2. Core concepts and vocabulary
- **INSERT VALUES / INSERT SELECT / INSERT EXEC**: insert literal rows, the result of a query, or the result set of a procedure.
- **SELECT INTO**: creates a new table from a query result and fills it. Can be minimally logged.
- **BULK INSERT / bcp**: load files fast.
- **IDENTITY**: column property that generates increasing numbers per table. Read the last value with `SCOPE_IDENTITY()`.
- **SEQUENCE**: an independent object that generates numbers (`NEXT VALUE FOR`), shareable across tables.
- **DELETE vs. TRUNCATE**: `DELETE` removes rows one by one (logged per row, fires triggers, can filter). `TRUNCATE TABLE` deallocates all pages (logs only deallocations, resets identity, needs no row filter, is blocked by referencing foreign keys). Both can be rolled back inside a transaction.
- **UPDATE … FROM / DELETE … FROM**: T-SQL extension that uses a join to decide which rows to change and which values to use.
- **MERGE**: one statement that compares a *source* with a *target* and runs INSERT, UPDATE and/or DELETE depending on whether rows match.
- **OUTPUT clause**: returns the rows affected by a modification, with `inserted.*` (new values) and `deleted.*` (old values). Can write them into a table (`OUTPUT … INTO`).
- **Upsert**: "update if exists, otherwise insert".
- **Atomicity**: a single statement either fully succeeds or fully fails; partial changes are rolled back.
- **Minimal logging**: logging only page allocations instead of each row. Possible for bulk operations under the SIMPLE or BULK_LOGGED recovery model with conditions (e.g. `TABLOCK`).
- **Batching**: splitting a large change into many small transactions.

### 3. How it works, step by step
**What happens inside one UPDATE:**
1. The plan reads the rows to change (seek/scan) using U locks, then converts them to X locks.
2. For each row: log record written (old and new values), data page changed in the buffer pool, each nonclustered index containing a changed column updated (delete + insert of the index entry).
3. Constraints are checked, triggers fire (as part of the same transaction).
4. On commit, the log is hardened to disk. Data pages are written later by checkpoint.

All rows are evaluated *all-at-once*: in `UPDATE T SET a = b, b = a`, both expressions see the *old* values, so this swaps the columns.

**Halloween protection.** If an update moves rows forward in the index being scanned (e.g. raising salaries while scanning an index on salary), a row could be updated twice. The engine separates reading from writing, usually with an *Eager Table Spool* in the plan. That spool is extra cost you may see in update plans.

**MERGE logic:**
```
MERGE target AS t
USING source AS s ON <match condition>
WHEN MATCHED [AND <extra>]            THEN UPDATE … | DELETE
WHEN NOT MATCHED [BY TARGET]          THEN INSERT …
WHEN NOT MATCHED BY SOURCE [AND …]    THEN UPDATE … | DELETE ;   -- the semicolon is mandatory
```
1. The engine performs a full outer join between source and target on the `ON` condition.
2. Each row is classified: matched, not matched by target (only in source), not matched by source (only in target).
3. The corresponding action runs. If one target row matches more than one source row, MERGE fails with error 8672. (An `UPDATE … FROM` would silently pick one of them.)

### 4. Worked example
**Upsert with MERGE, auditing with OUTPUT:**
```sql
CREATE TABLE dbo.Customers
( custid int PRIMARY KEY, companyname nvarchar(40) NOT NULL, phone varchar(24) NULL );

CREATE TABLE dbo.CustomersStage
( custid int PRIMARY KEY, companyname nvarchar(40) NOT NULL, phone varchar(24) NULL );

MERGE dbo.Customers WITH (HOLDLOCK) AS tgt
USING dbo.CustomersStage AS src
   ON tgt.custid = src.custid
WHEN MATCHED AND EXISTS (SELECT tgt.companyname, tgt.phone
                         EXCEPT
                         SELECT src.companyname, src.phone)     -- update only if something changed (NULL-safe)
    THEN UPDATE SET tgt.companyname = src.companyname,
                    tgt.phone       = src.phone
WHEN NOT MATCHED THEN
    INSERT (custid, companyname, phone) VALUES (src.custid, src.companyname, src.phone)
OUTPUT $action AS act, inserted.custid,
       deleted.phone AS oldphone, inserted.phone AS newphone;
```
`$action` returns `INSERT`, `UPDATE` or `DELETE` per row. The `EXISTS … EXCEPT` trick compares columns NULL-safely and skips rows that didn't change, which avoids pointless log records and trigger firings.

**The same upsert without MERGE** (often recommended for single-row OLTP upserts):
```sql
BEGIN TRAN;
UPDATE dbo.Customers WITH (UPDLOCK, SERIALIZABLE)
SET companyname = @name, phone = @phone
WHERE custid = @custid;

IF @@ROWCOUNT = 0
    INSERT dbo.Customers (custid, companyname, phone) VALUES (@custid, @name, @phone);
COMMIT;
```
The `UPDLOCK, SERIALIZABLE` hints hold a key-range lock on the missing key, so two sessions can't both decide "not there" and both insert (which would cause a primary key violation).

**UPDATE with a join, and DELETE with OUTPUT INTO an archive:**
```sql
UPDATE od
SET od.discount += 0.05
FROM dbo.OrderDetails AS od
JOIN dbo.Orders AS o ON o.orderid = od.orderid
WHERE o.custid = 1;

DELETE FROM dbo.Orders
OUTPUT deleted.orderid, deleted.orderdate, deleted.custid
INTO dbo.OrdersArchive (orderid, orderdate, custid)
WHERE orderdate < '20200101';
```

**Deleting a large number of rows in batches:**
```sql
DECLARE @rows int = 1;
WHILE @rows > 0
BEGIN
    DELETE TOP (5000) FROM dbo.EventLog
    WHERE logdate < '20250101';
    SET @rows = @@ROWCOUNT;
    -- optional: CHECKPOINT (SIMPLE recovery) or let log backups run (FULL), brief WAITFOR DELAY
END;
```
One `DELETE` of 50 million rows is a single transaction: the log must hold all of it until it commits, it can escalate to a table lock and block everyone, a failure rolls back for as long as it ran, and on an availability group it floods the replicas. Batches of a few thousand rows stay below the lock-escalation threshold (≈5,000 locks per statement), let the log be reused between batches, and keep blocking short. Needs an index on `logdate`, or each batch scans the table.

### 5. Under the hood
- **Logging.** INSERT logs the new row, DELETE the old row, UPDATE old and new values of changed columns (or delete+insert when the row moves). Every nonclustered index adds its own log records. Wide tables with many indexes have expensive modifications.
- **In-place vs. split updates.** If the new row no longer fits on its page, a **page split** moves half the rows to a new page (heavily logged). On a heap, the row moves and a **forwarding pointer** is left behind.
- **Updating a clustering key** changes the row's location: delete + insert in the clustered index and an update of *every* nonclustered index (they store the clustering key).
- **TRUNCATE** logs page and extent deallocations only, so it is nearly instant and still transactional.
- **Minimal logging** for `INSERT … SELECT` needs `TABLOCK` on the target and SIMPLE or BULK_LOGGED recovery. `SELECT INTO` is minimally logged under those models by default.
- **MERGE caveats.** MERGE has had a long list of bugs (some fixed, some with filtered indexes, indexed views, triggers and partitioning). It is *not* atomic against concurrency by default: two concurrent MERGEs can both see "not matched" and both insert, so add `HOLDLOCK` (SERIALIZABLE) on the target. Triggers fire once per action type, not once per MERGE.
- **OUTPUT … INTO** cannot target a table with enabled triggers or that participates in a foreign key. When the target table has triggers, `OUTPUT` without `INTO` fails unless you use `OUTPUT INTO` a table variable.

### 6. Performance and design implications
- Use set-based statements; never update in a cursor when a join can describe the change.
- Batch big DELETE/UPDATE jobs. Choose a batch size that stays under lock escalation and keeps each transaction short.
- For purging by date on very large tables, partition the table and use `TRUNCATE TABLE … WITH (PARTITIONS (n))` or partition switching: metadata-only operations.
- Skip no-op updates (compare old and new values) to cut log and trigger work.
- In ETL, consider replacing row-by-row "Execute T-SQL" or OLE DB Command steps with staged set-based statements; per-row tasks are a classic cause of slow loads.
- Watch the log: `sys.dm_db_log_space_usage`, `log_reuse_wait_desc` in `sys.databases`.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `INSERT … SELECT … WITH (TABLOCK)` | Bulk-style insert, minimal logging possible | Loads |
| `SELECT … INTO new_table` | Create + fill a table | Staging, copies |
| `TRUNCATE TABLE` | Remove all rows fast | Staging resets |
| `DELETE TOP (n)` / `UPDATE TOP (n)` | Limit rows per statement | Batching |
| `OUTPUT inserted.*, deleted.*` | Return/record changed rows | Auditing, getting generated keys |
| `MERGE … WITH (HOLDLOCK)` | Upsert/sync in one statement | Set-based sync from staging |
| `SCOPE_IDENTITY()` | Last identity in this scope | After single-row insert |
| `NEXT VALUE FOR seq` | Next sequence value | Shared numbering |
| `@@ROWCOUNT` | Rows affected by last statement | Upsert logic, loops |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| `MERGE` for set-based sync from a staging table | High-concurrency single-row upserts; complex triggers/indexed views → separate UPDATE + INSERT |
| UPDATE-then-INSERT with `UPDLOCK, SERIALIZABLE` | Forgetting the hints → duplicate-key races |
| `TRUNCATE` | Table referenced by FKs, or you need a WHERE → `DELETE` |
| Batched deletes | Simple one-off delete of a few thousand rows — no need |
| `UPDATE … FROM` with joins | Source has duplicate keys → nondeterministic result; MERGE would error instead |
| `IDENTITY` | Need gap-free numbers → neither identity nor sequences guarantee no gaps |

### 9. Common mistakes
- Deleting millions of rows in one statement → log growth, lock escalation, long rollback → batch with `TOP`.
- MERGE without `HOLDLOCK` under concurrency → duplicate key errors → add `HOLDLOCK` or use the explicit pattern.
- Forgetting the semicolon after MERGE → error 10713.
- `UPDATE … FROM` with a source that has duplicates → random value applied → deduplicate the source first.
- Using `@@IDENTITY` → returns identities from triggers → use `SCOPE_IDENTITY()` or `OUTPUT`.
- Updating rows that didn't change → useless log and trigger work → filter with `EXCEPT`-style comparison.
- `NOT MATCHED BY SOURCE THEN DELETE` with a partial source → deletes everything not in this batch → filter the target with a CTE.

### 10. 🎯 Interview kit
**30-second answer:** INSERT, UPDATE, DELETE and MERGE are atomic set-based statements. T-SQL adds joins in UPDATE/DELETE, TOP for batching, and OUTPUT to capture inserted/deleted rows. MERGE syncs a source into a target in one statement but needs HOLDLOCK for concurrency safety and has known pitfalls. For very large deletes I batch with DELETE TOP (n) in a loop, or use partition truncation, to control log growth and blocking.

**Structure a longer answer:** statement types → what one modification costs (locks, log, indexes, triggers) → upsert patterns and race conditions → large-volume strategies → a real purge job you designed.

**Likely follow-ups:**
- *DELETE vs. TRUNCATE?* → per-row logging, filters, triggers vs. page deallocation, identity reset, FK restriction; both transactional.
- *Why batch?* → log reuse, lock escalation (~5,000 locks), blocking, rollback time, replica lag.
- *MERGE problems?* → concurrency races without HOLDLOCK, multiple matches error, bug history, trigger behavior.
- *How to get generated IDs from a multi-row insert?* → `OUTPUT inserted.id INTO @t`.

**Signals interviewers look for:** awareness of logging and locking cost; safe upsert under concurrency; batching instinct.

### 11. Check yourself
- [ ] Write an upsert with MERGE and with UPDATE+INSERT, and explain the race each prevents.
- [ ] Delete 1 million rows in batches and watch `sys.dm_db_log_space_usage` between batches.
- [ ] Use OUTPUT INTO to archive deleted rows.
- [ ] Explain why `SET a = b, b = a` swaps values.
- [ ] List four differences between DELETE and TRUNCATE.
<!-- explainer:end -->

## References

- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 6: CTEs, MERGE, Pivoting, JSON, and XML › MERGE with OUTPUT Clause
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 6: CTEs, MERGE, Pivoting, JSON, and XML › MERGE Statement for UPSERT Operations
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 6: CTEs, MERGE, Pivoting, JSON, and XML › Common Pitfalls with MERGE
- *T-SQL Fundamentals (4th ed.)* — Itzik Ben-Gan — Data ��difi�ati�n
- *Beginning T-SQL* — Kathi Kellenberger, Lee Everest — Writing Advanced Queries › The MERGE Statement
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 6: CTEs, MERGE, Pivoting, JSON, and XML › Simple UPSERT Example
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 6: CTEs, MERGE, Pivoting, JSON, and XML › MERGE vs. Traditional Approach
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 6: CTEs, MERGE, Pivoting, JSON, and XML › Adding DELETE Logic
- *100 SQL Server Mistakes and How to Avoid Them* — Peter A. Carter — SSIS development › 6.3.2 Converting Execute T-SQL Statement tasks to data flows
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 6: CTEs, MERGE, Pivoting, JSON, and XML › MERGE Method

