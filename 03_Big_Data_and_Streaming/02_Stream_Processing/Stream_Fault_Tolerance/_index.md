---
id: "03_Big_Data_and_Streaming/02_Stream_Processing/Stream_Fault_Tolerance"
level: 4
type: "curriculum"
curriculum: "data_engineering"
---

# Fault Tolerance and Exactly-Once

**L4 Advanced** · [↑ Stream Processing](<../_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** A stream job that never ends must survive crashes without losing or double-counting events, which it achieves with checkpoints, atomic commit of state and output, and idempotent writes — together giving "effectively exactly-once".

### 1. The big picture

A batch job that fails can simply be rerun: its input is immutable and its output is discarded or replaced, so the visible result is as if it ran exactly once. A stream job cannot wait for a finish line before exposing results; outputs are emitted continuously and the job runs for months. When a task dies after emitting some results, how do you restart without producing duplicates or skipping events?

The goal is **exactly-once semantics** (more precisely *effectively-once*): the observable effect of each event on the output is as if it was processed exactly one time, even if the processing was actually attempted more than once. Three technique families get there:

1. **Microbatching and checkpointing** — break the stream into small blocks or save state snapshots at barriers.
2. **Atomic commit** — tie together output, state and input position so they succeed or fail as a unit.
3. **Idempotence** — make repeated application of the same operation harmless.

### 2. Core concepts and vocabulary

- **At-most-once** — events may be lost, never duplicated.
- **At-least-once** — never lost, may be duplicated (retry after failure).
- **Exactly-once / effectively-once** — each event affects results once, even if processing was retried.
- **Microbatching** — treating the stream as many tiny batch jobs (Spark Streaming, e.g., ~1 second).
- **Checkpoint** — a consistent snapshot of operator state plus input offsets written to durable storage (Flink uses barriers flowing with the data).
- **Checkpoint barrier** — a marker injected into the stream; when an operator sees it, it snapshots its state.
- **Atomic commit** — all-or-nothing across state, output and input offset (transaction).
- **Idempotent operation** — applying it once or many times has the same result (e.g., `SET x = 5`, not `x = x + 1`).
- **Write-ahead log / changelog** — durable record of state updates used to rebuild state.
- **Sink** — the external system receiving results.
- **foreachBatch** — Spark Structured Streaming hook that runs batch logic (including upserts) on every micro-batch.

### 3. How it works, step by step

**A. Checkpointing (Flink-style)**

1. The job periodically injects a barrier at each source.
2. Each operator, upon receiving the barrier on all inputs, snapshots its local state to durable storage and forwards the barrier.
3. The source offsets at the barrier are stored with the state snapshot.
4. Once all operators have acknowledged, the checkpoint is complete.
5. On failure, the job restarts from the last completed checkpoint: state restored, sources rewound to the saved offsets, events replayed.
6. Events after the checkpoint are re-processed, so output may be emitted twice *unless* the sink participates (below).

**B. Microbatching (Spark-style)**

1. Input is cut into small batches, each with a defined offset range (recorded in a write-ahead log before processing).
2. Each micro-batch is processed as a deterministic batch job and its output committed with its batch id.
3. After a crash, the failed batch is re-run over the same offset range; the sink either overwrites or ignores duplicates by batch id.

**C. Atomic commit across output and state**

1. Output records, state updates and the input offset are written in one transaction (or one atomic checkpoint-complete commit).
2. If anything fails, all roll back; the retry starts from the last committed offset with no partial effects.
3. Works well *within* one system (Kafka transactions, Flink two-phase commit sinks); it is harder across heterogeneous systems with no shared transaction protocol.

**D. Idempotent writes**

1. Give every output a deterministic key (event id, or topic-partition-offset).
2. Write using upsert/`INSERT … ON CONFLICT DO UPDATE` or a conditional write.
3. A replayed event rewrites the same row with the same value — no change.
4. Where the operation is not naturally idempotent (increment, append), store metadata such as "last applied offset" together with the value and skip anything at or below it. This needs the processing to be deterministic and ordered; fencing is required when failing over so a "zombie" instance can't write.

### 4. Worked example

Count page views per user, write to PostgreSQL, using Spark Structured Streaming with `foreachBatch`.

```python
def upsert_counts(batch_df, batch_id):
    # idempotent: the whole batch is applied in one DB transaction,
    # and batch_id is stored so a replay of the same batch is skipped
    rows = [(r.user_id, r["count"]) for r in batch_df.collect()]
    with pg.transaction() as tx:
        if tx.scalar("SELECT 1 FROM applied_batches WHERE id = %s", batch_id):
            return                                  # already applied -> no duplicate
        tx.execute_many("""
            INSERT INTO views(user_id, n) VALUES (%s, %s)
            ON CONFLICT (user_id) DO UPDATE SET n = views.n + EXCLUDED.n
        """, rows)
        tx.execute("INSERT INTO applied_batches VALUES (%s)", batch_id)

(events.groupBy("user_id").count()
       .writeStream
       .option("checkpointLocation", "s3://chk/views")
       .foreachBatch(upsert_counts)
       .start())
```

Failure walk-through: the job crashes after the database commit but before Spark records batch 42 as done. On restart Spark re-executes batch 42 from the same offsets; the `applied_batches` check sees id 42 and skips it. Without that table, `n = n + count` would double count; this is the difference between at-least-once and effectively-once.

### 5. Under the hood

- **Why exactly-once is "effectively".** Processing may physically happen more than once; what is exactly-once is the *effect on state and output*. External side effects (sending an email, calling an API) are outside the guarantee unless made idempotent (use a request id).
- **Atomic commit revisited.** Distributed transactions have a bad reputation, but *within* a stream processor they work: the framework controls both the state and the output and can use a lightweight protocol (e.g., two-phase commit with the checkpoint). The problem is crossing system boundaries where no common protocol exists.
- **State recovery.** Options: periodically snapshot state to durable storage; or keep state in a local store that is replicated to a changelog topic (Kafka Streams) so it can be rebuilt by replay; or rely on an external store as state. Small state can even be rebuilt from the input log by replaying a window.
- **Determinism requirement.** Replay-based recovery assumes operators are deterministic. Using wall-clock time, random numbers or unordered iteration breaks exactly-once because a re-run produces different output.
- **Fencing.** After failover, an old instance (paused by a long GC) may wake and write; use epochs/fencing tokens so the sink rejects stale writers.
- **Steal/force from databases.** The write-ahead logging concepts of databases (log before data, flush rules) are the ancestors of checkpoint + replay: log the intent, then apply, so recovery can redo or undo.
- **Durability** underlying all of this means writing to non-volatile storage and replication; without it, an acknowledged event can still vanish.

### 6. Performance and design implications

- Checkpoint interval trades recovery time (replay length) against overhead and latency of transactional sinks (outputs become visible only at commit).
- Larger state means larger snapshots; use incremental checkpoints (RocksDB-backed state).
- Exactly-once sinks add commit latency; idempotent upserts are cheaper and more portable.
- Microbatch latency floor equals batch interval; per-event engines can be lower.
- Keep state bounded (TTL, windows) so snapshots and recovery stay fast.

### 7. Commands and settings you should know

| Command / setting | What it does | When to use |
|---|---|---|
| `checkpointLocation` (Spark) | Durable store for offsets and state | Any production streaming query |
| Flink `execution.checkpointing.interval` | Time between checkpoints | Tune recovery vs overhead |
| Kafka `enable.idempotence=true` | Dedupes producer retries | Prevent duplicate appends |
| Kafka transactions (`transactional.id`) | Atomic write of output and offsets | Consume-transform-produce loops |
| `foreachBatch` | Custom per-batch sink logic | Upserts, multi-sink writes |
| `INSERT … ON CONFLICT` / `MERGE` | Idempotent upsert | Sinks that support it |
| `delivery.guarantee` options | At-least-once vs exactly-once sinks | Connector configuration |

### 8. Trade-offs and decisions

| Choose this when | Avoid / watch out when |
|---|---|
| **Idempotent upserts** to key-value/relational sinks | Sink only supports append and has no keys |
| **Transactional/2PC sinks** within one platform | Sink is a system without transaction support |
| **At-least-once** + dedupe downstream | Duplicates cause financial effects |
| **Short checkpoint interval** for fast recovery | State is huge and snapshots are costly |
| **Microbatch** for simplicity and throughput | Sub-second latency is required |

### 9. Common mistakes

- **Non-idempotent sink with at-least-once delivery** → duplicates after restart → upsert by key or track batch id.
- **Non-deterministic operators** → replays diverge → pass event time, seed randomness from data.
- **Assuming exactly-once covers external calls** → emails sent twice → use idempotency keys.
- **No checkpoint location** → restart starts from scratch or loses state → always configure it.
- **Unbounded state** → checkpoint times balloon → windows and TTLs.
- **Zombie writers after failover** → stale data overwrites → fencing tokens.

### 10. 🎯 Interview kit

**30-second answer:** Streaming systems recover by periodically snapshotting operator state with input offsets, and on failure restarting from the snapshot and replaying. To keep outputs from duplicating, they either commit output and offsets atomically or write idempotently, giving effectively-once results.

**Structure a longer answer:** contrast with batch rerun → delivery semantics → checkpoint/replay mechanism → output side (atomic commit vs idempotence) → determinism and external side effects → trade-offs.

**Likely follow-ups:**
- *Is exactly-once really possible?* → Effects, yes within a system; side effects need idempotence.
- *Checkpointing vs microbatching?* → Both rewind to a consistent point; microbatch makes the unit explicit.
- *What if the sink is not transactional?* → Idempotent writes with deterministic keys.
- *How do you rebuild state?* → From snapshots, or by replaying a changelog/the input log.
- *Why must operators be deterministic?* → Replay must reproduce the same output.

**Signals interviewers look for:** distinguishes processing from effect, mentions idempotence and atomic commit, knows about replay and state recovery.

### 11. Check yourself

- [ ] Explain why batch jobs are naturally fault tolerant and streaming jobs are not.
- [ ] Walk through a crash between "write to sink" and "commit offset" and show how each technique handles it.
- [ ] Make a counter-increment sink idempotent.
- [ ] Describe what a checkpoint barrier does.
- [ ] Name three things that break deterministic replay.
<!-- explainer:end -->

## References

- *Designing Data-Intensive Applications* — Martin Kleppmann — Fault Tolerance › Atomic commit revisited
- *Designing Data-Intensive Applications* — Martin Kleppmann — Fault Tolerance › Idempotence
- *Designing Data-Intensive Applications* — Martin Kleppmann — F
- *Learning Spark (2nd ed.)* — Jules S. Damji, Brooke Wenig, Tathagata Das, Denny Lee — Streaming Data Sources and Sinks › Using foreachBatch()
- *Designing Data-Intensive Applications* — Martin Kleppmann — Chapter 7. Transactions › Durability
- *Hadoop: The Definitive Guide* — Tom White — ZooKeeper › Recoverable exceptions
- *Database Internals* — Alex Petrov — Transaction Processing and Recovery › Steal and Force Policies

