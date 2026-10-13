---
id: "TOOLS/SQL_Server/T-SQL/Dialects_and_Portability/06_Transactions_and_Concurrency_Across_Engines"
level: 5
type: "concept"
topic: "TOOLS/SQL_Server/T-SQL/Dialects_and_Portability"
tools: ["SQL_Server"]
---

# 6 · Transactions and Concurrency Across Engines

**L5** · [↑ Series index](<_index.md>) · Prev: [Parser, Binder and Error Messages](<05_Parser_Binder_and_Error_Messages.md>) · Next: [Procedural Code](<07_Procedural_Code_T-SQL_vs_PLSQL_vs_PLpgSQL.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** The same `SELECT … ; UPDATE …` sequence can block, deadlock, read stale data or fail on one engine and sail through on another, because the *default* transaction mechanism — locks in SQL Server, row versions in PostgreSQL/Oracle/MySQL — is part of the engine, not the language.

### 1. Why concurrency is a dialect issue

The SQL standard defines four isolation levels in terms of **phenomena** (dirty read, non-repeatable read, phantom). It does not say *how* to achieve them. So engines choose:

| Engine | Default mechanism for reads | Readers block writers? | Writers block readers? |
|---|---|---|---|
| **SQL Server** (default `READ COMMITTED`) | **Shared locks**, released immediately after the read | Yes (a reader waits for an uncommitted writer's X-lock) | Yes (a writer waits for readers' short S-locks) |
| **SQL Server with RCSI / SNAPSHOT** | **Row versions** in `tempdb` (or PVS with ADR) | No | No |
| **PostgreSQL** (`READ COMMITTED`) | **MVCC** snapshots per statement | No | No |
| **Oracle** (`READ COMMITTED`) | **Undo-based consistent reads** per statement | No | No |
| **MySQL InnoDB** (`REPEATABLE READ` default) | **MVCC** snapshot per transaction | No (non-locking reads) | No |

> **Azure SQL Database** enables `READ_COMMITTED_SNAPSHOT` by default; a box SQL Server instance does not. That alone can make the same T-SQL behave differently on-prem vs in Azure. *Verify the setting for your own service tier.*

### 2. Isolation levels side by side

| Level | SQL Server implementation | PostgreSQL | Oracle | MySQL (InnoDB) |
|---|---|---|---|---|
| Read uncommitted | Supported (`NOLOCK` hint ≈ this) | Treated as read committed | Not supported | Supported (dirty reads possible) |
| Read committed | Locking (or RCSI versions) | MVCC, per-statement snapshot | Per-statement snapshot | per-statement snapshot (RC setting) |
| Repeatable read | Holds S-locks to commit | **Snapshot** (per-transaction) — also prevents phantoms | n/a (use SERIALIZABLE/read-only) | Default; snapshot reads, gap locks for locking reads |
| **Snapshot** | `SNAPSHOT` (needs DB option ON) | = Repeatable read | `SERIALIZABLE` ≈ snapshot isolation | – |
| Serializable | **Range locks** (`KEY-RANGE`) — blocking | **SSI** (predicate tracking, aborts with serialization failure) | Snapshot-based; may raise ORA-08177 | locking reads |

Take-aways:
- "Serializable" is not one thing: SQL Server blocks; PostgreSQL aborts (so *you must retry* on SQLSTATE `40001`).
- Oracle's "SERIALIZABLE" is snapshot isolation with write-skew still possible.
- `REPEATABLE READ` in SQL Server and PostgreSQL are *different animals*.

### 3. Statement-level quirks that affect application code

#### 3.1 `NOLOCK`
`WITH (NOLOCK)` / `READ UNCOMMITTED` can return **dirty rows, rows twice, or skip rows** during page splits — not just "slightly stale" data. PostgreSQL has no equivalent need because readers never wait. In SQL Server, use RCSI instead of sprinkling `NOLOCK`. See [Read Committed Locking Anomalies](<../../SQL_Server_Platform/Read_Committed_Locking_Anomalies.md>).

#### 3.2 Locking reads
| Need | T-SQL | PG / Oracle / MySQL |
|---|---|---|
| Lock rows I'm about to update | `SELECT … FROM t WITH (UPDLOCK, ROWLOCK) WHERE …` | `SELECT … FOR UPDATE` |
| Queue pattern (skip locked rows) | `WITH (UPDLOCK, READPAST, ROWLOCK)` | `FOR UPDATE SKIP LOCKED` |
| Fail rather than wait | `SET LOCK_TIMEOUT 0` | `FOR UPDATE NOWAIT` |
| Application lock | `sp_getapplock` | advisory locks (`pg_advisory_lock`) / `DBMS_LOCK` |

```sql
-- T-SQL queue worker: take one unclaimed message without blocking others
;WITH next_msg AS (
    SELECT TOP (1) *
    FROM   dbo.Queue WITH (UPDLOCK, READPAST, ROWLOCK)
    WHERE  Status = 'new'
    ORDER  BY Id
)
UPDATE next_msg SET Status = 'processing'
OUTPUT inserted.Id, inserted.Payload;
```

#### 3.3 Autocommit vs implicit transactions
| Engine | Default | How a multi-statement transaction starts |
|---|---|---|
| SQL Server | **Autocommit** — every statement commits itself | `BEGIN TRAN[SACTION]` explicitly (or `SET IMPLICIT_TRANSACTIONS ON`) |
| PostgreSQL | Autocommit in psql/drivers (each statement is its own transaction) | `BEGIN` |
| Oracle | **No autocommit** — first DML starts a transaction | `COMMIT`/`ROLLBACK`; DDL auto-commits |
| MySQL | Autocommit ON | `START TRANSACTION`/`BEGIN`; DDL implicit commit |

A script written for Oracle that never says `COMMIT` becomes *auto-committed per statement* on SQL Server (a failure halfway leaves partial changes) — and a script written for SQL Server that omits `BEGIN TRAN` has no atomicity at all.

#### 3.4 Nested transactions and savepoints
```sql
BEGIN TRAN;            -- @@TRANCOUNT = 1
  BEGIN TRAN;          -- @@TRANCOUNT = 2  (not a real nested transaction)
  COMMIT;              -- @@TRANCOUNT = 1  (nothing committed yet)
ROLLBACK;              -- rolls back EVERYTHING, @@TRANCOUNT = 0
```
- Inner `COMMIT` only decrements the counter; the *outermost* `COMMIT` is the real one; **any `ROLLBACK` rolls back all** (unless it names a savepoint).
- Savepoints: `SAVE TRANSACTION sp1; … ROLLBACK TRANSACTION sp1;` — compare to `SAVEPOINT` / `ROLLBACK TO` in PG/Oracle/MySQL.

#### 3.5 DDL inside transactions
SQL Server and PostgreSQL support **transactional DDL** (`CREATE TABLE` can be rolled back). Oracle and MySQL implicitly commit before and after DDL. Migration scripts must be designed accordingly.

### 4. Error behaviour inside a transaction

| Engine | After a runtime error in a transaction |
|---|---|
| SQL Server (default) | Statement fails, **transaction remains open**, next statements run → partial commits possible |
| SQL Server `SET XACT_ABORT ON` | Error ⇒ whole transaction rolled back, batch stops |
| PostgreSQL | Transaction enters **aborted state**: every command fails until `ROLLBACK` (or `ROLLBACK TO SAVEPOINT`) |
| Oracle | Statement-level rollback only; transaction continues |
| MySQL | Statement-level; depends on error and engine |

Best practice for T-SQL procedures that change data: `SET XACT_ABORT ON; SET NOCOUNT ON;` + `TRY…CATCH` with `IF @@TRANCOUNT > 0 ROLLBACK; THROW;`.

### 5. Deadlocks and retries

| | SQL Server | PostgreSQL | Oracle | MySQL |
|---|---|---|---|---|
| Error | Msg **1205** | SQLSTATE **40P01** | ORA-**00060** | Error **1213** (`40001`) |
| Victim chosen by | lowest `DEADLOCK_PRIORITY`, then least log written | the session that detects it | the session that detects it | InnoDB picks smallest transaction |
| What the app must do | **Retry the whole transaction** | same | same | same |

Common deadlock cause in T-SQL: **conversion deadlock** (two sessions take S then both want U/X on the same key) — fix by `UPDLOCK` on the initial read, or by RCSI which eliminates many reader/writer cycles. See [Deadlocks](<../../05_Concurrency/Deadlocks/_index.md>) and [Blocking Analysis](<../../05_Concurrency/Blocking_Analysis/_index.md>).

### 6. Locks, escalation and long transactions

- **SQL Server** may **escalate** row/page locks to a table lock after ~5,000 locks per statement/object (configurable per table, `LOCK_ESCALATION`). Batch `DELETE`/`UPDATE` of large ranges therefore blocks the table. Fix: chunk (`TOP (n)` loop), or partition switching. See [Lock Escalation](<../../05_Concurrency/Lock_Escalation/_index.md>).
- PostgreSQL and Oracle have no lock escalation; they have **bloat / undo** costs instead (dead tuples + vacuum; undo retention / `ORA-01555`).
- SQL Server's version store lives in `tempdb` (or **PVS** in the user DB with Accelerated Database Recovery); long-running open transactions under RCSI/SNAPSHOT **inflate** it. Monitor `sys.dm_tran_version_store_space_usage`.

### 7. Worked example — the same app code, two engines

Transfer money between two accounts:

```sql
-- T-SQL, correct under default READ COMMITTED
SET XACT_ABORT ON;
BEGIN TRAN;
  UPDATE dbo.Account WITH (UPDLOCK, ROWLOCK) SET Balance -= @amt WHERE Id = @from AND Balance >= @amt;
  IF @@ROWCOUNT = 0 BEGIN ROLLBACK; THROW 50001, 'Insufficient funds', 1; END;
  UPDATE dbo.Account SET Balance += @amt WHERE Id = @to;
COMMIT;
```
```sql
-- PostgreSQL equivalent (works under default READ COMMITTED)
BEGIN;
UPDATE account SET balance = balance - :amt WHERE id = :from AND balance >= :amt;
-- check rows affected in the driver; ROLLBACK if 0
UPDATE account SET balance = balance + :amt WHERE id = :to;
COMMIT;
```

Differences:
- **Locking:** SQL Server needs the explicit `UPDLOCK` on a *separate read* to avoid conversion deadlocks; a single `UPDATE … WHERE` is atomic so the hint is mostly redundant here. PostgreSQL's `UPDATE` takes a row-level lock; concurrent updaters block, then re-evaluate the `WHERE` on the new row version (the `EvalPlanQual` mechanism).
- **Deadlock risk:** order updates by id (lowest first) on both engines.
- **Retries:** both need application retry on deadlock; PostgreSQL also on serialization failure at higher isolation.

### 8. Performance and design implications

- **Turn on RCSI** for OLTP SQL Server databases that mix readers and writers — closest to PG/Oracle semantics; test for code relying on blocking (rare, but e.g. queue patterns).
- **Keep transactions short** everywhere; open transaction + user think-time is a universal anti-pattern.
- **Measure:** `sys.dm_tran_locks`, `sys.dm_os_waiting_tasks`, `sys.dm_exec_requests` (`blocking_session_id`), Extended Events `xml_deadlock_report`, Query Store wait stats.
- **ETL load design:** bulk loads to staging with `TABLOCK` (minimal logging) then partition-switch or `INSERT … SELECT` into targets; avoid long-running single transactions that escalate locks.

### 9. Common mistakes

- Porting Oracle code and forgetting `BEGIN TRAN`/`COMMIT`.
- Using `NOLOCK` everywhere "for speed" → wrong totals.
- Assuming serialization errors can't happen in PostgreSQL at `REPEATABLE READ`/`SERIALIZABLE`.
- Leaving a transaction open after an error (T-SQL without `XACT_ABORT`) → locks held, log can't truncate.
- Treating `@@TRANCOUNT > 1` as "my transaction is isolated".

### 10. 🎯 Interview kit

**30-second answer:** "SQL Server's default READ COMMITTED uses shared locks, so readers and writers block each other; PostgreSQL and Oracle use MVCC. I enable READ_COMMITTED_SNAPSHOT to get MVCC-style behaviour in SQL Server. I also remember autocommit differs and that errors don't automatically roll back in T-SQL unless XACT_ABORT is on."

**Likely follow-ups:** *Serializable on PostgreSQL vs SQL Server?* → SSI aborts vs range locks block. *What is lock escalation?* → promotion of many fine locks to a table lock; not in PG/Oracle. *How do you implement a work queue in T-SQL?* → `UPDLOCK, READPAST` (≈ `SKIP LOCKED`). *Why does the inner COMMIT not commit?* → `@@TRANCOUNT` is a nesting counter.

### 11. Check yourself

- [ ] State the default read mechanism of SQL Server, PostgreSQL, Oracle, MySQL.
- [ ] Write the T-SQL equivalent of `SELECT … FOR UPDATE SKIP LOCKED`.
- [ ] What does `SET XACT_ABORT ON` change?
- [ ] What happens with `BEGIN TRAN; BEGIN TRAN; COMMIT; ROLLBACK;`?
- [ ] Why can Azure SQL Database behave differently from an on-prem SQL Server for the same query?

**Where this connects:** [Isolation and Row Versioning](<../../05_Concurrency/Isolation_and_Row_Versioning/_index.md>) · [Lock Modes and Granularity](<../../05_Concurrency/Lock_Modes_and_Granularity/_index.md>) · [MERGE Pitfalls](<../MERGE_Pitfalls_and_Error_Handling.md>) · [Oracle isolation](<../../../Oracle/Oracle_Platform/_index.md>) · [PostgreSQL](<../../../PostgreSQL/_index.md>) · Next → [Procedural Code](<07_Procedural_Code_T-SQL_vs_PLSQL_vs_PLpgSQL.md>)
<!-- explainer:end -->
