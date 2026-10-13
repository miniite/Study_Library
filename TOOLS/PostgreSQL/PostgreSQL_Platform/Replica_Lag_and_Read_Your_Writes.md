---
id: "TOOLS/PostgreSQL/PostgreSQL_Platform/Replica_Lag_and_Read_Your_Writes"
level: 4
type: "concept"
topic: "TOOLS/PostgreSQL/PostgreSQL_Platform"
tools: ["PostgreSQL", "AWS"]
---

# Replica Lag and Read Your Writes

**L4 Advanced** · [↑ PostgreSQL Platform](<_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** PostgreSQL streaming replicas apply WAL asynchronously, so a read sent to a replica right after a write to the primary can miss it — you manage this by measuring lag, routing the right reads to the primary (or to a caught-up replica), and choosing a synchronous-commit level that matches how much staleness each feature can tolerate.

### 1. The big picture
Scaling reads by adding replicas is the cheapest way to take load off a primary. The price is **replication lag**: the replica is always some distance behind. Most of the time it is milliseconds; during a bulk load, a long-running query conflict or a network hiccup it can be minutes. The user-visible symptom is the *read-your-writes anomaly*: a user saves a profile, the page reloads from a replica, and the old value appears. Their own write seems to have vanished.

This lesson sits between replication topology (primary/standby, cascading, logical) and application design (routing, caching, idempotency). The book view of the same problem in leader-based systems is "eventual consistency" and "read-after-write consistency"; distributed stores like Cassandra solve it with tunable consistency levels, whereas PostgreSQL streaming replication keeps a *single writer* and gives you lag-aware routing and synchronous-commit options instead.

### 2. Core concepts and vocabulary
- **WAL (write-ahead log)** — the sequential log of every change; replicas replay it.
- **LSN (log sequence number)** — a byte position in the WAL; the common currency for measuring "how far".
- **Streaming replication** — standby connects to the primary's walsender and receives WAL continuously.
- **Hot standby** — a replica that accepts read-only queries while replaying WAL.
- **Replication lag** — the gap between primary's current WAL position and what the replica has *received*, *flushed* or *replayed* (three different numbers).
- **Replay lag vs. write/flush lag** — data is only visible to queries once *replayed*; a replica can have received WAL it hasn't applied yet.
- **Read-your-writes** — a session always sees its own earlier writes.
- **Monotonic reads** — a session never sees data go "back in time" when successive reads hit different replicas.
- **`synchronous_commit`** — per-transaction setting for how far WAL must travel before `COMMIT` returns.
- **Replication slot** — server-side bookmark that stops the primary from discarding WAL the replica still needs.
- **Recovery conflict** — a replica query clashing with WAL replay (e.g. vacuum removed rows the query still needs), causing a cancel or a delay.

### 3. How it works, step by step
1. A client commits on the primary; the commit record is written to WAL and flushed locally.
2. The walsender process streams the WAL to each standby's walreceiver.
3. The standby **writes** the WAL to its own files, then **flushes** (fsync), then the startup process **replays** it into data pages. Only replayed changes are visible to read queries.
4. Depending on `synchronous_commit`, the primary's `COMMIT` returns after: local flush (`local`), standby write (`remote_write`), standby flush (`on`), or standby replay (`remote_apply`). With `off`, it does not even wait for local flush.
5. Standbys report their positions back; the primary exposes them in `pg_stat_replication` (`write_lsn`, `flush_lsn`, `replay_lsn`, and the `*_lag` intervals).
6. If a replica query needs rows that replay wants to remove, replay waits up to `max_standby_streaming_delay` (then cancels the query) — or the standby sets `hot_standby_feedback` so the primary keeps old row versions longer, at the cost of bloat.

| Setting | Commit waits for | Data-loss risk on primary failure | Replica sees it at commit time? |
|---|---|---|---|
| `off` | nothing durable | recent commits may be lost even locally | no |
| `local` | local WAL flush | lost if failover to an unsynced replica | no |
| `remote_write` | standby OS received it | tiny | not guaranteed |
| `on` (with sync standby) | standby flushed it | none for that standby | not guaranteed |
| `remote_apply` | standby replayed it | none | **yes** |

### 4. Worked example
```sql
-- On the primary: how far behind is each replica?
SELECT application_name,
       state,
       sync_state,
       pg_wal_lsn_diff(pg_current_wal_lsn(), replay_lsn) AS replay_bytes_behind,
       write_lag, flush_lag, replay_lag
FROM pg_stat_replication;

-- On a replica: how stale is it, in time?
SELECT now() - pg_last_xact_replay_timestamp() AS replay_delay,
       pg_last_wal_receive_lsn() AS received,
       pg_last_wal_replay_lsn()  AS replayed;
```
Reading it: if `replay_bytes_behind` is large but `flush_lag` is small, the network is fine and the standby is slow *applying* (single-threaded replay, IO bound, or blocked by a recovery conflict). If `write_lag` is large, the bottleneck is network or the primary's WAL generation. Note that `now() - pg_last_xact_replay_timestamp()` also grows when the primary is simply **idle** — no new transactions means an old timestamp, not necessarily lag.

**Fixing a read-your-writes bug — LSN-token approach:**
```sql
-- 1. After the write, on the primary, capture the commit position
SELECT pg_current_wal_lsn();            -- e.g. 0/3000148   (store it in the user's session)

-- 2. On the replica, before serving that user's next read:
SELECT pg_last_wal_replay_lsn() >= '0/3000148'::pg_lsn AS caught_up;
```
If `caught_up` is false, route the read to the primary (or wait briefly and retry). The pattern is the same in every driver: *remember the LSN; read from a replica only when it has replayed past it.*

### 5. Under the hood
- Replay is **single-process** per standby (the startup process), even if the primary produced WAL from hundreds of concurrent backends. A burst of heavy writes can therefore outrun replay and create lag that only shrinks when the burst ends. Prefetching (`recovery_prefetch`, PG 15+) helps IO-bound replay.
- Lag has *two* ways to be harmful: **staleness** (users see old data) and **WAL retention** (the primary keeps WAL for a lagging slot, which can fill the disk — protect with `max_slot_wal_keep_size`).
- Synchronous replication uses `synchronous_standby_names` with `FIRST n (…)` or `ANY n (…)` quorum syntax. `ANY 1 (a, b)` tolerates one slow replica; a single named sync standby makes every commit wait for it, and if it dies, writes stall until you change the setting.
- `remote_apply` makes `COMMIT` wait for replay, giving read-your-writes **for reads on that standby** — but not monotonic reads across several standbys unless all are in the sync set.
- Cassandra-style quorum math (R + W > N) does not apply: PostgreSQL has one writer, so the only stale readers are replicas, and "consistency level" is expressed by *where you read* and *what the commit waited for*.

### 6. Performance and design implications
- Treat lag as an SLO: alert on `replay_lag` seconds, not just bytes (bytes mean little without a rate).
- Route by **freshness need**, not blanket policy: profile page after edit → primary or LSN-checked replica; dashboards and search → any replica; reports → a dedicated replica with a large `max_standby_streaming_delay`.
- Sticky routing ("read from primary for N seconds after a write by this user") is the simplest app-level fix; it costs primary load only for recently-active writers.
- `synchronous_commit = remote_apply` raises commit latency by one network round trip plus replay time; use it per transaction (`SET LOCAL synchronous_commit = 'remote_apply'`) only for writes that must be read immediately elsewhere.
- Long queries on replicas cause conflicts: either accept cancellations, raise the delay, or enable `hot_standby_feedback` and watch bloat on the primary.
- Pooler awareness: PgBouncer does not route reads; use a driver or proxy (pgpool, HAProxy+Patroni roles, app-level routing) that understands primary vs. replica endpoints.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `pg_stat_replication` | Per-replica write/flush/replay LSN and lag (on primary) | Lag dashboards |
| `pg_last_wal_replay_lsn()` / `pg_last_xact_replay_timestamp()` | Replica's replay position/time | Freshness check before reading |
| `pg_current_wal_lsn()` | Primary's write position | Capture token after a write |
| `pg_wal_lsn_diff(a, b)` | Bytes between two LSNs | Convert positions to lag |
| `synchronous_commit` | How far commit waits | Per-transaction durability/visibility |
| `synchronous_standby_names` | Defines sync set / quorum | Zero-RPO or read-after-write |
| `hot_standby_feedback` | Replica tells primary which rows it still needs | Long replica queries |
| `max_standby_streaming_delay` | How long replay waits on conflicting queries | Reporting replicas |
| `max_slot_wal_keep_size` | Cap WAL held by slots | Protect primary disk |
| `recovery_min_apply_delay` | Deliberately delay replay | "Oops" protection replica |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| Async replicas + sticky/LSN routing — read scaling with bounded staleness | You need zero data loss on failover (async can lose the tail of WAL) |
| `synchronous_commit = on` with a quorum — durability without a single point of stall | Cross-region sync standbys: every commit pays the WAN RTT |
| `remote_apply` for a few critical writes | Using it globally — throughput drops, one slow standby hurts everyone |
| `hot_standby_feedback = on` for long reports | Primary already suffers bloat — it delays vacuum cleanup |
| Reading from the primary after a write | Primary is the bottleneck — you've defeated the purpose of replicas |

### 9. Common mistakes
- Sending all reads to replicas "for scale" → users see their own writes disappear → add sticky or LSN-aware routing.
- Measuring lag only in seconds on an idle system → false alarms or false calm → combine byte lag, `replay_lag` and a heartbeat table.
- Assuming synchronous replication means replicas are instantly readable → only `remote_apply` guarantees visibility → pick the level deliberately.
- Leaving an abandoned replication slot → primary disk fills with WAL → drop unused slots, set `max_slot_wal_keep_size`.
- Running long analytic queries on a replica with default settings → cancellations (`conflict with recovery`) → dedicated replica or feedback.
- Caching a replica read, then trusting it as fresh → compounding staleness → include versions/ETags.

### 10. 🎯 Interview kit
**30-second answer:** Streaming replicas apply WAL after the primary commits, so a read there can be stale. I measure lag with `pg_stat_replication` and `pg_last_wal_replay_lsn`, fix read-your-writes by routing recent writers to the primary or checking a replica has replayed past the user's commit LSN, and use `synchronous_commit` levels (up to `remote_apply`) only where the business needs it.

**Structure a longer answer:** clarify the consistency requirement per feature → explain write/flush/replay pipeline and where lag appears → routing strategies (sticky, LSN token, per-transaction `remote_apply`) → trade-offs (latency, availability, bloat) → close with an incident story (e.g. a bulk update created 10 minutes of lag).

**Likely follow-ups:**
- *Difference between write, flush and replay lag?* → received vs. durable vs. visible to queries.
- *Does synchronous replication guarantee read-your-writes?* → only with `remote_apply`, and only on those standbys.
- *What happens if the sync standby goes down?* → commits block unless quorum or the setting is changed.
- *Why can a replica query be cancelled?* → recovery conflict with vacuum cleanup/locks; tune delay or feedback.
- *How is this different from Cassandra?* → single writer vs. tunable quorum reads/writes.

**Signals interviewers look for:** you separate staleness from durability; you name concrete views and functions; you route by requirement instead of "all reads to replicas".

### 11. Check yourself
- [ ] Draw the path of a commit from primary to a visible row on a standby, naming write/flush/replay.
- [ ] Write the query that shows replay lag per replica on the primary.
- [ ] Implement read-your-writes using an LSN token and explain the fallback.
- [ ] Compare `on` and `remote_apply` and say which gives read-after-write.
- [ ] Explain why a replica can look "lagging" on an idle primary.
- [ ] List two ways a lagging or dead replica can hurt the primary.
<!-- explainer:end -->

## Implementations (tools)

- [PostgreSQL](<../_index.md>)
- [AWS](<../../AWS/_index.md>)

## Practice questions

- [Intermediate] [A LISTEN/NOTIFY payload hit the 8000-byte limit and the client got truncated data; how would you restructure to send only an id in the payload and fetch the row, and why is that more robust?](<../../../PRACTICE/TOOLS/PostgreSQL/PostgreSQL_Platform/Q-36330_A_LISTEN_NOTIFY_payload_hit_the_8000-byte_limit_and_the_clie.md>)
- [Deep] [You need to run analytics queries without disturbing OLTP; would you use a logical replica, a physical read replica, or a separate ETL to a warehouse, and how does each handle long-running scans differently?](<../../../PRACTICE/TOOLS/PostgreSQL/PostgreSQL_Platform/Q-36272_You_need_to_run_analytics_queries_without_disturbing_OLTP_wo.md>)
- [Deep] [You need change data capture from Postgres to Kafka; compare Debezium reading the logical replication stream against polling an updated_at column, on completeness (deletes) and consistency guarantees.](<../../../PRACTICE/TOOLS/PostgreSQL/PostgreSQL_Platform/Q-36326_You_need_change_data_capture_from_Postgres_to_Kafka_compare_.md>)
- [Deep] [What is read replica and how does streaming replication work in PostgreSQL?](<../../../PRACTICE/TOOLS/PostgreSQL/PostgreSQL_Platform/Q-36376_What_is_read_replica_and_how_does_streaming_replication_work.md>)
- [Deep] [A reporting query aggregates over 100M rows and blocks the OLTP workload; how would you decide between a materialized view, a read replica, or offloading to a warehouse like Redshift/Snowflake?](<../../../PRACTICE/TOOLS/PostgreSQL/PostgreSQL_Platform/Q-35957_A_reporting_query_aggregates_over_100M_rows_and_blocks_the_O.md>)
- [Deep] [Your service reads a config row on every request and it's a Postgres hotspot; how would you decide between a materialized cache in-process, LISTEN/NOTIFY invalidation, and a short TTL, and how do you avoid stampedes?](<../../../PRACTICE/TOOLS/PostgreSQL/PostgreSQL_Platform/Q-36073_Your_service_reads_a_config_row_on_every_request_and_it_s_a_.md>)
- [Expert] [What is a failover and how would a read replica be promoted to primary in a managed setup like RDS or Aurora?](<../../../PRACTICE/TOOLS/PostgreSQL/PostgreSQL_Platform/Q-35909_What_is_a_failover_and_how_would_a_read_replica_be_promoted_.md>)
- [Expert] [You migrated to PgBouncer transaction mode and prepared statements started failing; explain the root cause and the options to keep prepared statements or driver-side caching working.](<../../../PRACTICE/TOOLS/PostgreSQL/PostgreSQL_Platform/Q-35947_You_migrated_to_PgBouncer_transaction_mode_and_prepared_stat.md>)

## Related concepts

- [Read Your Writes in MySQL](<../../MySQL/MySQL_Platform/Read_Your_Writes_in_MySQL.md>) *(MySQL Platform)*
- [Zero Downtime Sharding Migrations](<../../../07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Distributed_Joins_in_MPP_Systems/Zero_Downtime_Sharding_Migrations.md>) *(Distributed Systems Core)*
- [Read Your Writes with Replicas](<../../../06_Cloud_Computing/Cloud_Foundations/Cloud_Fundamentals/Read_Your_Writes_with_Replicas.md>) *(Cloud Fundamentals)*

## References

- *Building Data Integration Solutions* — Jay Borthen — Creating a PostgreSQL Database in the Rocky Linux AWS EC2 Instance › If a presigned URL was used earlier,
- *Database Reliability Engineering* — Laine Campbell, Charity Majors — Infrastructure Engineering › Disabling Swap
- *Data Contracts: Developing Production-Grade Pipelines at Scale* — Chad Sanderson, Mark Freeman — Implementing Data Contracts › Run Unit Tests and See Error for Missing Asset in Catalog

