---
id: "TOOLS/Hadoop/01_HDFS/HDFS_Architecture/HDFS_HA_and_Federation"
level: 5
type: "curriculum"
curriculum: "data_engineering"
---

# HDFS High Availability and Federation

**L5 Expert** · [↑ HDFS Architecture](<../_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** Hadoop 2 removes the namenode as a single point of failure with an active/standby pair that shares an edit log (usually through a quorum journal), a ZooKeeper-based failover controller and fencing — and federation separately scales the namespace by adding independent namenodes.

### 1. The big picture
In the original HDFS design the namenode was a single point of failure (SPOF). Replicating its metadata and running a secondary namenode protects against *data loss* but not *unavailability*: if the namenode is down, every client — including every MapReduce job — cannot read, write or list files. Recovery meant starting a new namenode from a metadata copy, and it could not serve anyone until it had (i) loaded the fsimage, (ii) replayed the edit log and (iii) received enough block reports to leave safe mode. On big clusters that cold start can take **30 minutes or more**.

Because unplanned namenode failures are rare, the practical pain was *planned* maintenance: every restart for patching or reconfiguration meant the same long outage. HDFS High Availability (HA) solves both. Federation is an orthogonal feature that solves a different limit: namenode memory.

### 2. Core concepts and vocabulary
- **Active / standby namenode** — a pair of namenodes; only the active serves clients.
- **Shared edit log** — the journal both namenodes use so the standby can stay current.
- **Quorum Journal Manager (QJM)** — purpose-built HDFS service storing the shared edit log on a set of **journal nodes**; each edit must be written to a majority.
- **Failover controller (ZKFC)** — small process beside each namenode that monitors it and drives failover; the default implementation uses **ZooKeeper** to guarantee a single active node.
- **Graceful failover** — admin-initiated, orderly role swap.
- **Ungraceful failover** — triggered by a detected failure.
- **Fencing** — making sure the old active can no longer do damage.
- **STONITH** — "shoot the other node in the head": power it off.
- **Logical URI** — a name like `hdfs://mycluster` mapped to the two namenode addresses.
- **Federation** — several namenodes, each with its own *namespace volume* and *block pool*.
- **Client-side mount table (`viewfs://`)** — maps path prefixes to namenodes.

### 3. How it works, step by step
**Architecture changes required for HA**
1. Both namenodes use highly available shared storage for the edit log. The standby reads to the end of it, then keeps tailing new entries.
2. Datanodes send block reports and heartbeats to **both** namenodes, because block locations live only in memory.
3. Clients are configured to try both namenodes, transparently.
4. The secondary namenode disappears: the standby takes the periodic checkpoints.

**Automatic failover**
1. Each namenode's ZKFC heartbeats its local namenode and holds a ZooKeeper session; the one that has the lock marks its namenode active.
2. The active fails (or its ZKFC session expires).
3. The surviving ZKFC fences the old active.
4. It then acquires the lock and promotes its namenode: the standby finishes reading the shared edit log, leaves standby mode and starts serving.
5. Clients that fail to reach the old node try the other address and carry on.

Since the standby already has an up-to-date namespace *and* current block reports, takeover takes seconds, not tens of minutes.

### 4. Worked example
```xml
<!-- hdfs-site.xml -->
<property><name>dfs.nameservices</name><value>mycluster</value></property>
<property><name>dfs.ha.namenodes.mycluster</name><value>nn1,nn2</value></property>
<property><name>dfs.namenode.rpc-address.mycluster.nn1</name><value>host-a:8020</value></property>
<property><name>dfs.namenode.rpc-address.mycluster.nn2</name><value>host-b:8020</value></property>
<property><name>dfs.namenode.shared.edits.dir</name>
  <value>qjournal://jn1:8485;jn2:8485;jn3:8485/mycluster</value></property>
<property><name>dfs.client.failover.proxy.provider.mycluster</name>
  <value>org.apache.hadoop.hdfs.server.namenode.ha.ConfiguredFailoverProxyProvider</value></property>
<property><name>dfs.ha.fencing.methods</name><value>sshfence</value></property>
<property><name>dfs.ha.automatic-failover.enabled</name><value>true</value></property>
```
```bash
hdfs haadmin -getServiceState nn1       # active
hdfs haadmin -failover nn1 nn2          # graceful failover for maintenance
hdfs dfs -ls hdfs://mycluster/          # clients use the logical name only
```
The logical name `mycluster` hides which host is active. Three journal nodes tolerate the loss of one (a majority of two must acknowledge each edit).

### 5. Under the hood
**QJM:** each edit is sent to all journal nodes and committed when a majority acknowledges. Only one namenode is allowed to write to the journal at a time (each writer holds an *epoch*; journal nodes reject writes from older epochs). This is why QJM is preferred over NFS: it prevents the "split-brain" write problem by design.

**Why fencing is still needed:** after an ungraceful failover you cannot be sure the failed namenode really stopped — a slow network or partition may just have *looked* like a failure. With QJM the old active cannot corrupt the edit log, but it might still serve **stale reads** to clients that reach it. Hence the layers of fencing:
1. SSH fencing: log in and kill the namenode process (a sensible default with QJM).
2. Revoke the node's access to shared storage (NFS vendor command) — necessary with an NFS filer, which can't restrict to a single writer.
3. Disable its network port through a remote management command.
4. STONITH: cut power with a controllable power distribution unit — last resort.

**Client failover** is handled by the client library, configured via the logical URI → list of namenode addresses; it tries each until one answers.

**Federation internals:** each namenode manages a namespace volume (metadata) plus a block pool (all blocks for that namespace). Namenodes do not communicate; one failing doesn't impact the others. Datanodes register with all namenodes and store blocks of multiple pools, so *storage is shared while the namespace is partitioned*. Clients use `viewfs://` mount tables; e.g., `/user` → nn1, `/share` → nn2. Federation exists because the namenode holds every file/block reference in RAM — the number of objects, not terabytes, is the scale limit.

### 6. Performance and design implications
- Failover time: seconds with HA vs 30+ minutes for cold standby; tune ZooKeeper session timeouts to balance fast detection against false failovers (GC pauses can look like failure).
- Journal nodes should be on separate machines with low-latency disks; every metadata mutation waits for a majority of them.
- The standby's checkpointing replaces the secondary; no additional machine needed.
- Federation adds *capacity* and *isolation* but each namespace is still its own SPOF unless it also has HA.
- Rolling upgrades become practical: upgrade the standby, fail over, upgrade the former active.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `hdfs haadmin -getServiceState` | Show active/standby | Monitoring, scripts |
| `hdfs haadmin -failover a b` | Graceful failover | Planned maintenance |
| `dfs.namenode.shared.edits.dir` | Location of the QJM / shared log | HA setup |
| `dfs.ha.fencing.methods` | Fencing commands (`sshfence`, `shell(...)`) | HA setup |
| `dfs.ha.automatic-failover.enabled` | ZKFC-driven failover | Production HA |
| `ha.zookeeper.quorum` | ZooKeeper ensemble for the controllers | HA setup |
| `viewfs://` / `fs.viewfs.mounttable.*` | Client mount table | Federation |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| QJM (3 or 5 journal nodes) for the shared log | An NFS filer as shared storage — it's a SPOF itself and needs strong fencing |
| Automatic failover for production | Dev/test clusters where the operational complexity isn't justified |
| Federation when the namespace outgrows one heap or needs tenant isolation | Expecting it to give availability |
| Several fencing methods in order | Relying on a single fence that may be unreachable |
| Odd number of journal nodes | Even numbers (no extra fault tolerance) |

### 9. Common mistakes
- Believing metadata replication + secondary namenode = HA → still hours of outage on failure → deploy an active/standby pair.
- No working fencing method → failover refuses or risks split-brain → configure and test `sshfence`/shell fencing.
- Fewer than three journal nodes → one failure stops all writes → run 3 (or 5).
- Putting journal nodes/ZooKeeper on the same busy machines as heavy jobs → false failovers → dedicate them.
- Mixing up federation and HA → wrong capacity plan → federation partitions the namespace; HA duplicates each namenode.

### 10. 🎯 Interview kit
**30-second answer:** HDFS HA runs an active and a standby namenode sharing an edit log via a quorum of journal nodes; datanodes report to both; ZooKeeper-backed failover controllers promote the standby and fence the old active. Federation is separate: multiple namenodes each own part of the namespace.

**Structure a longer answer:** clarify (availability vs scale problem) → mechanism (shared edits, block reports to both, ZKFC, fencing) → trade-offs (QJM vs NFS, timeouts) → scenario (planned upgrade with graceful failover).

**Likely follow-ups:**
- *Why not just rely on the secondary namenode?* → it lags and isn't a standby; recovery needs fsimage load + replay + block reports.
- *What is fencing and why?* → ensure the former active can't write or serve; avoids split-brain.
- *Why 3 journal nodes?* → majority writes; tolerates one failure.
- *Federation vs HA?* → scale vs availability.
- *How do clients find the active?* → logical URI + failover proxy provider.

**Signals interviewers look for:** you mention block reports to both nodes, fencing, and keep federation and HA separate.

### 11. Check yourself
- [ ] List the four architectural changes HA needs.
- [ ] Explain why a restarted namenode takes so long to serve again.
- [ ] Describe what QJM guarantees and why it simplifies fencing.
- [ ] Walk through an ungraceful failover from the failure to the first served request.
- [ ] Draw a federated cluster with two namenodes and shared datanodes.
<!-- explainer:end -->

## References

- *Hadoop: The Definitive Guide* — Tom White — The Hadoop Distributed Filesystem › HDFS High Availability
- *Hadoop: The Definitive Guide* — Tom White — E › F
- *Hadoop: The Definitive Guide* — Tom White — Administering Hadoop › Note
- *Hadoop: The Definitive Guide* — Tom White — Q
- *Fundamentals of Data Engineering* — Joe Reis, Matt Housley — Storage › Data durability and availability
- *Hadoop: The Definitive Guide* — Tom White — The Hadoop Distributed Filesystem › Failover and fencing
- *Fundamentals of Data Engineering* — Joe Reis, Matt Housley — The Future of Data Engineering › Availability Zones

