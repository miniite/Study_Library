---
id: "03_Big_Data_and_Streaming/02_Stream_Processing/Databases_and_Streams"
level: 4
type: "curriculum"
curriculum: "data_engineering"
---

# Databases and Streams

**L4 Advanced** · [↑ Stream Processing](<../_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** A database's changes are themselves an ordered stream of events, so capturing that stream (CDC) and replaying it is a more reliable way to keep caches, search indexes and warehouses in sync than dual writes or periodic dumps.

### 1. The big picture

Almost every non-trivial application stores the same data in several places: an OLTP database for transactions, a cache for speed, a full-text index for search, a warehouse for analytics. Each copy is optimized for its own access pattern, but they all must reflect the same truth. How do you keep them in sync?

There are three ways:

1. **Periodic full dumps / ETL** — slow and always stale.
2. **Dual writes** — application code writes to each system itself. Fast, but it breaks in subtle ways.
3. **Change data capture** — treat the database's own write log as a stream and let every other system consume it.

The deep insight: *a write to a database is an event*. A replication log is just a stream of write events produced by the leader and applied by followers. If every replica applies the same events in the same order, all replicas converge to the same state (state-machine replication). Extending that idea beyond the database's own replicas gives you derived data systems that are always "just another follower".

### 2. Core concepts and vocabulary

- **System of record** — the authoritative store; others are *derived data*.
- **Derived data system** — cache, index, materialized view, warehouse table built from the system of record.
- **Dual write** — application writes to two or more systems directly.
- **Change data capture (CDC)** — observing all changes in a database and exposing them as a stream.
- **Replication log / WAL / binlog / oplog** — the database's ordered log of changes.
- **Log-based CDC** — parsing that log (Debezium, Maxwell, GoldenGate) instead of using triggers or polling.
- **Trigger-based CDC** — database triggers copy changes to a changelog table; fragile and slower.
- **Batch/timestamp CDC** — query for rows with `updated_at` after the last run.
- **Initial snapshot** — a consistent copy of the database at a known log position, needed to bootstrap a new consumer.
- **Log compaction** — keeping only the latest event per key so the log stays bounded but still reconstructs current state.
- **Tombstone** — an event with a null value meaning "this key was deleted"; removed during compaction.
- **Event sourcing** — a design where events, not current state, are the source of truth (see *Event Sourcing*).
- **Idempotent consumer** — one that can apply the same event twice with the same result.

### 3. How it works, step by step

**Why dual writes fail.** Two clients update the same item X concurrently. Client 1 sets X=A, client 2 sets X=B. The database sees A then B (final B). The search index receives the writes in the opposite order — B then A (final A). The systems are now permanently inconsistent, no error was raised, and neither knows. A second problem: one write may succeed while the other fails, leaving them out of step with no automatic repair.

**How CDC fixes it.**

1. The application writes **only** to the system-of-record database.
2. The database appends the change to its commit log in a single total order.
3. A CDC connector reads that log (log-based) and publishes each change as an event to a log-based broker topic, preserving order per key.
4. Each derived system has its own consumer applying events in order: index updater, cache invalidator, warehouse loader.
5. Because the order is identical, all derived systems converge to the same state as the database (after replication lag).

**Bootstrapping a new consumer.**

1. If the full log history exists, replay from offset 0.
2. Otherwise take a **consistent snapshot** tied to a known log position (offset/LSN).
3. Load the snapshot, then apply log events from that position onward.

**Log compaction as an alternative.**

1. Every change event is keyed by the row's primary key.
2. The broker periodically discards older events for the same key, keeping the latest.
3. A new consumer starts at offset 0 of the compacted topic and sees the latest value for every key — a full copy without a separate snapshot.
4. Deleting a row writes a tombstone; compaction later removes both the key and tombstone.
5. Disk usage depends on the size of the current data, not the number of historical writes.

### 4. Worked example

Keep a search index in sync with a PostgreSQL `products` table.

```text
products table ──(logical decoding / WAL)──► Debezium ──► Kafka topic "shop.public.products"
                                                       │
                                  compacted, key = product id
                                                       ├──► search-index updater
                                                       ├──► cache invalidator
                                                       └──► warehouse loader
```

A sample change event (simplified):

```json
{
  "key":   {"id": 4711},
  "value": {
    "before": {"id": 4711, "price": 19.99},
    "after":  {"id": 4711, "price": 17.99},
    "op": "u",
    "source": {"lsn": 903418, "ts_ms": 1760000000000}
  }
}
```

Walk through:

1. The price update commits in PostgreSQL; the WAL records it.
2. Debezium reads the WAL via logical decoding and emits the event keyed by `id`, so all changes to product 4711 go to the same partition, in order.
3. The search updater upserts `{id:4711, price:17.99}`. If it crashes and re-reads the event, the upsert is idempotent — no harm.
4. A brand-new warehouse loader starts at offset 0 of the compacted topic and builds the full table.

### 5. Under the hood

- **Ordering guarantees.** Order is only meaningful per key/partition. CDC must keep all changes for a key in one partition; cross-partition order is not defined, so consumers must not assume a global order unless the broker offers a single partition.
- **Asynchrony.** CDC is normally asynchronous: the database commits without waiting for consumers. A slow consumer does not slow writers, but replication lag applies — derived systems are eventually consistent.
- **Schema changes.** Log parsing has to handle DDL (added/dropped columns); a schema registry plus compatible encodings (Avro/Protobuf) avoids breaking consumers.
- **Triggers vs log reading.** Triggers add write-path overhead and run in the transaction; log reading adds almost none but depends on a log format that was historically internal.
- **Atomic with the write.** Because the log is part of the commit, CDC gets "write and event" atomically for free. Application-level "write DB, then publish" cannot — the outbox pattern (write an event row in the same transaction, relay it via CDC) recovers this.
- **Unbundling.** Treating the database log as the integration backbone is the "unbundling the database" view: indexes, caches and views are separate components fed by the same ordered log, with loose coupling and independent failure domains. This is simpler than cross-system distributed transactions (2PC), which lack a standard protocol across heterogeneous systems.
- **Batch CDC limitation.** Polling an `updated_at` column returns only the **latest** state per row in the interval, losing intermediate changes (a customer who withdrew money five times shows one final balance). Insert-only schemas fix this.

### 6. Performance and design implications

- Log-based CDC has minimal source overhead but needs retention of log segments (WAL) long enough for the connector — an unconsumed replication slot can fill the source's disk.
- Compaction keeps topic size proportional to distinct keys, enabling cheap rebuilds.
- Consumers must be **idempotent**; delivery is typically at-least-once.
- Snapshot-then-stream bootstraps can be heavy; throttle them and run off a replica when possible.
- Monitor connector lag (source LSN vs consumed LSN) and replication-slot size.

### 7. Commands and settings you should know

| Command / setting | What it does | When to use |
|---|---|---|
| `wal_level = logical` (PostgreSQL) | Enables logical decoding | Required for log-based CDC |
| `binlog_format = ROW` (MySQL) | Row-level change events | Required for Debezium/Maxwell |
| Replication slot | Remembers consumer position, retains WAL | Monitor to avoid disk exhaustion |
| `cleanup.policy=compact` (Kafka) | Enables log compaction | Changelog / table-like topics |
| `snapshot.mode` (Debezium) | Controls initial snapshot behaviour | Bootstrapping a connector |
| Outbox table | Event rows written in the business transaction | Reliable "DB write + event" |

### 8. Trade-offs and decisions

| Choose this when | Avoid / watch out when |
|---|---|
| **Log-based CDC** for ongoing sync of several derived systems | The source cannot expose its log or you lack access |
| **Compacted topic** to rebuild consumers without snapshots | You need full history, not just latest values |
| **Timestamp polling** for simple, low-volume needs | You need deletes or every intermediate change |
| **Outbox pattern** when events must be atomic with business data | Volume is huge and outbox contention hurts |
| **Event log + idempotent consumers** across heterogeneous systems | You truly need synchronous cross-system consistency |

### 9. Common mistakes

- **Dual writes** → permanent, silent divergence under concurrency or partial failure → write once, derive the rest from the log.
- **Unmonitored replication slot** → source disk fills → alert on slot lag and set retention limits.
- **Assuming global ordering** → inconsistent state across partitions → key by entity id.
- **Non-idempotent consumers** → duplicates double-apply → upserts or dedupe by event id/offset.
- **Forgetting deletes** → "zombie" rows in the index → handle tombstones.
- **Skipping the snapshot** → new index missing rows that were not recently changed → snapshot at a known offset, then stream.

### 10. 🎯 Interview kit

**30-second answer:** Instead of having the application write to the database, cache and search index separately (dual writes, which race and diverge), you write only to the database and capture its commit log as a stream. Derived systems consume that ordered change stream, so they converge to the same state. Log compaction and snapshots let you bootstrap new consumers.

**Structure a longer answer:** the sync problem → why dual writes break (ordering race, partial failure) → CDC mechanism (log parsing, broker, per-key order) → bootstrapping (snapshot/compaction) → trade-offs (lag, schema changes, idempotence) → scenario.

**Likely follow-ups:**
- *Trigger-based vs log-based CDC?* → Triggers are intrusive and slow; log-based is robust but format-dependent.
- *How do you handle deletes?* → Tombstone events removed on compaction.
- *What if the consumer is down for days?* → It resumes from its offset if retention allows, else re-snapshot.
- *CDC vs 2PC across systems?* → An ordered log with idempotent consumers is simpler and more available than distributed transactions.
- *How do you make a DB write and event publish atomic?* → Outbox pattern.

**Signals interviewers look for:** can explain the dual-write race concretely; knows CDC is asynchronous; mentions snapshot/compaction and idempotence.

### 11. Check yourself

- [ ] Draw the timeline of the dual-write race that leaves the index with a stale value.
- [ ] Explain why a replication log is "a stream of events".
- [ ] Describe how to add a new search index to a system whose log has been truncated.
- [ ] Explain how log compaction bounds storage but preserves current state.
- [ ] List two reasons log-based CDC is preferred over triggers.
<!-- explainer:end -->

## In this section (study in order)

1. [Event Sourcing](<Event_Sourcing/_index.md>) (L5 Expert)
2. [State Streams and Immutability](<Immutability_and_Derived_Views/_index.md>) (L5 Expert)

## References

- *Designing Data-Intensive Applications* — Martin Kleppmann — Databases and Streams › Keeping Systems in Sync
- *Designing Data-Intensive Applications* — Martin Kleppmann — Q › R
- *Designing Data-Intensive Applications* — Martin Kleppmann — Databases and Streams › Initial snapshot
- *Fundamentals of Data Engineering* — Joe Reis, Matt Housley — Ingestion › Continuous CDC
- *Designing Data-Intensive Applications* — Martin Kleppmann — Databases and Streams › Change Data Capture
- *Designing Data-Intensive Applications* — Martin Kleppmann — Databases and Streams › API support for change streams
- *Designing Data-Intensive Applications* — Martin Kleppmann — Unbundling Databases › Making unbundling work
- *Fundamentals of Data Engineering* — Joe Reis, Matt Housley — Ingestion › CDC and database replication
- *Designing Data-Intensive Applications* — Martin Kleppmann — Databases and Streams › Implementing change data capture
- *Designing Data-Intensive Applications* — Martin Kleppmann — Databases and Streams › Log compaction

