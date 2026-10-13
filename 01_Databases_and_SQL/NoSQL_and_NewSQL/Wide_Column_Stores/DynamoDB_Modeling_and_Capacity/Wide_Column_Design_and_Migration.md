---
id: "01_Databases_and_SQL/NoSQL_and_NewSQL/Wide_Column_Stores/Wide_Column_Design_and_Migration"
level: 5
type: "deep-dive"
topic: "01_Databases_and_SQL/NoSQL_and_NewSQL/Wide_Column_Stores"
tools: ["AWS", "Cassandra"]
---

# Wide Column Design and Migration

**L5 Expert** · [↑ Wide Column Stores](<../_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** In a wide-column store you model tables from your queries, choose partition keys that spread load and bound partition size, accept denormalization and eventual consistency, and migrate between engines only with dual writes, shadow reads and a measured cutover.

### 1. The big picture
Cassandra and DynamoDB reward the same discipline: you cannot join or filter freely, so the *query defines the table*. The partition key decides data placement and load distribution; the clustering/sort key decides order within the partition and which range queries are cheap. Mistakes here (hot partitions, unbounded partitions, wrong consistency) are expensive to fix because they require rewriting data, which is why this lesson also covers migration and the organisational guardrails around the choice of engine.

This lesson builds on the replication/read path lesson (how Cassandra stays available) and the DynamoDB capacity lesson (units and limits). Here the focus is design decisions and the migration path.

### 2. Core concepts and vocabulary
- **Partition key** — decides which node/partition holds the row; determines distribution and the unit of atomic, single-partition access.
- **Clustering key (Cassandra) / sort key (DynamoDB)** — orders rows inside a partition; range queries on it are sequential reads.
- **Wide row / wide partition** — a partition with very many clustered rows; useful for time series until it becomes unbounded.
- **Hot partition / hot key** — one key receiving disproportionate traffic.
- **Bucketing** — adding a time bucket or shard number to the partition key to bound size or spread writes.
- **Write sharding** — splitting a hot logical key into N physical keys and combining on read.
- **Denormalization / query table** — storing the same data in multiple tables, one per access pattern.
- **LSM tree / read amplification** — storage layout that makes writes append-only and reads touch multiple files.
- **Last-write-wins (LWW)** — conflict policy picking the highest timestamp.
- **Lightweight transaction (LWT)** — Cassandra's Paxos-based compare-and-set (`IF NOT EXISTS`, `IF col = x`).
- **Shadow read / dual write** — migration techniques to compare engines before cutover.

### 3. How it works, step by step
**Modeling method**
1. List the queries (who asks, filters, order, page size, rate, freshness).
2. For each query, define a table (or index) whose partition key is the equality filter and whose clustering key is the sort/range column.
3. Check partition size: rows per partition × row size, over the retention period. Target well below ~100 MB and 100k rows for Cassandra; for DynamoDB keep the item collection bounded and below throughput ceilings.
4. Check write distribution: will one key take a disproportionate share? If so, bucket or shard.
5. Decide the consistency requirement per query (see below).
6. Write the application so that all copies are updated (batch/transaction/stream) and define the repair path if one fails.

**Partition vs clustering key, and on-disk effect.** Rows with the same partition key are stored together, sorted by clustering columns in the declared order (`CLUSTERING ORDER BY (ts DESC)`). A query for the newest N rows is a short sequential read at the head of the partition; the opposite order may need reverse reads, which are slower. Partition key columns are hashed, so range queries across partition keys are not supported efficiently.

**Why wide-column writes are fast.** Writes append to a log and update memory (LSM), so there is no read-before-write or random page update. The cost shows up on reads: a row may live in the memtable and several SSTables (read amplification), solved with Bloom filters, caches and compaction. This is also why delete-heavy patterns hurt: tombstones must be read and skipped.

### 4. Worked example
**Chat history.** Requirement: show the latest messages of a conversation, page backwards, with some channels having millions of messages.
```sql
-- Cassandra
CREATE TABLE messages (
  conversation_id uuid,
  bucket          text,        -- e.g. '2026-10' (month), bounds partition size
  sent_at         timeuuid,
  sender_id       uuid,
  body            text,
  PRIMARY KEY ((conversation_id, bucket), sent_at)
) WITH CLUSTERING ORDER BY (sent_at DESC);
```
The partition key is `(conversation_id, bucket)`: ordered within a month, bounded by activity per month, and the application walks buckets backwards for older pages. In DynamoDB the same idea is `pk = CONV#<id>#<yyyy-mm>`, `sk = <timeuuid or ULID>`; paginate with `LastEvaluatedKey`, and use consistent reads where missing a just-written message is unacceptable.

**Counters without a hot key.** One global counter row is a single hot partition. Shard it:
```text
pk = ACTIVE_USERS#<region>#<0..15>   sk = 'C'    value (atomic add)
read = sum of the 16 shards per region (cache the sum briefly)
```
Writes spread across 16 partitions per region; the cost is a 16-key read and the fact that the total is only eventually accurate. For truly exact global totals, write events and aggregate asynchronously.

**Wallet balance that must not double-spend.** Use a single-writer-per-account model: keep the balance in one partition, serialise updates with a conditional write (`version = :expected` in DynamoDB; LWT `IF version = ?` in Cassandra), and make operations idempotent with a request ID stored in the same partition/transaction. In DynamoDB, `TransactWriteItems` gives ACID within a region; Global Tables replicate asynchronously with LWW, so two regions writing the same account can lose an update. Give each account a **home region** (region-affinity key) so only one region writes it, and route other regions to it, or fail over by explicit promotion. Cassandra LWT across DCs pays several WAN round trips (Paxos) and still needs a clear contention policy. Neither is "free": for money, many teams keep the ledger in a relational database or a single-region strongly consistent store and use the wide-column store for projections.

### 5. Under the hood
- **Distribution.** Cassandra hashes the partition key with Murmur3 over the token ring; DynamoDB hashes internally and splits partitions by size and load. A monotonic key (timestamp, auto-increment) hashes uniformly in Cassandra but in DynamoDB and in range-partitioned stores (HBase) all current writes land in the last range. The usual warning "never use a timestamp as partition key" is mainly about **time-ordered partition keys with a small active set**: even with hashing, every write for "now" hits the same partition key, so all traffic targets one replica set.
- **Counters.** Cassandra counters are special distributed counters (not idempotent: a retried increment can double count); DynamoDB atomic `ADD` is also non-idempotent on retries unless guarded by a condition. Use event tables plus aggregation if exactness matters.
- **LWW and clock skew.** Timestamps come from clients/coordinators; skew can make an older write win. During migrations, mismatches commonly come from (1) timestamp precision/skew, (2) TTL semantics (Cassandra TTL per cell vs DynamoDB TTL per item, deleted lazily within roughly 48 h), (3) encoding (blobs, decimals, collections, null vs missing attribute, empty strings), and (4) writes landing during the comparison window.
- **Consistency levels vs units.** Cassandra: tune per query via CL and RF. DynamoDB: eventual vs strong read, plus transactions. Both can be made strongly consistent at a latency cost; neither gives multi-partition ACID without an explicit mechanism.
- **Multi-tenancy.** Cassandra isolates by keyspace/table, but I/O, compaction and cache are shared, so noisy neighbours are an operational concern; DynamoDB isolates throughput per table with quotas and on-demand.

### 6. Performance and design implications
- **Cost model.** Cassandra cost is nodes × hours (over-provisioned clusters are a typical hidden bill; right-size by measuring CPU, disk, compaction headroom and per-table traffic). DynamoDB cost is per request/capacity unit and GSIs; per-access-pattern cost is directly visible. Attribute spend by table and tag.
- **Operational load.** Cassandra: repair, compaction tuning, upgrades, capacity planning, JVM GC. DynamoDB: key design, capacity mode, limits. For a startup with two backend engineers that expects 100× growth, managed DynamoDB's reduced operational burden usually outweighs lock-in; revisit when cost per request or feature gaps dominate.
- **Observability to provide per team.** Throttles, consumed vs provisioned capacity, top partition keys (Contributor Insights), p99 latency, partition size and tombstone metrics, and cost per table. Making this visible changes behaviour more than policy documents.
- **Schema review.** Require each new table to document the queries, key choice, expected row/partition growth, TTL/delete behaviour and consistency requirement. Keep it fast with a checklist and automated linting (e.g. unbounded list attributes, timestamp-only partition keys, filter-heavy queries).

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `CLUSTERING ORDER BY` | Fixes sort within a partition | Newest-first reads |
| `nodetool tablestats` / `cfstats` | Partition size, SSTables, tombstones | Detect wide partitions |
| `cassandra-stress` | Load and stress test | Validate a data model before launch |
| `IF NOT EXISTS` / `IF col = ?` (LWT) | Paxos compare-and-set | Rare uniqueness/CAS needs |
| `ConditionExpression` | DynamoDB conditional write | Optimistic concurrency |
| `TransactWriteItems` | Multi-item ACID (region-local) | Invariants spanning items |
| DynamoDB Streams / Cassandra CDC | Change log | Derived tables, migration |
| Export to S3 / `sstableloader` / Spark | Bulk copy | Backfill phase of migration |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| Wide-column store: huge write volume, known key-based queries, multi-AZ/DC availability | Ad-hoc queries, joins, strong multi-row transactions: a relational database (e.g. Postgres) fits better |
| Default to a relational DB and require a justified exception for NoSQL | Setting the rule so rigidly that clear time-series/high-write cases are blocked |
| Shared multi-tenant table | Teams with different SLAs, noisy neighbours, chargeback needs: table (or cluster) per service |
| Write-time denormalization (copies) | Many copies with complex updates: consistency repair burden |
| Move hot aggregates to a cache or relational store | Adding another engine without an owner |
| Build (run Cassandra) | Small team, rapid growth: buy managed |
| DynamoDB Streams CDC | Mixed estates: choose one org CDC standard to avoid fragmented pipelines |

### 9. Common mistakes
- Modeling the entities first and the queries later → full-table scans or many round trips → start from queries.
- Partition key with "now" in it, or unbounded growth → hot or huge partitions → bucket by time, shard by hash.
- Choosing `ONE` for both reads and writes → stale reads → use quorum or strong reads where staleness has a business cost (e.g. OTP verification).
- Treating a follower or like count as a transaction → negative or wrong counts under concurrency → use bounded counters with conditional updates, or accept approximation and reconcile asynchronously.
- Migrating with a single cutover → no way back → dual write, shadow read, compare, then shift traffic gradually.
- Ignoring the comparison semantics → 0.3% mismatches that are "noise" → classify every mismatch (timestamp ordering, TTL, encoding, in-flight write) before deciding.

### 10. 🎯 Interview kit
**30-second answer:** Design from queries: partition key for distribution and bounded size, clustering/sort key for order and ranges, denormalize per access pattern, shard hot keys, pick consistency per operation, and treat a migration as dual write → backfill → shadow read → gradual cutover with rollback.

**Structure a longer answer:** requirements and queries → keys and bucket strategy → consistency/concurrency → failure modes (hot key, unbounded growth, tombstones) → operations and cost → migration or governance plan.

**Likely follow-ups:**
- *Why does a timestamp partition key hurt?* → all current writes target one key/replica set; add a bucket and shard.
- *Why are LSM writes fast and reads costlier?* → append-only writes; reads merge memtable and SSTables; Bloom filters and compaction reduce it.
- *Is 0.3% shadow-read mismatch acceptable?* → Not until each mismatch is classified; clock-skew LWW, TTL differences and encoding are expected causes, but data in money/identity paths should be at zero unexplained mismatches.
- *Can you get global consistency on an AP store?* → Only by constraining writers (home region, single leader per entity) or paying coordination (LWT/transactions) and accepting latency and failure trade-offs.

**Signals interviewers look for:** query-first thinking, concrete bucketing/sharding, awareness of LWW and clock skew, pragmatic engine selection, and an incremental migration plan.

### 11. Check yourself
- [ ] Redesign a table with a timestamp partition key into a bounded, well-distributed one.
- [ ] Explain read amplification and name three things that reduce it.
- [ ] Sketch a sharded global counter and state the accuracy trade-off.
- [ ] Describe a Cassandra → DynamoDB migration with validation gates and rollback.
- [ ] Write a one-paragraph guardrail on when a team must justify a wide-column store over Postgres.
<!-- explainer:end -->

## Implementations (tools)

- [AWS](<../../../../TOOLS/AWS/_index.md>)
- [Cassandra](<../../../../TOOLS/Cassandra/_index.md>)

## Practice questions

- [Intermediate] [Explain the difference between a Cassandra partition key and clustering key, and how the clustering order affects both on-disk storage and the efficiency of range queries.](<../../../../PRACTICE/01_Databases_and_SQL/NoSQL_and_NewSQL/Wide_Column_Stores/Q-05936_Explain_the_difference_between_a_Cassandra_partition_key_and.md>)
- [Deep] [A teammate proposes CL=ALL for reads to 'be safe'. Explain the availability cost, what happens when one replica is down, and how you'd talk them into QUORUM instead.](<../../../../PRACTICE/01_Databases_and_SQL/NoSQL_and_NewSQL/Wide_Column_Stores/Q-05983_A_teammate_proposes_CL_ALL_for_reads_to_be_safe_Explain_the_.md>)
- [Deep] [A principal on another team ships a design that puts a monotonic timestamp partition key on a global write-heavy table. In the design review, how do you explain the multi-region hot-shard failure it will cause at scale and steer them without undermining their authority?](<../../../../PRACTICE/01_Databases_and_SQL/NoSQL_and_NewSQL/Wide_Column_Stores/Q-05852_A_principal_on_another_team_ships_a_design_that_puts_a_monot.md>)
- [Expert] [You must store time-series metrics at 1M points/min and query the last hour per host. Compare a Cassandra TWCS bucketed model vs a DynamoDB composite-key model. Which do you pick and why?](<../../../../PRACTICE/01_Databases_and_SQL/NoSQL_and_NewSQL/Wide_Column_Stores/Q-05918_You_must_store_time-series_metrics_at_1M_points_min_and_quer.md>)
- [Expert] [Your org runs both Cassandra and DynamoDB across teams and the duplication is bleeding money and expertise. As principal, how do you decide which one to standardize on, and how do you migrate the losing side without stalling every roadmap for a year?](<../../../../PRACTICE/01_Databases_and_SQL/NoSQL_and_NewSQL/Wide_Column_Stores/Q-05810_Your_org_runs_both_Cassandra_and_DynamoDB_across_teams_and_t.md>)
- [Expert] [Define the reference architecture for feeding a wide-column operational store into an analytics lakehouse without hammering production. Compare DynamoDB S3 export + Athena, Streams-to-Kinesis-to-warehouse, and Cassandra CDC, and pick a default with justification.](<../../../../PRACTICE/01_Databases_and_SQL/NoSQL_and_NewSQL/Wide_Column_Stores/Q-05833_Define_the_reference_architecture_for_feeding_a_wide-column_.md>)
- [Expert] [Design a hybrid architecture where the write path lands in a wide-column store for scale but a relational read-model is maintained via CDC for complex reporting. Justify the added complexity, and describe how you keep the two from drifting.](<../../../../PRACTICE/01_Databases_and_SQL/NoSQL_and_NewSQL/Wide_Column_Stores/Q-05868_Design_a_hybrid_architecture_where_the_write_path_lands_in_a.md>)
- [Expert] [You're the tie-breaker on whether a new payments service uses Cassandra LWT or DynamoDB TransactWriteItems for its critical invariants. Walk through the head-to-head on correctness guarantees, cost, latency, and multi-region behavior, and make the call.](<../../../../PRACTICE/01_Databases_and_SQL/NoSQL_and_NewSQL/Wide_Column_Stores/Q-05895_You_re_the_tie-breaker_on_whether_a_new_payments_service_use.md>)

## Related concepts

- [Partitioning and Bucketing Design](<../../../../02_Data_Engineering/Storage_and_Lakehouse/Partitioning_and_Bucketing/Partitioning_and_Bucketing_Design.md>) *(Partitioning and Bucketing)*
- [Ordering and Partition Keys](<../../../../07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Ordering_and_Partition_Keys.md>) *(Distributed Systems Core)*
- [Zero Downtime Sharding Migrations](<../../../../07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Distributed_Joins_in_MPP_Systems/Zero_Downtime_Sharding_Migrations.md>) *(Distributed Systems Core)*
- [Distributed Counters and Consistency](<../../../../07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Distributed_Joins_in_MPP_Systems/Distributed_Counters_and_Consistency.md>) *(Distributed Systems Core)*
- [HA Failover Design](<../../../Database_Operations/High_Availability/HA_Failover_Design.md>) *(High Availability)*
- [Choosing a Database for the Workload](<../../../Database_Strategy/Database_Selection/Choosing_a_Database_for_the_Workload.md>) *(Database Selection)*

## References

- *Building Data Integration Solutions* — Jay Borthen — Data Stores and Management Systems › Apache Cassandra and HBase
- *Building Data Integration Solutions* — Jay Borthen — Data Stores and Management Systems › AWS Keyspaces
- *Cassandra: The Definitive Guide* — Jeff Carpenter, Eben Hewitt — has emerged as the key technology for orchestrating the deployment and maintenance of containers
- *Cassandra: The Definitive Guide* — Jeff Carpenter, Eben Hewitt — The strength and relevance of any technology depend on the investment of individuals in a vibrant
- *Cassandra: The Definitive Guide* — Jeff Carpenter, Eben Hewitt — POLYGLOT PERSISTENCE One of the benefits of microservice architecture is that each service is
- *Cassandra: The Definitive Guide* — Jeff Carpenter, Eben Hewitt — When choosing to use Cassandra as the primary underlying database across multiple services, it is
- *AWS Certified Data Engineer Associate Study Guide* — Sakti Mishra et al. — Overview of AWS Analytics and Auxiliary Services › Amazon Redshift
- *AWS Certified Data Engineer Associate Study Guide* — Sakti Mishra et al. — Sample template to create a glue database\ › HA for Amazon MSK
- *AWS Certified Data Engineer Associate Study Guide* — Sakti Mishra et al. — Chapter 6 › Chapter 9
- *AWS Certified Data Engineer Associate Study Guide* — Sakti Mishra et al. — B › C
- *Ultimate AWS Data Engineering* — Rathish Mohan — Architecting for Scalability Data Replication Techniques › Setting Up DynamoDB Global Tables for Cross-Region Replication
- *Ultimate AWS Data Engineering* — Rathish Mohan — Partitioning and Sharding Optimizing Data Management › Best Practices for Partitioning and Sharding in Distributed Systems
- *Data Engineering Design Patterns* — Bartosz Konieczny — Data Ingestion Design Patterns › Metadata
- *Data Engineering Design Patterns* — Bartosz Konieczny — Idempotency Design Patterns › Metadata limits
- *Azure Data Fundamentals* — Michael John Pena — Azure Cosmos DB › Cassandra API in Azure Cosmos DB

