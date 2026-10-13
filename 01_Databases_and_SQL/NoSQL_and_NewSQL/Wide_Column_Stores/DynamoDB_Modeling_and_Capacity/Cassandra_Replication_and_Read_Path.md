---
id: "01_Databases_and_SQL/NoSQL_and_NewSQL/Wide_Column_Stores/Cassandra_Replication_and_Read_Path"
level: 5
type: "deep-dive"
topic: "01_Databases_and_SQL/NoSQL_and_NewSQL/Wide_Column_Stores"
tools: ["Cassandra", "AWS"]
---

# Cassandra Replication and Read Path

**L5 Expert** · [↑ Wide Column Stores](<../_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** Cassandra copies every partition to RF nodes, lets any node coordinate a request, and lets you choose per query how many replicas must answer, so you trade consistency against latency and availability on every operation.

### 1. The big picture
Cassandra is a peer-to-peer, wide-column store: there is no leader, every node is equal, and data is spread over a token ring by hashing the partition key. Replication copies each partition to several nodes, and the *consistency level* (CL) decides how many of those copies must respond before the client gets an answer. Everything else in this lesson (coordinators, hinted handoff, read repair, anti-entropy repair, tombstones, gc_grace_seconds) exists to make "any replica may be missing a write" a survivable condition.

This sits under the wide-column family: it explains *why* Cassandra stays writable during failures, and *why* operators must run repair on a schedule. The sibling lessons cover the DynamoDB equivalent and the data-modeling rules that make these mechanisms cheap.

### 2. Core concepts and vocabulary
- **Token / ring** — the partition key is hashed to a token; each node owns ranges of tokens.
- **vnode (virtual node)** — each physical node owns many small token ranges (default 16 in recent versions, 256 historically) instead of one big range. Adding or removing a node moves many small ranges from many peers in parallel, so streaming is spread out and no manual token calculation is needed.
- **Replication factor (RF)** — how many nodes store each partition. Set per keyspace.
- **Replication strategy** — `SimpleStrategy` (single DC, testing) or `NetworkTopologyStrategy` (per-DC RF, rack/AZ aware; use this in production).
- **Coordinator** — the node the client happens to contact. It is not special; it routes the request to the replicas for that partition key.
- **Consistency level (CL)** — number of replicas that must acknowledge a request: `ONE`, `TWO`, `THREE`, `QUORUM`, `LOCAL_QUORUM`, `EACH_QUORUM`, `ALL`, `ANY` (writes only).
- **Quorum** — `floor(RF/2) + 1`. For RF=3 that is 2; for RF=5 it is 3.
- **Hint** — a record a coordinator stores for a replica that was down during a write, replayed when it returns.
- **Tombstone** — a delete marker. A delete in Cassandra is a write.
- **gc_grace_seconds** — how long a tombstone must be kept before compaction may purge it (default 864000 s = 10 days).
- **Repair (anti-entropy)** — comparing replicas with Merkle trees and streaming the differences.
- **Read repair** — fixing stale replicas opportunistically when a read notices a mismatch.

### 3. How it works, step by step
**Write at RF=3, CL=QUORUM**
1. The client sends the mutation to any node; that node becomes the coordinator.
2. The coordinator hashes the partition key, finds the 3 replicas, and sends the mutation to **all three** (not just two) at once. With `NetworkTopologyStrategy`, remote-DC replicas get one forwarded message that the remote DC's node fans out locally.
3. Each replica appends to its **commit log** (durability) and applies to its **memtable** (in memory), then acks.
4. As soon as 2 acks arrive, the coordinator answers the client. The third replica is still written; if it is slow or down, the coordinator stores a hint (see below).
5. Memtables later flush to immutable **SSTables**; compaction merges SSTables in the background.

**Read at CL=QUORUM**
1. Coordinator picks replicas (preferring fast ones by snitch/latency score). It asks one for the full data and others for a **digest** (hash) of the data.
2. If digests match, it returns the data. If not, it fetches full data from the disagreeing replicas, takes the cell with the **newest write timestamp** per column, returns that, and writes the winner back to stale replicas (**read repair**).
3. On each replica a read may need the memtable plus several SSTables; the replica merges them and applies tombstones.

**Why QUORUM reads + QUORUM writes give read-your-writes:** if W + R > RF, the write set and read set must overlap in at least one replica that has the latest value. With RF=3, 2 + 2 > 3. If you drop reads to `ONE`, then W=2, R=1 gives 3, not > 3, so a read can land on the one replica that missed the write and return stale data.

**Hinted handoff**
1. A replica is unreachable (down or partitioned) during a write.
2. The coordinator stores a hint locally (in the hints directory), for at most `max_hint_window` (default 3 hours of the node being down).
3. When gossip reports the node up, hints are replayed.
4. Hints do **not** count toward the CL (except `ANY`): they are not readable, only a catch-up aid. Hints are best-effort; a node that was down longer than the window needs a repair.

Riak-style *sloppy quorums* are different: a write may be accepted by non-replica nodes to hit the quorum count. Cassandra does not do this for normal CLs, which is why its hints never make a read see the data.

### 4. Worked example
```sql
CREATE KEYSPACE shop
  WITH replication = {'class': 'NetworkTopologyStrategy', 'dc1': 3};

CREATE TABLE shop.orders_by_user (
  user_id uuid, order_ts timestamp, order_id uuid, total decimal,
  PRIMARY KEY ((user_id), order_ts)
) WITH CLUSTERING ORDER BY (order_ts DESC)
  AND gc_grace_seconds = 864000;

CONSISTENCY LOCAL_QUORUM;
INSERT INTO shop.orders_by_user (user_id, order_ts, order_id, total)
VALUES (3f1c..., toTimestamp(now()), uuid(), 19.99);

-- trace what the coordinator did
TRACING ON;
SELECT * FROM shop.orders_by_user WHERE user_id = 3f1c... LIMIT 10;
```
Read the trace top to bottom: *"Sending READ message to /10.0.0.2"* shows which replicas the coordinator picked; *"Merging data from memtables and N sstables"* shows read amplification; *"Read N live rows and M tombstone cells"* is the number to watch for delete-heavy tables.

**AZ failure scenario.** RF=3 with one replica in each of 3 AZs and one AZ lost: 2 of 3 replicas are alive. `QUORUM` (2) and `LOCAL_QUORUM` (2) still succeed for both reads and writes, so you keep full availability *and* read-your-writes. `ALL` and `THREE` fail. Note the question-bank claim that you need to drop writes to `ONE` is unnecessary. The durability risk you accept is that, while the AZ is down, each write exists on only 2 nodes; one more node loss before the AZ returns loses quorum for the partitions it holds.

**Delete-then-reinsert resurrection.** Tombstones and data compare by timestamp. A DELETE written with timestamp T and a later INSERT at T+1 is fine. Trouble comes from (a) client-supplied timestamps that go backwards, which makes the old tombstone win over the new data, and (b) a replica that missed the delete and is brought back after `gc_grace_seconds` passed: the other replicas have already purged the tombstone, so repair streams the stale row back and the deleted data reappears ("zombie"). Fix: repair every node at least once within `gc_grace_seconds`.

### 5. Under the hood
- **Storage engine = LSM tree.** Writes are sequential (commit log append + memtable insert), so they are cheap; reads pay by checking memtable plus possibly many SSTables (read amplification). Mitigations per SSTable: **Bloom filter** (probabilistic "definitely not here"), **partition index/summary**, **key cache**, and compaction to reduce SSTable count. Strategies: size-tiered (write-heavy), leveled (read-heavy, more write I/O), time-window (TTL time series).
- **Coordinator selection and speculative retry.** A slow replica can be bypassed by `speculative_retry` sending an extra read to another replica after a latency percentile.
- **Failure detection.** Gossip plus the phi-accrual failure detector decides when a node is "down" and so when hints start.
- **Anti-entropy repair.** Each replica builds a Merkle tree over a token range; trees are compared and only mismatching ranges are streamed. *Incremental* repair marks repaired SSTables to avoid re-validating them. Repair is I/O and CPU heavy, so it is usually run per token subrange and scheduled (e.g. via Reaper).
- **Tombstone types.** Cell, row, range and partition tombstones, plus TTL expiry (which creates tombstones at expiry). They live in SSTables until compaction can drop them after gc_grace. `tombstone_warn_threshold` (1000) and `tombstone_failure_threshold` (100000) abort or warn on reads scanning too many.
- **Last-write-wins.** Conflict resolution is per cell by timestamp; ties broken by value. Clock skew between clients/nodes therefore silently decides winners, so run NTP.
- **Bootstrap.** A new node picks vnodes, streams the data for its ranges from existing replicas while not serving reads for them, then joins. Reads during bootstrap are served by the old owners, so consistency is not degraded; the cost is streaming I/O, throttled by `stream_throughput_outbound`.

### 6. Performance and design implications
- Latency at CL=ONE or LOCAL_QUORUM is local-DC latency; EACH_QUORUM/QUORUM across DCs pays WAN round trips. Prefer `LOCAL_QUORUM` for both reads and writes in multi-DC.
- Multi-DC write amplification is inherent: every write ships to each DC's replicas. To cut cross-region traffic, reduce the number of DCs that hold full RF for a given keyspace (e.g. RF 3 local + 1 remote), not just lower the CL: the write still goes to all replicas whatever the CL.
- Dropping RF from 5 to 3 saves nodes but moves quorum from 3-of-5 (tolerates 2 failures) to 2-of-3 (tolerates 1). Evaluate per region by failure domains, repair behavior and the lost headroom during maintenance.
- Watch: `nodetool tpstats` (dropped MUTATION messages = overloaded, pending/blocked pools), `nodetool tablestats` (SSTable count, max partition size, tombstones per read), `nodetool proxyhistograms` (coordinator latency), GC pauses in `gc.log`, disk iowait.
- Wide partitions (hundreds of MB) hurt compaction, repair and read latency; cap them with bucketed partition keys.
- Heavy analytics on the operational cluster causes latency spikes; move it to a Spark job against a separate DC or an exported copy.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `CONSISTENCY LOCAL_QUORUM;` (cqlsh) | Sets CL for the session | Testing a CL choice |
| `TRACING ON;` | Per-request trace | Explaining a slow read |
| `nodetool status` | Ring, load, up/down | First check in any incident |
| `nodetool repair -pr` | Repair node's primary ranges | Scheduled anti-entropy |
| `nodetool tpstats` | Thread-pool stats, dropped messages | Write timeouts under load |
| `nodetool tablestats ks.tbl` | SSTables, partition size, tombstones | Wide partitions / delete storms |
| `nodetool compactionstats` | Pending compactions | Reads slowing over time |
| `nodetool decommission` / `removenode` | Remove a node safely / when dead | Scale-in |
| `gc_grace_seconds` | Tombstone retention | Lower only if repair cadence allows |
| `max_hint_window` | How long hints are stored | Tune for planned downtime |
| `speculative_retry` | Extra read after percentile | Tail-latency control |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| `LOCAL_QUORUM` read+write: strong in-DC consistency, low latency | You need cross-DC read-your-writes: use `QUORUM`/`EACH_QUORUM` and accept WAN latency |
| `ONE` for non-critical, high-volume telemetry | Reads that must see recent writes, such as OTP or balance data |
| `ALL` for rare admin-grade operations | Anything user-facing: one slow node blocks you |
| Vnodes for easy scaling | Very large vnode counts slow repair; keep the default modest |
| TTL for expiry | A hot table with high-rate TTL expiry produces tombstone storms; use time-window compaction and time-bucketed partitions so whole SSTables expire |
| Lightweight transactions (Paxos) for compare-and-set | High-throughput paths: LWT costs several round trips |

### 9. Common mistakes
- Assuming QUORUM depends on node count → it depends on **RF** → compute `floor(RF/2)+1`.
- Reading at `ONE` after writing at `QUORUM` and expecting fresh data → R+W ≤ RF → use QUORUM reads or accept staleness.
- Never running repair → deleted rows resurrect after gc_grace → schedule repair more often than gc_grace_seconds.
- Delete-then-reinsert in a loop on one partition (queue pattern) → tombstones pile up, reads scan them → model with time buckets and let partitions age out.
- Adding or removing several nodes at once at peak → streaming saturates I/O → one at a time, off-peak, throttled.
- Relying on hints for consistency → hints expire and are not counted toward CL → repair is the real guarantee.
- Client-supplied timestamps from unsynchronised clocks → silently lost writes → use server timestamps or sync clocks.

### 10. 🎯 Interview kit
**30-second answer:** Cassandra replicates each partition to RF nodes; any node can coordinate; the CL says how many replicas must ack. QUORUM reads and writes overlap so you read your writes. Hints, read repair and anti-entropy repair converge replicas; deletes are tombstones kept for gc_grace_seconds, so repair must run inside that window.

**Structure a longer answer:** clarify the workload and RF/DC layout → write path (coordinator → all replicas → commit log/memtable → ack at CL) → read path (digest vs data, timestamps, read repair, SSTable merge) → failure handling (hints, repair, gc_grace) → trade-offs and a real scenario such as an AZ outage.

**Likely follow-ups:**
- *What happens to the third replica on a QUORUM write?* → It still receives the write; if down, a hint is stored.
- *Why can one read touch many SSTables?* → LSM writes flush immutable files; each may hold part of the row; compaction reduces but never eliminates this.
- *What if a node is down longer than gc_grace and rejoins?* → It may hold data the cluster deleted; without a prior repair it can resurrect it. Wipe and rebootstrap, or repair first.
- *Why is a delete a write?* → Replicas must learn about it later; a tombstone with a timestamp is the only way to beat older data.

**Signals interviewers look for:** correct quorum math, R+W>RF reasoning, knowing hints ≠ consistency, awareness of tombstone and repair operations, and choosing LOCAL_QUORUM deliberately.

### 11. Check yourself
- [ ] Compute quorum for RF=3, 5 and for a 2-DC RF=3+3 cluster, and say which CLs survive one AZ loss.
- [ ] Trace a QUORUM write with one replica down: who stores what, and when the client is acked.
- [ ] Explain exactly how a deleted row can reappear and which setting and operation prevent it.
- [ ] Read a `TRACING ON` output and point out SSTables consulted and tombstones read.
- [ ] Explain why vnodes make adding a node cheaper and safer than single tokens.
<!-- explainer:end -->

## Implementations (tools)

- [Cassandra](<../../../../TOOLS/Cassandra/_index.md>)
- [AWS](<../../../../TOOLS/AWS/_index.md>)

## Practice questions

- [Deep] [Explain the difference between logical and physical deletes in Cassandra, why a delete is itself a write (tombstone), and how a naive 'delete then reinsert' pattern can resurrect old data.](<../../../../PRACTICE/01_Databases_and_SQL/NoSQL_and_NewSQL/Wide_Column_Stores/Q-05794_Explain_the_difference_between_logical_and_physical_deletes_.md>)
- [Deep] [How do you implement cursor-based pagination over Cassandra clustering columns (paging_state vs manual token) and keep it stable when new rows are inserted mid-scroll?](<../../../../PRACTICE/01_Databases_and_SQL/NoSQL_and_NewSQL/Wide_Column_Stores/Q-05950_How_do_you_implement_cursor-based_pagination_over_Cassandra_.md>)
- [Deep] [When would you denormalize a computed/derived field into the item vs compute it on read? Give a case in a wide-column store where the write-time denormalization creates a consistency headache.](<../../../../PRACTICE/01_Databases_and_SQL/NoSQL_and_NewSQL/Wide_Column_Stores/Q-05981_When_would_you_denormalize_a_computed_derived_field_into_the.md>)
- [Deep] [Your org's incident history shows repeated Cassandra outages from adding/removing nodes during peak. As principal, design a safe fleet-scaling operational standard and the automation/guardrails that make node lifecycle changes routine instead of terrifying.](<../../../../PRACTICE/01_Databases_and_SQL/NoSQL_and_NewSQL/Wide_Column_Stores/Q-05911_Your_org_s_incident_history_shows_repeated_Cassandra_outages.md>)
- [Expert] [Cassandra reads on a table are slow and getting slower over weeks. You suspect tombstones. How do you confirm it, and what causes tombstone buildup? Include gc_grace_seconds in your answer.](<../../../../PRACTICE/01_Databases_and_SQL/NoSQL_and_NewSQL/Wide_Column_Stores/Q-05845_Cassandra_reads_on_a_table_are_slow_and_getting_slower_over_.md>)
- [Expert] [Explain read repair chance vs full/blocking read repair in Cassandra. If read_repair_chance is 0, what other mechanism keeps replicas converged, and how often should you run it?](<../../../../PRACTICE/01_Databases_and_SQL/NoSQL_and_NewSQL/Wide_Column_Stores/Q-05938_Explain_read_repair_chance_vs_full_blocking_read_repair_in_C.md>)
- [Expert] [Explain speculative retry (rapid read protection) in Cassandra and how it trades extra load for tail-latency improvement. When can it make an overloaded cluster worse?](<../../../../PRACTICE/01_Databases_and_SQL/NoSQL_and_NewSQL/Wide_Column_Stores/Q-05962_Explain_speculative_retry_rapid_read_protection_in_Cassandra.md>)
- [Expert] [What's the difference between Cassandra's counter columns and a normal update-based counter, why are counters non-idempotent, and what breaks if a counter write is retried after a timeout?](<../../../../PRACTICE/01_Databases_and_SQL/NoSQL_and_NewSQL/Wide_Column_Stores/Q-05997_What_s_the_difference_between_Cassandra_s_counter_columns_an.md>)

## Related concepts

- [Zero Downtime Sharding Migrations](<../../../../07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Distributed_Joins_in_MPP_Systems/Zero_Downtime_Sharding_Migrations.md>) *(Distributed Systems Core)*
- [HA Failover Design](<../../../Database_Operations/High_Availability/HA_Failover_Design.md>) *(High Availability)*
- [Distributed Counters and Consistency](<../../../../07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Distributed_Joins_in_MPP_Systems/Distributed_Counters_and_Consistency.md>) *(Distributed Systems Core)*
- [Primary-replica Databases](<../../../../07_Distributed_Systems_and_Design/System_Design/System_Design_Core/Clickstream_Counting_Pipeline/Primary-replica_Databases.md>) *(System Design Core)*
- [Read Your Writes in MySQL](<../../../../TOOLS/MySQL/MySQL_Platform/Read_Your_Writes_in_MySQL.md>) *(MySQL Platform)*

## References

- *Cassandra: The Definitive Guide* — Jeff Carpenter, Eben Hewitt — has emerged as the key technology for orchestrating the deployment and maintenance of containers
- *Cassandra: The Definitive Guide* — Jeff Carpenter, Eben Hewitt — The strength and relevance of any technology depend on the investment of individuals in a vibrant
- *Cassandra: The Definitive Guide* — Jeff Carpenter, Eben Hewitt — Virtual machine images A frequent model for deployment of Cassandra is to package one of the
- *Cassandra: The Definitive Guide* — Jeff Carpenter, Eben Hewitt — about to design, you'll have to declare it again. Reservation Physical Data Model Now, let's
- *Database Internals* — Alex Petrov — File Formats › Versioning
- *Database Internals* — Alex Petrov — Anti-Entropy and Dissemination › Hinted Handoff
- *Designing Data-Intensive Applications* — Martin Kleppmann — Partitioning › Request Routing
- *Database Reliability Engineering* — Laine Campbell, Charity Majors — Operational Visibility › Distributed Systems Aggregation
- *Database Reliability Engineering* — Laine Campbell, Charity Majors — Data Storage, Indexing, and Replication › Replication availability and capacity
- *Analytics Optimization with Columnstore Indexes in Microsoft SQL Server* — Edward Pollack — NoN-Clustered ColumNstore INdexes oN rowstore tables - LUSTERED COLUM STORE! DEXESO OWSTORE › Managing Hot, Warm, and Cold Transactional Data

