---
id: "01_Databases_and_SQL/Transactions_and_Concurrency/Deadlocks/Deadlock_Retry_Strategy"
level: 4
type: "concept"
topic: "01_Databases_and_SQL/Transactions_and_Concurrency/Deadlocks"
tools: []
---

# Deadlock Retry Strategy

**L4 Advanced** · [↑ Deadlocks](<_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** Deadlocks cannot be fully eliminated in a busy lock-based system, so every transaction that can be a victim must be written so the whole unit can be safely re-run, with bounded attempts, backoff and no duplicated side effects.

### 1. The big picture
You reduce deadlocks by design (consistent lock order, short transactions, good indexes, versioned reads), but a residual rate always remains under concurrency. The engine's answer is to roll back one session and return an error. If the application surfaces that error to the user, a harmless transient becomes a failure. A **retry strategy** turns it back into a short delay.

Retrying is not just a loop around a call. It has requirements: the unit being retried must be the *entire* transaction, the operation must be idempotent or free of external side effects, attempts must be bounded and spaced, and retries at several layers must not multiply (retry storms). It also has a policy side: when a deadlock is rare enough, "retry and move on" is an acceptable, communicated decision; when frequent, retry only masks a design problem.

### 2. Core concepts and vocabulary
- **Deadlock victim error** — SQL Server 1205; Oracle ORA-00060; PostgreSQL SQLSTATE 40P01 (and 40001 serialization failure); MySQL 1213.
- **Transient error** — failure expected to succeed on a later attempt.
- **Idempotent operation** — running it twice has the same effect as once.
- **Exponential backoff** — wait grows (e.g., 50 ms, 100 ms, 200 ms…) between attempts.
- **Jitter** — random variation added to backoff so retriers do not collide in lockstep.
- **Retry budget** — maximum attempts/total time allowed.
- **Retry storm** — nested or concurrent retries amplifying load during a fault.
- **Circuit breaker** — stops calling a failing dependency for a period.
- **Outbox pattern** — record external effects in the same transaction, send them after commit.
- **Idempotency key** — unique request id that makes a replay harmless.

### 3. How it works, step by step
A robust retry wrapper:

1. Define the **unit of work** as a function containing `BEGIN TRAN ... COMMIT` and nothing outside it that changes state (no emails, no HTTP calls inside).
2. Execute it; on success return.
3. On exception, classify it: is it a deadlock/serialization failure (or lock timeout, if the policy says so)? Only these are retried; constraint violations, syntax errors, permissions are not.
4. Ensure the failed transaction has been rolled back (the engine already did for deadlock; always check connection state before reuse).
5. Wait: `delay = base * 2^attempt + random(0, jitter)`.
6. Re-run the entire unit **from the start**, re-reading any data it depends on (values read earlier may be stale).
7. Stop after N attempts (commonly 3-5) and surface the error with context (correlation id, attempt count).
8. Log each deadlock retry so frequency stays visible.

When to surface rather than retry: the operation is not idempotent and you cannot make it so, the user's input may have been invalidated by the other transaction (they should see new data), or the time budget is exhausted.

### 4. Worked example
Server side (T-SQL), bounded loop:

```sql
DECLARE @retry int = 3;
WHILE @retry > 0
BEGIN
    BEGIN TRY
        BEGIN TRAN;
            UPDATE Inventory SET Qty = Qty - 1 WHERE ProductID = 5 AND Qty > 0;
            IF @@ROWCOUNT = 0 THROW 50002, 'Out of stock', 1;
            INSERT OrderLine(OrderID, ProductID, Qty) VALUES (@OrderID, 5, 1);
        COMMIT;
        BREAK;                                   -- success
    END TRY
    BEGIN CATCH
        IF @@TRANCOUNT > 0 ROLLBACK;
        IF ERROR_NUMBER() = 1205 AND @retry > 1  -- deadlock and attempts left
        BEGIN
            SET @retry -= 1;
            WAITFOR DELAY '00:00:00.100';         -- crude backoff
        END
        ELSE THROW;                               -- anything else, or out of attempts
    END CATCH
END
```

Application side (pseudo-code with jitter):

```text
for attempt in 1..5:
    try:
        run_transaction(work)          # whole unit, new connection state
        return
    except DeadlockOrSerializationFailure:
        if attempt == 5: raise
        sleep(0.05 * 2**attempt + random(0, 0.05))
```
Note `work` must re-read inputs each time and must write using an idempotency key if it can be replayed by an upstream retry.

### 5. Under the hood
- When the deadlock monitor picks the victim, the engine rolls back that transaction itself (the session's `@@TRANCOUNT` becomes 0 or the transaction is doomed); the client simply sees the error. Rolling back may take time proportional to the work already done, which is why retried long transactions are expensive.
- Backoff without jitter makes every victim retry at the same instant and often recreate the same cycle; jitter breaks the symmetry.
- Retries interact with `SET XACT_ABORT` and with nested transactions: retrying only an inner part while an outer transaction continues is invalid because the outer transaction has already been rolled back. Always retry at the outermost boundary.
- Multi-layer retries: if the service retries 3 times and the caller retries 3 times and a gateway retries 3 times, one fault produces 27 attempts. Retries multiply under load, turning a brief deadlock spike into overload.
- Distributed systems use idempotent consumers (processed-message table with unique id) so at-least-once delivery and retries do not double-apply.

### 6. Performance and design implications
- Retries add latency only on the rare path; watch the *rate*: more than a few deadlocks per minute indicates a pattern to fix, not tolerate.
- Cap total retry time under the caller's timeout; otherwise requests pile up.
- Hot rows (leaderboards, counters) deadlock and retry endlessly; redesign: sharded counters, append-only events with periodic aggregation, queues/single writer, or in-memory/Redis counters flushed in batches.
- Keep retried transactions small and fast; their cost is paid again every attempt.
- Metrics to expose: deadlocks/sec, retry attempts histogram, retry exhaustion count, latency percentiles.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `ERROR_NUMBER() = 1205` in CATCH | Detects deadlock victim | T-SQL retry loop |
| `THROW` | Rethrows the original error | After attempts exhausted |
| `WAITFOR DELAY` | Server-side pause | Simple backoff in procedures |
| `SET DEADLOCK_PRIORITY LOW` | Make a job the preferred victim | Background work |
| `SET LOCK_TIMEOUT` | Bound lock waits | Different class of transient failure |
| Idempotency key + unique constraint | Make replays harmless | Payments, webhooks |
| Polly / resilience4j / tenacity | Library retry policies | Application tier |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| Automatic retry | Operation has non-idempotent external effects you cannot gate |
| Few attempts with jitter | High contention hot row → fix design |
| Retry in one layer only | Several layers each retry → storm |
| Surface to user | Their decision depends on data that just changed |
| Accept rare deadlock + retry | Frequency is rising or victims are important jobs |
| Make batch the victim | Batch is the only path to meeting a deadline |

### 9. Common mistakes
- Retrying only the failing statement → the rest of the transaction was rolled back → retry the whole unit.
- Retrying inside an outer transaction → outer is already doomed → retry at the outermost boundary.
- Unbounded loops, no backoff → hammering, livelock → cap attempts, add jittered backoff.
- Side effects inside the transaction (email, payment API) → duplicates on retry → move after commit via outbox or use idempotency keys.
- Treating every error as retryable → repeats constraint violations → whitelist transient errors.
- Not logging retries → deadlock problem invisible → count and alert.
- Stacked retries across service, upstream and client → retry storm → one retry layer, propagate a deadline and budget.

### 10. 🎯 Interview kit
**30-second answer:** Deadlocks are inevitable under concurrency, so I wrap the whole transaction in a bounded retry that re-runs only on deadlock or serialization errors, with exponential backoff and jitter. The operation must be idempotent, with side effects moved outside the transaction, and I keep one retry layer to avoid storms.

**Structure a longer answer:** clarify → core mechanism → trade-offs → close with a real scenario.
Say what is retried and when, show the loop, then idempotency/outbox, then layering and metrics, then when to fix the cause instead.

**Likely follow-ups:**
- *Retry or surface?* → retry transient and idempotent; surface when user context changed or retries exhausted.
- *What must retry never do?* → run inside a half-dead outer transaction, repeat non-idempotent effects, or loop unbounded.
- *Retry storm between microservice and upstream?* → single retry owner, deadline propagation, budgets, circuit breaker, jitter.
- *Leaderboard hot rows?* → partition keys, append events, aggregate asynchronously, cache.
- *Confirm a deadlock fix?* → canary/shadow traffic, compare deadlock counters and retry rates before full rollout.
- *Oracle?* → catch ORA-00060, rollback, re-run block.

**Signals interviewers look for:** you retry the whole transaction, protect against duplicates and amplification, and still treat the deadlock source as something to fix.

### 11. Check yourself
- [ ] Write a retry loop that handles 1205 correctly.
- [ ] Explain why jitter matters.
- [ ] List three side effects that must not be inside a retried transaction.
- [ ] Describe a retry storm and its mitigation.
- [ ] Decide when to leave a rare deadlock in place and how to justify it.
<!-- explainer:end -->

## Practice questions

- [Intermediate] [Describe how you'd implement a robust application-side deadlock retry with exponential backoff and what it must never do.](<../../../PRACTICE/01_Databases_and_SQL/Transactions_and_Concurrency/Deadlocks/Q-23051_Describe_how_you_d_implement_a_robust_application-side_deadl.md>)
- [Intermediate] [How do you decide whether to retry a deadlocked transaction automatically or surface the error to the user?](<../../../PRACTICE/01_Databases_and_SQL/Transactions_and_Concurrency/Deadlocks/Q-23090_How_do_you_decide_whether_to_retry_a_deadlocked_transaction_.md>)
- [Intermediate] [When is it acceptable to leave a rare deadlock in place and simply rely on retry, and how do you communicate that decision?](<../../../PRACTICE/01_Databases_and_SQL/Transactions_and_Concurrency/Deadlocks/Q-23130_When_is_it_acceptable_to_leave_a_rare_deadlock_in_place_and_.md>)
- [Intermediate] [How would you design a rollback-and-retry wrapper in PL/SQL that safely re-runs the failed transaction on ORA-00060?](<../../../PRACTICE/01_Databases_and_SQL/Transactions_and_Concurrency/Deadlocks/Q-23143_How_would_you_design_a_rollback-and-retry_wrapper_in_PL_SQL_.md>)
- [Deep] [How would you architect a leaderboard update that many users hit simultaneously to avoid deadlocks and hotspots?](<../../../PRACTICE/01_Databases_and_SQL/Transactions_and_Concurrency/Deadlocks/Q-23096_How_would_you_architect_a_leaderboard_update_that_many_users.md>)
- [Deep] [A microservice retries on deadlock but the upstream also retries, causing a retry storm; how do you make the retry strategy safe end-to-end?](<../../../PRACTICE/01_Databases_and_SQL/Transactions_and_Concurrency/Deadlocks/Q-23102_A_microservice_retries_on_deadlock_but_the_upstream_also_ret.md>)
- [Expert] [How would you use a canary or shadow deployment to confirm a deadlock fix under real traffic before full rollout?](<../../../PRACTICE/01_Databases_and_SQL/Transactions_and_Concurrency/Deadlocks/Q-23131_How_would_you_use_a_canary_or_shadow_deployment_to_confirm_a.md>)

## Related concepts

- [Contention and Slow-burn Incidents](<../../../07_Distributed_Systems_and_Design/Distributed_Systems/Concurrency_Patterns/Contention_and_Slow-burn_Incidents.md>) *(Concurrency Patterns)*
- [Row Locks Two-phase Locking and Lock Footprint](<../Locking/Row_Locks_Two-phase_Locking_and_Lock_Footprint.md>) *(Locking)*
- [Lock Granularity Design](<../../../07_Distributed_Systems_and_Design/System_Design/Low_Level_Design/Compensating_Transactions/Lock_Granularity_Design.md>) *(Low Level Design)*

## References

- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 10: Transactions, Concurrency, and Isolation Levels › Deadlock Retry Logic
- *SQL Server Tutorial* (YouTube series) — Part 85: Retry logic for deadlock exceptions ([watch](https://www.youtube.com/watch?v=xWRsoD2twxI))

