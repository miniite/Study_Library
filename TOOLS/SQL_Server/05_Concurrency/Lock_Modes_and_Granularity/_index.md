---
id: "TOOLS/SQL_Server/05_Concurrency/Lock_Modes_and_Granularity"
level: 4
type: "curriculum"
curriculum: "sql_server"
---

# Lock Modes and Granularity

**L4 Advanced** · [↑ Concurrency, Locking and Blocking](<../_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** SQL Server locks resources at several levels (row/key, page, partition, table, database) in different modes (S, U, X, intent, schema, range). The compatibility matrix decides whether a request is granted or waits.

### 1. The big picture
To understand any blocking or deadlock you must be able to answer: *which resource* was locked, *in which mode*, *for how long*, and *what mode the waiter wanted*. This lesson gives you that vocabulary. The parent lesson (*Concurrency*) explained why locks exist. *Lock Escalation*, *Blocking Analysis* and *Deadlocks* use these terms constantly.

Two dimensions describe every lock:
- **Granularity**: how much data the lock covers. Fine-grained locks (rows) allow more concurrency but cost more memory and CPU to manage. Coarse locks (tables) are cheap to manage but block more.
- **Mode**: what the holder intends to do (read, maybe-update, write, change schema…), and therefore what others may do at the same time.

### 2. Core concepts and vocabulary
**Resources (granularity), from fine to coarse**
- **RID**: a row in a heap (file:page:slot).
- **KEY**: a row in an index (B-tree), identified by a hash of the key value. In `sys.dm_tran_locks` it appears as e.g. `(8194443284a0)`.
- **PAGE**: an 8 KB data or index page.
- **EXTENT**: 8 contiguous pages (used during allocation).
- **HOBT / partition**: a heap or B-tree partition (only for escalation when `LOCK_ESCALATION = AUTO` on partitioned tables).
- **OBJECT**: a whole table (including all its indexes), view, or procedure.
- **DATABASE**: every connection holds a shared DB lock on its current database. That prevents dropping or restoring it while in use.
- Others: **FILE**, **APPLICATION** (`sp_getapplock`), **METADATA**, **ALLOCATION_UNIT**, **XACT** (optimized locking).

**Modes**
- **S (Shared)**: reading. Many sessions can hold S on the same resource.
- **U (Update)**: "I'm reading this to maybe change it." Only one U at a time; compatible with S. It is converted to X when the change happens. It exists to prevent the classic *conversion deadlock* (two sessions holding S, both trying to convert to X).
- **X (Exclusive)**: writing. Incompatible with everything (except readers that use row versioning, which take no S lock).
- **Intent locks — IS, IX, SIX**: placed on the *parents* (page, table) of a resource you lock at a lower level. IS = "I hold S somewhere inside", IX = "I hold X somewhere inside", SIX = "S on the whole thing plus X on some parts". They let the engine check a table-level request against thousands of row locks with a single comparison.
- **Sch-S (Schema stability)**: taken while compiling/running queries. Prevents the schema from changing under them. Compatible with everything except Sch-M.
- **Sch-M (Schema modification)**: taken by DDL (`ALTER TABLE`, index rebuild switch, `TRUNCATE`). Incompatible with *everything*, including `NOLOCK` readers (which still take Sch-S).
- **BU (Bulk Update)**: taken by bulk loads with `TABLOCK` into a heap. Multiple BU holders can load in parallel.
- **Key-range locks** (`RangeS-S`, `RangeS-U`, `RangeI-N`, `RangeX-X`): used under SERIALIZABLE to lock the *gap* between keys, so no phantom row can be inserted.

### 3. How it works, step by step
**Compatibility matrix** (✓ = granted together, ✗ = the requester waits)

| Requested ↓ / Held → | IS | S | U | IX | SIX | X |
|---|---|---|---|---|---|---|
| **IS** | ✓ | ✓ | ✓ | ✓ | ✓ | ✗ |
| **S** | ✓ | ✓ | ✓ | ✗ | ✗ | ✗ |
| **U** | ✓ | ✓ | ✗ | ✗ | ✗ | ✗ |
| **IX** | ✓ | ✗ | ✗ | ✓ | ✗ | ✗ |
| **SIX** | ✓ | ✗ | ✗ | ✗ | ✗ | ✗ |
| **X** | ✗ | ✗ | ✗ | ✗ | ✗ | ✗ |

How an `UPDATE ... WHERE OrderID = 42` locks, top-down:
1. **Sch-S** on the table for the plan (and DATABASE S from the connection).
2. **IX** on the OBJECT (table).
3. **IX** on the PAGE holding the row.
4. **U** on the KEY while the row is located and qualified.
5. Convert **U → X** on the KEY when the row is modified. Every index that contains a changed column also gets X key locks on its entries.
6. Hold the X locks (and the intent locks) until **COMMIT/ROLLBACK**.

How long locks last depends on the mode and isolation level:
- **X**: always until the transaction ends.
- **S under READ COMMITTED**: released as soon as the row (or page) has been read.
- **S under REPEATABLE READ / SERIALIZABLE**: held until the transaction ends.
- **U**: released if the row doesn't qualify; converted to X if it does.

**Granularity choice.** The engine picks row or page locks per statement, based on estimated rows and plan shape. A scan of a big table may start with page locks. During execution it may **escalate** to a table lock (next lesson).

### 4. Worked example
```sql
-- Session 1: modify one row, keep the transaction open
BEGIN TRAN;
UPDATE Sales.SalesOrderHeader SET Comment = 'x' WHERE SalesOrderID = 43659;

-- Session 1 (or another window): what does it hold?
SELECT resource_type, resource_description, request_mode, request_status,
       OBJECT_NAME(p.object_id) AS obj, p.index_id
FROM sys.dm_tran_locks AS l
LEFT JOIN sys.partitions AS p ON p.hobt_id = l.resource_associated_entity_id
WHERE l.request_session_id = @@SPID   -- run in session 1
ORDER BY resource_type;
-- DATABASE  S
-- OBJECT    IX    (SalesOrderHeader)
-- PAGE      IX    (index_id 1)
-- KEY       X     (index_id 1, hash value)

-- Session 2 tries a table lock: IX held vs S requested → incompatible → waits
SELECT COUNT(*) FROM Sales.SalesOrderHeader WITH (TABLOCK);

-- Session 3: shows the WAIT row for session 2 (OBJECT, S, WAIT)
ROLLBACK; -- in session 1
```
Try the same `UPDATE` with `WHERE YEAR(OrderDate) = 2011` (no usable index). You'll see many more KEY or PAGE locks, or one OBJECT X lock after escalation. Locking footprint follows the access path.

Lock hints change mode or granularity for one table reference:
```sql
SELECT ... FROM dbo.Queue WITH (UPDLOCK, ROWLOCK, READPAST) WHERE Status = 'New';
-- take U (not S), stay at row level, skip rows others have locked → work-queue pattern
SELECT ... FROM dbo.Big WITH (TABLOCKX);   -- one exclusive table lock
SELECT ... FROM dbo.T WITH (HOLDLOCK);     -- same as SERIALIZABLE for this table
```

### 5. Under the hood
- **Lock memory.** Each lock structure is about 64–128 bytes. Lock memory comes from the lock manager's memory clerk (`OBJECTSTORE_LOCK_MANAGER`). One million row locks ≈ 100+ MB. That's one reason for escalation.
- **KEY hash.** Key locks are on a 6-byte hash of the index key, not the key itself. Rare hash collisions can make two different keys block each other.
- **Lock partitioning.** On machines with 16+ CPUs, IS/IX/S locks on objects are partitioned per scheduler to reduce lock-manager spinlock contention. You may see `OBJECT` locks with a `resource_lock_partition` value. An X table lock must then acquire all partitions.
- **Conversion.** U→X and S→X are *conversions*. Two sessions holding S and both wanting X is the conversion deadlock. U locks prevent it, because only one session can hold U.
- **Page-lock disabling.** `ALLOW_PAGE_LOCKS = OFF` on an index forbids page locks. That also prevents `ALTER INDEX REORGANIZE` on it.
- **Read-only and `NOLOCK` readers** still take **Sch-S**. That's why an online index rebuild's final Sch-M phase can block, and be blocked by, `NOLOCK` queries.

### 6. Performance and design implications
- Locking footprint = rows touched, not rows returned. **Indexes that let a statement seek** dramatically reduce locks and blocking.
- **Keep transactions short.** X locks live until commit.
- **Avoid unnecessary SERIALIZABLE** (some frameworks and `TransactionScope` default to it). Range locks cause blocking and deadlocks.
- Use `UPDLOCK` for "read then update" patterns to avoid conversion deadlocks.
- DDL in busy systems waits for Sch-M, and everything queued behind it waits too. Use `WAIT_AT_LOW_PRIORITY` for online operations.
- Monitor `LCK_M_*` waits by mode (`LCK_M_S`, `LCK_M_U`, `LCK_M_X`, `LCK_M_IX`, `LCK_M_SCH_M`…). The suffix tells you which mode the *waiter* requested.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `sys.dm_tran_locks` | every granted/waiting lock with resource and mode | live analysis |
| `sp_lock` (deprecated) / `sp_WhoIsActive @get_locks = 1` | lock summary per session | quick checks |
| `ROWLOCK`, `PAGLOCK`, `TABLOCK`, `TABLOCKX` hints | force granularity | rare, targeted |
| `UPDLOCK`, `XLOCK`, `HOLDLOCK`, `READPAST` hints | change mode / duration / skip locked rows | queues, read-then-update |
| `ALTER INDEX ... SET (ALLOW_ROW_LOCKS / ALLOW_PAGE_LOCKS = OFF)` | restrict granularity per index | very special cases |
| `sp_getapplock` / `sp_releaseapplock` | lock an arbitrary named resource | serialize a code path |
| `ALTER INDEX ... REBUILD WITH (ONLINE = ON (WAIT_AT_LOW_PRIORITY (...)))` | gentle Sch-M acquisition | DDL on busy tables |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| Row/key locks: OLTP point access | huge batches (lock memory, escalation) |
| Page locks: medium ranges | hot pages with many writers |
| Table lock (`TABLOCK`): bulk loads, maintenance windows | anything concurrent |
| `UPDLOCK`: read-then-write in one transaction | held on many rows for a long time |
| `READPAST`: work queues | reports that must see all rows |

### 9. Common mistakes
- Thinking `NOLOCK` takes no locks → it still takes Sch-S and can be blocked by DDL → plan DDL accordingly.
- Reading `LCK_M_IX` as "someone holds IX" → it's the *waiter* that wants IX → look at what the holder has.
- Read-then-update with S locks under REPEATABLE READ → conversion deadlocks → use `UPDLOCK`.
- Forcing `ROWLOCK` on million-row updates → lock memory explodes and escalation happens anyway → batch the work.
- Ignoring that intent locks block table-level requests → surprise waits on `TABLOCK` or DDL.

### 10. 🎯 Interview kit
**30-second answer:** SQL Server locks at key/RID, page, partition, table and database level. The main modes are shared for reads, update for "read with intent to modify" (which avoids conversion deadlocks), exclusive for writes, and intent locks (IS, IX, SIX) on parent levels so table-level checks are cheap. There are also schema locks: Sch-S for queries and Sch-M for DDL, which blocks everything. Under SERIALIZABLE, key-range locks protect gaps. X locks are held until the transaction ends. S locks under READ COMMITTED are released right after the read. The compatibility matrix decides who waits.
**Structure a longer answer:** granularity hierarchy → modes → compatibility → lock duration by isolation level → how an UPDATE locks top-down → diagnosis with `sys.dm_tran_locks` → design implications (indexes, short transactions, hints).
**Likely follow-ups:** *Why do U locks exist?* → to prevent S→X conversion deadlocks. · *Can NOLOCK be blocked?* → yes, by Sch-M. · *What's SIX?* → a shared lock on the whole object plus intent to take X on parts, e.g. a SERIALIZABLE scan that updates some rows.
**Signals interviewers look for:** you know the matrix for S/U/X/IX, explain intent locks and U locks, and connect lock footprint to indexes.

### 11. Check yourself
- [ ] Fill in the S/U/X/IS/IX compatibility matrix from memory.
- [ ] List the locks an indexed single-row UPDATE takes, top to bottom.
- [ ] Explain the conversion deadlock and how U locks prevent it.
- [ ] Show in `sys.dm_tran_locks` how a non-SARGable UPDATE locks more than an indexed one.
- [ ] Build a work-queue dequeue with `UPDLOCK, READPAST, ROWLOCK` and test it from two sessions.
<!-- explainer:end -->

## References

- *T-SQL Fundamentals (4th ed.)* — Itzik Ben-Gan — Transactions and concurrency
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 10: Transactions, Concurrency, and Isolation Levels › Lock Granularity
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 10: Transactions, Concurrency, and Isolation Levels › Lock Compatibility
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 10: Transactions, Concurrency, and Isolation Levels › Lock Modes
- *SQL Server 2025 Query Performance Tuning (7th ed.)* — Grant Fritchey — FULL_BOOK_NEEDS_MANUAL_SPLIT (cont.) › Chapter 16 Blocking And Blocked Processes
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 10: Transactions, Concurrency, and Isolation Levels › Hands-on Labs
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 10: Transactions, Concurrency, and Isolation Levels › Locking Best Practices Summary
- *Expert Performance Indexing in SQL Server* — Jason Strate, Grant Fritchey — Page Lock
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 10: Transactions, Concurrency, and Isolation Levels › Lock Hints
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 16: Best Practices for T-SQL Development and Database Design › Principles of Database Security

