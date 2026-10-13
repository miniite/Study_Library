---
type: "practice"
difficulty: "Intermediate"
---

# What is the difference between the TRUNCATE and DELETE commands in terms of rollback and triggers?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ SQL vs NoSQL](<../../../../01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/SQL_vs_NoSQL.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

#### Triage

What's really being asked: Both TRUNCATE TABLE t and DELETE FROM t empty a table, so when do they behave differently — specifically around (a) whether the operation can be rolled back inside a transaction and (b) whether it fires row triggers? A naive answer says "both delete all rows"; the interviewer wants the semantics that bite you in production.

What the interviewer is probing for: Do you understand that DELETE is a DML row-by-row logged operation while TRUNCATE is (in most engines) a DDL metadata/deallocation operation? Do you know the engine-specific nuances (Oracle vs SQL Server vs Postgres vs MySQL) instead of parroting one vendor's rule as universal? Do you know the practical consequences: triggers, identity/auto-increment reset, WHERE clause, permissions, foreign keys, locking, and speed?

Clarifying question I'd ask: Which database? Because the rollback answer flips between Oracle and SQL Server/Postgres. I'll give the general model, then call out the vendor differences explicitly — that precision is what earns the point.

Plan: define each command, then answer the two named axes (rollback, triggers) precisely, then round out with the other differences that matter and a "when to use which" rule.

What each command actually does

DELETE FROM t [WHERE ...] is DML. The engine locates each qualifying row, writes an entry to the transaction/undo log for every row (so it can be undone), removes it, updates indexes, and fires row-level DELETE triggers per row. Because it's per-row and fully logged, it's slower on large tables but flexible: you can filter with WHERE, and it participates fully in your transaction.

TRUNCATE TABLE t removes all rows by deallocating the data pages / resetting the storage rather than deleting rows one at a time. It's classified as DDL in most engines. There's no WHERE clause — it's all or nothing. It's near-instant and barely touches the transaction log (it logs page deallocations, not individual rows). It typically resets identity/auto-increment counters and requires higher privileges (ALTER/DROP-level), not just DELETE permission.

Axis 1 — Rollback

This is the axis with the most vendor variation, so be precise:

