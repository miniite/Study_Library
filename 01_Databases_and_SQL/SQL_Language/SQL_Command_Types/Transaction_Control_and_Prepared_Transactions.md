---
id: "01_Databases_and_SQL/SQL_Language/SQL_Command_Types/Transaction_Control_and_Prepared_Transactions"
level: 4
type: "concept"
topic: "01_Databases_and_SQL/SQL_Language/SQL_Command_Types"
tools: ["MySQL"]
---

# Transaction Control and Prepared Transactions

**L4 Advanced** · [↑ SQL Command Types](<_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** Transaction control (TCL) lets you group several statements so they succeed or fail as one unit — `BEGIN`, `COMMIT`, `ROLLBACK`, and `SAVEPOINT` for partial undo — and *prepared* (two-phase) transactions let that all-or-nothing promise span more than one database.

### 1. The big picture
Business facts rarely map to a single statement. "Move 100 from account A to account B" is two `UPDATE`s; "place an order" might be an `INSERT` into `orders`, several into `order_lines`, and a stock decrement. If the system dies between statement one and statement two, you must not be left with money that vanished. A **transaction** is the unit that guarantees **atomicity** (all or nothing) — the "A" in ACID — and **durability** once it is committed.

*Transaction control language* is the small set of commands that define those boundaries. Fundamentals of Data Engineering describes TCL as the commands that let you "define commit checkpoints" and the "conditions when actions will be rolled back". It sits next to DDL, DML and DCL, and it connects upward to **isolation levels and locking** (what other sessions see while your transaction is open) and downward to the **write-ahead log** (how COMMIT survives a crash).

Data-engineering pipelines care about the same idea at a different scale: Data Engineering Design Patterns notes that a pipeline transaction can be *local* to each task or span the *whole job*; with Spark + Delta Lake the transaction commits when a new entry lands in the commit log, and files written before a failure remain as orphans that readers never see. That is the same all-or-nothing semantics, implemented with a log instead of a lock manager.

### 2. Core concepts and vocabulary
- **Transaction** — a sequence of statements treated as one atomic, isolated, durable unit.
- **Autocommit** — the default in most clients: every statement is its own transaction, committed immediately.
- **Explicit transaction** — opened with `BEGIN` / `BEGIN TRAN` / `START TRANSACTION`, ended with `COMMIT` or `ROLLBACK`.
- **`COMMIT`** — makes all changes permanent and visible to others; releases locks.
- **`ROLLBACK`** — undoes everything since `BEGIN` (or since a savepoint) and releases locks.
- **Savepoint** — a named marker inside a transaction that you can roll back *to* without abandoning earlier work.
- **Implicit transaction** — a mode (e.g. SQL Server `SET IMPLICIT_TRANSACTIONS ON`) where the first DML silently opens a transaction that you must commit yourself.
- **Statement-level vs transaction-level rollback** — whether an error undoes only the failing statement or dooms the whole transaction (engine-specific).
- **Prepared transaction (two-phase commit, 2PC)** — a transaction that has completed phase 1 ("prepare": all work is durable and guaranteed committable) and waits for an external coordinator to say commit or abort.
- **Outbox pattern** — writing an event row in the same transaction as the business change, so both commit or neither does.

### 3. How it works, step by step
A normal transaction:

1. **Begin.** The engine assigns a transaction id and starts tracking locks / row versions.
2. **Do work.** Each `INSERT/UPDATE/DELETE` writes log records first (write-ahead logging) and modifies pages in memory; locks or MVCC versions keep other sessions from seeing half-done work.
3. **Savepoint (optional).** `SAVEPOINT s1;` remembers a position in the log.
4. **Decide.**
   - `COMMIT`: the engine forces the log record for the commit to durable storage, *then* reports success. Data pages can be written later; the log is what makes it durable.
   - `ROLLBACK`: the engine uses the log to undo each change in reverse order, then releases locks.
   - `ROLLBACK TO s1`: undo only the changes after `s1`; the transaction stays open.
5. **Release.** Locks are freed, row versions become visible/garbage-collectable.

The book's tiny example from *Ultimate T-SQL* shows the testing habit:

```sql
BEGIN TRAN;
DELETE FROM Employees WHERE EmployeeID = 999;
-- inspect @@ROWCOUNT / SELECT to verify
ROLLBACK TRAN;      -- undo while testing
-- COMMIT TRAN;     -- only when verified
```

Beginning T-SQL adds a subtlety: SQL Server allows *nested* `BEGIN TRAN`, but only the outermost `COMMIT` truly commits and a single `ROLLBACK` rolls back *everything*. `@@TRANCOUNT` tells you the depth. Nested "transactions" are only a counter, not independent units — use savepoints (`SAVE TRAN name` / `ROLLBACK TRAN name`) for partial undo.

### 4. Worked example
**Multi-step import with partial rollback (PostgreSQL):**

```sql
BEGIN;
INSERT INTO import_batch(id, started_at) VALUES (42, now());

SAVEPOINT customers_loaded;
INSERT INTO customers SELECT * FROM staging_customers;      -- step 1

SAVEPOINT orders_loaded;
INSERT INTO orders SELECT * FROM staging_orders;             -- step 2 fails: FK violation
-- ERROR: insert or update on table "orders" violates foreign key constraint

ROLLBACK TO SAVEPOINT customers_loaded;                      -- throws away step 2, keeps step 1
-- fix or skip bad rows, retry just the orders step
INSERT INTO orders SELECT * FROM staging_orders WHERE customer_id IN (SELECT id FROM customers);

COMMIT;
```

What happens: without the savepoint, the error would put the PostgreSQL transaction into an **aborted** state — "current transaction is aborted, commands ignored until end of transaction block" — and every further statement would fail until `ROLLBACK`. `ROLLBACK TO SAVEPOINT` returns it to a usable state. That message is the classic signal that you need savepoints or a retry around a unit of work.

**Outbox pattern (atomic row + event):**

```sql
BEGIN;
INSERT INTO orders(id, customer_id, total) VALUES (1001, 7, 59.90);
INSERT INTO outbox(aggregate_id, type, payload)
VALUES (1001, 'OrderPlaced', '{"order_id":1001,"total":59.90}');
COMMIT;      -- both rows or none
-- a separate relay process reads outbox and publishes to the queue, then marks rows sent
```

### 5. Under the hood
- **Durability and COMMIT.** Engines use *write-ahead logging*: a change is described in the log, and the log is flushed before the commit is acknowledged. After a crash, recovery **redoes** committed work and **undoes** uncommitted work from the log. This is why `COMMIT` can be fast (sequential log write) even though data pages are written lazily.
- **Synchronous vs asynchronous commit.** Synchronous commit waits for the log flush (and possibly a replica acknowledgment). Asynchronous commit (e.g. PostgreSQL `synchronous_commit = off`) returns earlier and may lose the last few transactions on a crash — but never corrupts. Use synchronous for payments, asynchronous for analytics-style writes where losing a second of data is acceptable.
- **Error semantics differ.** PostgreSQL aborts the whole transaction on any error (unless a savepoint is used). SQL Server, by default, rolls back only the failing statement for most errors (constraint violations), but severe errors and `SET XACT_ABORT ON` roll back everything — and you should almost always set `XACT_ABORT ON` in procedures that open transactions.
- **Savepoints are not free.** Each savepoint holds a subtransaction id and undo state; thousands of savepoints inside a long transaction cost memory and slow visibility checks. They also don't release locks acquired after them on rollback in all engines — the work is undone, but you stay in the same transaction.
- **Long transactions** pin old row versions (blocking `VACUUM` in PostgreSQL) and prevent log truncation (SQL Server, Oracle). An open transaction with a savepoint is still an open transaction — the remedy is to commit or roll back, not to "release" the savepoint.
- **Prepared transactions (2PC).** `PREPARE TRANSACTION 'gid'` in PostgreSQL (or the XA protocol in the Java/Oracle/MySQL world, MSDTC in SQL Server) writes the transaction's state durably and detaches it from the session. A transaction manager then calls `COMMIT PREPARED 'gid'` or `ROLLBACK PREPARED 'gid'`. Prepared transactions **keep their locks** across restarts, so an orphan can block a table indefinitely and holds back vacuum. They are disabled by default (`max_prepared_transactions = 0`) for exactly that reason.

### 6. Performance and design implications
- **Autocommit hurts throughput.** Each statement pays a log flush. Batching 1,000 inserts in one transaction can be 10–100× faster than 1,000 autocommits. But keep batches bounded (thousands, not millions) so locks and undo space stay manageable.
- **Keep transactions short.** Do not wait for user input or call external services inside one.
- **Lock ordering** prevents deadlocks: always touch rows/tables in the same order (e.g. by ascending id). When two sessions do `UPDATE ... WHERE balance > 0` in opposite order, the lock graph cycles and one is killed — retry the victim.
- **`SELECT ... FOR UPDATE` / `SKIP LOCKED`** participate in the same transaction: the first locks the selected rows until commit; `SKIP LOCKED` lets workers pull different rows from a queue table without blocking each other.
- **Idempotency matters on retry.** If a commit succeeds but the client never hears back, a naive retry can double-apply. Design writes with natural keys or idempotency keys (see *Upserts*).
- **Client disconnects.** If a client drops mid-transaction, the server rolls it back once it notices (connection timeout, TCP keepalive). Locks stay until then — one reason to set statement and idle-in-transaction timeouts.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `BEGIN` / `START TRANSACTION` / `BEGIN TRAN` | Opens explicit transaction | Any multi-statement change |
| `COMMIT` / `COMMIT TRAN` | Makes work permanent | End of unit of work |
| `ROLLBACK` | Undoes since begin | On error or test runs |
| `SAVEPOINT s` / `SAVE TRAN s` | Partial-undo marker | Multi-step imports, retries |
| `ROLLBACK TO [SAVEPOINT] s` | Undo back to marker, stay in txn | Recover from a failed sub-step |
| `RELEASE SAVEPOINT s` | Discards the marker, *keeps* work | Free resources; does not undo anything |
| `SET XACT_ABORT ON` (SQL Server) | Any runtime error rolls back all | Procedures with transactions |
| `SET AUTOCOMMIT=0` (MySQL) / `IMPLICIT_TRANSACTIONS` | Changes default behaviour | Understand drivers/ORMs |
| `PREPARE TRANSACTION 'g'`, `COMMIT PREPARED`, `ROLLBACK PREPARED` | 2PC participant commands | Distributed coordinators only |
| `idle_in_transaction_session_timeout` | Kills forgotten open transactions | Production safety net |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| One explicit transaction around a business operation | Wrapping unrelated work: it extends lock time |
| Savepoints to retry a failing sub-step | Hundreds of savepoints in a loop; consider batching or per-row `ON CONFLICT` instead |
| Batched commits (every N rows) for bulk loads | Needing all-or-nothing for the whole load — then use staging + a single swap |
| Synchronous commit for money | Analytics ingestion where throughput is the goal |
| Outbox + relay for DB-and-message atomicity | Trying to include a message broker in the DB transaction by hand |
| 2PC when two resource managers *must* agree | Everywhere else: prefer sagas/outbox, because prepared transactions hold locks and block recovery |

### 9. Common mistakes
- Forgetting to `COMMIT` in an explicit transaction → session holds locks; others block → add timeouts and use `try/finally` in code.
- Confusing `RELEASE SAVEPOINT` with rollback → work is not undone → use `ROLLBACK TO`.
- Believing SQL Server nested `BEGIN TRAN` makes independent transactions → one `ROLLBACK` kills them all → use savepoints.
- Leaving autocommit on for a 100k-row load → partial data after failure, plus slow → one transaction (or chunked, idempotent batches).
- Committing per row in an import and failing halfway → partial data → wrap in one transaction, or load to staging and swap.
- Leaving prepared transactions orphaned → permanent locks, vacuum blocked → monitor `pg_prepared_xacts` and resolve them.
- Assuming one service's `COMMIT` is instantly visible to another → isolation level or replica lag hides it → read from the primary or wait for the replica.

### 10. 🎯 Interview kit
**30-second answer:** "A transaction groups statements so they're atomic and durable: `BEGIN`, then `COMMIT` to persist or `ROLLBACK` to undo. Savepoints give partial rollback inside one transaction. Durability comes from the write-ahead log flushed at commit; isolation and locks decide what others see."

**Structure a longer answer:** clarify (single DB? engine?) → mechanism (log, locks/MVCC, commit protocol) → trade-offs (batch size, sync vs async commit, 2PC cost) → a scenario like the import above.

**Likely follow-ups:**
- *What if you never commit?* → changes are invisible to others, locks held, rolled back on disconnect; long-open transaction blocks cleanup.
- *What happens on client disconnect?* → server rolls back and releases locks once it notices.
- *Statement-level vs transaction-level rollback?* → engine-specific; PostgreSQL aborts the whole transaction on error, SQL Server often only the statement unless `XACT_ABORT`.
- *How do you guarantee durability after a crash?* → write-ahead log flushed at commit; redo/undo recovery.
- *Sync vs async commit?* → latency vs risk of losing last commits; payments sync, analytics async.
- *How do you do atomic DB write + event?* → outbox table in the same transaction, relay publishes.
- *When is 2PC justified?* → cross-resource atomicity; costs: blocking coordinator, locks held.

**Signals interviewers look for:** you know autocommit, can explain savepoints vs rollback, can tie `COMMIT` to the log, and know the hazards of long-running or prepared transactions.

### 11. Check yourself
- [ ] Draw the lifecycle of a transaction from `BEGIN` to `COMMIT`, marking where the log is flushed.
- [ ] Rewrite a loop of per-row commits as a safe all-or-nothing batch.
- [ ] Explain "current transaction is aborted" and show how a savepoint recovers.
- [ ] Contrast `ROLLBACK TO SAVEPOINT` and `RELEASE SAVEPOINT`.
- [ ] State two reasons prepared transactions are risky if left open.
<!-- explainer:end -->

## Implementations (tools)

- [MySQL](<../../../TOOLS/MySQL/_index.md>)

## Practice questions

- [Basic] [What does the COMMIT command do in a transaction?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Command_Types/Q-22903_What_does_the_COMMIT_command_do_in_a_transaction.md>)
- [Intermediate] [After a series of INSERTs, if you never COMMIT, what happens to the data?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Command_Types/Q-22983_After_a_series_of_INSERTs_if_you_never_COMMIT_what_happens_t.md>)
- [Intermediate] [An application relies on autocommit and each INSERT is its own transaction, hurting throughput. How do you batch commits safely without risking partial loss?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Command_Types/Q-22875_An_application_relies_on_autocommit_and_each_INSERT_is_its_o.md>)
- [Deep] [What is the default transaction behavior called autocommit and how does it affect DML?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Command_Types/Q-22984_What_is_the_default_transaction_behavior_called_autocommit_a.md>)
- [Deep] [What is the effect of a COMMIT on locks held by a transaction?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Command_Types/Q-23017_What_is_the_effect_of_a_COMMIT_on_locks_held_by_a_transactio.md>)
- [Deep] [How do you make a data-backfill UPDATE resumable so a mid-run failure doesn't force you to restart from zero?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Command_Types/Q-22896_How_do_you_make_a_data-backfill_UPDATE_resumable_so_a_mid-ru.md>)
- [Deep] [Explain how autocommit interacts with connection pools and why a leaked open transaction can hold locks for minutes. How do you detect it?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Command_Types/Q-22918_Explain_how_autocommit_interacts_with_connection_pools_and_w.md>)
- [Deep] [Explain why Oracle's FLASHBACK or Postgres PITR is your safety net for irreversible DML, and how you'd rehearse the recovery.](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Command_Types/Q-22945_Explain_why_Oracle_s_FLASHBACK_or_Postgres_PITR_is_your_safe.md>)

## Related concepts

- [Commit Rollback and Savepoints](<../../Transactions_and_Concurrency/ACID_Transactions/Commit_Rollback_and_Savepoints.md>) *(ACID Transactions)*
- [Read Committed Locking Anomalies](<../../../TOOLS/SQL_Server/SQL_Server_Platform/Read_Committed_Locking_Anomalies.md>) *(SQL Server Platform)*
- [Lock Holding and Optimistic Concurrency](<../../Transactions_and_Concurrency/ACID_Transactions/Lock_Holding_and_Optimistic_Concurrency.md>) *(ACID Transactions)*

## References

- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 1: Getting Started with T-SQL › Use Transactions for Safe Changes
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 5: Views, Functions, and Stored Procedures › Transactions and Error Handling
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 10: Transactions, Concurrency, and Isolation Levels › BEGIN TRANSACTION
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 10: Transactions, Concurrency, and Isolation Levels › COMMIT TRANSACTION
- *Think Distributed Systems* — Dominik Tornow — Distributed transactions › Phase 1: Prepare
- *Fundamentals of Data Engineering* — Joe Reis, Matt Housley — Chapter 8. Queries, Modeling, and Transformation › TCL
- *Beginning T-SQL* — Kathi Kellenberger, Lee Everest — Managing Transactions › Rolling Back a Transaction
- *Data Engineering Design Patterns* — Bartosz Konieczny — Idempotency Design Patterns › Consequences

