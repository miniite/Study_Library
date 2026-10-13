---
id: "01_Databases_and_SQL/NoSQL_and_NewSQL/NewSQL/NewSQL_Evaluation_and_Resilience"
level: 5
type: "concept"
topic: "01_Databases_and_SQL/NoSQL_and_NewSQL/NewSQL"
tools: []
---

# NewSQL Evaluation and Resilience

**L5 Expert** · [↑ NewSQL](<_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** Choosing and operating a NewSQL database means pricing the consensus latency you will pay for strong consistency, deciding which failures you must survive, and proving it with your own workload — not vendor benchmarks.

### 1. The big picture
NewSQL systems (Spanner, CockroachDB, YugabyteDB, TiDB, plus Vitess/Citus as "sharded SQL" alternatives) promise SQL, horizontal scale and serializable transactions. They deliver that by running **consensus per shard** and, for cross-shard writes, two-phase commit. That is real engineering — but not "free": every write waits for a quorum, geographic distribution adds network round trips, and operations (schema change, garbage collection, rebalancing) behave differently from a single node. The evaluation question is therefore *"which failure domain do I need to survive, which latency can I afford, and who operates it?"*

### 2. Core concepts and vocabulary
- **Quorum** — a majority of replicas (2 of 3, 3 of 5). Raft/Paxos commit when a quorum acknowledges.
- **Flexible Paxos** — a refinement: only the quorum used in the *leader-election* phase must intersect the quorum used in the *replication* phase (|Q1| + |Q2| > N); two replication quorums need not intersect, because the leader serialises them. It lets you shrink the replication quorum for latency while enlarging the election quorum.
- **Survival goal** (CockroachDB) — `ZONE` survival (lose one AZ) or `REGION` survival (lose an entire region); the latter places replicas in ≥3 regions and raises write latency.
- **Table locality** — `REGIONAL BY TABLE`, `REGIONAL BY ROW`, `GLOBAL`: where leaseholders live and how reads are served.
- **TrueTime** — Spanner's GPS+atomic-clock API returning an interval `[earliest, latest]`; commits wait out the interval.
- **HLC (hybrid logical clock)** — CockroachDB's timestamp combining physical time and a logical counter, with a *max clock offset* uncertainty window (default 500 ms) — a *clock*, unrelated to HyperLogLog.
- **MVCC garbage collection** — old versions are kept for `gc.ttlseconds` (CockroachDB; default 4 h in recent versions, 25 h earlier) and can be pinned longer by *protected timestamps*.
- **Changefeed / change stream** — CDC output (CockroachDB `CREATE CHANGEFEED`, Spanner change streams) ordered per key, with at-least-once delivery.
- **Retryable error** — a serialization failure (`SQLSTATE 40001`) the client must retry.

### 3. How it works, step by step — an evaluation
1. **Write down the workload facts**: peak read/write QPS, key access distribution, transaction shape (single row? multi-row? cross-entity?), data size and growth, regions and users per region, RPO/RTO, regulatory residency.
2. **Name the failure model**: lose a node? a zone? a region? a cloud? That decides survival goal and replica count.
3. **Compute the latency floor**: a strongly consistent write ≥ the round-trip time to the quorum's slowest *required* replica (+ commit wait/uncertainty). For regions 5 ms apart that is ~10 ms; across continents 80–150 ms.
4. **Check feature fit**: wire protocol (PostgreSQL vs MySQL), ORM support, supported isolation levels, triggers/stored procedures, extensions, foreign keys, sequences.
5. **Model cost**: Spanner bills compute (processing units/nodes) and storage separately, plus replication/network; CockroachDB bills nodes/vCPU (self-hosted or Cloud). Replication factor multiplies storage. Index count multiplies write cost.
6. **Prototype with shadow traffic** or a replay of a production trace; measure p50/p99/p99.9, contention under hot keys, behaviour under failures (kill a node, partition a region), and schema changes.
7. **Plan operations**: backups and PITR, upgrades, online schema change, monitoring, capacity headroom, on-call skills.

### 4. Worked example
Survival goals and locality in CockroachDB:
```sql
ALTER DATABASE app SET PRIMARY REGION "us-east1";
ALTER DATABASE app ADD REGION "us-west1";
ALTER DATABASE app ADD REGION "europe-west1";

-- Survive the loss of an entire region (needs 3+ regions)
ALTER DATABASE app SURVIVE REGION FAILURE;

-- Rows live near their owner; reads/writes for a user stay in that user's region
ALTER TABLE users SET LOCALITY REGIONAL BY ROW;

-- Small, read-mostly reference data: fast reads everywhere, slower writes
ALTER TABLE currencies SET LOCALITY GLOBAL;
```
What changes: with `SURVIVE REGION FAILURE` each range needs a quorum that crosses regions, so *every write* now includes a cross-region round trip. Adding a region to a database that was `ZONE`-surviving can silently change replica placement — the "write latency went up when we added a region" surprise. Verify with `SHOW ZONE CONFIGURATION FROM DATABASE app;` and `SHOW RANGES FROM TABLE users;` before and after.

A retry-safe transaction pattern (any NewSQL):
```python
for attempt in range(max_retries):
    try:
        with conn.begin():
            conn.execute("UPDATE accounts SET bal = bal - %s WHERE id = %s", (amt, src))
            conn.execute("UPDATE accounts SET bal = bal + %s WHERE id = %s", (amt, dst))
        break
    except SerializationFailure:      # SQLSTATE 40001
        sleep(backoff(attempt))
```
Rule: the body must be idempotent from the client's view (no side effects like emails inside the retried block).

### 5. Under the hood
- **Why Spanner and CockroachDB differ.** Spanner uses TrueTime to assign globally meaningful timestamps; a write waits out ε (single-digit ms) so external consistency holds without coordination. CockroachDB has no special hardware; reads that encounter a value inside their uncertainty window are *restarted at a higher timestamp*, which can show up as retries on contended keys. Both give serializable isolation; Spanner also gives external consistency for read-write transactions by default.
- **GC and MVCC bloat.** Deleted/overwritten values stay until older than the GC TTL *and* not pinned. Protected timestamps — used by backups and changefeeds — hold GC back. A stalled changefeed or a long-running backup therefore makes storage balloon and slows scans over tombstones. Remedy: shorten TTL for hot churn tables, cancel/repair the stuck job, and alert on protected-timestamp age.
- **Large transactions.** Each transaction has intents/locks proportional to the write set. Very large ones are slow to resolve and strain Raft. Batch in chunks, use bulk-ingest paths that write below the SQL layer, and use admission control to protect foreground traffic.
- **Schema changes.** Online schema change backfills a new index in the background using a sequence of states (delete-only → write-only → public). It doesn't take table-wide locks but does consume I/O and can stall behind long transactions; run it with canaries on a staging cluster with production-size data.
- **Connection model.** Any node can be a gateway; each connection costs memory on that node. A connection pool (application side, or PgBouncer-like proxy in transaction mode) keeps total sessions bounded; remember that session-level state does not survive transaction-mode pooling.
- **Compatibility gaps.** "PostgreSQL-compatible" means the wire protocol and most dialect; behaviour differs for advisory locks, some isolation semantics, sequences (non-contiguous/unordered), `SERIAL` performance (hotspot), extensions, and error/retry codes.

### 6. Performance and design implications
- Use **UUIDs or hash-sharded indexes** for high-write append tables; do not use sequential integers/timestamps as the leading key.
- **Index discipline.** Each secondary index is another replicated write; covering (`STORING`) indexes trade write cost for read hops.
- **Locality.** Pin data to its users' region; use `GLOBAL` tables only for small read-heavy lookups.
- **Latency budget.** Report the *tax* honestly: consistency costs a quorum round trip; PACELC says even without partitions you trade latency for consistency.
- **Capacity plan.** Expect throughput to scale with nodes only if keys distribute; test with hot-key skew, because uniform benchmarks flatter every system.
- **Observability.** Track p99 by statement fingerprint, retries per transaction, hot ranges, GC queue age, replication lag of followers, changefeed lag.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `SURVIVE ZONE\|REGION FAILURE` | Sets the failure domain the database tolerates | Fix the failure model first |
| `SET LOCALITY …` | Decides leaseholder/replica placement per table | Latency by geography |
| `gc.ttlseconds` zone config | MVCC retention | Trade PITR/AS OF window for storage |
| Protected timestamps view (`crdb_internal`) | Shows what pins GC | Diagnose storage growth |
| `CREATE CHANGEFEED` / Spanner change stream | CDC to Kafka/Pub/Sub | Event-driven downstream, outbox |
| `IMPORT`, `COPY`, batch writes | Bulk ingestion | Large loads |
| `BACKUP … WITH revision_history`, Spanner backups/PITR | Recovery points | Regulated retention plan |
| Admission control settings | Prioritises foreground over background work | Protect SLOs under load |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| Spanner: GCP-native, global consistency, managed ops, TrueTime guarantees | Strong need to avoid cloud lock-in or run on-prem |
| CockroachDB: PostgreSQL-ish, multi-cloud/self-host, rich locality controls | Team can't carry an operational burden of a distributed DB |
| YugabyteDB: reuses PostgreSQL query layer, good Postgres compat | Mixed workloads that need very specific Postgres extensions |
| TiDB: MySQL compat, built-in HTAP (TiFlash columnar replicas) | Workload relies on MySQL features TiDB hasn't implemented |
| Postgres + read replicas/Citus or Vitess | Workload fits one node or a clean shard key and cross-shard transactions are rare — simpler and cheaper |

### 9. Common mistakes
- Taking vendor TPC-C numbers as capacity → production skew and transaction mix differ → shadow traffic and your own replay.
- Sequential primary keys on append tables → range hotspot → hash-sharded index or UUID.
- Assuming 2PC across microservices is fine because the database is distributed → cross-service 2PC isn't offered by the DB → use outbox + CDC + idempotent consumers, or saga.
- Ignoring GC/changefeed interactions → storage explosion → monitor protected timestamps, set lag alerts.
- Adding a region without checking zone configs → higher write latency → review survival goal and placement first.
- No retry loop → 40001 reaches users → centralise a retry helper.
- Zero-downtime migration without a rollback path → can't back out → dual-write with verification, CDC backfill, shadow reads, then cutover with the old system kept in sync until confidence.

### 10. 🎯 Interview kit
**30-second answer:** NewSQL gives horizontally scalable SQL with serializable transactions by running consensus per shard. The price is a quorum round trip per write, 2PC for cross-shard writes, and a different operating model. I choose by failure domain, latency budget, compatibility and operations — and validate with shadow traffic.

**Structure a longer answer:** requirements and failure model → latency floor → compatibility and ops → cost → prove with replay → rollout/rollback.

**Likely follow-ups:**
- *Spanner vs CockroachDB?* → TrueTime/managed/GCP vs HLC/multi-cloud/self-hostable; both serializable; lock-in and ops maturity dominate.
- *Why is a region-failure survival goal slower?* → every write needs a cross-region quorum.
- *How do you do a 30 TB MySQL migration with no downtime?* → schema/compat check, bulk load + CDC catch-up, dual-write or shadow reads, verification, staged cutover, reverse replication for rollback.
- *What's Flexible Paxos for?* → shrinking the commit quorum by enlarging the election quorum, since only phases need to intersect.

**Signals interviewers look for:** quantified latency/cost; awareness of GC, retries, hotspots; honest "when not to use NewSQL"; migration risk management.

### 11. Check yourself
- [ ] State what write latency a REGION-survival database has relative to ZONE survival and why.
- [ ] Explain how a stuck changefeed grows storage.
- [ ] Design a shadow-traffic evaluation for a 20 TB ledger.
- [ ] Write a retry-safe transfer transaction.
- [ ] Argue for Postgres + sharding over NewSQL for one concrete workload.
<!-- explainer:end -->

## Practice questions

- [Deep] [Walk me through how you would decide between Spanner and CockroachDB for a new globally-distributed financial ledger, and what org-level factors (cloud lock-in, TrueTime vs HLC, ops maturity) dominate that decision.](<../../../PRACTICE/01_Databases_and_SQL/NoSQL_and_NewSQL/NewSQL/Q-33797_Walk_me_through_how_you_would_decide_between_Spanner_and_Coc.md>)
- [Deep] [How do you approach connection management and pooling for CockroachDB at 100k+ concurrent app connections, given per-connection memory and the gateway-node model, and where does a proxy like pgbouncer help or hurt?](<../../../PRACTICE/01_Databases_and_SQL/NoSQL_and_NewSQL/NewSQL/Q-33825_How_do_you_approach_connection_management_and_pooling_for_Co.md>)
- [Deep] [How would you architect a saga or outbox pattern on CockroachDB for cross-service transactions, and why can't you just rely on distributed 2PC across microservices even though the DB itself is distributed?](<../../../PRACTICE/01_Databases_and_SQL/NoSQL_and_NewSQL/NewSQL/Q-33852_How_would_you_architect_a_saga_or_outbox_pattern_on_Cockroac.md>)
- [Deep] [Explain how you'd handle very large transactions or bulk imports into CockroachDB without hitting per-transaction limits or overwhelming Raft, using IMPORT, batching, and admission control.](<../../../PRACTICE/01_Databases_and_SQL/NoSQL_and_NewSQL/NewSQL/Q-33878_Explain_how_you_d_handle_very_large_transactions_or_bulk_imp.md>)
- [Expert] [A team wants to migrate a 30 TB sharded MySQL fleet to CockroachDB with zero downtime; design the multi-phase migration strategy including dual-write, CDC backfill, cutover, and rollback safety.](<../../../PRACTICE/01_Databases_and_SQL/NoSQL_and_NewSQL/NewSQL/Q-33801_A_team_wants_to_migrate_a_30_TB_sharded_MySQL_fleet_to_Cockr.md>)
- [Expert] [You inherit a CockroachDB cluster where GC and MVCC garbage are ballooning storage; explain how gc.ttlseconds, protected timestamps, and long-running changefeeds interact, and how you'd remediate.](<../../../PRACTICE/01_Databases_and_SQL/NoSQL_and_NewSQL/NewSQL/Q-33824_You_inherit_a_CockroachDB_cluster_where_GC_and_MVCC_garbage_.md>)
- [Expert] [A team reports that adding a region to CockroachDB unexpectedly increased write latency for existing data; explain what happened with survival goals and replica placement, and how you'd have prevented the surprise.](<../../../PRACTICE/01_Databases_and_SQL/NoSQL_and_NewSQL/NewSQL/Q-33848_A_team_reports_that_adding_a_region_to_CockroachDB_unexpecte.md>)
- [Expert] [How do you plan for and mitigate the operational blast radius of a bad schema migration or index build on a production NewSQL cluster serving millions of users; what guardrails and canaries do you build?](<../../../PRACTICE/01_Databases_and_SQL/NoSQL_and_NewSQL/NewSQL/Q-33866_How_do_you_plan_for_and_mitigate_the_operational_blast_radiu.md>)

## Related concepts

- [Zero Downtime Sharding Migrations](<../../../07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Distributed_Joins_in_MPP_Systems/Zero_Downtime_Sharding_Migrations.md>) *(Distributed Systems Core)*
- [HA Failover Design](<../../Database_Operations/High_Availability/HA_Failover_Design.md>) *(High Availability)*
- [Primary-replica Databases](<../../../07_Distributed_Systems_and_Design/System_Design/System_Design_Core/Clickstream_Counting_Pipeline/Primary-replica_Databases.md>) *(System Design Core)*
- [Choosing a Database for the Workload](<../../Database_Strategy/Database_Selection/Choosing_a_Database_for_the_Workload.md>) *(Database Selection)*
- [Quality Attribute Trade-off Scenarios](<../../../07_Distributed_Systems_and_Design/Software_Architecture/Trade_off_Analysis/Quality_Attribute_Trade-off_Scenarios.md>) *(Trade off Analysis)*

## References

- *Azure Data Engineering Cookbook* — Ahmad Osama — Analyzing Data with Azure Synapse Analytics › How it works...

