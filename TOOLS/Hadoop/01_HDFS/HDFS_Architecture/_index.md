---
id: "TOOLS/Hadoop/01_HDFS/HDFS_Architecture"
level: 4
type: "curriculum"
curriculum: "data_engineering"
---

# HDFS Architecture

**L4 Advanced** · [↑ HDFS](<../_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** HDFS architecture is a metadata master (namenode) plus many block-storing workers (datanodes), with replication, checksums and rack-aware placement providing durability, and federation/caching/balancing extending it at scale.

### 1. The big picture
The HDFS overview covered *what* HDFS is. This lesson is about the *components and their contracts*: what the namenode owns, what datanodes own, how blocks are protected, how the namespace scales past one machine's memory, and how an operator keeps the cluster healthy. Children of this topic go deeper: the read/write path (how a byte actually travels) and high availability/federation (how the master stops being a single point of failure).

A useful mental model is that HDFS separates **metadata** (small, hot, in RAM on one machine) from **data** (huge, cold, spread over thousands of disks). Almost every design decision follows from that split.

### 2. Core concepts and vocabulary
- **Namespace volume** — a namenode's metadata for its part of the tree.
- **Block pool** — all blocks that belong to one namespace; datanodes hold block pools for each namenode they register with.
- **Block report / heartbeat** — periodic datanode messages: which blocks I have, and I'm alive.
- **Checksum** — a CRC-32C per 512 bytes of data guarding against corruption.
- **DataBlockScanner** — background thread that re-verifies stored blocks to detect bit rot.
- **Cache directive / cache pool** — instruction to keep a file's blocks in datanode off-heap memory, grouped for permissions and quotas.
- **Federation / ViewFileSystem** — multiple independent namenodes, glued client-side with mount tables (`viewfs://`).
- **Balancer** — daemon that moves blocks from over- to under-utilised datanodes.
- **Include / exclude files** — cluster membership control, used for commissioning and decommissioning.

### 3. How it works, step by step
**Component roles**
| Component | Owns | Persisted? |
|---|---|---|
| Namenode | Directory tree, file → block list, permissions, replication per file | Yes: fsimage + edit log |
| Namenode (memory) | Block → datanode locations | No: rebuilt from block reports |
| Datanode | Block bytes + checksum metadata | Yes: local disks |
| Client | API (`FileSystem`), checksum verification | – |

**Healing a corrupt block**
1. A client reading a block detects a checksum mismatch.
2. It reports block + datanode to the namenode, then raises `ChecksumException` to the application.
3. The namenode marks that replica corrupt and stops sending clients to it.
4. It schedules re-replication from a healthy replica to another datanode.
5. Once the replication factor is restored, the corrupt replica is deleted.

**Decommissioning a datanode**
1. Add the node to the exclude file (`dfs.hosts.exclude`); leave the include file alone.
2. `hdfs dfsadmin -refreshNodes` (and `yarn rmadmin -refreshNodes` for node managers).
3. Watch the web UI: *Decommission In Progress* → *Decommissioned* once its blocks are copied elsewhere.
4. Remove the node from the include file, refresh again, then shut it down.

Precedence for datanodes: not in include → may not connect; in include and not in exclude → connects; in both → connects only to be decommissioned.

### 4. Worked example
```xml
<!-- hdfs-site.xml: a datanode with 4 disks, namenode metadata in 3 places -->
<property><name>dfs.namenode.name.dir</name>
  <value>file:///disk1/nn,file:///disk2/nn,file:///mnt/nfs/nn</value></property>
<property><name>dfs.datanode.data.dir</name>
  <value>/disk1/dn,/disk2/dn,/disk3/dn,/disk4/dn</value></property>
<property><name>dfs.namenode.checkpoint.dir</name>
  <value>file:///disk1/snn</value></property>
```
```bash
hdfs cacheadmin -addPool lookups
hdfs cacheadmin -addDirective -path /dims/country -pool lookups -replication 1
hadoop fs -checksum /dims/country/part-0000     # compare two copies
hdfs balancer -threshold 5                        # even out to within 5%
```
`name.dir` is *redundant* (the same metadata is written to each). `data.dir` is *striped* (each block lands on exactly one directory, round-robin). The cache directive keeps a small join table in memory so schedulers can place tasks on the node holding it.

### 5. Under the hood
- **Datanode on-disk format:** block-pool directory → `finalized` and `rbw`; `blk_N` + `blk_N_gen.meta` (header with version/type, then checksum list). Sub-directories created every 64 blocks keep directory sizes sane.
- **Namenode memory:** roughly one object per file, directory and block, so total file count drives the heap. Each block's replica locations are in memory only; after a restart the namenode sits in *safe mode* until enough blocks are reported (`dfs.namenode.replication.min` replicas for the threshold fraction).
- **Checksum timing:** the *last datanode in the write pipeline* verifies what it receives; clients verify on every read; each datanode logs verification times, which helps spot dying disks.
- **Federation:** namenodes never talk to each other. A failure of one affects only its namespace. Datanodes register with every namenode and are shared, so storage is not partitioned though the namespace is. Clients resolve paths via a mount table in configuration.
- **Block caching:** blocks are cached off-heap on a datanode, one replica by default; the namenode tracks cached locations and schedulers prefer those nodes.
- **Balancing:** the balancer respects rack-aware placement and throttles itself (`dfs.datanode.balance.bandwidthPerSec`, default 1 MB/s). Only one runs at a time and it stops when every node is within the threshold of the cluster mean utilisation.

### 6. Performance and design implications
- Spread writes: using at least as many `distcp` maps as nodes avoids filling the writer's disk with first replicas.
- Reads benefit from several disks per datanode because concurrent block reads hit separate spindles.
- Checksum overhead is under 1% of storage and little CPU thanks to hardware-assisted CRC-32C.
- Raising the balancer bandwidth speeds rebalancing but competes with jobs; do it in quiet windows.
- Namenode heap sizing is the capacity plan: estimate files + blocks, not terabytes.
- Cached blocks speed up repeatedly read small datasets; caching large data wastes memory.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `hdfs dfsadmin -refreshNodes` | Re-reads include/exclude files | Commission/decommission |
| `dfs.hosts`, `dfs.hosts.exclude` | Membership files | Controlled scale in/out |
| `hdfs cacheadmin` | Manage cache pools/directives | Pin hot lookup data |
| `hdfs balancer -threshold N` | Rebalance blocks | After adding nodes |
| `hdfs fsck` | Health, corrupt/missing blocks | Regular checks, before upgrade |
| `-ignoreCrc` with `-get` | Read despite checksum failure | Inspect corrupt file to salvage |
| `dfs.bytes-per-checksum` | Checksum chunk size (512) | Rarely changed |
| `viewfs://` URIs | Client mount table over federated namenodes | Federated clusters |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| Federation: namespace too big for one heap, or you need isolation per tenant | You only need availability (use HA) |
| Cache directives for small, hot, reused data | Large scans that churn memory |
| Decommissioning via exclude file | Pulling several nodes at once — blocks of all three replicas can be lost |
| Running the balancer regularly on growing clusters | Needing absolute even distribution at the cost of job interference |
| Big blocks | Small clusters or tiny jobs |

### 9. Common mistakes
- Powering off datanodes en masse → missing blocks → decommission properly and watch under-replication.
- Putting the include file update before refresh/decommission is finished → node dropped while its blocks are unique → remove from include only at the end.
- Assuming federation gives HA → one namenode failure still takes its namespace offline → combine federation with HA pairs.
- Ignoring balance → hot datanodes, poor task locality → balancer after expansion, enough `distcp` maps.
- Disabling checksum verification permanently → silent corruption → only for one-off salvage.
- Putting HDFS metadata and data on the same single disk → one failure loses both → separate disks, NFS copy.

### 10. 🎯 Interview kit
**30-second answer:** A namenode keeps the namespace and block map in memory with persistent fsimage/edit log; datanodes store replicated blocks with checksums and report to the namenode. Corruption is repaired from replicas, namespace scalability comes from federation, and the balancer and decommissioning keep the cluster healthy.

**Structure a longer answer:** clarify (scale, availability needs) → core mechanism (metadata vs data split, replication, checksums) → trade-offs (heap bound, small files, federation vs HA) → scenario (adding 20 nodes, rebalancing, retiring old ones).

**Likely follow-ups:**
- *How does HDFS detect bit rot?* → read-time checksums plus the background block scanner.
- *What is a block pool?* → the set of blocks of one namespace; datanodes host several under federation.
- *Federation vs HA?* → federation scales the namespace, HA removes the single point of failure.
- *How do you safely remove a datanode?* → exclude file, `-refreshNodes`, wait for decommission to complete.
- *How much memory does a namenode need?* → proportional to number of files + blocks.

**Signals interviewers look for:** correct split of persisted vs in-memory state, awareness of safe mode, and operational fluency with decommission/balancer.

### 11. Check yourself
- [ ] Draw the roles of namenode, datanode and client and what each stores.
- [ ] Explain, in order, what happens when a client reads a corrupt block.
- [ ] Describe federation using the terms namespace volume and block pool.
- [ ] Walk through decommissioning a datanode and the include/exclude table.
- [ ] State why `dfs.namenode.name.dir` and `dfs.datanode.data.dir` behave differently with multiple directories.
<!-- explainer:end -->

## In this section (study in order)

1. [HDFS Read and Write Path](<HDFS_Read_and_Write_Path/_index.md>) (L5 Expert)
2. [HDFS High Availability and Federation](<HDFS_HA_and_Federation/_index.md>) (L5 Expert)

## References

- *Hadoop: The Definitive Guide* — Tom White — N
- *Hadoop: The Definitive Guide* — Tom White — Setting Up a Hadoop Cluster › Tip
- *Hadoop: The Definitive Guide* — Tom White — Administering Hadoop › Decommissioning old nodes
- *Hadoop: The Definitive Guide* — Tom White — The Hadoop Distributed Filesystem › Block Caching
- *Hadoop: The Definitive Guide* — Tom White — Hadoop IO › Data Integrity in HDFS
- *Hadoop: The Definitive Guide* — Tom White — Setting Up a Hadoop Cluster › HDFS
- *Hadoop: The Definitive Guide* — Tom White — D
- *Hadoop: The Definitive Guide* — Tom White — The Hadoop Distributed Filesystem › HDFS Concepts
- *Hadoop: The Definitive Guide* — Tom White — The Hadoop Distributed Filesystem › Keeping an HDFS Cluster Balanced
- *Fundamentals of Data Engineering* — Joe Reis, Matt Housley — Choosing Technologies Across the Data Engineering Lifecycle › The Distributed Monolith Pattern

