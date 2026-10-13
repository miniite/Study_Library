---
id: "03_Big_Data_and_Streaming/02_Stream_Processing/Partitioned_Logs"
level: 4
type: "curriculum"
curriculum: "data_engineering"
---

# Partitioned Logs

**L4 Advanced** · [↑ Stream Processing](<../_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** A partitioned log broker (Kafka-style) appends every message to an ordered, durable, replayable log per partition and lets each consumer track its own offset — combining database-style storage with messaging-style low latency.

### 1. The big picture

Classic message brokers delete a message once it is acknowledged. That makes messaging transient: you cannot add a new consumer and have it read yesterday's events, and a consumer bug that mishandles messages is not recoverable. Databases, by contrast, keep data durably and let you re-read it, but they don't notify clients of new data.

A **log-based message broker** takes the best of both: it is an append-only file (a *log*) of messages. Producers append to the end; consumers read sequentially and are notified of new data. To scale beyond one disk, the log is split into **partitions**, each its own ordered log possibly on a different machine, and a **topic** is a group of partitions. Apache Kafka, Amazon Kinesis Streams and Twitter's DistributedLog are of this kind.

Understanding partitioned logs is the key to modern streaming: ordering, parallelism, replay, backpressure and fault tolerance all follow from the log design.

### 2. Core concepts and vocabulary

- **Log** — append-only sequence of records on disk, ordered by position.
- **Partition** — one log; ordering is guaranteed only inside a partition.
- **Topic** — a named set of partitions that carry the same type of messages.
- **Offset** — a monotonically increasing sequence number of a message within its partition.
- **Segment** — a file chunk of a partition log; old segments are deleted or compacted as a unit.
- **Producer** — appends messages; picks the partition (by key hash or round robin).
- **Consumer group** — a set of consumers sharing a topic; each partition is assigned to exactly one member of the group.
- **Consumer offset** — the position a consumer group has read up to, stored (committed) by the broker or consumer.
- **Retention** — how long (or how much) data is kept before segments are discarded.
- **Log compaction** — keeping only the latest message per key.
- **Replication / leader-follower** — each partition has copies on several brokers for durability.
- **Consumer lag** — head offset minus the consumer's committed offset.

### 3. How it works, step by step

1. **Append.** A producer sends a message. The broker appends it to the end of the chosen partition and assigns the next offset.
2. **Partition choice.** With a message key, `hash(key) mod partitions` keeps all events of one entity in one partition, so their relative order is preserved.
3. **Replicate.** The partition leader copies the record to followers; the producer ack level decides when the write is "committed".
4. **Read.** A consumer reads messages sequentially from its current offset. The broker returns a batch; the consumer processes it.
5. **Track.** The consumer periodically records its offset. After a crash or restart, it resumes from the last committed offset.
6. **Scale.** Within a consumer group, partitions are divided among consumers. Adding consumers beyond the partition count gives no extra parallelism. A different group gets its own full copy (fan-out), tracked by its own offsets.
7. **Retain.** The broker deletes the oldest segments when retention is reached (time or size), or compacts them.

```text
topic "clicks" (3 partitions)
 P0: [0][1][2][3][4][5] ──►  consumer A1 (group A)
 P1: [0][1][2][3]       ──►  consumer A2 (group A)
 P2: [0][1][2][3][4]    ──►  consumer A2 (group A)       group B reads all three independently
```

### 4. Worked example

```bash
# Create a topic with 6 partitions, replication factor 3, 7-day retention
kafka-topics.sh --create --topic clicks --partitions 6 --replication-factor 3 \
  --config retention.ms=604800000 --bootstrap-server broker1:9092

# Inspect consumer-group progress
kafka-consumer-groups.sh --describe --group dashboard --bootstrap-server broker1:9092
```

Typical output (simplified):

| PARTITION | CURRENT-OFFSET | LOG-END-OFFSET | LAG |
|---|---|---|---|
| 0 | 120450 | 120452 | 2 |
| 1 | 118000 | 130900 | 12900 |
| 2 | 119777 | 119780 | 3 |

Reading it: partition 1's consumer is far behind (lag 12,900) — a slow or stuck consumer or a hot partition. Because the data is retained, nothing is lost; the consumer simply catches up. If a bug in the dashboard consumer corrupted yesterday's results, you reset the group's offset to yesterday and replay — impossible with a delete-on-ack queue.

### 5. Under the hood

- **Sequential I/O.** Appending and sequentially reading a log is the fastest disk pattern; brokers rely on the OS page cache and can sustain millions of messages per second. Throughput scales by adding partitions and brokers.
- **Message ordering.** Total order exists within a partition only. Across partitions there is no ordering guarantee. If you need a global order, you need one partition (and give up parallelism). Per-key order via key partitioning is the usual compromise.
- **Offsets replace per-message acks.** The consumer's progress is a single number, so the broker keeps no per-message bookkeeping. Messages before the offset are processed; after it, not. This is similar to a replication log's *log sequence number* — the consumer is effectively a follower reading the leader's log.
- **At-least-once by default.** Process then commit the offset: a crash between them replays some messages (duplicates). Commit before processing risks loss. Exactly-once needs idempotence or transactions that tie the output and the offset commit together.
- **Slow consumers** do not harm the broker or other consumers. If a consumer falls behind by more than the retention, it misses messages (effectively drop). The broker is a large bounded buffer, so you can monitor lag and alert before data is lost.
- **Disk space.** A circular buffer in effect: the log is bounded by retention. A 6 TB disk at 150 MB/s of writes holds about 11 hours; in practice 100 MB/s rates are rare, so days or weeks of retention are common.
- **Replay is cheap and non-destructive**, which makes the log batch-friendly: experiments and new consumers can read history without affecting production consumers — the same immutability that makes batch jobs safe to rerun.
- **Parallelism limits.** At most one consumer per partition per group; processing time per message limits per-partition throughput; head-of-line blocking occurs if a message is slow to process.
- **Compaction** enables a topic to act like a changelog table (see *Databases and Streams*).

### 6. Performance and design implications

- Choose the partition count for the target peak consumer parallelism (hard to reduce later, expensive to over-provision).
- Pick keys with good cardinality; one hot key makes one hot partition.
- Batch writes and reads; compress batches.
- Keep per-message processing fast, or you will see head-of-line blocking within a partition.
- Retention sets how long a consumer may be down and still catch up.
- Monitor lag per partition, under-replicated partitions, and disk usage.

### 7. Commands and settings you should know

| Command / setting | What it does | When to use |
|---|---|---|
| `kafka-topics.sh --create/--describe` | Manage topics and partitions | Capacity planning |
| `kafka-consumer-groups.sh --describe` | Show offsets and lag | Health checks |
| `--reset-offsets --to-datetime ...` | Rewind or skip a group | Replay after a bug |
| `retention.ms` / `retention.bytes` | How long/how much data is kept | Sizing replay window |
| `cleanup.policy=compact` | Keep latest per key | Changelog topics |
| `acks=all`, `min.insync.replicas` | Producer durability level | Avoid loss on failover |
| `enable.auto.commit` | Automatic vs manual offset commits | Control delivery semantics |

### 8. Trade-offs and decisions

| Choose this when | Avoid / watch out when |
|---|---|
| High throughput, many independent consumers, replay | You need per-message routing, priorities or delays |
| Ordering per key is enough | You need global total order at high rate |
| Pipelines, CDC, event sourcing, stream processing | Tasks vary hugely in duration (head-of-line blocking) — use a work queue |
| You want slow consumers isolated from fast ones | You cannot afford operating a replicated cluster |

### 9. Common mistakes

- **More consumers than partitions** → idle consumers → increase partitions or accept the limit.
- **Committing offsets before processing** → message loss on crash → commit after processing.
- **Hot key** → one overloaded partition → salt or choose a better key.
- **Retention shorter than outage window** → data lost for the lagging consumer → size retention and alert on lag.
- **Expecting global order** → events from different partitions interleave → key by entity or use a single partition.
- **Treating the log as a queue with per-message delete** → not possible; use offsets and compaction.

### 10. 🎯 Interview kit

**30-second answer:** A log-based broker stores messages in append-only, partitioned logs on disk. Each message has an offset; consumers keep their own offset, so they can be slow, restarted or replayed without affecting others. Order is guaranteed per partition; scale comes from partitions and consumer groups.

**Structure a longer answer:** why delete-on-ack brokers fall short → log + partitions + offsets → consumer groups → ordering/parallelism limits → delivery semantics → retention/compaction → scenario.

**Likely follow-ups:**
- *How do you guarantee order?* → Key-based partitioning; ordering per partition only.
- *How do you scale consumers?* → Add partitions and group members, up to partition count.
- *What happens when a consumer is slow?* → Lag grows; broker unaffected; data lost only if retention expires.
- *At-least-once vs exactly-once?* → Offset commit timing; exactly-once via idempotent writes or transactions.
- *Log vs AMQP-style queue?* → Replay and throughput vs per-message routing and parallel slow tasks.

**Signals interviewers look for:** can reason about partitions, offsets and ordering; mentions lag, retention and replay; knows the limits of parallelism.

### 11. Check yourself

- [ ] Explain why a log broker lets you add a new consumer that reads history.
- [ ] Describe what offsets are and what happens on a consumer crash.
- [ ] Work out maximum consumer parallelism for a 12-partition topic.
- [ ] Explain why ordering is only per partition.
- [ ] Describe what happens if a consumer is offline longer than the retention period.
<!-- explainer:end -->

## References

- *Designing Data-Intensive Applications* — Martin Kleppmann — Partitioned Logs › Consumer offsets
- *Designing Data-Intensive Applications* — Martin Kleppmann — Partitioned Logs › Using logs for message storage
- *Designing Data-Intensive Applications* — Martin Kleppmann — Partitioned Logs › When consumers cannot keep up with producers
- *Designing Data-Intensive Applications* — Martin Kleppmann — Partitioned Logs › Disk space usage
- *Designing Data-Intensive Applications* — Martin Kleppmann — Partitioned Logs
- *Designing Data-Intensive Applications* — Martin Kleppmann — Partitioned Logs › Replaying old messages
- *Designing Data-Intensive Applications* — Martin Kleppmann — Chapter 11. Stream Processing › Message brokers compared to databases
- *Designing Data-Intensive Applications* — Martin Kleppmann — Partitioned Logs › Logs compared to traditional messaging
- *Fundamentals of Data Engineering* — Joe Reis, Matt Housley — Data Generation in Source Systems › Stream partitions
- *Designing Data-Intensive Applications* — Martin Kleppmann — Storage and Retrieval › Downsides of LSM-trees

