---
id: "TOOLS/MySQL/MySQL_Platform/Deadlocks_in_MySQL"
level: 4
type: "concept"
topic: "TOOLS/MySQL/MySQL_Platform"
tools: ["MySQL"]
---

# Deadlocks in MySQL

**L4 Advanced** · [↑ MySQL Platform](<_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** InnoDB locks individual index records (and the gaps between them) for the length of a transaction, so concurrent transactions that touch the same rows in different orders can deadlock — you prevent it with short transactions, consistent lock order, good indexes and retry logic, and you diagnose it from the deadlock report and `performance_schema`.

### 1. The big picture
A transaction groups statements into one all-or-nothing unit (`START TRANSACTION` … `COMMIT` / `ROLLBACK`). To keep concurrent transactions from corrupting each other, InnoDB uses **MVCC** for plain reads (readers don't block writers) and **locks** for writes and locking reads. Locks are what produce waits, "Lock wait timeout exceeded", and deadlocks.

This lesson gathers the whole concurrency-correctness toolbox that MySQL interviews and production incidents revolve around: ACID basics, which locks InnoDB takes and why, how deadlocks form and are detected, how to design idempotent writes (unique keys, upserts, dedup tables), and how to debug lock waits under load. It connects forward to isolation levels and to Read-Your-Writes consistency with replicas.

### 2. Core concepts and vocabulary
- **Transaction / ACID** — Atomicity (all or nothing), Consistency (constraints hold before and after, e.g., a transfer keeps the total balance), Isolation (concurrent transactions don't see each other's partial work), Durability (after `COMMIT` the change survives a crash; redo log flushed per `innodb_flush_log_at_trx_commit=1`).
- **Autocommit** — default `ON`: each statement is its own transaction. `START TRANSACTION` (or `BEGIN`) suspends it until `COMMIT`/`ROLLBACK`. A crash or dropped connection before `COMMIT` rolls the transaction back.
- **Record lock** — lock on one index record. **Gap lock** — lock on the space between index records (blocks inserts there). **Next-key lock** — record + preceding gap; the default for locking reads and writes under `REPEATABLE READ`. **Insert intention lock** — a gap-lock variant signalling an intent to insert.
- **Shared (S) / exclusive (X) lock** — `SELECT … FOR SHARE` (formerly `LOCK IN SHARE MODE`) takes S; `SELECT … FOR UPDATE`, `UPDATE`, `DELETE` take X. S+S compatible; anything with X conflicts.
- **Table lock vs row lock** — InnoDB locks rows (actually index records); `LOCK TABLES` and DDL use table/metadata locks.
- **Deadlock** — a cycle of waits: T1 holds A and wants B; T2 holds B and wants A.
- **Lock wait timeout** — a transaction waited longer than `innodb_lock_wait_timeout` (default 50 s) → error 1205; *not* a deadlock.
- **Idempotent operation** — running it twice has the same effect as once.
- **Optimistic concurrency** — no lock held while the user thinks; a `version` column detects that someone else changed the row.

### 3. How it works, step by step — how a deadlock happens and resolves
1. T1: `UPDATE accounts SET bal = bal - 10 WHERE id = 1;` → X lock on row 1.
2. T2: `UPDATE accounts SET bal = bal - 5 WHERE id = 2;` → X lock on row 2.
3. T1: `UPDATE accounts SET bal = bal + 10 WHERE id = 2;` → waits for T2.
4. T2: `UPDATE accounts SET bal = bal + 5 WHERE id = 1;` → waits for T1 → cycle.
5. InnoDB's **deadlock detector** (`innodb_deadlock_detect=ON`) finds the cycle immediately and rolls back the **victim** — the transaction with the least work (fewest modified rows). The victim's client receives **error 1213** (`ER_LOCK_DEADLOCK`, SQLSTATE 40001).
6. The other transaction proceeds. The application should **retry the whole transaction**.

Plain `SELECT`s under `REPEATABLE READ`/`READ COMMITTED` use consistent snapshots and take no row locks, so they neither cause nor suffer these waits; the trouble comes from writes, `FOR UPDATE`/`FOR SHARE`, and gap/next-key locks from range conditions on indexes.

### 4. Worked example
```sql
-- Prevent the cycle above: always lock in a fixed order (lower id first)
START TRANSACTION;
SELECT id, bal FROM accounts WHERE id IN (1, 2) ORDER BY id FOR UPDATE;
UPDATE accounts SET bal = bal - 10 WHERE id = 1;
UPDATE accounts SET bal = bal + 10 WHERE id = 2;
COMMIT;
```
`ORDER BY id` makes every transfer acquire locks in ascending id order, so two transfers can never wait on each other in a cycle.

**Idempotent create-order** (double-click, retry, replayed message):
```sql
CREATE TABLE orders (
  order_id   BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
  user_id    BIGINT UNSIGNED NOT NULL,
  idem_key   CHAR(36)        NOT NULL,
  total      DECIMAL(12,2)   NOT NULL,
  UNIQUE KEY uq_orders_idem (user_id, idem_key)
);

INSERT INTO orders (user_id, idem_key, total)
VALUES (42, 'b6c1…-uuid-from-client', 99.00)
ON DUPLICATE KEY UPDATE order_id = LAST_INSERT_ID(order_id);   -- returns the existing id
SELECT LAST_INSERT_ID();
```
The unique key makes the second call a safe no-op that returns the first order's id. `affected_rows` is 1 for insert, 0 or 2 for the duplicate path depending on whether values changed — use `LAST_INSERT_ID(order_id)` for a stable id.

**Exactly-once consumer with a dedup table:**
```sql
START TRANSACTION;
INSERT INTO processed_events (event_id) VALUES (?);   -- PK; duplicate → error 1062 → skip
-- ... apply the event's effect in the same transaction ...
COMMIT;
```
The marker and the effect commit together, so a crash can't leave one without the other. Purge old markers by `processed_at` in small batches, keeping the window longer than the maximum possible redelivery delay.

**Atomic counter/upsert without a racy read:**
```sql
INSERT INTO counters (k, n) VALUES ('likes:post:7', 1)
ON DUPLICATE KEY UPDATE n = n + 1;
```
A `SELECT` then `INSERT` is racy: two sessions both see "no row" and both insert.

**Debugging** (8.0):
```sql
SHOW ENGINE INNODB STATUS\G              -- "LATEST DETECTED DEADLOCK" section
SELECT * FROM performance_schema.data_lock_waits;
SELECT * FROM sys.innodb_lock_waits\G    -- who blocks whom, with the SQL
SET GLOBAL innodb_print_all_deadlocks = ON;  -- log every deadlock to the error log
```

### 5. Under the hood
- **Locks live on index records.** An `UPDATE … WHERE col = x` locks every index record it scans, plus the clustered-index record of each matched row. With no usable index, InnoDB scans the whole clustered index and (under `REPEATABLE READ`) locks *every* row/gap it visits — which looks like a table lock and is a classic deadlock/timeout generator. **Add the index.**
- **Gap and next-key locks** exist to prevent phantoms under `REPEATABLE READ`. Two sessions can both hold a gap lock on the same gap (they don't conflict with each other) and then both try to insert into it, each waiting on the other's gap → deadlock. Dropping to `READ COMMITTED` (`SET TRANSACTION ISOLATION LEVEL READ COMMITTED`) disables most gap locking and removes this class, at the price of non-repeatable reads and phantoms.
- **Deadlock detection cost.** The detector walks the wait-for graph; with very high concurrency on hot rows, detection itself can become a bottleneck, and some setups disable it (`innodb_deadlock_detect=OFF`) and rely on a short `innodb_lock_wait_timeout` — only with strong retry/timeout handling.
- **Victim selection.** Weight is based on rows modified/locked; rolled-back work is small by design. Statements that fail with a lock wait timeout roll back **only the statement** by default (`innodb_rollback_on_timeout=OFF`), leaving the transaction open and still holding earlier locks — the application must roll back explicitly.
- **MVCC and undo.** Updates write an undo record and a version pointer; non-locking reads rebuild old versions via the undo chain. Long-running transactions force undo to be retained (history list growth) — another reason to keep transactions short.
- **Auto-increment locking** (`innodb_autoinc_lock_mode`): `INSERT`s into a table with `AUTO_INCREMENT` may take a brief table-level AUTO-INC lock or a lightweight mutex (mode 2 default in 8.0), producing gaps but few waits.
- **Durability.** On `COMMIT` with `innodb_flush_log_at_trx_commit=1` and `sync_binlog=1`, the redo and binary logs are fsync'd; committed data survives power loss. Values 0/2 trade durability for throughput.

### 6. Performance and design implications
- **Keep transactions short:** no user think-time, no HTTP calls, no queue publishes inside a transaction. A third-party call inside a transaction holds locks for its whole latency; do the call before/after, and use an outbox table for reliable messaging.
- **Touch rows in a consistent order** (sort IDs, lock parents before children, same table order everywhere).
- **Make the access path an index lookup** so you lock few records.
- **Batch large changes** (`DELETE … LIMIT 1000` loops, `UPDATE … WHERE id BETWEEN`) to bound lock footprint and undo size.
- **Retry on 1213 and 1205** with a bounded, jittered backoff, re-running the *entire* transaction and re-reading state.
- **Choose the lock deliberately:** `FOR UPDATE` for read-modify-write on a few hot rows; optimistic `UPDATE … SET …, version = version + 1 WHERE id = ? AND version = ?` (check `ROW_COUNT()`) for low-contention user edits; atomic SQL (`SET n = n + 1`) when no read is needed.
- **Hot-row counters** (likes, stock) serialize on one row's X lock; shard the counter (N slots) or buffer increments.
- **Append-only ledgers:** insert-only rows with a monotonic key and a unique `(account_id, seq)` to forbid reordering; balances come from `SUM` or periodic snapshot rows, never from updating a single balance row under heavy concurrency.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `START TRANSACTION; … COMMIT; ROLLBACK;` / `SAVEPOINT s; ROLLBACK TO s;` | Transaction control / partial rollback | Multi-statement units |
| `SELECT … FOR UPDATE` / `FOR SHARE` (`NOWAIT`, `SKIP LOCKED`) | Take X / S locks; fail fast or skip locked rows | Read-modify-write; job queues |
| `SHOW ENGINE INNODB STATUS` | Latest deadlock, transactions, lock info | Post-mortem |
| `performance_schema.data_locks`, `data_lock_waits`, `sys.innodb_lock_waits` | Live lock and waiter listing | Diagnose waits |
| `innodb_lock_wait_timeout` | Seconds to wait before error 1205 | Tune per workload |
| `innodb_print_all_deadlocks` | Log every deadlock | Production visibility |
| `innodb_deadlock_detect` | Enable/disable detector | Extreme hot-row contention only |
| `SET TRANSACTION ISOLATION LEVEL …` | Per-transaction isolation | `READ COMMITTED` to cut gap locks |
| `INSERT … ON DUPLICATE KEY UPDATE`, `INSERT IGNORE`, `REPLACE` | Upsert variants | Idempotency (prefer ODKU; `REPLACE` deletes+inserts) |
| `autocommit`, `ROW_COUNT()` | Statement mode; rows changed | Optimistic locking checks |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| Pessimistic `FOR UPDATE` — high contention, short critical section | User think-time between read and write (locks held too long) |
| Optimistic version column — low contention, long edits, REST APIs (return 409) | Hot rows with constant conflicts → retry storms |
| `READ COMMITTED` — fewer gap locks, fewer deadlocks | You depend on repeatable reads within a transaction |
| `INSERT … ON DUPLICATE KEY UPDATE` — idempotent upsert | Multiple unique keys (it may update the wrong row) and auto-increment gaps |
| `SKIP LOCKED` queues — many workers | Strict ordering requirements |
| Unique key + dedup table | Unbounded growth without a purge job |

### 9. Common mistakes
- **Doing remote calls inside a transaction** → locks held across latency/timeouts → move outside, use outbox.
- **Missing index on the `WHERE`/join column** → full-range locks, deadlocks/timeouts → index it.
- **Inconsistent update order across code paths** → deadlock cycles → fixed global ordering.
- **Not retrying 1213** → user-visible failures → catch and retry the whole transaction.
- **Assuming a lock wait timeout rolled back everything** → partial work and held locks → explicit `ROLLBACK`.
- **`SELECT` then `INSERT` for uniqueness** → race → unique key + upsert.
- **Large `DELETE … WHERE created_at < …`** → long X locks and replica lag → chunk it and loop.
- **Mixing `LOCK TABLES` with transactions** → implicit commits and surprises → avoid.
- **Using `REPEATABLE READ` range updates on non-indexed columns** → gap locks across the table → index or lower isolation.

### 10. 🎯 Interview kit
**30-second answer:** InnoDB uses row/index-record locks held until commit; a deadlock is a wait cycle that InnoDB detects and breaks by rolling back one transaction (error 1213). Prevent it with short transactions, consistent lock order, proper indexes and lower isolation where acceptable; handle it by retrying the transaction. Diagnose with `SHOW ENGINE INNODB STATUS` and `performance_schema.data_lock_waits`.

**Structure a longer answer:** ACID/transaction basics → MVCC vs locking reads → record/gap/next-key locks → deadlock formation and detection → prevention → diagnosis → idempotent designs and retry contract.

**Likely follow-ups:**
- *Deadlock vs lock wait timeout?* → cycle detected immediately (1213) vs plain long wait that times out (1205).
- *What does `FOR UPDATE` do under `REPEATABLE READ` on a range?* → next-key locks, blocking inserts into the range.
- *How do you debug intermittent 1205 at peak?* → find blockers via `sys.innodb_lock_waits`, long transactions in `information_schema.innodb_trx`, add indexes, shorten transactions.
- *Idempotent POST?* → client idempotency key + unique constraint + `ON DUPLICATE KEY UPDATE`, return the original result.
- *What happens if the app crashes mid-transaction?* → uncommitted changes are rolled back on connection loss/crash recovery.
- *Shared vs exclusive lock?* → many readers vs one writer.

**Signals interviewers look for:** you name gap/next-key locks, you retry correctly, you use the diagnostics, and you keep external calls outside the transaction.

### 11. Check yourself
- [ ] Draw a two-transaction deadlock and show the fix by lock ordering.
- [ ] Explain what a gap lock protects against and how `READ COMMITTED` changes things.
- [ ] Locate the last deadlock and the blocking transaction using MySQL 8.0 views.
- [ ] Write an idempotent insert with a unique key and return the existing id.
- [ ] Explain why a missing index can make an `UPDATE` lock far more rows than it changes.
- [ ] Design a retry wrapper (what to catch, how many times, what to re-read).
<!-- explainer:end -->

## Implementations (tools)

- [MySQL](<../_index.md>)

## Practice questions

- [Basic] [What is START TRANSACTION and how do COMMIT and ROLLBACK complete a transaction?](<../../../PRACTICE/TOOLS/MySQL/MySQL_Platform/Q-33421_What_is_START_TRANSACTION_and_how_do_COMMIT_and_ROLLBACK_com.md>)
- [Intermediate] [What happens if your application crashes in the middle of a multi-statement transaction?](<../../../PRACTICE/TOOLS/MySQL/MySQL_Platform/Q-33571_What_happens_if_your_application_crashes_in_the_middle_of_a_.md>)
- [Deep] [A transaction updates two rows in different tables and occasionally deadlocks with a reverse-order transaction — how do you enforce a canonical lock ordering in code?](<../../../PRACTICE/TOOLS/MySQL/MySQL_Platform/Q-33464_A_transaction_updates_two_rows_in_different_tables_and_occas.md>)
- [Deep] [How would you design a schema so a 'reorder items by drag' operation persists cheaply, avoiding rewriting every row's position on each move?](<../../../PRACTICE/TOOLS/MySQL/MySQL_Platform/Q-33530_How_would_you_design_a_schema_so_a_reorder_items_by_drag_ope.md>)
- [Deep] [What is a deadlock in MySQL and how does InnoDB detect and resolve one?](<../../../PRACTICE/TOOLS/MySQL/MySQL_Platform/Q-33236_What_is_a_deadlock_in_MySQL_and_how_does_InnoDB_detect_and_r.md>)
- [Deep] [How would you design a schema and unique constraints so a coupon can be redeemed at most once per user and at most N times globally, race-safe?](<../../../PRACTICE/TOOLS/MySQL/MySQL_Platform/Q-33201_How_would_you_design_a_schema_and_unique_constraints_so_a_co.md>)
- [Deep] [A service intermittently sees "Duplicate entry for key PRIMARY" on an INSERT it believes is unique — how do you trace whether it's a retry, replication, or app-side ID reuse?](<../../../PRACTICE/TOOLS/MySQL/MySQL_Platform/Q-33284_A_service_intermittently_sees_Duplicate_entry_for_key_PRIMAR.md>)
- [Deep] [How would you design a schema and query to detect and block duplicate financial transactions within a short time window (same amount, payer, payee)?](<../../../PRACTICE/TOOLS/MySQL/MySQL_Platform/Q-33327_How_would_you_design_a_schema_and_query_to_detect_and_block_.md>)

## Related concepts

- [Lock Granularity](<../../../01_Databases_and_SQL/Database_Internals/DBMS_Concepts/Lock_Granularity.md>) *(DBMS Concepts)*
- [SKIP LOCKED Job Queues](<../../../01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/SKIP_LOCKED_Job_Queues.md>) *(SQL Fundamentals)*
- [Row Locks Two-phase Locking and Lock Footprint](<../../../01_Databases_and_SQL/Transactions_and_Concurrency/Locking/Row_Locks_Two-phase_Locking_and_Lock_Footprint.md>) *(Locking)*
- [Unique Constraints for Dedup](<../../../01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Unique_Constraints_for_Dedup.md>) *(SQL Fundamentals)*
- [Deadlocks and Lock Ordering](<../../PostgreSQL/PostgreSQL_Platform/Deadlocks_and_Lock_Ordering.md>) *(PostgreSQL Platform)*

