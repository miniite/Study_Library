---
id: "01_Databases_and_SQL/Transactions_and_Concurrency/Isolation_Levels/Lost_Updates_and_Write_Skew"
level: 4
type: "concept"
topic: "01_Databases_and_SQL/Transactions_and_Concurrency/Isolation_Levels"
tools: ["SQL_Server"]
---

# Lost Updates and Write Skew

**L4 Advanced** · [↑ Isolation Levels](<_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** Two concurrent transactions each make a decision based on data the other is about to change. A **lost update** silently overwrites a write. **Write skew** breaks a business rule even though neither transaction overwrote anything.

### 1. What it is
- **Lost update.** T1 and T2 both *read* the same row (balance = 100), both compute a new value in the application (100 − 30 and 100 − 50), and both *write* it back. The second write wins (balance = 50), and T1's change disappears with no error.
- **Write skew.** T1 and T2 read the **same set** of rows, check a rule ("at least one doctor stays on call"), and then each updates a **different** row. Neither write conflicts with the other at the row level, but together they break the rule (zero doctors on call).

Both are *read-modify-write* races. The difference is **what** gets broken: a value (lost update) or an invariant spanning several rows (write skew).

### 2. Why it matters
- They cause money, inventory and booking errors that **don't raise errors**, so tests and logs rarely catch them.
- They show up only under concurrency (load, retries, double-clicks), which makes them classic production-only bugs.
- Choosing the wrong fix either leaves the bug in place or adds blocking and deadlocks.

### 3. How it works (per isolation level)
| Level / mode | Lost update on read-then-write? | Write skew? | Why |
|---|---|---|---|
| READ COMMITTED (default in most engines) | **Possible** | Possible | Each statement sees the latest committed data; nothing remembers what you read earlier. |
| REPEATABLE READ, locking (SQL Server) | Prevented, but usually as a **deadlock** with one victim | Possible | Shared locks are held on the rows you read, so the other writer blocks. |
| REPEATABLE READ, InnoDB (MySQL) | **Possible** | Possible | Plain SELECT is a snapshot read, but UPDATE writes against the current row, so there's no conflict check. |
| Snapshot / RR in PostgreSQL, SQL Server SNAPSHOT | Prevented (**first committer wins**, the loser gets a serialization or update-conflict error) | **Possible** | The conflict check only compares rows that were *written*, and write-skew transactions write different rows. |
| SERIALIZABLE | Prevented | Prevented | PostgreSQL SSI detects the dangerous read/write dependency. SQL Server uses range locks. MySQL turns reads into locking reads. |

Note: `UPDATE t SET qty = qty - 1 WHERE id = 7` is **atomic** at any level. Lost updates come from reading a value into the application and writing a computed value back.

### 4. Key details to remember
- "First committer wins" protects only rows **both** transactions wrote, which is why snapshot isolation stops lost updates but not write skew.
- SQL Server's `READ_COMMITTED_SNAPSHOT` gives statement-level consistency and does **not** detect update conflicts. `SNAPSHOT` gives transaction-level consistency and raises an update-conflict error.
- Oracle's "SERIALIZABLE" is really snapshot isolation, so write skew is still possible.
- The fixes trade concurrency for safety: atomic SQL < optimistic version check < row lock (`FOR UPDATE`) < SERIALIZABLE.

### 5. Trade-offs and when to use each fix
| Fix | Use when | Watch out for |
|---|---|---|
| Atomic update (`SET x = x - :n WHERE x >= :n`) | The rule fits in one row and one statement | Check the rows-affected count (0 means the rule failed) |
| Optimistic lock (`WHERE id=? AND version=?`) | Conflicts are rare, or the user-facing edit spans time | Handle "0 rows updated" with a retry or user message |
| `SELECT … FOR UPDATE` on what you read | Short transactions with high contention | Lock ordering (deadlocks); it can't lock rows that don't exist yet (phantoms) |
| Materialize the conflict (lock a parent row or a counter row) | Write skew on a *set* of rows | You must add the extra row and lock it consistently |
| SERIALIZABLE + retry loop | Complex invariants that are hard to lock by hand | Aborts under load; **every** caller must retry |
| Constraint (unique, check, exclusion) | The rule can be declared | Not every rule is expressible |

### 6. Common mistakes
- Believing READ COMMITTED prevents lost updates → it doesn't for read-then-write in application code → use an atomic update or a version check.
- Treating snapshot isolation as fully serializable → write skew slips through → lock the rows you read or use SERIALIZABLE.
- Raising the isolation level with no retry logic → serialization failures become user-facing errors → wrap transactions in a bounded retry.
- `FOR UPDATE` on a check that returns **zero rows** → nothing gets locked → lock a parent or anchor row instead.

### 7. 🎯 Interview kit
**30-second answer:** "A lost update happens when two transactions read the same value and both write back a computed result, so one write silently disappears. Write skew is similar, but they update *different* rows after reading a shared set, so no write conflicts and yet an invariant breaks. READ COMMITTED allows both. Snapshot isolation stops lost updates with first-committer-wins but still allows write skew. Only SERIALIZABLE, or explicitly locking what you read, prevents write skew."

**Structure a longer answer:**
1. Clarify: which engine and default level? Is the logic one SQL statement or read-in-app-then-write? How hot is the row?
2. Core idea: draw the T1/T2 timeline (read, read, write, write) and point to where the information is lost.
3. Trade-offs: compare atomic SQL, optimistic versioning, `FOR UPDATE` and SERIALIZABLE on throughput, deadlocks and retries.
4. Close: "For a counter I'd use a single atomic UPDATE. For a multi-row rule like on-call coverage, I'd lock a parent row or run SERIALIZABLE with retries."

**Likely follow-ups:**
- *Why doesn't snapshot isolation stop write skew?* → It only checks write-write conflicts on the same rows.
- *How would you detect this in production?* → Invariant checks or reconciliation queries, conflict and retry metrics, and load tests with concurrent clients.
- *Loyalty points get over-redeemed under load: lost update or write skew?* → One balance row means lost update (use an atomic decrement). Points spread across several ledger rows means write skew (lock the account row).

**Signals interviewers look for:** naming the anomaly correctly, knowing engine-specific behaviour (InnoDB vs PostgreSQL vs SQL Server), and pairing any stricter isolation level with a retry strategy.

### 8. Check yourself
- [ ] Draw the timeline of a lost update and of write skew from memory.
- [ ] Explain why `UPDATE … SET x = x - 1` is safe but `SELECT x` then `UPDATE … SET x = :newx` is not.
- [ ] Pick the fix for the on-call doctors example and justify it.

*See the References section of this note for the books this lesson draws on.*
<!-- explainer:end -->

## Implementations (tools)

- [SQL Server](<../../../TOOLS/SQL_Server/_index.md>)

## Practice questions

- [Intermediate] [How does isolation level impact the correctness of a 'get-or-create' upsert pattern across concurrent requests? Compare INSERT ON CONFLICT versus SELECT-then-INSERT.](<../../../PRACTICE/01_Databases_and_SQL/Transactions_and_Concurrency/Isolation_Levels/Q-30082_How_does_isolation_level_impact_the_correctness_of_a_get-or-.md>)
- [Intermediate] [Two transactions increment the same counter under READ COMMITTED and one update is lost. Walk me through why, and three ways to prevent the lost update.](<../../../PRACTICE/01_Databases_and_SQL/Transactions_and_Concurrency/Isolation_Levels/Q-30269_Two_transactions_increment_the_same_counter_under_READ_COMMI.md>)
- [Intermediate] [What happens to isolation guarantees when you mix autocommit statements with an explicit transaction in the same session? Give a bug this causes.](<../../../PRACTICE/01_Databases_and_SQL/Transactions_and_Concurrency/Isolation_Levels/Q-30272_What_happens_to_isolation_guarantees_when_you_mix_autocommit.md>)
- [Deep] [You need account balance checks to never overdraft under concurrency. Compare SELECT FOR UPDATE at READ COMMITTED versus running at SERIALIZABLE.](<../../../PRACTICE/01_Databases_and_SQL/Transactions_and_Concurrency/Isolation_Levels/Q-30192_You_need_account_balance_checks_to_never_overdraft_under_con.md>)
- [Deep] [Under snapshot isolation, why does the 'first-committer-wins' rule prevent lost updates but not write skew? Illustrate with two updates.](<../../../PRACTICE/01_Databases_and_SQL/Transactions_and_Concurrency/Isolation_Levels/Q-30211_Under_snapshot_isolation_why_does_the_first-committer-wins_r.md>)
- [Deep] [Explain how deadlocks become more likely as you raise isolation level, using a simple two-transaction example that deadlocks at SERIALIZABLE but not READ COMMITTED.](<../../../PRACTICE/01_Databases_and_SQL/Transactions_and_Concurrency/Isolation_Levels/Q-30221_Explain_how_deadlocks_become_more_likely_as_you_raise_isolat.md>)
- [Deep] [What is a 'write-write conflict' under snapshot isolation and how does the database detect and resolve it at commit time?](<../../../PRACTICE/01_Databases_and_SQL/Transactions_and_Concurrency/Isolation_Levels/Q-30224_What_is_a_write-write_conflict_under_snapshot_isolation_and_.md>)
- [Deep] [Describe write skew with a concrete example (e.g. on-call scheduling). Which isolation levels allow it and which prevent it?](<../../../PRACTICE/01_Databases_and_SQL/Transactions_and_Concurrency/Isolation_Levels/Q-30236_Describe_write_skew_with_a_concrete_example_e_g_on-call_sche.md>)

## Related concepts

- [Range Locks and Retries](<../Locking/Range_Locks_and_Retries.md>) *(Locking)*
- [Isolation Anomalies](<../../SQL_Language/SQL_Fundamentals/Isolation_Anomalies.md>) *(SQL Fundamentals)*
- [Read Committed Locking Anomalies](<../../../TOOLS/SQL_Server/SQL_Server_Platform/Read_Committed_Locking_Anomalies.md>) *(SQL Server Platform)*

## References

- *T-SQL Fundamentals (4th ed.)* — Itzik Ben-Gan — Transactions and concurrency
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 10: Transactions, Concurrency, and Isolation Levels › Monitoring Isolation Level Effects
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 10: Transactions, Concurrency, and Isolation Levels › Snapshot Isolation and Row Versioning
- *SQL Server 2025 Query Performance Tuning (7th ed.)* — Grant Fritchey — FULL_BOOK_NEEDS_MANUAL_SPLIT (cont.) › Chapter 15 Reduce Query Resource Use
- *SQL Server 2025 Query Performance Tuning (7th ed.)* — Grant Fritchey — FULL_BOOK_NEEDS_MANUAL_SPLIT (cont.) › Chapter 16 Blocking And Blocked Processes
- *Designing Data-Intensive Applications* — Martin Kleppmann — Weak Isolation Levels › No dirty writes
- *Designing Data-Intensive Applications* — Martin Kleppmann — Snapshot Isolation and Repeatable Read › Implementing snapshot isolation
- *Designing Data-Intensive Applications* — Martin Kleppmann — Write Skew and Phantoms › Characterizing write skew
- *100 SQL Server Mistakes and How to Avoid Them* — Peter A. Carter — Optimization › 10.12 #65 Not considering optimistic isolation levels
- *SQL Server Tutorial* (YouTube series) — Part 77: Difference between snapshot isolation and read committed snapshot ([watch](https://www.youtube.com/watch?v=-fLEztcGPUA))

