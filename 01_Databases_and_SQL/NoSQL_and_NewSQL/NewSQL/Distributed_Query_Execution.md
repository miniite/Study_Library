---
id: "01_Databases_and_SQL/NoSQL_and_NewSQL/NewSQL/Distributed_Query_Execution"
level: 5
type: "concept"
topic: "01_Databases_and_SQL/NoSQL_and_NewSQL/NewSQL"
tools: []
---

# Distributed Query Execution

**L5 Expert** · [↑ NewSQL](<_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** A NewSQL database splits every table into key ranges replicated by consensus, so a query is planned centrally but executed as many small flows that run next to the data — and the cost model is dominated by how many ranges, nodes and regions a query or transaction has to touch.

### 1. The big picture
A single-node planner (PostgreSQL, SQL Server) assumes that "reading a row" costs a page fetch. In a distributed SQL system (Google Spanner, CockroachDB, YugabyteDB, TiDB) a row lives in one *range/split* out of thousands, each range is replicated three or more times, and one replica (the leader or leaseholder) is allowed to serve consistent reads and order writes. The optimizer therefore has two jobs: pick a good logical plan, and decide **where each operator runs** so that filters, partial aggregates and joins happen *at the data* instead of dragging rows over the network.

This lesson sits under *NewSQL*: it explains the execution side. Replication/consensus is covered by the consensus lessons; evaluation of products is in the companion lesson on NewSQL evaluation and resilience.

### 2. Core concepts and vocabulary
- **Split / range** — a contiguous slice of the primary-key space (Spanner: *split*; CockroachDB/TiKV: *range*/*region*). The unit of replication, leadership and rebalancing.
- **Replica group** — the 3–5 copies of one split kept in sync by Paxos (Spanner) or Raft (CockroachDB, TiKV, YugabyteDB).
- **Leader / leaseholder** — the replica that serves reads and sequences writes for a split.
- **Gateway / coordinator node** — the node that receives the SQL, plans it, and stitches results back together.
- **Distributed union / flow** — the plan operator (Spanner: *distributed union*; CockroachDB: *DistSQL flow*) that ships a sub-plan to the nodes holding the data and merges the streams.
- **Pushdown** — moving filters, projections, limits and partial aggregation into the sub-plan that runs at the data.
- **Interleaving / co-location** — storing child rows physically inside or beside their parent row so a parent–child join is local to one split (Spanner `INTERLEAVE IN PARENT`, CockroachDB `INTERLEAVE` historically, now locality/colocation features).
- **Two-phase commit (2PC)** — the protocol that makes a write touching several splits atomic.
- **TrueTime / commit wait** — Spanner's bounded-uncertainty clock; a writer waits out the uncertainty before releasing locks.
- **External consistency** — if transaction T1 commits before T2 starts, T1's timestamp is smaller than T2's (linearizable ordering, even across shards).

### 3. How it works, step by step
**A read query**
1. The client connects to any node (the gateway). The SQL is parsed and optimized there, using table statistics (row counts, histograms) gathered per table, not per split.
2. The optimizer looks up key spans: `WHERE user_id = 42` becomes a single-key span; `WHERE created_at > …` with no usable index becomes a *full scan of every split*.
3. Spans are mapped to splits, and each split to its leader/leaseholder. The plan is cut into **flows** — one per node that owns relevant ranges.
4. Each flow runs the pushed-down operators locally: scan → filter → project → partial aggregate/top-N.
5. Partial results stream back to the gateway (or to an intermediate node for a distributed hash join/aggregation), where final aggregation, `ORDER BY` merge and `LIMIT` are applied.
6. The answer is returned. Latency ≈ slowest flow + network hops; cost ≈ rows scanned across *all* splits touched.

**A write transaction**
1. Writes inside the transaction are buffered; reads inside it take locks or use MVCC timestamps.
2. If all mutated keys sit in one split, the commit is a single consensus round in that replica group (one quorum round trip).
3. If keys span splits, one split's leader becomes the **2PC coordinator**: *prepare* is replicated in each participant group, then *commit* is replicated. That is two quorum round trips plus the coordinator hop — the "multi-partition tax".
4. In Spanner the commit timestamp is chosen from TrueTime and the leader performs **commit wait** (typically a handful of ms) so no later transaction can observe an earlier timestamp. In CockroachDB, a hybrid logical clock plus an uncertainty interval can force a read to restart instead.

| Operation | Typical cost driver |
|---|---|
| Point read by full primary key | 1 split, 1 leader round trip (or local replica if stale read) |
| Range scan on a key prefix | Number of splits the range spans |
| Full table scan | All splits, all nodes — plus you pay for the bytes scanned |
| Single-split write | 1 consensus round |
| Cross-split write | 2PC: ≥2 consensus rounds + commit wait |
| Multi-region strong read | Round trip to the leader's region unless served by a follower with a bounded/exact stale read |

### 4. Worked example
Spanner-style schema where the hierarchy decides locality:
```sql
CREATE TABLE Customers (
  CustomerId INT64 NOT NULL,
  Name STRING(100),
) PRIMARY KEY (CustomerId);

CREATE TABLE Orders (
  CustomerId INT64 NOT NULL,
  OrderId    INT64 NOT NULL,
  Total      NUMERIC,
  CreatedAt  TIMESTAMP,
) PRIMARY KEY (CustomerId, OrderId),
  INTERLEAVE IN PARENT Customers ON DELETE CASCADE;
```
```sql
-- Local: all orders for one customer live in the same split as that customer
SELECT c.Name, o.OrderId, o.Total
FROM Customers c JOIN Orders o USING (CustomerId)
WHERE c.CustomerId = 42;

-- Global: no key prefix → fans out to every split, then merges
SELECT DATE(o.CreatedAt) d, SUM(o.Total)
FROM Orders o
GROUP BY d;
```
Read the plan (Spanner: query plan visualizer / `EXPLAIN ANALYZE`; CockroachDB: `EXPLAIN (DISTSQL) …` or `EXPLAIN ANALYZE`). In the first query you see a single *distributed union* with a small row count. In the second you see a distributed union over all splits with a *local hash aggregate* under it and a *global aggregate* above it; "rows scanned" equals the table size. The first query scales with customers, the second with table size — nothing in the SQL text tells you that, only the key layout does.

In CockroachDB the equivalent check is:
```sql
EXPLAIN ANALYZE SELECT count(*) FROM orders WHERE created_at > now() - interval '1 day';
-- look at: "distribution: full", "rows read", "KV time", "network usage", "regions"
```

### 5. Under the hood
- **Keys are the whole story.** Every table and index is a sorted key–value map; secondary indexes are separate key ranges, so using an index may mean a *second* hop to the base row (an "index join" / "back join") on a different split. `STORING` (CockroachDB) or `STORING` clause (Spanner) covering columns avoid that hop at the price of write amplification.
- **Load-based splitting.** Ranges are split when they grow or get hot, and merged when cold. A monotonically increasing key (timestamp, auto-increment) makes the last split hot because every insert lands there; the fix is to spread keys — hash/shard prefix, bit-reversed sequence, or UUID — accepting that range scans by time become fan-outs.
- **Optimizer differences from a single node.** Cost models include network and RPC cost per split; join order and join *algorithm* choices (lookup join vs hash join vs merge join) depend on how many rows must cross the network. A naive join between two large tables that are not co-located turns into one RPC per row for a lookup join — the classic "cross-region RPC explosion".
- **Vectorised execution.** CockroachDB and TiDB process columnar batches inside the flow for scans/aggregations; Spanner offers batched operators ("batch scan") so lookup joins ship key batches rather than single keys.
- **Mutation limits.** A single Spanner commit is bounded (documented limits are on the order of tens of thousands of mutations — counting each indexed column — and a commit size of about 100 MB; check current quotas). CockroachDB similarly recommends batching writes in the low thousands of rows per statement. Bulk ingestion therefore uses many moderate transactions or a bulk path (Spanner Dataflow connector/batch-write, CockroachDB `IMPORT`/`COPY`), never one giant transaction.
- **Stale reads.** A bounded-staleness or exact-staleness read picks a timestamp in the past and can be served by the nearest replica that is caught up to that timestamp, avoiding the leader hop and any lock waiting. Spanner exposes `max_staleness`/`exact_staleness`; CockroachDB offers `AS OF SYSTEM TIME` and follower reads.

### 6. Performance and design implications
- **Design the key before the schema.** Put the column you always filter by first; co-locate rows that are updated together; avoid keys that start with a timestamp.
- **Keep transactions single-split where possible.** Aggregate roots (customer + their orders) in one hierarchy; use idempotent, retry-safe writes across roots rather than 2PC chatter.
- **Measure** with the engine's own tooling: Spanner *query statistics* tables (`SPANNER_SYS.QUERY_STATS_TOP_MINUTE`, lock stats, read stats, key visualizer), CockroachDB DB Console → *Statements*, *Hot ranges*, `crdb_internal.cluster_queries`, `EXPLAIN ANALYZE`.
- **Watch scan bytes.** Compute is billed in processing units/nodes, so a full distributed scan hurts everyone's latency, not just yours. Put guard rails on ad-hoc queries (query timeouts, statement limits, read-only analytical replicas or an export to a warehouse).
- **Read from the right place.** Use stale reads for analytics dashboards; keep strong reads for money-moving paths.
- **Latency floor.** A strongly consistent global write cannot be faster than a quorum round trip between the replicas' regions plus commit wait; place the leader near the writers and accept slower reads elsewhere, or pin data to a region.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `EXPLAIN ANALYZE` (CockroachDB, YugabyteDB, TiDB) | Runs the query and reports rows, time and network per operator | Find which operator fans out |
| `EXPLAIN (DISTSQL)` | Gives a diagram of flows and nodes | See how the plan is distributed |
| Spanner query plan + `SPANNER_SYS` stats | Per-query CPU, rows scanned, latency, locks | Hunt the expensive statements |
| `AS OF SYSTEM TIME` / `exact_staleness` | Read a past consistent snapshot, from nearest replica | Analytics, dashboards, backfills |
| `INTERLEAVE IN PARENT` / table families / locality | Co-locate related rows | Parent–child access patterns |
| `SHOW RANGES FROM TABLE` / key visualizer | Shows splits and hot keys | Diagnose hotspots |
| Leader/lease preferences, `ALTER … SET LOCALITY` | Puts the writer-side replica in a chosen region | Reduce write latency for a user base |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| Interleave/co-locate child with parent for hierarchical access | Hierarchy may change later (re-parenting means a full rewrite), or one parent can grow to a huge size (a hot, unsplittable root) |
| Hash-shard or UUID keys for very high insert rate | You also need frequent time-ordered range scans — each becomes a fan-out |
| Strong reads everywhere | Cross-region read latency matters more than seeing the last write |
| Covering/storing indexes | Write-heavy tables — each extra index adds a replicated write |
| Single global database | A regional data-residency law makes pinning rows by region necessary |

### 9. Common mistakes
- Treating the cluster as "Postgres with more nodes" → unbounded scans and cross-split transactions → decide access paths from the key, check plans.
- Sequential keys on a hot insert path → one split takes all writes → shard/hash/UUID the key prefix.
- Giant transactions or single-statement bulk loads → hit mutation/size limits or starve consensus → batch to a few thousand rows, or use the bulk path.
- Assuming a join across two large, unrelated tables is cheap → thousands of cross-region RPCs → co-locate, denormalise, or pre-aggregate.
- Missing index on a frequent filter → a full distributed scan on every call → add the index and verify with `EXPLAIN`.
- Forgetting retry handling → serialization/`40001` errors surface to users → wrap transactions in a retry loop.

### 10. 🎯 Interview kit
**30-second answer:** A distributed SQL engine plans on the node you connect to, but cuts the plan into flows that run on whichever nodes lead the key ranges involved, pushing filters and partial aggregates down so only reduced data crosses the network. Single-split transactions need one consensus round; cross-split ones add two-phase commit.

**Structure a longer answer:** clarify the access pattern → show how keys map to splits and leaders → explain flows/pushdown and 2PC → discuss hotspots, stale reads and co-location → close with a concrete schema decision.

**Likely follow-ups:**
- *Why can a JOIN explode into cross-region RPCs?* → lookup joins issue per-row/batch RPCs to the split that owns the key; unrelated tables are not co-located.
- *How do monotonically increasing keys hurt?* → all inserts hit the last split/leader; splitting can't help because the load is on one key range.
- *What is commit wait?* → the leader holds the commit until the clock-uncertainty interval has passed to guarantee external consistency; costs milliseconds of tail latency.
- *When is a stale read right?* → read-heavy, tolerance of seconds-old data, wants nearest replica.

**Signals interviewers look for:** you reason from keys → splits → leaders; you quote the cost of 2PC and cross-region hops; you know that limits exist on transaction size; you mention measuring with plans/stats instead of guessing.

### 11. Check yourself
- [ ] Explain the path of a `SELECT … WHERE customer_id = ?` from gateway to leaseholder and back.
- [ ] Draw a cross-split write as 2PC and count the consensus rounds.
- [ ] Choose a primary key for a 50k-inserts/second event table and justify it.
- [ ] Say when interleaving helps and what its migration hazard is.
- [ ] Read an `EXPLAIN ANALYZE` and identify the operator that fans out to all nodes.
<!-- explainer:end -->

## Practice questions

- [Deep] [Spanner charges for compute in processing units/nodes and storage separately; model the cost of a workload doing 50k QPS multi-region reads and 5k QPS writes, and how you would optimize spend without losing consistency.](<../../../PRACTICE/01_Databases_and_SQL/NoSQL_and_NewSQL/NewSQL/Q-33802_Spanner_charges_for_compute_in_processing_units_nodes_and_st.md>)
- [Deep] [Explain Spanner's interleaved tables and how they physically co-locate child rows with parents; when does interleaving materially improve latency, and what are the migration hazards if hierarchy changes later?](<../../../PRACTICE/01_Databases_and_SQL/NoSQL_and_NewSQL/NewSQL/Q-33807_Explain_Spanner_s_interleaved_tables_and_how_they_physically.md>)
- [Deep] [Spanner offers stale reads with bounded staleness; explain the consistency semantics, how staleness bounds interact with TrueTime, and when you'd use exact-staleness reads for a read-heavy analytics path.](<../../../PRACTICE/01_Databases_and_SQL/NoSQL_and_NewSQL/NewSQL/Q-33810_Spanner_offers_stale_reads_with_bounded_staleness_explain_th.md>)
- [Deep] [Explain how Spanner's query optimizer and distributed execution differ from a single-node Postgres planner, and why a naive JOIN across large interleaved tables can explode into cross-region RPCs.](<../../../PRACTICE/01_Databases_and_SQL/NoSQL_and_NewSQL/NewSQL/Q-33817_Explain_how_Spanner_s_query_optimizer_and_distributed_execut.md>)
- [Deep] [Spanner enforces a mutation limit per transaction and 256 MB row/commit size limits; how do these constraints shape your data model and bulk-write patterns for a high-throughput ingestion pipeline?](<../../../PRACTICE/01_Databases_and_SQL/NoSQL_and_NewSQL/NewSQL/Q-33820_Spanner_enforces_a_mutation_limit_per_transaction_and_256_MB.md>)
- [Deep] [Explain how Spanner's split points and load-based splitting work, why monotonically increasing keys cause hotspots, and the bit-reversed/UUID mitigations you'd recommend for a high-ingest table.](<../../../PRACTICE/01_Databases_and_SQL/NoSQL_and_NewSQL/NewSQL/Q-33830_Explain_how_Spanner_s_split_points_and_load-based_splitting_.md>)
- [Deep] [Spanner's leader-region placement determines write latency; explain how you'd choose leader regions for a write-heavy workload with users concentrated in two continents, and the trade-off against read latency elsewhere.](<../../../PRACTICE/01_Databases_and_SQL/NoSQL_and_NewSQL/NewSQL/Q-33835_Spanner_s_leader-region_placement_determines_write_latency_e.md>)
- [Deep] [Design a global user-identity/auth store on Spanner that must be strongly consistent worldwide with sub-100ms local reads; how do you reconcile that with the write-consensus latency floor?](<../../../PRACTICE/01_Databases_and_SQL/NoSQL_and_NewSQL/NewSQL/Q-33851_Design_a_global_user-identity_auth_store_on_Spanner_that_mus.md>)

## Related concepts

- [Stream Join Data Structures](<../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Stream_Join_Data_Structures.md>) *(Data Structures and Algorithms)*
- [Primary-replica Databases](<../../../07_Distributed_Systems_and_Design/System_Design/System_Design_Core/Clickstream_Counting_Pipeline/Primary-replica_Databases.md>) *(System Design Core)*
- [Database Cost Optimization](<../../Database_Strategy/Cost_Optimization/Database_Cost_Optimization.md>) *(Cost Optimization)*

## References

- *Database Internals* — Alex Petrov — Distributed Transactions › Distributed Transactions with Spanner

