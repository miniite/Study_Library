---
id: "07_Distributed_Systems_and_Design/Distributed_Systems/02_Replication_and_Partitioning"
level: 3
type: "curriculum"
curriculum: "data_engineering"
---

# Replication and Partitioning

**L3 Practitioner** · [↑ Distributed Systems](<../_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** Replication keeps copies of the same data on several nodes for redundancy and read speed; partitioning splits one big dataset across nodes for capacity; real systems combine both.

### 1. The big picture
When one machine is no longer enough (too much data, too many requests, or too little fault tolerance) you spread data across nodes. There are two orthogonal tools. **Replication** answers "what if a node dies, and how do I serve more reads?" by keeping the *same* data in several places. **Partitioning** (sharding) answers "what if the data or write rate exceeds one machine?" by giving each node a *different slice*. Typically each partition is replicated, so every node stores several partitions, and each partition has a few replicas on different nodes (a leader for one, followers for others).

Shared-nothing is the dominant architecture: each node uses its own CPU, memory and disks, and nodes coordinate only over the network. That is cheap and scalable but means all the complexity of coordination lands in software, which is what the following lessons cover.

### 2. Core concepts and vocabulary
- **Replica** — a node holding a copy of the data (or of a partition).
- **Replication** — keeping copies of the same data on multiple nodes.
- **Partition / shard** — a subset of the dataset assigned to a node; also called a region (HBase), tablet (Bigtable), vnode (Cassandra), vBucket (Couchbase).
- **Leader (primary)** and **follower (secondary, read replica)** — the replica that accepts writes and those that copy it.
- **Replication factor (RF)** — number of copies of each piece of data.
- **Shared-nothing** — nodes share no memory or disk; cooperation is over the network.
- **Hot spot / skew** — a partition receiving disproportionate load.
- **Rebalancing** — moving partitions between nodes when nodes join or leave.

### 3. How it works, step by step
Replication of a write in a leader-based system:
1. The client sends the write to the leader.
2. The leader stores it locally and appends it to a replication log.
3. Followers consume the log and apply writes in the same order.
4. Reads can go to the leader or to followers.

Partitioning of a record:
1. Compute the record's partition from its key (key range or hash of key).
2. Route the request to the node owning that partition (via a router or client-side metadata).
3. That node serves the request; its replicas copy the change.

Combined picture: *4 nodes, 2 partitions, RF=2* → partition A lives on node 1 (leader) and node 3 (follower); partition B on node 2 (leader) and node 4 (follower). Losing any node leaves all data available; writes scale with the number of partitions.

### 4. Worked example
```text
Dataset: 6 TB, 20k writes/s, 100k reads/s. One node: 2 TB disk, 5k writes/s.
Partitioning: ceil(6/2)=3 and 20k/5k=4  -> at least 4 partitions
Replication:  RF=3 for fault tolerance  -> 12 partition-replicas
Placement:    spread so no node holds two replicas of the same partition
Reads:        100k/s spread over 3 replicas per partition -> ~8k reads/s per replica
```
Check storage: 6 TB × RF 3 = 18 TB of raw copies, at 2 TB per node that is at least 9 nodes, so storage (not write rate) sets the node count here. The exercise shows storage, write throughput and replication factor all drive sizing; the largest requirement wins.

### 5. Under the hood
- Replication is about *consistency of copies*: every follower must see the same writes in the same order or they diverge. Strategies: single-leader, multi-leader, leaderless (next lessons).
- Partitioning is about *routing and balance*: key-range partitioning supports range scans but risks hot spots; hash partitioning spreads load but loses ordering. Secondary indexes complicate both (document-partitioned vs term-partitioned).
- The two interact: transactions across partitions need distributed commit; replication lag matters per partition; rebalancing must copy replicas without breaking availability.
- Even a "single" large machine uses NUMA, so partitioning of work is needed there too.

### 6. Performance and design implications
- Replication scales *reads* and availability, not writes (single leader still takes all writes).
- Partitioning scales *writes and storage*; cross-partition queries and joins become expensive scatter-gather operations.
- More replicas = more write amplification and replication-lag exposure; more partitions = more metadata and rebalancing work.
- Choose partition keys to match access patterns and avoid hot keys.

### 7. Commands and settings you should know
| Concept | Typical knob | When to use |
|---|---|---|
| Replication factor | `replication_factor` (Cassandra), `min.insync.replicas` + `replication.factor` (Kafka) | Setting durability level |
| Partition count | `num.partitions` (Kafka), shard count (Elasticsearch `number_of_shards`) | Sizing parallelism; often hard to change later |
| Read replicas | Postgres streaming replicas, MySQL replicas | Scale reads, offload analytics |
| Consistency level | Quorum settings | Trade latency against freshness |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| Replication only: dataset fits one node, you need HA and read scale | Writes saturate the leader |
| Partitioning: data/writes exceed one node | Workload is dominated by cross-partition joins/transactions |
| Both: large, critical system | Operational maturity is low; complexity multiplies |

### 9. Common mistakes
- Thinking replication gives write scalability → single leader remains bottleneck → partition.
- Choosing a timestamp as partition key → all writes hit the newest partition → hash or compound key.
- Assuming replicas are instantly consistent → stale reads → design for replication lag.
- Too few partitions → cannot add nodes later without resharding → over-provision partitions modestly.

### 10. 🎯 Interview kit
**30-second answer:** Replication copies the same data to several nodes for fault tolerance and read throughput; partitioning splits data across nodes for capacity and write throughput. They are independent and usually combined: each partition is replicated.

**Structure a longer answer:** state the two goals → describe each mechanism → show combined layout → discuss trade-offs (lag, hot spots, cross-partition queries) → give an example (Kafka topic partitions with RF=3).

**Likely follow-ups:**
- *Does replication help write throughput?* → no, not with a single leader.
- *Key-range vs hash partitioning?* → range supports scans, hash balances load.
- *What breaks when partitioning?* → joins, global secondary indexes, multi-partition transactions.

**Signals interviewers look for:** clean separation of concepts, awareness of skew, and linking to concrete systems.

### 11. Check yourself
- [ ] Draw a cluster of 4 nodes, 2 partitions, RF=2.
- [ ] Explain which problem each mechanism solves.
- [ ] Name two ways partitioning complicates queries.
- [ ] Explain why shared-nothing pushes complexity into software.
<!-- explainer:end -->

## In this section (study in order)

1. [Leader-Follower Replication](<Leader_Follower_Replication/_index.md>) (L4 Advanced)
2. [Multi-Leader Replication](<Multi_Leader_Replication/_index.md>) (L4 Advanced)
3. [Leaderless Replication and Quorums](<Leaderless_Replication/_index.md>) (L4 Advanced)
4. [Anti-Entropy and Gossip](<Anti_Entropy_and_Gossip/_index.md>) (L5 Expert)
5. [Partitioning Key-Value Data](<Partitioning/_index.md>) (L4 Advanced)

## References

- *Designing Data-Intensive Applications* — Martin Kleppmann — Partitioning › Partitioning and Replication
- *Designing Data-Intensive Applications* — Martin Kleppmann — Chapter 5. Replication › Leaders and Followers
- *Designing Data-Intensive Applications* — Martin Kleppmann — Encoding and Evolution › Replication Versus Partitioning
- *Designing Data-Intensive Applications* — Martin Kleppmann — Partitioning › Chapter 6. Partitioning
- *Designing Data-Intensive Applications* — Martin Kleppmann — P
- *Designing Data-Intensive Applications* — Martin Kleppmann — Q › R
- *Designing Data-Intensive Applications* — Martin Kleppmann — Knowledge, Truth, and Lies › The leader and the lock
- *Designing Data-Intensive Applications* — Martin Kleppmann — Glossary

