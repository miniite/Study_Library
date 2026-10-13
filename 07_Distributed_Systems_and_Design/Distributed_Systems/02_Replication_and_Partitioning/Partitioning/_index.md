---
id: "07_Distributed_Systems_and_Design/Distributed_Systems/02_Replication_and_Partitioning/Partitioning"
level: 4
type: "curriculum"
curriculum: "data_engineering"
---

# Partitioning Key-Value Data

**L4 Advanced** · [↑ Replication and Partitioning](<../_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** Partitioning assigns each key to exactly one partition so data and load spread evenly; key-range partitioning keeps keys sorted for range scans but risks hot spots, while hash (and consistent-hash) partitioning spreads load evenly but gives up ordered scans.

### 1. The big picture
Replication copies the same data; partitioning (sharding) splits a dataset that is too big, or too write-heavy, for one node. The goal is that 10 nodes handle 10× the data and 10× the throughput of one. That only holds if the split is fair. **Skew** means some partitions hold more data or traffic than others; the extreme case is a **hot spot**, one partition taking nearly all the load while nine nodes idle.

The naive fix, assigning records to nodes at random, balances perfectly but makes it impossible to know where a record lives, so every read must query all nodes. A usable scheme must map a key deterministically to a partition. The two classic families are key-range and hash partitioning; consistent hashing is a hash variant designed to minimize data movement when the cluster changes.

### 2. Core concepts and vocabulary
- **Partition (shard)** — a slice of the keyspace owned by one node (and replicated to others).
- **Skew / hot spot** — uneven data or load; a hot key concentrates traffic.
- **Key-range partitioning** — each partition owns a contiguous range of keys (Bigtable, HBase, early MongoDB, RethinkDB).
- **Hash partitioning** — partition = f(hash(key)); hash range is divided among partitions (Cassandra, MongoDB hashed, DynamoDB, Riak).
- **Consistent hashing** — hash ring where each node owns the arc up to its position; adding/removing a node moves only about K/n keys from neighbours.
- **Compound key** — first part hashed to choose partition, remaining parts sorted within partition (Cassandra).
- **Virtual nodes (vnodes)** — many ring positions per physical node to smooth balance.

### 3. How it works, step by step
**Key range**:
1. Choose boundaries (manual, or automatic splitting as partitions grow).
2. A key is routed by binary search over boundaries.
3. Inside a partition keys stay sorted (SSTables/LSM), so range scans touch one or few partitions.
Risk: access patterns can concentrate. If the key is a timestamp, all of today's writes hit one partition while others sit idle. Fix: prefix the key with something else (sensor ID, then time).

**Hash**:
1. Apply a hash function (MD5, Murmur3; *not* the language's built-in `hashCode`, which may differ between processes) to the key.
2. Map the hash value into a partition's hash range.
3. Neighbouring keys scatter across partitions: range queries must be sent to all partitions (scatter-gather). Cassandra's compound primary key keeps range queries on the second column within one partition.
Hot keys are still possible: a celebrity user ID is one key. Application-level mitigation: append a random 2-digit suffix to split writes over 100 keys, at the cost of merging reads.

**Consistent hashing**:
1. Place nodes (and their vnodes) on a ring of hash values.
2. A key goes to the first node clockwise from its hash.
3. A node joining takes over part of its successor's arc; a node leaving hands its arc to its successor, so only neighbours move data.

### 4. Worked example
```text
Key = sensor_id || timestamp

Range-by-timestamp only:   all current writes -> newest partition (hot)
Key (sensor_id, timestamp): writes spread over sensors; per-sensor time range
                             scans stay inside one partition (compound key)

Ring: hash space 0..99, nodes A@10 B@40 C@70
  key hash 55 -> C (first node clockwise)
  add D@55: only keys in (40,55] move from C to D; A and B are untouched
```
Check: with 4 nodes and naive `hash(key) mod 4`, adding a 5th node changes the owner of ~80% of keys; consistent hashing moves ~20%.

### 5. Under the hood
- **`mod N` is a trap**: changing N relocates almost every key, hence fixed number of partitions or consistent hashing.
- **Hash function choice** matters for uniformity, not security; MD5 or Murmur3 are typical.
- **Vnodes** give each physical node many small arcs so adding a node takes load evenly from many peers and heterogeneous machines can take proportionally more.
- **Skewed workloads** cannot be solved by hashing alone; the system or application must detect and split hot keys.
- **Range partitioning** adapts boundaries to data by splitting a partition at a size threshold (e.g., HBase splits regions at ~10 GB).
- **Metadata**: the mapping key range/hash → node must be stored and cached by routers (see Rebalancing and Routing).

### 6. Performance and design implications
- Single-key operations scale linearly with partitions if load is uniform.
- Range scans: cheap under range partitioning; scatter-gather and tail-latency amplified under hashing.
- Choose partition keys from access patterns: high cardinality, even access, aligned with the queries.
- Cross-partition joins, transactions and unique constraints are expensive; model data to co-locate what is read together.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| Cassandra `PRIMARY KEY ((a), b, c)` | Partition key a, clustering b,c | Composite partitioning |
| MongoDB `sh.shardCollection(ns, {k: "hashed"})` | Hashed sharding | Even write distribution |
| Kafka partitioner / key | Hash of key picks partition | Per-key ordering |
| `num_tokens` (Cassandra vnodes) | Number of vnodes | Balance vs repair overhead |
| HBase region pre-splitting | Create boundaries up front | Avoid initial hot region |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| Key range: range scans, time-ordered or lexicographic queries | Monotonic keys (timestamps, sequences) as leading key |
| Hash: uniform write/read load on point lookups | Range queries are central |
| Compound (hash + sorted): per-entity ordered data | Entity is itself massive or hot |
| Consistent hashing: elastic clusters | Need strict range ordering |

### 9. Common mistakes
- Sequential IDs/timestamps as partition key → hot partition → hash prefix or composite key.
- `hash(key) % N` with variable N → mass reshuffle → fixed partitions or consistent hashing.
- Using a language's native hash → inconsistent partition across clients → use a stable hash.
- Ignoring celebrity keys → a single hot partition → salt or split hot keys.

### 10. 🎯 Interview kit
**30-second answer:** Partitioning splits data so each key lives in one partition. Key-range partitioning preserves order for scans but risks hot spots; hash partitioning balances load but loses range queries; consistent hashing minimizes movement when nodes change. Choose the key by access pattern and handle hot keys explicitly.

**Structure a longer answer:** goal & skew → range scheme → hash scheme → hybrids (compound keys, vnodes) → hot key handling → cross-partition costs.

**Likely follow-ups:**
- *Why not hash mod N?* → rebalancing moves almost everything.
- *How do you handle a hot key?* → salting/key splitting, caching, dedicated partition.
- *How to do range queries on hashed data?* → scatter-gather or compound key.

**Signals interviewers look for:** reasoning from access patterns, awareness of hot spots, and explicit mention of trade-offs.

### 11. Check yourself
- [ ] Show why timestamp keys create a hot spot and fix it.
- [ ] Compute how many keys move under mod N vs consistent hashing when adding a node.
- [ ] Explain vnodes.
- [ ] Describe how a compound key combines both schemes.
<!-- explainer:end -->

## In this section (study in order)

1. [Secondary Indexes in Partitioned Data](<Secondary_Indexes_Partitioned/_index.md>) (L5 Expert)
2. [Rebalancing and Request Routing](<Rebalancing_and_Routing/_index.md>) (L5 Expert)

## References

- *Designing Data-Intensive Applications* — Martin Kleppmann — Partitioning › Partitioning by Hash of Key
- *Designing Data-Intensive Applications* — Martin Kleppmann — Partitioning › Skewed Workloads and Relieving Hot Spots
- *Designing Data-Intensive Applications* — Martin Kleppmann — Partitioning › Partitioning of Key-Value Data
- *Database Internals* — Alex Petrov — Distributed Transactions › Consistent Hashing
- *Designing Data-Intensive Applications* — Martin Kleppmann — Partitioning › Partitioning by Key Range
- *Designing Data-Intensive Applications* — Martin Kleppmann — Storage and Retrieval › Hash Indexes
- *Designing Data-Intensive Applications* — Martin Kleppmann — S
- *Designing Data-Intensive Applications* — Martin Kleppmann — Partitioning › Fixed number of partitions
- *Fundamentals of Data Engineering* — Joe Reis, Matt Housley — Modeling Streaming Data › Shuffle hash join
- *Designing Data-Intensive Applications* — Martin Kleppmann — Partitioning › Dynamic partitioning

