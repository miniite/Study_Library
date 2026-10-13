---
id: "TOOLS/Hadoop/03_Ecosystem/HBase/HBase_Architecture"
level: 5
type: "curriculum"
curriculum: "data_engineering"
---

# HBase Architecture and Schema Design

**L5 Expert** · [↑ HBase](<../_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** HBase turns random writes into sequential ones — every write goes to a write-ahead log and an in-memory MemStore, is later flushed to immutable HFiles on HDFS, and compactions keep reads fast — while regions, the master and ZooKeeper keep the cluster balanced and available.

### 1. The big picture
This lesson opens the box described in the HBase overview. The design is a classic **LSM-tree** (log-structured merge tree) wrapped in a distributed system. Because HDFS only supports append and whole-file operations, HBase cannot update pages in place like a B-tree; instead it buffers changes in memory, writes them out as sorted immutable files, and merges files in the background. Understanding that pipeline explains almost every HBase behaviour: fast writes, read amplification, compaction storms, and row-key design rules.

### 2. Core concepts and vocabulary
- **WAL (HLog)** — write-ahead log on HDFS; every mutation is appended here first for durability.
- **MemStore** — sorted in-memory buffer, one per column family per region.
- **HFile (StoreFile)** — immutable, sorted file on HDFS containing key-values, indexes and Bloom filters; the MemStore is flushed into one.
- **Store** — one column family inside one region (a MemStore plus its HFiles).
- **BlockCache** — read cache of HFile blocks on the RegionServer.
- **Compaction** — minor (merge a few small HFiles) or major (rewrite all HFiles of a store, purge deletes/expired versions).
- **Region split** — a region that grows beyond a threshold is divided into two at a middle key.
- **HMaster** — assigns regions, handles DDL and load balancing; **RegionServer** serves reads/writes.
- **ZooKeeper** — tracks live servers, the active master and the location of `hbase:meta`.
- **Tombstone** — marker written by a delete; physically removed at major compaction.

### 3. How it works, step by step
**Write path**
1. Client locates the region (ZooKeeper → `hbase:meta` → RegionServer, cached).
2. RegionServer appends the edit to the **WAL** (durable on HDFS replicas).
3. Edit is inserted into the **MemStore** of the right store.
4. Client gets an acknowledgement.
5. When the MemStore reaches its limit it is **flushed** to a new HFile.
6. As HFiles accumulate, **compaction** merges them.

**Read path**
1. Region located as above.
2. The server merges results from the **MemStore**, the **BlockCache** and the relevant **HFiles** (Bloom filters and key ranges skip files that can't contain the row).
3. The newest version wins; tombstones hide deleted cells.

**Failure path**
1. A RegionServer's ZooKeeper session expires.
2. HMaster splits the dead server's WAL by region and reassigns its regions to other servers.
3. New hosts replay their part of the WAL into MemStores; the regions come back online.

**Region name and catalogue.** `hbase:meta` rows are keyed by region name `table,startRow,creationTimestamp.md5hash.` — e.g. `TestTable,xyz,1279729913622.1b6e176f….` Finding a row = find the largest region start key ≤ the row key.

### 4. Worked example
```text
hbase> create 'metrics', {NAME=>'d', VERSIONS=>1, COMPRESSION=>'SNAPPY', BLOOMFILTER=>'ROW'},
         {SPLITS => ['2','4','6','8']}        # pre-split into 5 regions
hbase> put 'metrics', '3|sensor17|20240501T1200', 'd:temp', '21.4'
hbase> flush 'metrics'            # force MemStore -> HFile
hbase> major_compact 'metrics'    # rewrite HFiles, drop tombstones
hbase> status 'detailed'          # see regions per server, memstore size
```
Key design: the leading digit is a salt (e.g. `hash(sensor) mod 10`), so inserts spread across the pre-split regions instead of hammering one. A range scan for one sensor must now issue one scan per salt bucket — the cost of avoiding hotspots. After `flush` you can see new files under `/hbase/data/default/metrics/<region>/d/` in HDFS.

### 5. Under the hood
- **HFile layout**: data blocks (~64 KB, optionally compressed), index blocks, Bloom filter blocks, trailer. Blocks are the unit of caching and I/O.
- **MemStore flush** is triggered by size (per-region and global memory fraction) or WAL count; flushing is why a very large number of families produces many small files.
- **Read amplification**: a get may consult MemStore + N HFiles; compaction reduces N. Write amplification: each cell is rewritten each time it is compacted.
- **Major compaction** is expensive — schedule off-peak or trigger manually. It also restores data locality (files re-written on the local DataNode).
- **Splits** are cheap at first (daughters reference parent files) and completed by compaction.
- **Consistency**: single-row operations are atomic and strongly consistent because a row lives in exactly one region on one server at a time.
- Compare with B-tree stores: LSM trades read cost for write throughput; see the storage-engine material for the general comparison.

### 6. Performance and design implications
- **Row key**: avoid monotonic prefixes; pre-split new tables; keep keys short (stored with every cell).
- **Heap split**: BlockCache vs MemStore share RegionServer heap; read-heavy → larger cache, write-heavy → larger MemStores.
- **Column families**: few; put data with different access/TTL patterns in different families.
- **Compaction tuning**: avoid compaction storms by pre-splitting and limiting simultaneous major compactions.
- **Observe**: RegionServer metrics (memstore size, store file count, compaction queue, block cache hit ratio), the web UI on the master, `status`.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `flush 't'` / `compact 't'` / `major_compact 't'` | Force flush / compaction | Maintenance, before snapshots |
| `split 't'` / pre-split `SPLITS=>[...]` | Divide regions | Load distribution |
| `hbase.hregion.memstore.flush.size` | MemStore flush threshold | Write tuning |
| `hbase.hregion.max.filesize` | Split threshold | Region sizing |
| `hfile.block.cache.size` | Heap fraction for BlockCache | Read tuning |
| `hbase.hregion.majorcompaction` | Major compaction period | Avoid peak-hour storms |
| `BLOOMFILTER`, `COMPRESSION`, `IN_MEMORY` | Family options | Read/size tuning |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| Salted/hashed keys for write-heavy ingest | You need ordered range scans across all data |
| Big MemStore, fewer flushes | Short recovery time is critical (more WAL to replay) |
| Frequent compactions for read latency | Cluster is I/O-saturated |
| Row-level atomicity suffices | You need multi-row transactions |

### 9. Common mistakes
- Many column families → many tiny flushes and compactions → merge families.
- Ignoring GC and heap sizing → RegionServer pauses exceed the ZooKeeper timeout → server declared dead → tune heap/GC.
- Triggering major compaction at peak → latency spikes → schedule or automate off-peak.
- Single region on a fresh table under bulk load → one server overloaded → pre-split or bulk-load HFiles.

### 10. 🎯 Interview kit
**30-second answer:** HBase writes to a WAL and MemStore, flushes sorted immutable HFiles to HDFS, and compacts them in the background — an LSM design. Tables are split into regions by row-key range, served by RegionServers, coordinated by an HMaster and ZooKeeper.

**Structure a longer answer:** clarify → core mechanism → trade-offs → close with a real scenario.

**Likely follow-ups:**
- *What happens when a RegionServer dies?* → ZooKeeper session expiry, WAL splitting and replay, region reassignment.
- *Why are writes fast?* → sequential WAL + in-memory insert.
- *Why can reads be slow?* → read amplification across HFiles; compaction/Bloom filters/cache mitigate.
- *How to design the row key?* → distribution vs scan locality trade-off.

**Signals interviewers look for:** end-to-end write and read path, failure recovery and key-design trade-offs.

### 11. Check yourself
- [ ] Draw the write path and say where durability is guaranteed.
- [ ] Explain minor vs major compaction and what a tombstone is.
- [ ] Describe recovery from RegionServer failure.
- [ ] Choose a row key and pre-split plan for an IoT time-series table.
<!-- explainer:end -->

## References

- *Hadoop: The Definitive Guide* — Tom White — HBase
- *Hadoop: The Definitive Guide* — Tom White — HBase › UI
- *Hadoop: The Definitive Guide* — Tom White — HBase › Online Queries
- *Hadoop: The Definitive Guide* — Tom White — HBase › Note
- *Hadoop: The Definitive Guide* — Tom White — G › H
- *Hadoop: The Definitive Guide* — Tom White — HBase › Backdrop
- *Hadoop: The Definitive Guide* — Tom White — HBase › Clients
- *Database Internals* — Alex Petrov — M
- *Hadoop: The Definitive Guide* — Tom White — HBase › Bulk load
- *Hadoop: The Definitive Guide* — Tom White — HBase › Tip

