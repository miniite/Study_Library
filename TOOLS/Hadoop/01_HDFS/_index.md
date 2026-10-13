---
id: "TOOLS/Hadoop/01_HDFS"
level: 3
type: "curriculum"
curriculum: "data_engineering"
---

# HDFS

**L3 Practitioner** · [↑ Hadoop](<../_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** HDFS is a distributed filesystem that chops big files into large blocks, spreads (and copies) those blocks over many cheap machines, and keeps one master node that knows where everything is.

### 1. The big picture
When a dataset no longer fits on one machine you have to split it across several. A *distributed filesystem* manages storage over a network of machines, and because it is network-based it inherits all the difficulties of network programming — the hardest being to survive machine failure without losing data. **HDFS** (Hadoop Distributed Filesystem) is Hadoop's flagship answer: it is designed for very large files, written once and read many times, on commodity hardware where failures are normal rather than exceptional.

HDFS is the storage layer that YARN, MapReduce, Hive, Spark and HBase sit on. Everything above it assumes two things HDFS gives: *capacity that grows by adding machines*, and *data that survives disk and node loss through replication*. Its sub-topics (architecture, read/write path, high availability, operations) each zoom into one part of the picture drawn here.

### 2. Core concepts and vocabulary
- **Block** — the unit HDFS stores and replicates; 128 MB by default (far larger than the few KB of a local filesystem). A file smaller than a block only uses its real size on disk.
- **Namenode** — the single master (per namespace) that holds the filesystem tree and the file → block mapping.
- **Datanode** — a worker that stores blocks and reports which blocks it holds.
- **Namespace image (fsimage) and edit log** — the two files in which the namenode persists metadata: a snapshot plus a journal of changes since.
- **Secondary namenode** — despite the name *not* a standby; it periodically merges the edit log into the image (a checkpoint).
- **Client** — library that presents a POSIX-like interface and talks to the namenode for metadata and directly to datanodes for data.
- **Replication factor** — number of copies of each block (default 3).
- **Rack awareness** — knowledge of network topology used to place replicas on different racks.

### 3. How it works, step by step
1. **Split.** A file written to HDFS is cut into block-sized pieces; each piece is an independent unit.
2. **Place.** The namenode chooses datanodes for every block; replicas go to different nodes, and not all on one rack.
3. **Store.** Datanodes keep each block as an ordinary local file (plus a checksum file).
4. **Record.** The namenode logs the change in its edit log and updates the in-memory namespace.
5. **Report.** Datanodes send periodic block reports and heartbeats. The namenode does **not** persist block locations; it rebuilds the map from these reports at startup.
6. **Read.** A client asks the namenode where the blocks are, then streams data straight from the nearest datanodes. The namenode is never in the data path.

Why such large blocks? If a seek costs about 10 ms and the disk transfers 100 MB/s, a block of ~100 MB makes the seek only 1% of the transfer time, so reading a large file runs at disk speed. The argument can't be pushed too far: map tasks usually work on one block at a time, so too few blocks means too few parallel tasks.

Block-sized abstraction has further benefits: a file can be bigger than any single disk, the storage layer stays simple (fixed-size units, easy to count and replicate), and replication per block gives fault tolerance and availability cheaply.

### 4. Worked example
```bash
# put a 300 MB file into HDFS, then see how it was laid out
hdfs dfs -mkdir -p /data/logs
hdfs dfs -put access_2024.log /data/logs/
hdfs fsck /data/logs/access_2024.log -files -blocks -locations
# -> 3 blocks: 128 MB + 128 MB + 44 MB, each listed with 3 datanode locations
```
`fsck` shows the block list and which datanodes hold each replica. The last block is only 44 MB on disk — HDFS does not pad it to 128 MB. Stop one of the datanodes holding a replica and, after the heartbeat timeout, the namenode schedules a new copy so the factor returns to 3.

### 5. Under the hood
**Namenode persistence.** The *fsimage* holds a serialized inode tree (replication level, times, permissions, block size, block list; for directories, quotas). It does not contain datanode locations. Each mutation is first appended to the *edits* file. Without intervention the edit log grows forever, and restart time (replaying every transaction while the filesystem is offline) would grow with it.

**Checkpointing** solves that. The secondary namenode:
1. asks the primary to roll to a new edits file;
2. fetches the latest fsimage and edits over HTTP;
3. loads the image into memory, replays the edits and writes a merged fsimage;
4. sends it back; the primary stores it as a `.ckpt` file;
5. the primary renames it to make it current.

![The checkpointing process](<../../../_assets/Hadoop_The_Definitive_Guide_Tom_White/hddg_1101.png>)

By default this happens hourly (`dfs.namenode.checkpoint.period`) or after one million transactions (`dfs.namenode.checkpoint.txns`). Because it loads the whole image, the secondary needs about as much memory as the primary. It lags the primary, so it is *not* a failover target; true failover needs HA (see the HA lesson).

**Datanode layout.** Under `dfs.datanode.data.dir` there is a block-pool directory containing `finalized/` and `rbw/` (replicas being written). Blocks are files `blk_<id>` with a `.meta` file holding checksums. When a directory reaches 64 blocks a subdirectory is created so directories stay small. With several data directories on different disks, blocks are written round-robin; replication is across datanodes, never across disks of one node.

**Integrity.** Every 512 bytes (`dfs.bytes-per-checksum`) get a 4-byte CRC-32C, under 1% overhead. Datanodes verify on write, clients verify on read, and a background scanner catches bit rot. A corrupt replica is reported to the namenode, which re-replicates from a good copy and then deletes the bad one.

**Scaling limits.** The namenode keeps every file and block reference in RAM, so the *number of files and blocks* — not total bytes — bounds a namespace. Federation addresses this by running several namenodes, each owning a namespace volume and its block pool, with clients using a `viewfs://` mount table.

### 6. Performance and design implications
- Many small files are the classic problem: each costs namenode memory regardless of size and creates many tiny tasks. Combine them (sequence files, Parquet, compaction).
- Data is for streaming reads of big ranges; HDFS gives high throughput but not low-latency random access, and files are append-only (no in-place edits).
- Mount data disks with `noatime` to avoid writing access times on every read.
- Use one data directory per physical disk to spread I/O; use a few namenode dirs, including a remote NFS, for metadata redundancy.
- Hot, small lookup data can be pinned in off-heap datanode memory with *cache directives* in *cache pools*.
- Keep the cluster balanced; bulk loads via `distcp -m 1` would make the single writing node hold the first replica of everything.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `hdfs namenode -format` | Creates empty namenode storage | Once, on a brand-new cluster (datanodes need no format) |
| `fs.defaultFS` | Namenode URI and default filesystem (port 8020) | Every client and server config |
| `dfs.namenode.name.dir` | Where namenode metadata is written (one copy per dir) | Set several: local disks + NFS |
| `dfs.datanode.data.dir` | Block storage dirs (round-robin) | One per local disk |
| `dfs.namenode.checkpoint.dir` | Secondary's checkpoint dirs | Secondary node setup |
| `dfs.replication` | Copies per block (3 default; 1 pseudo-distributed) | Tune per cluster/file |
| `hdfs fsck <path> -files -blocks -locations` | Inspect blocks, replicas, health | Debugging missing/corrupt data |
| `hdfs dfsadmin -saveNamespace` | Force a checkpoint (in safe mode) | Before maintenance |
| `start-balancer.sh -threshold N` | Move blocks to even out utilisation (default 10%, 1 MB/s per node) | After adding nodes or lopsided loads |
| `hadoop fs -checksum <file>` | File checksum | Compare two copies |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| Large files, sequential scans, write-once data, batch analytics | Millions of tiny files, low-latency random reads, frequent in-place updates |
| Commodity servers with replication for durability | Needing POSIX semantics or concurrent writers (use a different store) |
| Larger blocks (256 MB+) for huge scans | Small clusters where big blocks leave too few parallel tasks |
| Federation to scale namespace size | Wanting HA — federation does not give failover by itself |
| NFS gateway for mounting HDFS | FUSE-DFS (the NFS gateway is the more robust choice) |

### 9. Common mistakes
- Treating the secondary namenode as a hot standby → data loss when the primary dies → configure HA with a standby namenode.
- Writing namenode metadata to a single disk → total filesystem loss if it fails → several `name.dir` entries incl. NFS.
- Loading millions of small files → namenode heap exhaustion and slow jobs → pack into larger files.
- Shutting down several datanodes at once → under-replicated or missing blocks → decommission through the exclude file.
- Running `distcp` with too few maps → unbalanced cluster → default of 20 maps per node, then run the balancer.
- Expecting HDFS to hold full blocks of space for small files → misjudged capacity; a 1 MB file uses 1 MB (times replication).

### 10. 🎯 Interview kit
**30-second answer:** HDFS splits files into big (128 MB) blocks, replicates each (3×) across datanodes, and a namenode keeps the namespace and block map in memory with an edit log + fsimage on disk. Clients get metadata from the namenode and stream data directly from datanodes.

**Structure a longer answer:** clarify (what workload — big sequential or small random?) → core mechanism (blocks, namenode/datanodes, replication, checksums) → trade-offs (small files, namenode memory, no random writes) → close with a scenario (nightly 5 TB log ingest and Spark scans).

**Likely follow-ups:**
- *Why is the block size so large?* → amortise seek cost so transfer dominates.
- *What does the secondary namenode do?* → merges edits into fsimage (checkpoint); not a failover.
- *What if the namenode dies?* → without HA the filesystem is down; metadata must be recovered from the saved copies.
- *How is corruption handled?* → checksums, background scanner, re-replication from good copies.
- *Where does the namenode get block locations?* → from datanode block reports, not from disk.

**Signals interviewers look for:** you know the namenode is metadata-only and out of the data path, you can name the small-files problem and its cause, and you distinguish checkpointing from HA.

### 11. Check yourself
- [ ] Explain why a 1 MB file does not waste a 128 MB block.
- [ ] Walk through the five-step checkpoint process.
- [ ] Say what is and is not stored in the fsimage.
- [ ] Describe how a corrupt replica is detected and repaired.
- [ ] Explain what limits namenode scalability and how federation helps.
- [ ] Run `fsck -files -blocks -locations` and interpret the output.
<!-- explainer:end -->

## In this section (study in order)

1. [HDFS Architecture](<HDFS_Architecture/_index.md>) (L4 Advanced)
2. [HDFS Command Line and Administration](<HDFS_Operations/_index.md>) (L4 Advanced)

## References

- *Hadoop: The Definitive Guide* — Tom White — The Hadoop Distributed Filesystem › Namenodes and Datanodes
- *Hadoop: The Definitive Guide* — Tom White — The Hadoop Distributed Filesystem › Chapter 3. The Hadoop Distributed Filesystem
- *Hadoop: The Definitive Guide* — Tom White — Administering Hadoop › HDFS data and metadata upgrades
- *Hadoop: The Definitive Guide* — Tom White — Administering Hadoop › Balancer
- *Hadoop: The Definitive Guide* — Tom White — Administering Hadoop › Datanode directory structure
- *Hadoop: The Definitive Guide* — Tom White — Administering Hadoop › Note
- *Hadoop: The Definitive Guide* — Tom White — Setting Up a Hadoop Cluster › Configuring Hadoop
- *Hadoop: The Definitive Guide* — Tom White — The Hadoop Distributed Filesystem › FUSE
- *Hadoop: The Definitive Guide* — Tom White — Summary › Configuration
- *Hadoop: The Definitive Guide* — Tom White — Setting Up a Hadoop Cluster › Hadoop Daemon Addresses and Ports