SQL Server: TRUNCATE is transactional and can be rolled back. TRUNCATE inside BEGIN TRAN ... ROLLBACK restores the data. It logs page deallocations (minimally logged), which is enough to undo. So in SQL Server both are rollback-able.
PostgreSQL: TRUNCATE is also transactional and MVCC-safe — issue it inside a transaction block and ROLLBACK brings the rows back. Postgres treats it as a DDL statement that is still fully transactional.
Oracle: TRUNCATE is DDL and includes an implicit COMMIT. You cannot roll it back — the commit before/after the statement finalizes it (and it can't sit inside your transaction; it ends the current one). This is the classic "TRUNCATE can't be rolled back" claim — it's true for Oracle.
MySQL/InnoDB: TRUNCATE performs an implicit commit and is not rollback-able; internally it often drops and recreates the table. Like Oracle, treat it as irreversible.

DELETE, by contrast, is DML in every engine: uncommitted, it always rolls back with ROLLBACK; once committed it's gone (recoverable only via backups/PITR/flashback). So the honest one-liner is: DELETE is always rollback-able while uncommitted; TRUNCATE is rollback-able in SQL Server and PostgreSQL, but NOT in Oracle or MySQL (implicit commit).

Axis 2 — Triggers
DELETE fires row-level (FOR EACH ROW) and statement-level DELETE/AFTER DELETE/BEFORE DELETE triggers, once per row (row triggers) or per statement (statement triggers). This is why DELETE can enforce cascading audit logic, soft-delete bookkeeping, etc.
TRUNCATE does NOT fire DELETE triggers. Because it operates on storage, not rows, the DBMS never "sees" individual row deletions, so no per-row DELETE trigger runs.
PostgreSQL is the exception with nuance: it supports a dedicated TRUNCATE trigger (CREATE TRIGGER ... AFTER TRUNCATE) that does fire on truncate — but ordinary DELETE triggers still do not.
In Oracle/SQL Server/MySQL there's no per-row trigger firing on truncate at all.

Practical consequence: if you rely on DELETE triggers for auditing or downstream cascades, TRUNCATE silently bypasses them — a common data-integrity bug. Never TRUNCATE a table whose deletes must be audited unless you've accounted for it.

Other differences worth stating (rounds out a senior answer)
WHERE clause: DELETE can filter; TRUNCATE cannot — it's the whole table (or specific partitions in some engines).
Identity / AUTO_INCREMENT: TRUNCATE resets the counter to the seed; DELETE does not (next insert continues from the last value).
Foreign keys: TRUNCATE is usually blocked if the table is referenced by an FK (SQL Server/MySQL refuse; Postgres requires CASCADE). DELETE respects FKs row-by-row and can trigger ON DELETE CASCADE.
Speed & log: TRUNCATE is dramatically faster on big tables (deallocates pages, minimal logging); DELETE writes per-row log entries and is slow at scale — for "delete all rows" TRUNCATE is the right tool.
Locking: TRUNCATE typically takes a table/schema lock (short but exclusive); DELETE takes row/page locks and holds them for the transaction.
Permissions: TRUNCATE needs DDL-level rights (ALTER); DELETE needs only DELETE. This is a real access-control difference.
DROP vs TRUNCATE vs DELETE: DROP removes the table structure itself; TRUNCATE keeps the structure, empties data, resets storage; DELETE keeps structure and storage allocation, removes rows.
When to use which
Emptying an entire staging/temp table fast, don't need triggers/rollback, want the identity reset → TRUNCATE.
Removing some rows, or needing triggers/audit/cascade, or needing to undo inside a transaction on Oracle/MySQL → DELETE (add a WHERE, or DELETE in batches for very large sets to control log growth/locks).
Summary — what a strong answer emphasizes

Frame it as DML (DELETE, row-by-row, fully logged) vs DDL (TRUNCATE, storage-level). On rollback: DELETE is always undoable while uncommitted; TRUNCATE is undoable in SQL Server and Postgres but not in Oracle/MySQL (implicit commit) — naming the engine is what makes the answer correct. On triggers: DELETE fires per-row DELETE triggers; TRUNCATE fires none — except Postgres's dedicated TRUNCATE trigger. Then add identity reset, no WHERE, FK restrictions, speed, locking, and permissions.

#### Common mistakes
stating "TRUNCATE can never be rolled back" as universal (false for SQL Server/Postgres); claiming TRUNCATE fires DELETE triggers; forgetting it resets AUTO_INCREMENT; assuming it works while an FK references the table.

**Comparison:**

```
Aspect	DELETE	TRUNCATE
Classification	DML (row-by-row)	DDL (storage/page deallocation)
WHERE clause	Yes (can filter)	No (whole table)
Rollback (uncommitted)	Always (all engines)	Yes in SQL Server & PostgreSQL; NO in Oracle & MySQL (implicit COMMIT)
Fires DELETE triggers	Yes (per row / per statement)	No (except Postgres AFTER TRUNCATE trigger)
Identity / AUTO_INCREMENT	Not reset	Reset to seed
Transaction log	Per-row entries (heavy)	Page deallocations only (minimal)
Speed on large table	Slow	Very fast
FK-referenced table	Allowed (row-by-row, honors CASCADE)	Blocked (SQL Server/MySQL) / needs CASCADE (Postgres)
Locking	Row/page locks	Table/schema lock (brief, exclusive)
Privilege needed	DELETE	ALTER / DDL-level
```

**Code:**

```sql
-- DELETE: filtered, transactional, fires triggers, keeps identity counter
BEGIN TRANSACTION;
  DELETE FROM orders WHERE status = 'cancelled';  -- WHERE allowed
  -- ... any DELETE triggers on orders fire per row ...
ROLLBACK;   -- rows come back (uncommitted) in every engine

-- TRUNCATE: whole table, resets identity, no DELETE triggers, no WHERE
TRUNCATE TABLE staging_events;   -- fast page deallocation

-- SQL Server / PostgreSQL: TRUNCATE is rollback-able
BEGIN TRANSACTION;
  TRUNCATE TABLE staging_events;
ROLLBACK;   -- SQL Server & Postgres: data restored

-- Oracle / MySQL: TRUNCATE issues an IMPLICIT COMMIT -> cannot roll back
TRUNCATE TABLE staging_events;   -- irreversible here

-- PostgreSQL-only: a trigger that DOES fire on TRUNCATE
CREATE TRIGGER trg_after_trunc
AFTER TRUNCATE ON staging_events
FOR EACH STATEMENT EXECUTE FUNCTION log_truncate();

-- FK reference blocks TRUNCATE (must use DELETE or Postgres CASCADE)
TRUNCATE TABLE customers;          -- errors if orders.customer_id -> customers
TRUNCATE TABLE customers CASCADE;  -- Postgres: also truncates dependents
```

**Example input / output:**

```text
Scenario A — DELETE rollback (any engine)
  Table orders has 5 rows.
  BEGIN; DELETE FROM orders;  -> 0 rows visible in this txn
  ROLLBACK;                   -> 5 rows back.  [PASS everywhere]

Scenario B — TRUNCATE rollback
  SQL Server:  BEGIN TRAN; TRUNCATE TABLE orders; ROLLBACK;  -> 5 rows back
  PostgreSQL:  BEGIN;      TRUNCATE orders;        ROLLBACK;  -> 5 rows back
  Oracle:      TRUNCATE TABLE orders;  (implicit COMMIT)      -> 0 rows, gone
  MySQL:       TRUNCATE TABLE orders;  (implicit COMMIT)      -> 0 rows, gone

Scenario C — Triggers (AFTER DELETE trigger writes to audit_log)
  DELETE FROM orders;   -> audit_log gains 5 rows (trigger fired per row)
  TRUNCATE TABLE orders;-> audit_log unchanged (DELETE trigger NOT fired)
                          (Postgres AFTER TRUNCATE trigger, if defined, fires once)

Scenario D — Identity reset (id IDENTITY/AUTO_INCREMENT, last id = 5)
  DELETE all; INSERT one   -> new id = 6  (counter preserved)
  TRUNCATE;   INSERT one   -> new id = 1  (counter reset to seed)
```

**Flow:**

1. Yes
2. No: empty the whole tablefast
3. Yes
4. No
5. Oracle/MySQL and needrollback
6. SQL Server/Postgres, or noundo needed
7. Need to remove rows?
8. Some rows only, or needtriggers/audit?
9. Use DELETE ... WHERE
10. Table referenced by an FK?
11. Engine + must undo inside atxn?
12. Use TRUNCATE TABLE
13. Resets identity, skips DELETEtriggers

