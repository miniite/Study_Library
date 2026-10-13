---
id: "TOOLS/Hadoop/03_Ecosystem/ZooKeeper"
level: 4
type: "curriculum"
curriculum: "data_engineering"
---

# ZooKeeper

**L4 Advanced** · [↑ Hadoop Ecosystem](<../_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** ZooKeeper is a small, replicated, strongly consistent hierarchical store that distributed systems use for coordination — membership, leader election, configuration and locks — so each system doesn't have to write its own consensus code.

### 1. The big picture
Distributed programs repeatedly need the same primitives: "who is alive?", "who is the leader?", "what is the current configuration?", "only one of us may do this". Getting these right across failures and network partitions is hard (it needs consensus). ZooKeeper packages them as a service. Think of it as a **highly available filesystem for tiny pieces of coordination data**, not as a place to store big data.

In the Hadoop ecosystem it provides NameNode HA failover, HBase master election and region-server tracking, Kafka (older versions) broker and controller metadata, Hive locks, and many other uses. It runs as an **ensemble** of an odd number of servers (3 or 5), and tolerates the loss of a minority.

### 2. Core concepts and vocabulary
- **Znode** — a node in the hierarchy; both holds a small amount of data (like a file, <1 MB, usually bytes) *and* can have children (like a directory).
- **Persistent znode** — stays until explicitly deleted.
- **Ephemeral znode** — tied to a client **session**; deleted automatically when the session ends. Cannot have children.
- **Sequential znode** — ZooKeeper appends a monotonically increasing counter to the name (`lock-0000000007`).
- **Session** — a client's connection state with a timeout; kept alive by heartbeats.
- **Watch** — a one-shot notification the client registers on a znode (data change, child list change, deletion).
- **Ensemble / quorum** — the set of servers; writes need acknowledgement from a majority.
- **Leader / follower** — the leader orders all writes; followers serve reads and replicate.
- **ZAB (ZooKeeper Atomic Broadcast)** — the protocol that gives total ordering and atomic delivery of updates.
- **Epoch / zxid** — leader term number and the transaction ID (epoch + counter) that totally orders changes.

### 3. How it works, step by step
**Data model — group membership example.** Parent znode `/zoo` is the group; each live member creates an ephemeral child `/zoo/duck`, `/zoo/goat`. When a member crashes its session expires and its znode disappears, so listing the children always shows live members.

```
/
└── zoo               (persistent)
    ├── duck          (ephemeral — owned by session A)
    └── goat          (ephemeral — owned by session B)
```

**Write path (ZAB, simplified)**
1. Client sends a write to any server; a follower forwards it to the leader.
2. Leader assigns a zxid and **proposes** the change to followers.
3. Each follower logs it and **acknowledges**.
4. When a majority has acknowledged, the leader **commits** and followers apply it in order.
5. The server the client is connected to replies.

**Reads** are served locally by whichever server the client is connected to — fast, but possibly slightly stale (`sync` forces catching up).

**Leader election in ZAB** has three phases: *discovery* (prospective leader learns the latest epoch from everyone and proposes a higher one), *synchronization* (lagging followers are brought up to date and acknowledge the new leader), then *broadcast* (normal operation: two message rounds per write).

### 4. Worked example
```text
$ zkCli.sh -server zk1:2181
[zk] create /zoo ""
[zk] create -e /zoo/duck "host-a"       # ephemeral
[zk] create -s /locks/job- ""           # sequential: /locks/job-0000000001
[zk] ls /zoo watch                      # get notified when membership changes
[zk] get /zoo/duck
[zk] stat /zoo
```
Distributed-lock recipe: each contender creates an ephemeral+sequential znode under `/lock`; the one with the lowest number holds the lock; everyone else watches only the znode just before theirs (avoiding a thundering herd). When the holder's session dies, its znode vanishes and the next contender is notified.

Java API: `new ZooKeeper(hosts, sessionTimeoutMs, watcher)` returns immediately, so wait for the `SyncConnected` event (e.g. with a `CountDownLatch`) before using it; `zk.create(path, data, ACL, CreateMode.EPHEMERAL)` registers a member.

### 5. Under the hood
- All state is held **in memory** for speed, with a transaction log and periodic snapshots on disk; a hard size limit per znode keeps it small.
- Guarantees: sequential consistency (updates from one client applied in order), atomicity (all or nothing), single system image, durability once acknowledged, and timeliness (clients see updates within a bounded time).
- ZAB differs from Paxos/Raft in details but delivers a long-lived leader with a total order of events; recovery streams missing messages from one up-to-date process.
- **Quorum math**: an ensemble of 2f+1 survives f failures — 3 servers tolerate 1, 5 tolerate 2. Even-sized ensembles add cost without extra tolerance.
- Session timeouts trade failure-detection speed against false positives (long GC pauses can expire sessions).
- Watches are one-shot and delivered before the client sees the new data; re-register after each event.

### 6. Performance and design implications
- Read-heavy workloads scale with more followers; writes don't (everything goes through the leader and a quorum).
- Keep the ensemble small (3 or 5), on dedicated machines with a dedicated disk for the transaction log.
- Don't store application data or high-frequency counters in znodes; store *coordination state*.
- Monitor with four-letter commands / metrics (`ruok`, `mntr`, outstanding requests, latency, znode count).
- Avoid watches on huge numbers of nodes (watch storms) and avoid very large child lists.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `zkServer.sh start|status` | Start a server / see leader-follower role | Operations |
| `zkCli.sh` | Interactive client | Inspection, debugging |
| `create [-e] [-s]`, `get`, `set`, `ls`, `delete`, `stat` | znode operations | Everyday use |
| `tickTime`, `initLimit`, `syncLimit` | Time base and election/sync timeouts | Tuning |
| `dataDir`, `dataLogDir` | Snapshot dir / transaction log dir | Put log on its own disk |
| `server.N=host:2888:3888` | Ensemble membership (quorum and election ports) | Cluster setup |
| `clientPort` (2181) | Client connection port | Firewall/config |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| Leader election, locks, config, membership across services | Storing bulk data or acting as a database |
| You need strong, ordered consistency | You need massive write throughput |
| Odd-sized small ensemble per region/cluster | Stretching one ensemble across high-latency WANs |
| Newer systems offering built-in Raft (e.g. KRaft) may remove the dependency | You already run it for Hadoop/HBase — reuse it |

### 9. Common mistakes
- Even number of servers (e.g. 4) → no more fault tolerance than 3 → use 3 or 5.
- Long GC pauses in clients → session expiry → spurious failovers → tune heap, increase timeout.
- Using it as a queue/DB with large payloads → latency and memory blow-up → keep data tiny.
- Forgetting watches are one-shot → missed events → re-register in the callback.
- Sharing the transaction-log disk with other I/O → slow commits → dedicated disk.

### 10. 🎯 Interview kit
**30-second answer:** ZooKeeper is a replicated coordination service with a filesystem-like namespace of znodes. It offers ordered, consistent updates via ZAB, ephemeral nodes for liveness, sequential nodes for ordering and watches for notification, which are the building blocks for election, locks and membership.

**Structure a longer answer:** clarify → core mechanism → trade-offs → close with a real scenario.

**Likely follow-ups:**
- *Why odd ensemble sizes?* → majority quorum; 2f+1 tolerates f failures.
- *Ephemeral vs persistent znode?* → session-bound vs explicit deletion.
- *How would you implement leader election?* → lowest sequential ephemeral znode wins, others watch predecessor.
- *Are reads linearizable?* → no, served locally; use `sync` for freshness.

**Signals interviewers look for:** you tie primitives to recipes and understand the cost and limits of consensus.

### 11. Check yourself
- [ ] Explain ephemeral, sequential and persistent znodes with one use for each.
- [ ] Describe the ZAB write path and what a majority means.
- [ ] Implement a lock recipe on paper without a thundering herd.
- [ ] List three Hadoop-ecosystem components that depend on ZooKeeper and why.
<!-- explainer:end -->

## References

- *Hadoop: The Definitive Guide* — Tom White — X › Z
- *Hadoop: The Definitive Guide* — Tom White — ZooKeeper › Note
- *Hadoop: The Definitive Guide* — Tom White — ZooKeeper › Group Membership in ZooKeeper
- *Hadoop: The Definitive Guide* — Tom White — ZooKeeper › The ZooKeeper Service
- *Hadoop: The Definitive Guide* — Tom White — ZooKeeper › Consistency
- *Hadoop: The Definitive Guide* — Tom White — ZooKeeper › Joining a Group
- *Hadoop: The Definitive Guide* — Tom White — ZooKeeper › Implementation
- *Database Internals* — Alex Petrov — Chapter 14. Consensus › Zookeeper Atomic Broadcast (ZAB)
- *Hadoop: The Definitive Guide* — Tom White — ZooKeeper › Creating the Group
- *Hadoop: The Definitive Guide* — Tom White — ZooKeeper › APIs

