---
id: "07_Distributed_Systems_and_Design/Distributed_Systems/02_Replication_and_Partitioning/Partitioning/Rebalancing_and_Routing"
level: 5
type: "curriculum"
curriculum: "data_engineering"
---

# Rebalancing and Request Routing

**L5 Expert** · [↑ Partitioning Key-Value Data](<../_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** Rebalancing moves partitions between nodes as the cluster or data changes, using strategies that move as little data as possible, and request routing (via a coordinator, routing tier or client) makes sure each request finds the node that currently owns the key.

### 1. The big picture
Clusters change: data grows, query load rises, nodes die or are added. Partition ownership must shift to keep load and storage balanced, while the database keeps serving requests. That shift is **rebalancing**. After rebalancing, clients need to know where a key now lives, which is the **request routing** (service discovery) problem. Both are control-plane concerns that determine operational pain at scale.

Requirements for rebalancing: after it, load should be fairly shared; the database must keep accepting reads and writes during it; and no more data than necessary should move, to be fast and minimize network and I/O load.

### 2. Core concepts and vocabulary
- **Rebalancing** — moving partitions between nodes.
- **Fixed number of partitions** — create many more partitions than nodes up front; only whole partitions move.
- **Dynamic partitioning** — partitions split when too large and merge when small.
- **Partitioning proportional to nodes** — fixed partitions per node; new nodes split existing partitions at random.
- **Pre-splitting** — creating initial partitions up front for dynamic systems to avoid a single starting partition.
- **Request routing / service discovery** — determining which node serves a key.
- **Routing tier / partition-aware client / gossip** — three routing approaches.
- **Coordination service (ZooKeeper, etcd)** — authoritative store of partition→node mapping.

### 3. How it works, step by step
**Strategies**
1. *Fixed partitions*: e.g., 1,000 partitions over 10 nodes (100 each). A new node steals a few partitions from each existing node until balanced; only whole partitions move, assignment is the only thing that changes. Partition count must be chosen well: too large → costly management and recovery; too small → each partition is huge and rebalances slowly. Used by Riak, Elasticsearch, Couchbase, Kafka.
2. *Dynamic*: a partition exceeding a size threshold (say 10 GB) splits in two; one half may move to another node. Shrunken partitions merge. Number of partitions adapts to data volume. HBase, MongoDB, RethinkDB.
3. *Proportional to nodes*: Cassandra gives each node a fixed number of vnodes (default 256 in older versions); a joining node picks random split points, takes over half of some existing partitions.

**Automatic vs manual**: fully automatic rebalancing is convenient but dangerous: it is expensive (rerouting plus bulk copy), and combined with failure detection can cascade (an overloaded node looks dead, is rebalanced away, load shifts and overloads others). A common compromise is automatic proposal, human approval.

**Routing approaches** for a request to key K:
1. Client contacts any node (e.g., via round-robin load balancer); the node serves it or forwards it (Cassandra, Riak, gossip).
2. A routing tier acts as a partition-aware load balancer (MongoDB `mongos`).
3. Clients know the partition map and connect directly (Kafka clients with metadata).
All need consensus on the current mapping: ZooKeeper tracks it and routers subscribe to changes (Kafka controller, HBase, SolrCloud); gossip disseminates state among nodes (Cassandra).

### 4. Worked example
```text
Cluster: 4 nodes, 64 fixed partitions (16 each). Add node 5.
Target: 64/5 ≈ 12–13 each.
Each old node gives ~3-4 partitions -> node 5 gets 13.
Moved data = 13/64 ≈ 20% of the dataset.  Everything else untouched.
During the move the old owner keeps serving the partition;
once copied, the mapping in ZooKeeper flips and routers update.
```
Compare: `hash(key) mod N` would relocate ~80% of keys on the same change.

### 5. Under the hood
- **Move protocol**: copy snapshot of the partition to the new owner, stream changes made during the copy, then atomically switch ownership in the metadata store; the old owner redirects stragglers and deletes its copy. Throttle bandwidth so foreground traffic is not starved.
- **Hot partitions are not fixed by rebalancing alone**: splitting by size ignores load; some systems split by load too (DynamoDB adaptive capacity, Bigtable load-based splitting).
- **Stale routing**: a client with an old map may reach the wrong node; the node responds with a redirect or error and the client refreshes metadata.
- **Failure detection coupling**: rebalancing triggered by failure detection should wait out a grace period to avoid thrash.
- **Compare with Dynamo-style rings**: consistent hashing makes rebalancing local to ring neighbours; fixed-partition schemes decouple partitioning from membership.

### 6. Performance and design implications
- Rebalancing competes with production I/O; schedule and throttle, and size partitions so a move takes minutes, not hours.
- Many small partitions improve balance and recovery parallelism but add metadata and file handles.
- Keep routing metadata small and cached; the coordination service should not be on the request path of every query.
- Kafka: partition count cannot easily be decreased and changing it breaks key→partition mapping; plan for growth.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `nodetool move`, `nodetool decommission`, `nodetool cleanup` | Ring changes and cleanup | Cassandra topology changes |
| `kafka-reassign-partitions.sh` / Cruise Control | Moves partitions between brokers | Balancing Kafka |
| Elasticsearch `cluster.routing.allocation.*` | Controls shard allocation | Throttle / pin shards |
| MongoDB balancer (`sh.startBalancer`, `sh.setBalancerState`) | Chunk migration control | Maintenance windows |
| HBase `split`, `balance_switch` | Manual region split / balancer toggle | Hot region mitigation |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| Fixed partitions: predictable, simple moves | You cannot estimate data growth |
| Dynamic splitting: unknown/changing data size | Starting empty without pre-splitting (one hot partition) |
| Proportional to nodes: elastic Cassandra-like clusters | Need strong control of placement |
| Manual approval of rebalance | Large fleets where humans become the bottleneck |

### 9. Common mistakes
- Auto-rebalancing on failure detection with short timeouts → cascading load shifts → delay and approve.
- Too few partitions → cannot add nodes meaningfully → over-provision partitions.
- Rebalancing at peak without throttling → latency spikes → cap streaming throughput.
- Clients caching routing forever → errors after moves → TTL/refresh on redirect.

### 10. 🎯 Interview kit
**30-second answer:** Rebalancing reassigns partitions to keep load even as nodes change; fixed partition counts, dynamic splitting and proportional-to-nodes strategies all move only whole partitions rather than rehashing everything. Routing finds the owner via any-node forwarding, a routing tier, or partition-aware clients, using a coordination service or gossip for the map.

**Structure a longer answer:** why rebalance → three strategies with examples → automatic vs manual → routing options → failure modes.

**Likely follow-ups:**
- *Why not mod N?* → huge movement.
- *How does a request find a partition after a move?* → metadata update plus redirect/refresh.
- *What is the danger of automatic rebalancing?* → cascades when combined with failure detection.

**Signals interviewers look for:** quantifying data movement, awareness of operational risk, and naming real systems.

### 11. Check yourself
- [ ] Compute data moved when adding a 5th node to 4 with fixed partitions.
- [ ] Explain dynamic splitting and the pre-splitting problem.
- [ ] Compare the three routing approaches.
- [ ] Describe how ZooKeeper participates.
<!-- explainer:end -->

## References

- *Designing Data-Intensive Applications* — Martin Kleppmann — Partitioning › Strategies for Rebalancing
- *Designing Data-Intensive Applications* — Martin Kleppmann — Partitioning › Request Routing
- *Database Internals* — Alex Petrov — Implementing B-Trees › Rebalancing
- *Designing Data-Intensive Applications* — Martin Kleppmann — Partitioning › Rebalancing Partitions
- *Designing Data-Intensive Applications* — Martin Kleppmann — M
- *Fundamentals of Data Engineering* — Joe Reis, Matt Housley — Designing Good Data Architecture › Tight Versus Loose Coupling: Tiers, Monoliths, and Microservices
- *Designing Data-Intensive Applications* — Martin Kleppmann — Partitioning › Operations: Automatic or Manual Rebalancing
- *Database Internals* — Alex Petrov — B-Tree Variants › Consolidation and Garbage Collection
- *Designing Data-Intensive Applications* — Martin Kleppmann — P
- *Designing Data-Intensive Applications* — Martin Kleppmann — S

