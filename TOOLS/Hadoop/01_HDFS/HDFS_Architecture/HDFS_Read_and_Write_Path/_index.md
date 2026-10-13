---
id: "TOOLS/Hadoop/01_HDFS/HDFS_Architecture/HDFS_Read_and_Write_Path"
level: 5
type: "curriculum"
curriculum: "data_engineering"
---

# HDFS Read and Write Path

**L5 Expert** · [↑ HDFS Architecture](<../_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** In HDFS the namenode only answers "where?" and "who?"; the bytes themselves flow directly between the client and a pipeline of datanodes, with packets, acknowledgements, rack-aware placement and an explicit coherency model deciding what readers can see.

### 1. The big picture
Understanding the read and write paths explains most HDFS behaviour you will meet in practice: why a file looks empty while a writer is still running, why a write survives a datanode crash, why the first replica lands on the client's own node, and why the namenode does not become a bandwidth bottleneck. This lesson follows a single `open()`/`read()` and a single `create()`/`write()`/`close()` through the client library, the namenode and the datanodes.

It sits under HDFS architecture (components and their roles) and next to high availability (what happens when the namenode itself fails). Beneath it are the on-disk structures covered in the HDFS overview: blocks, checksums, edit log.

### 2. Core concepts and vocabulary
- **`DistributedFileSystem`** — the client-side `FileSystem` implementation for HDFS; makes the RPCs to the namenode.
- **`FSDataInputStream` / `FSDataOutputStream`** — stream objects returned to the application; they wrap `DFSInputStream` / `DFSOutputStream`, which hide datanode and namenode communication.
- **Packet** — a small chunk (including its checksums) into which the output stream splits written data.
- **Data queue / ack queue** — the two internal queues of `DFSOutputStream`: packets waiting to be sent, and packets sent but not yet acknowledged by all datanodes.
- **`DataStreamer`** — thread that asks the namenode for block allocations and sends packets down the pipeline.
- **Replica pipeline** — ordered chain of datanodes that receive the same block.
- **Network distance** — hop count in the topology tree (node → rack → data centre), used to find the "closest" replica.
- **Coherency model** — rules for when written data becomes visible to readers.
- **`hflush()` / `hsync()`** — force data to datanode memory / to datanode disk.

### 3. How it works, step by step
**Reading a file**
1. The client calls `open()` on `DistributedFileSystem`.
2. It asks the namenode (RPC) for the locations of the first few blocks. For each block the namenode returns the datanodes holding a replica, **sorted by proximity** to the client (a client running on a datanode that has a replica reads locally).
3. The returned `FSDataInputStream` connects to the closest datanode of the first block.
4. Each `read()` streams data from that datanode; at the end of the block the stream closes it and connects to the best datanode for the next block. More block locations are fetched from the namenode in batches as needed. This is invisible to the application, which sees a continuous stream.
5. On a datanode error the stream remembers the bad node (so it is not retried for later blocks) and moves to the next closest replica. Checksums are verified on every read; a corrupt block is reported to the namenode before trying another replica.

Because the client contacts datanodes directly and the namenode only supplies block locations, HDFS scales to many concurrent clients: the data traffic is spread over the whole cluster.

**Writing a file**
1. `create()` on `DistributedFileSystem` → RPC to the namenode to add the file to the namespace *with no blocks*. The namenode checks that it doesn't exist and the caller may create it; otherwise an `IOException` is thrown.
2. The client writes; `DFSOutputStream` cuts the data into packets and appends them to the **data queue**.
3. `DataStreamer` asks the namenode to allocate a block and pick datanodes → this forms the **pipeline**.
4. The streamer sends each packet to datanode 1, which stores and forwards to datanode 2, which stores and forwards to datanode 3.
5. Packets move to the **ack queue**; a packet is removed only when *every* datanode in the pipeline has acknowledged it.
6. When a block is full the process repeats with a new block and a new pipeline.
7. `close()` flushes remaining packets, waits for acks, then tells the namenode the file is complete (the namenode waits for the blocks to be minimally replicated before returning success).

![A client writing data to HDFS](<../../../../../_assets/Hadoop_The_Definitive_Guide_Tom_White/hddg_0304.png>)

**Replica placement (default, replication 3)**
1. Replica 1 on the client's node (or a random not-too-busy node if the client is outside the cluster).
2. Replica 2 on a different rack, chosen at random (off-rack).
3. Replica 3 on the same rack as replica 2, on a different node.
4. Extra replicas go to random nodes, avoiding too many on one rack.

![A typical replica pipeline](<../../../../../_assets/Hadoop_The_Definitive_Guide_Tom_White/hddg_0305.png>)

This balances reliability (two racks), write bandwidth (the pipeline crosses only one inter-rack link), read performance (choice of two racks) and distribution (one block per write on the local rack).

### 4. Worked example
```java
Configuration conf = new Configuration();
FileSystem fs = FileSystem.get(URI.create("hdfs://namenode/"), conf);
Path p = new Path("/tmp/demo.txt");

FSDataOutputStream out = fs.create(p);
out.write("hello".getBytes("UTF-8"));
out.flush();
System.out.println(fs.getFileStatus(p).getLen());   // 0  -> content not guaranteed visible

out.hflush();
System.out.println(fs.getFileStatus(p).getLen());   // 5  -> in all datanodes' memory, visible to new readers

out.hsync();                                         // fsync to datanode disks (stronger)
out.close();                                         // implicit hflush, completes the file
```
Walkthrough: after `create()` the file exists in the namespace, so `exists()` is true, yet the length is `0` because the block being written is not visible. `hflush()` guarantees the data reached every datanode in the pipeline (in memory) and becomes visible to new readers; `hsync()` additionally forces it to disk, like POSIX `fsync`. Note that a data-centre power loss between `hflush()` and a disk write could still lose that data.

### 5. Under the hood
**Pipeline failure recovery** (transparent to the writer):
1. The pipeline is closed and un-acked packets from the ack queue are pushed back to the *front* of the data queue, so nodes downstream of the failed one don't miss data.
2. The surviving datanodes get a new *generation stamp* ("identity") for the current block, reported to the namenode, so the partial block on the failed node is deleted if it comes back.
3. The failed node is dropped from the pipeline and the rest of the block is written to the two good nodes.
4. The namenode notices the block is under-replicated and arranges another replica later.
If several nodes fail, the write still succeeds as long as `dfs.namenode.replication.min` (default 1) replicas are written; the rest are made asynchronously.

**Coherency:** the file is visible in the namespace immediately, but the block currently being written is not visible to other readers until it is complete or `hflush()` is called; completed blocks are visible. `close()` performs an implicit `hflush()`. The consequence for application design: if you must not lose the last record, call `hflush()`/`hsync()` at sensible points — but each call has overhead, so tune it against throughput.

**Network distance:** the topology is a tree. Two processes on the same node have distance 0, same rack 2, different racks in the same data centre 4, different data centres 6 (each step up to the common ancestor and back down counts 1+1). Clients and the namenode use this to sort replicas; the topology is supplied by a script or class configured via `net.topology.script.file.name`.

**Write-once consequence:** one writer, append only. This makes the pipeline and coherency model simple and is why random in-place updates are not possible.

### 6. Performance and design implications
- Writes are pipelined: the client sends each byte once, regardless of replication factor; the network cost of replication is shared between datanodes.
- The replica-1-on-local-node rule makes loads from cluster nodes cheap; loading from one node concentrates first replicas there (imbalance).
- Reads prefer local/rack-local replicas, giving the data locality that schedulers exploit.
- `hflush()` per record destroys throughput; batch it.
- Many concurrent small writers multiply namenode RPCs (create, addBlock, complete) — the namenode is the limiting resource, not the datanodes.
- Short-circuit local reads (reading local block files directly) bypass the datanode process for local clients.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `FileSystem.create()/open()` | Write / read entry points | Application code |
| `hflush()` / `hsync()` | Make data visible / durable | Streaming appenders, log writers |
| `dfs.replication` | Target copies per block | Per-file or cluster default |
| `dfs.namenode.replication.min` | Minimum replicas for a write to succeed | Rarely changed |
| `net.topology.script.file.name` | Rack awareness mapping | Multi-rack clusters |
| `hdfs fsck <p> -files -blocks -locations -racks` | Show replica locations by rack | Verify placement |
| `-ignoreCrc` | Read without checksum verification | Salvage a corrupt file |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| Default placement (local, off-rack, same-remote-rack) | Single-rack cluster (no rack awareness gain) |
| `hflush()` for streams that readers tail | Per-record flushing at high rate |
| `hsync()` for data you cannot lose on power failure | Throughput-critical bulk loads (close() is enough) |
| Higher replication for hot, widely read data | Wasting capacity on temp/intermediate data |
| Clients inside the cluster for fast ingest | Single ingest node (hot spot of first replicas) |

### 9. Common mistakes
- Reading a file while it is being written and expecting all data → sees only completed blocks → `hflush()` in the writer or wait for `close()`.
- Not calling `close()` (or crashing before) → file left open/under construction, last block's length unknown → lease recovery; always close in `finally`.
- Assuming `hflush()` is durable → data lost on whole-cluster power loss → `hsync()` when needed.
- Writing from one machine only → imbalance → parallel ingestion or run the balancer.
- Assuming the namenode relays data → misreading bottlenecks → it never carries block data.

### 10. 🎯 Interview kit
**30-second answer:** The client asks the namenode for block locations (sorted by proximity) and reads straight from datanodes. For writes the namenode allocates blocks and picks a datanode pipeline; the client streams packets through the pipeline and each packet is acked by all nodes before it's discarded.

**Structure a longer answer:** clarify (read or write; failures?) → core mechanism (RPC to namenode, pipeline, packets, ack queue) → trade-offs (coherency, flush cost, replica placement) → scenario (datanode dies mid-write).

**Likely follow-ups:**
- *What happens if a datanode fails during a write?* → pipeline rebuilt, ack-queue packets requeued, block gets new generation stamp, later re-replication.
- *Where are replicas placed?* → local node, off-rack, same rack as second on a different node.
- *Is data visible after `flush()`?* → not guaranteed; `hflush()` makes it visible; `hsync()` makes it durable.
- *Does data pass through the namenode?* → no, metadata only.
- *Can two clients write one file?* → no, single writer (lease), append-only.

**Signals interviewers look for:** correct separation of control and data path, understanding of the ack queue and recovery, and knowing the exact semantics of `hflush`/`hsync`.

### 11. Check yourself
- [ ] Draw the seven steps of a write, naming the queues.
- [ ] Explain what happens to packets when the second datanode dies.
- [ ] State the replica placement rule and why it uses two racks.
- [ ] Explain why a just-flushed file may show length 0.
- [ ] Compute the network distance between a client and replicas on same node, same rack, another rack.
- [ ] Describe how the read path picks and fails over between replicas.
<!-- explainer:end -->

## References

- *Hadoop: The Definitive Guide* — Tom White — The Hadoop Distributed Filesystem › Anatomy of a File Write
- *Hadoop: The Definitive Guide* — Tom White — The Hadoop Distributed Filesystem › Coherency Model
- *Hadoop: The Definitive Guide* — Tom White — The Hadoop Distributed Filesystem › The Design of HDFS
- *Hadoop: The Definitive Guide* — Tom White — Administering Hadoop › Data backups
- *Fundamentals of Data Engineering* — Joe Reis, Matt Housley — Storage › Local disk storage
- *Hadoop: The Definitive Guide* — Tom White — HBase › Praxis
- *Database Internals* — Alex Petrov — Anti-Entropy and Dissemination › Digest Reads
- *Hadoop: The Definitive Guide* — Tom White — Hadoop IO › Warning
- *Fundamentals of Data Engineering* — Joe Reis, Matt Housley — Storage › Example: AWS EMR with S3 and HDFS
- *Designing Data-Intensive Applications* — Martin Kleppmann — Storage and Retrieval › Making B-trees reliable

