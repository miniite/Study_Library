---
id: "02_Data_Engineering/Pipelines_and_Integration/Change_Data_Capture/Debezium_Transforms_and_Snapshots"
level: 4
type: "concept"
topic: "02_Data_Engineering/Pipelines_and_Integration/Change_Data_Capture"
tools: ["Apache_Kafka", "MySQL", "AWS"]
---

# Debezium Transforms and Snapshots

**L4 Advanced** · [↑ Change Data Capture](<_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** Debezium turns a database log into Kafka change events; to run it well you must understand the event envelope, what single message transforms should and shouldn't do, and how snapshots and replication slots behave.

### 1. The big picture
Debezium is a distributed set of source connectors for Kafka Connect. Each connector reads a database's log (MySQL binlog, PostgreSQL logical decoding, SQL Server CDC tables, Oracle LogMiner/XStream, MongoDB change streams) and publishes row-level changes to Kafka topics. A typical installation has Kafka (originally with ZooKeeper, newer versions with KRaft), Kafka Connect workers hosting the connectors, and the source database.

Three practical questions come up in every production deployment:
1. What does an event look like, and how do I reshape it for consumers? (envelope, `op`, SMTs)
2. How do I get the existing data in, and how do I redo it for one table without stopping everything? (snapshots, incremental snapshots)
3. How do I keep the connector from hurting the database? (replication slots, heartbeats, WAL/binlog retention)

### 2. Core concepts and vocabulary
- **Change event envelope** — the message structure: `before`, `after`, `source` (database, table, log position, timestamp), `op`, `ts_ms`, and optionally `transaction` info.
- **`op` field** — `c` create (insert), `u` update, `d` delete, `r` read (emitted during snapshot), `t` truncate (on supporting connectors).
- **Tombstone** — a message with the same key and a null value, emitted after a delete event so Kafka log compaction can remove that key.
- **Single Message Transform (SMT)** — a small, stateless transform applied to each message inside Kafka Connect (e.g. unwrap the envelope, route to another topic, mask a field).
- **Snapshot** — the initial consistent read of existing table data, emitted as `r` events before streaming begins.
- **Incremental snapshot** — chunked, on-demand re-snapshot of selected tables while streaming continues, using watermarks to reconcile chunks with live changes.
- **Replication slot (PostgreSQL)** — a server-side bookmark for a logical decoding consumer; the server retains WAL until the slot's consumer confirms it.
- **Publication (PostgreSQL)** — the set of tables whose changes are exposed to logical replication.
- **Heartbeat** — a periodic message that advances the connector's offset and slot position even when the captured tables are idle.
- **Schema history topic** — for MySQL/SQL Server/Oracle connectors, a topic recording DDL so past log entries can be decoded correctly.

### 3. How it works, step by step
**Startup and snapshot**
1. The connector connects and records the current log position.
2. In snapshot mode `initial`, it acquires a consistent view of the captured tables (using locks or a transaction snapshot, depending on the database) and reads all rows, emitting `r` events.
3. Once done, it switches to streaming from the recorded log position, so no change between snapshot start and streaming start is lost.
4. Offsets are committed to Kafka Connect's offset store regularly.

**Streaming**
1. The connector reads log records, decodes them using the table schema.
2. It groups changes into transactions and emits events in commit order, keyed by primary key.
3. Each event is serialized (Avro/JSON/Protobuf) and sent to a topic per table.

**Incremental snapshot (the DBLog-style algorithm)**
1. You send a signal (insert into a signaling table or a Kafka signal topic) naming tables to re-snapshot.
2. The connector reads the table in primary-key-ordered chunks (e.g. 1,024 rows).
3. For each chunk it writes a *low watermark* into the log, reads the chunk, then writes a *high watermark*.
4. Streaming events that arrive between the two watermarks and touch keys in the chunk win over the chunk's rows: those rows are dropped from the chunk buffer. This makes the combination correct without locking the table.
5. Remaining chunk rows are emitted as `r` events; processing continues with the next chunk. The stream is never paused.

### 4. Worked example
An update to a customer row, unwrapped for a warehouse sink:

```json
// raw envelope (simplified)
{
  "before": {"id": 7, "email": "a@x.com", "tier": "silver"},
  "after":  {"id": 7, "email": "a@x.com", "tier": "gold"},
  "source": {"db": "shop", "table": "customers", "lsn": 24023128, "ts_ms": 1760200000000},
  "op": "u",
  "ts_ms": 1760200000123
}
```

Connector config fragment using SMTs:

```properties
transforms=unwrap,route
transforms.unwrap.type=io.debezium.transforms.ExtractNewRecordState
transforms.unwrap.delete.handling.mode=rewrite
transforms.unwrap.add.fields=op,source.ts_ms
transforms.route.type=org.apache.kafka.connect.transforms.RegexRouter
transforms.route.regex=shop\.public\.(.*)
transforms.route.replacement=raw_$1
```

After `ExtractNewRecordState`, the value becomes a flat row `{id, email, tier, __op, __source_ts_ms, __deleted}`. With `delete.handling.mode=rewrite` deletes are kept as rows with `__deleted=true` instead of being dropped, which suits warehouse sinks that cannot process tombstones. `RegexRouter` renames topics.

To re-snapshot only `customers` on a live connector:

```sql
INSERT INTO debezium_signal (id, type, data)
VALUES ('rs-1', 'execute-snapshot',
        '{"data-collections": ["public.customers"], "type": "incremental"}');
```

### 5. Under the hood
- **Why tombstones exist.** Kafka compaction retains the last value per key. A delete event is a value (it says "deleted"); a following null-value message tells compaction the key can be removed entirely. If a sink doesn't understand nulls, or you want deletes preserved as data, disable tombstones (`tombstones.on.delete=false`) and use the delete-rewrite option. Downstream consequence: compacted topics will keep the delete event forever and consumers that rebuild state must know to treat `op=d` or `__deleted` as removal.
- **Replication slot mechanics (PostgreSQL).** The slot's `restart_lsn` pins WAL. If the connector is down or the captured tables are idle while *other* tables in the database generate WAL, the slot's confirmed position does not advance in a useful way and WAL accumulates, risking a full disk. Heartbeats (`heartbeat.interval.ms`, with an action query writing to a small heartbeat table) let the connector move its confirmed position. Set `max_slot_wal_keep_size` as a safety limit, and monitor `pg_replication_slots`.
- **Lost log position.** If the offset the connector needs refers to a binlog file or WAL segment that was purged, it cannot resume. Recovery: set the snapshot mode to re-snapshot (`when_needed` or a manual reset), accept the gap by taking a fresh snapshot, then reconcile. Prevention: adequate retention, lag alerts, and heartbeats.
- **Transaction metadata.** Debezium can emit begin/end markers and per-event transaction IDs and order. Consumers can use these to rebuild transactionally consistent views; most pipelines skip this and apply per-row.
- **Postgres 16/17 and publications.** Publications define what is replicated; filtering can be by table (and, in recent versions, by columns and row filters), which reduces volume. Newer versions allow logical decoding from standbys and improved slot management, which helps large fleets avoid loading primaries.
- **SMT discipline.** SMTs run in the connector's worker thread per message. Keep them stateless and cheap: unwrap, route, mask, rename, add metadata. Do joins, aggregations, enrichment and anything needing external lookups in a stream processor or the warehouse, where you can scale and test them independently.

### 6. Performance and design implications
- Snapshot of large tables takes time and creates read load; schedule it, prefer replicas when supported, and consider incremental snapshots for very large tables (hundreds of GB to TB) because they are resumable and don't block streaming.
- Large transactions spike event volume; tune `max.batch.size`, queue size and Kafka producer settings.
- One connector task per database for most relational connectors: throughput scales by splitting databases/tables across connectors, not by adding tasks.
- Monitor connector JMX metrics: `MilliSecondsBehindSource`, `QueueRemainingCapacity`, snapshot completion; plus source-side slot lag and WAL size.
- Topic retention in Kafka determines how far back consumers can replay; if you need a week of replay with 24-hour retention, store raw events in object storage (sink connector) and replay from there rather than re-snapshotting.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `snapshot.mode=initial / when_needed / never / schema_only` | Controls snapshot on start | Adopting a table, recovering from lost log |
| `signal.data.collection` + `execute-snapshot` signal | Triggers incremental snapshot | Re-sync one table on a live connector |
| `ExtractNewRecordState` SMT | Flattens envelope to the new row | Sinks that want plain rows |
| `tombstones.on.delete` | Emits null-value message after delete | Disable for non-compacting or null-intolerant sinks |
| `heartbeat.interval.ms`, `heartbeat.action.query` | Advances offsets on idle tables | Low-traffic tables on busy PostgreSQL databases |
| `publication.autocreate.mode` | How the publication is created | Control which tables are exposed |
| `provide.transaction.metadata=true` | Emits transaction markers | Transactionally consistent downstream rebuild |
| `pg_replication_slots`, `max_slot_wal_keep_size` | Monitor/limit retained WAL | Prevent disk fill |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| Incremental snapshot for multi-hundred-GB tables or a single misbehaving table | Table lacks a usable primary key ordering; very small tables where a blocking snapshot is simpler |
| Rewrite deletes into rows with a flag | Sinks that already honour tombstones and compaction |
| Disable tombstones for warehouse sinks | Topics that rely on compaction to bound size |
| SMTs for flattening, routing, masking | Anything stateful, cross-message, or needing lookups |
| Transaction metadata for strict consistency | Typical per-row analytics where eventual consistency is fine |

### 9. Common mistakes
- Treating `r` events as noise → snapshot rows are dropped by consumers and the table starts empty → handle `r` like an insert.
- Letting a low-traffic table's connector sit idle on a busy PostgreSQL → WAL grows without bound → enable heartbeats and slot limits.
- Putting heavy business logic in SMTs → slow, hard-to-test connectors → keep SMTs to shape-and-route.
- Skipping DDL tests → schema change breaks the connector in production → in staging test large transactions, adding/dropping/renaming columns, deletes, and restarts.
- Assuming topic retention equals replay capability → connector restarts from a recycled position or consumers miss history → keep raw events in durable storage.
- Forgetting REPLICA IDENTITY on PostgreSQL → updates and deletes lack the old row values → set it to FULL or a suitable index where you need before images.

### 10. 🎯 Interview kit
**30-second answer:** Debezium reads the database log and emits change events with before, after, op and source metadata to Kafka. You flatten and route with SMTs, load existing data with a snapshot, re-sync large tables with incremental snapshots that use watermarks so streaming never stops, and you guard the source with heartbeats and slot limits.

**Structure a longer answer:** envelope and op → snapshot → streaming → incremental snapshot and watermark correctness → source-side risks → testing and monitoring.

**Likely follow-ups:**
- *What do `c`, `r`, `u`, `d` mean?* → create, read (snapshot), update, delete.
- *Why disable tombstones?* → sink cannot handle null values or you need deletes as data; downstream must then treat the delete event or flag explicitly.
- *How do incremental snapshots stay correct?* → watermarks around each chunk let live events override stale chunk rows.
- *Low-traffic table, WAL growing?* → heartbeat plus action query, slot WAL limit, monitor `confirmed_flush_lsn`.
- *Offset points to recycled WAL?* → re-snapshot (or schema-only plus targeted backfill), then add retention and alerts.

**Signals interviewers look for:** concrete knowledge of the event shape, awareness of slot/WAL risk, and clarity on what belongs in a connector versus downstream.

### 11. Check yourself
- [ ] Write the envelope of an update event and mark which part you use for a MERGE.
- [ ] Explain the low and high watermark steps of an incremental snapshot.
- [ ] Explain why a PostgreSQL slot can fill the disk when your tables are idle.
- [ ] Decide between delete rewrite and tombstones for a warehouse sink.
- [ ] List four edge cases to test before promoting a connector to production.
<!-- explainer:end -->

## Implementations (tools)

- [Apache Kafka](<../../../TOOLS/Apache_Kafka/_index.md>)
- [MySQL](<../../../TOOLS/MySQL/_index.md>)
- [AWS](<../../../TOOLS/AWS/_index.md>)

## Practice questions

- [Basic] [What is Debezium and what problem does it solve for data engineers?](<../../../PRACTICE/02_Data_Engineering/Pipelines_and_Integration/Change_Data_Capture/Q-06021_What_is_Debezium_and_what_problem_does_it_solve_for_data_eng.md>)
- [Intermediate] [What does the op field in a Debezium message mean and what values can it take?](<../../../PRACTICE/02_Data_Engineering/Pipelines_and_Integration/Change_Data_Capture/Q-06074_What_does_the_op_field_in_a_Debezium_message_mean_and_what_v.md>)
- [Deep] [What is a Debezium single message transform (SMT) and give an example of its use?](<../../../PRACTICE/02_Data_Engineering/Pipelines_and_Integration/Change_Data_Capture/Q-06093_What_is_a_Debezium_single_message_transform_SMT_and_give_an_.md>)
- [Deep] [Why might you disable tombstone events in Debezium for a particular sink and what downstream behavior changes when you do?](<../../../PRACTICE/02_Data_Engineering/Pipelines_and_Integration/Change_Data_Capture/Q-06114_Why_might_you_disable_tombstone_events_in_Debezium_for_a_par.md>)
- [Deep] [How do you tune Debezium heartbeat and WAL retention on a low-traffic Postgres table so the replication slot doesn't lag and pin WAL indefinitely?](<../../../PRACTICE/02_Data_Engineering/Pipelines_and_Integration/Change_Data_Capture/Q-06132_How_do_you_tune_Debezium_heartbeat_and_WAL_retention_on_a_lo.md>)
- [Deep] [Describe how you'd validate a new Debezium connector in staging before production: what edge cases (large txns, DDL, deletes) must the test cover?](<../../../PRACTICE/02_Data_Engineering/Pipelines_and_Integration/Change_Data_Capture/Q-06149_Describe_how_you_d_validate_a_new_Debezium_connector_in_stag.md>)
- [Expert] [Why would you choose an incremental snapshot (DBLog/Debezium) over a blocking snapshot for a 2 TB table and what watermarking makes it correct?](<../../../PRACTICE/02_Data_Engineering/Pipelines_and_Integration/Change_Data_Capture/Q-06108_Why_would_you_choose_an_incremental_snapshot_DBLog_Debezium_.md>)
- [Expert] [Explain how logical replication slots and publications in Postgres 16/17 shape a Debezium setup, and what changed you'd exploit for large fleets.](<../../../PRACTICE/02_Data_Engineering/Pipelines_and_Integration/Change_Data_Capture/Q-06163_Explain_how_logical_replication_slots_and_publications_in_Po.md>)

## Related concepts

- [Handling Source Changes and Spikes](<../ETL_ELT/Handling_Source_Changes_and_Spikes.md>) *(ETL ELT)*
- [Quarantine Topics](<../../../TOOLS/Apache_Kafka/Kafka_Core/Quarantine_Topics.md>) *(Kafka Core)*
- [Data Warehouse Fundamentals](<../../Modeling_and_Warehousing/Data_Warehousing/Data_Warehouse_Fundamentals.md>) *(Data Warehousing)*

## References

- *Data Pipelines Pocket Reference* — James Densmore — Data Ingestion Extracting Data › Streaming Data Ingestions with Kafka and Debezium
- *AWS Certified Data Engineer Associate Study Guide* — Sakti Mishra et al. — Chapter 6 › Chapter 9

