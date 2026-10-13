---
id: "01_Databases_and_SQL/Database_Design/Normalization/Denormalization_and_Locking"
level: 5
type: "deep-dive"
topic: "01_Databases_and_SQL/Database_Design/Normalization"
tools: []
---

# Denormalization and Locking

**L5 Expert** · [↑ Normalization](<../_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** Denormalized columns turn many independent writes into contended writes on one row, so correctness depends on the isolation level and update pattern — read-modify-write loses updates, atomic in-place updates do not, and hot counters need sharding or async aggregation.

### 1. The big picture
Normalized data spreads writes across many rows: each comment is its own row, each order line its own row. The moment you add `post.comment_count` or `account.balance_cached`, every writer for that parent now converges on **one row**. That single row becomes a lock queue (a hot spot) and a source of lost-update bugs if the application reads the value, adds one, and writes it back.

This lesson explains how the concurrency model of the database interacts with denormalized aggregates: what each isolation level guarantees, which update patterns are safe, how to avoid hot-row contention, and how idempotency interacts with replays.

### 2. Core concepts and vocabulary
- **Lost update** — two transactions read the same value, both write based on it; one write overwrites the other.
- **Read-modify-write** — application reads a value, computes, writes it back (unsafe without protection).
- **Atomic in-place update** — `SET n = n + 1` evaluated by the engine under a row lock (safe).
- **Row lock / hot row** — an exclusive lock held until commit; many writers serialize on it.
- **Isolation level** — READ COMMITTED, REPEATABLE READ/SNAPSHOT, SERIALIZABLE; each prevents a different set of anomalies.
- **Optimistic concurrency control (OCC)** — version column checked at write time; retry on conflict.
- **Pessimistic locking** — `SELECT ... FOR UPDATE` / `UPDLOCK` to serialize before changing.
- **Sharded counter** — split one counter into N rows and sum them when reading.
- **Idempotency key** — identifier used to apply an event at most once even if delivered multiple times.

### 3. How it works, step by step
**Why read-modify-write loses updates under READ COMMITTED**
1. T1 reads `count = 10`. T2 reads `count = 10` (reads don't block reads).
2. T1 writes 11 and commits.
3. T2 writes 11 (based on its stale 10) and commits.
4. Two comments, count went up by one. READ COMMITTED permits this because each statement sees only committed data, but T2's earlier read is not protected.

**Safe patterns**
1. *Atomic update:* `UPDATE post SET comment_count = comment_count + 1 WHERE post_id = ?` — the engine takes the row lock, reads the latest committed value inside the statement, writes. No lost update under READ COMMITTED.
2. *Pessimistic:* `SELECT ... FOR UPDATE` then update — second transaction waits.
3. *Optimistic:* `UPDATE ... SET n = ?, version = version + 1 WHERE id = ? AND version = ?`; zero rows affected → retry.
4. *Higher isolation:* REPEATABLE READ (PostgreSQL) raises a serialization failure on the conflicting write; application must retry. SERIALIZABLE similarly.
5. *Compute instead of store:* `COUNT(*)` on demand with a good index.

### 4. Worked example
Session-level reproduction in PostgreSQL (two psql sessions, READ COMMITTED):
```sql
-- setup
CREATE TABLE post (post_id int PRIMARY KEY, comment_count int NOT NULL);
INSERT INTO post VALUES (1, 0);

-- Session A                          -- Session B
BEGIN;                                BEGIN;
SELECT comment_count FROM post;       SELECT comment_count FROM post;   -- both see 0
UPDATE post SET comment_count = 1;    
COMMIT;                               UPDATE post SET comment_count = 1; -- app computed 0+1
                                      COMMIT;     -- final = 1, expected 2  (lost update)

-- safe version of both sessions:
UPDATE post SET comment_count = comment_count + 1 WHERE post_id = 1;
```
Under the atomic form, B waits for A's row lock; when A commits, B re-evaluates the expression against the new row version (PostgreSQL's "EvalPlanQual") and the result is 2.

**Hot-row redesign — sharded counter**
```sql
CREATE TABLE post_counter (post_id int, shard smallint, n bigint NOT NULL DEFAULT 0,
                           PRIMARY KEY (post_id, shard));
-- writer picks a random shard 0..15
UPDATE post_counter SET n = n + 1 WHERE post_id = 1 AND shard = floor(random()*16)::int;
-- reader
SELECT SUM(n) FROM post_counter WHERE post_id = 1;
```
Contention drops about 16×. Another option is append-only "delta" rows folded periodically into the parent.

### 5. Under the hood
- **Locking engines (SQL Server default READ COMMITTED):** UPDATE takes an exclusive (X) lock on the row (or key) and holds it to commit; readers under READ COMMITTED (non-snapshot) take short shared locks, so a plain SELECT can block behind an uncommitted UPDATE. `READ_COMMITTED_SNAPSHOT` makes reads use row versions instead.
- **MVCC engines (PostgreSQL, Oracle, MySQL InnoDB):** readers don't block writers; writers still serialize on the same row. PostgreSQL creates a new tuple version per update, so heavy counter updates generate dead tuples and bloat — HOT updates and aggressive autovacuum help; counter columns should not be indexed.
- **Phantoms and aggregates:** a denormalized aggregate that depends on a *set* of rows (e.g. "total of all lines") can be wrong under REPEATABLE READ if a concurrent insert changes the set; SERIALIZABLE or locking the parent row closes it.
- **Deadlocks:** two transactions updating the parent and child rows in opposite orders deadlock. Fix with a consistent lock order (parent first), short transactions, and retry logic.
- **Replays:** at-least-once delivery (queues, stream processors) re-applies events; `n = n + 1` is not idempotent. Store the processed event id (unique constraint) in the same transaction as the increment, or compute the aggregate from the event log (idempotent upsert of a window total).
- **Distributed:** across shards or regions, a single synchronous counter becomes a coordination bottleneck; CRDT counters or async aggregation with eventual consistency are used.

### 6. Performance and design implications
- Write throughput on one row is bounded by lock-hold time ≈ transaction duration. Keep transactions short; do external calls outside them.
- Reads of a stored count are O(1); `COUNT(*)` is O(rows) but index-only and often fine up to millions with caching.
- Trade freshness for contention: async/batched counters (queue, micro-batch) turn thousands of row updates into one.
- Diagnose: PostgreSQL `pg_locks`, `pg_stat_activity` wait_event `Lock:transactionid`; SQL Server `sys.dm_tran_locks`, `sys.dm_os_waiting_tasks`, LCK_M_X waits, deadlock graph via Extended Events.
- Retries must be bounded and idempotent; surface serialization failures (`40001`) to a retry wrapper.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `UPDATE t SET n = n + 1` | atomic increment | default for counters |
| `SELECT ... FOR UPDATE` (PG/MySQL/Oracle) / `WITH (UPDLOCK, ROWLOCK)` (SQL Server) | lock before changing | read-then-decide logic |
| `SET TRANSACTION ISOLATION LEVEL ...` | choose isolation | when anomalies beyond lost update matter |
| `READ_COMMITTED_SNAPSHOT ON` | MVCC reads in SQL Server | reduce reader/writer blocking |
| version column + conditional UPDATE | optimistic concurrency | low contention, interactive edits |
| `INSERT ... ON CONFLICT DO NOTHING` + event table | idempotent apply | replays |
| `SKIP LOCKED` | queue-style workers | work distribution without contention |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| Atomic increment | the new value depends on other rows (use locking/serializable) |
| Optimistic version check | conflicts are frequent — retry storms |
| Pessimistic lock | long user think time while locked |
| Sharded counter | you need exact transactional reads cheaply |
| Async aggregate | read-your-writes is required |
| On-demand COUNT | table is huge and called per request |

### 9. Common mistakes
- Read in app, add one, write back → lost updates → atomic SQL or version check.
- Increment inside a long transaction → everyone queues → commit early.
- Counting events at-least-once without idempotency → inflated numbers → event id unique constraint.
- Updating parent then child in one path and reverse in another → deadlocks → fixed order.
- Indexing a frequently updated counter → every increment rewrites the index (and defeats HOT) → don't index it.
- Assuming SERIALIZABLE means no retries → serialization failures still happen → retry loop.

### 10. 🎯 Interview kit
**30-second answer:** A denormalized counter becomes a hot row. With read-modify-write at READ COMMITTED two transactions can overwrite each other's update; use an atomic `SET n = n + 1`, a locking read, or an optimistic version check — and for extreme concurrency shard the counter or aggregate asynchronously.

**Structure a longer answer:** show the lost-update interleaving → fix options and their costs → isolation level interaction → contention solutions → idempotency under replays.

**Likely follow-ups:**
- *Does SERIALIZABLE fix it?* → Yes, by aborting one transaction; you must retry.
- *Synchronous vs asynchronous aggregate?* → freshness and read-your-writes vs write throughput and isolation from failures.
- *Fan-out on write vs read for a feed?* → write: fast reads, heavy writes, celebrity hotspot; read: slow reads, simple writes; hybrid.
- *Make a streaming increment idempotent?* → processed-event table in the same transaction, or recompute from source offsets/windows.
- *Hot counter redesign?* → sharded counter, delta table + periodic fold, in-memory buffer/queue, or approximate counters.

**Signals interviewers look for:** the exact interleaving, knowing which pattern is safe at which isolation level, and operational diagnosis (locks, deadlocks, retries).

### 11. Check yourself
- [ ] Draw the interleaving that loses an update under READ COMMITTED.
- [ ] Write the atomic and optimistic versions of the update.
- [ ] Explain why `n = n + 1` is not idempotent and how to fix replays.
- [ ] Design a sharded counter and say what the read costs.
- [ ] Name two DMVs/views for finding blocking on a hot row.
<!-- explainer:end -->

## Practice questions

- [Deep] [Why can a heavily denormalized table increase write amplification in your loading job?](<../../../../PRACTICE/01_Databases_and_SQL/Database_Design/Normalization/Q-34214_Why_can_a_heavily_denormalized_table_increase_write_amplific.md>)
- [Deep] [When would you compute a denormalized aggregate synchronously in the write path versus asynchronously in a downstream streaming job?](<../../../../PRACTICE/01_Databases_and_SQL/Database_Design/Normalization/Q-34278_When_would_you_compute_a_denormalized_aggregate_synchronousl.md>)
- [Deep] [How do you measure and reduce write amplification when one logical update touches five denormalized copies across your platform?](<../../../../PRACTICE/01_Databases_and_SQL/Database_Design/Normalization/Q-34291_How_do_you_measure_and_reduce_write_amplification_when_one_l.md>)
- [Deep] [How does denormalization change your locking footprint and the risk of deadlocks in a high-write system?](<../../../../PRACTICE/01_Databases_and_SQL/Database_Design/Normalization/Q-33958_How_does_denormalization_change_your_locking_footprint_and_t.md>)
- [Expert] [When you denormalize per query pattern into Cassandra you get write amplification. How do you manage and bound it as query patterns grow?](<../../../../PRACTICE/01_Databases_and_SQL/Database_Design/Normalization/Q-34282_When_you_denormalize_per_query_pattern_into_Cassandra_you_ge.md>)
- [Expert] [Your denormalized counter column becomes a write hotspot under high concurrency. How would you redesign to avoid lock contention?](<../../../../PRACTICE/01_Databases_and_SQL/Database_Design/Normalization/Q-33913_Your_denormalized_counter_column_becomes_a_write_hotspot_und.md>)
- [Expert] [Explain the interaction between transaction isolation level and maintaining a denormalized aggregate consistently.](<../../../../PRACTICE/01_Databases_and_SQL/Database_Design/Normalization/Q-33955_Explain_the_interaction_between_transaction_isolation_level_.md>)
- [Expert] [How would you design a denormalized feed or timeline table and manage fan-out on write versus fan-out on read?](<../../../../PRACTICE/01_Databases_and_SQL/Database_Design/Normalization/Q-33982_How_would_you_design_a_denormalized_feed_or_timeline_table_a.md>)

## Related concepts

- [Distributed Counters and Consistency](<../../../../07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Distributed_Joins_in_MPP_Systems/Distributed_Counters_and_Consistency.md>) *(Distributed Systems Core)*
- [Index Tuning in Practice](<../../../Query_Performance/Indexing/Index_Tuning_in_Practice/_index.md>) *(Indexing)*
- [Primary-replica Databases](<../../../../07_Distributed_Systems_and_Design/System_Design/System_Design_Core/Clickstream_Counting_Pipeline/Primary-replica_Databases.md>) *(System Design Core)*

