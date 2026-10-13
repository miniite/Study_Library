---
id: "TOOLS/Hadoop/03_Ecosystem"
level: 3
type: "curriculum"
curriculum: "data_engineering"
---

# Hadoop Ecosystem

**L3 Practitioner** · [↑ Hadoop](<../_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** The Hadoop ecosystem is the set of tools built around HDFS and YARN — ingestion (Flume, Sqoop), higher-level processing (Pig, Hive, Crunch, Spark), serving (HBase) and coordination (ZooKeeper) — most of which now matter as legacy or as building blocks inside newer data platforms.

### 1. The big picture
Hadoop began as two things: HDFS for storage and MapReduce for processing. Writing raw MapReduce is verbose, so a ring of projects grew around the core, each removing a piece of friction:

| Need | Tool | Idea |
|---|---|---|
| Get streaming event data in | **Flume** | agents wiring sources → channels → sinks into HDFS |
| Get relational data in/out | **Sqoop** | bulk import/export between databases and Hadoop |
| Query with SQL | **Hive** | SQL-like HiveQL compiled to MapReduce/Tez/Spark, metadata in a *metastore* |
| Scripting dataflows | **Pig** | Pig Latin data-flow language, client-side only |
| Java pipelines | **Crunch / Cascading** | typed pipeline APIs that plan multi-job workflows |
| Fast in-memory/iterative processing | **Spark** | DAG engine, runs on YARN |
| Low-latency random access | **HBase** | column-family store on HDFS |
| Coordination | **ZooKeeper** | small replicated config/lock service |

The picture is a stack: HDFS (storage) → YARN (resources) → processing engines → query/scripting layers → applications. As the *Fundamentals of Data Engineering* authors put it, "Hadoop is dead — long live Hadoop": the pure MapReduce model has fallen by the wayside and tools like Pig are on life support for legacy jobs, but HDFS remains widely used and underpins many current engines (for example Spark on HDFS or on Amazon EMR with S3 and an ephemeral HDFS). Smaller organisations often find cloud services cheaper than running a small cluster.

### 2. Core concepts and vocabulary
- **Agent / source / channel / sink** — Flume's building blocks; a channel buffers events between source and sink.
- **Metastore** — Hive's catalog of tables, schemas and partitions (a relational database).
- **HiveQL** — SQL-like dialect; tables are mapped onto files in HDFS.
- **Pig Latin** — procedural data-flow scripting language; executes as MapReduce (or Tez/Spark) jobs.
- **Execution mode** — Pig: local or MapReduce (Tez/Spark emerging); Hive: engine setting.
- **ZooKeeper znode / ephemeral node / watch** — tree of small data nodes, nodes tied to a session, and change notifications.
- **Ensemble** — a ZooKeeper cluster of (usually 3 or 5) servers using majority votes.
- **Distributed monolith** — a cluster where all jobs share one set of dependencies and versions (a typical Hadoop pain).

### 3. How it works, step by step
**Flume ingestion topology**
1. Edge agents run next to web servers; a *source* tails logs or receives events.
2. Events go into a *channel* (memory or file) which holds them until delivered.
3. A *sink* forwards them — to another agent's source for aggregation, and finally to HDFS (or HBase/Solr).
4. Delivery is transactional between source→channel→sink so events are not lost when a stage restarts.

**Hive query lifecycle**
1. Client (CLI / HiveServer2) submits HiveQL; configuration comes from `hive-site.xml` (`fs.defaultFS`, resource manager address; local mode if unset).
2. The driver consults the **metastore** for table schemas, partitions and file locations.
3. The query is compiled into a plan of MapReduce/Tez/Spark stages.
4. Stages run on YARN, reading and writing HDFS files; results return to the client.

**Pig**: runs entirely on the client — nothing to install on the cluster. Statements are lazily built into a logical plan and executed when you `DUMP` or `STORE`.

**ZooKeeper use**: a process creates an *ephemeral* znode to claim leadership; others `watch` it; if the leader dies its session ends, the znode vanishes, watchers are notified and elect a new leader.

### 4. Worked example
```sql
-- Hive: expose HDFS files as a table and aggregate them
CREATE EXTERNAL TABLE weather (station STRING, year INT, temp INT)
  ROW FORMAT DELIMITED FIELDS TERMINATED BY '\t'
  LOCATION '/data/weather';

SELECT year, MAX(temp) FROM weather WHERE temp != 9999 GROUP BY year;
```
```pig
-- Pig Latin: the same logic as a data flow
records = LOAD '/data/weather' AS (station:chararray, year:int, temp:int);
valid   = FILTER records BY temp != 9999;
grouped = GROUP valid BY year;
maxtemp = FOREACH grouped GENERATE group, MAX(valid.temp);
STORE maxtemp INTO '/out/maxtemp';
```
```properties
# Flume agent: tail web logs into HDFS via a memory channel
a1.sources = r1  a1.channels = c1  a1.sinks = k1
a1.sources.r1.type = exec
a1.sources.r1.command = tail -F /var/log/web.log
a1.channels.c1.type = memory
a1.sinks.k1.type = hdfs
a1.sinks.k1.hdfs.path = /logs/web/%Y-%m-%d
a1.sources.r1.channels = c1
a1.sinks.k1.channel = c1
```
All three express "max temperature per year"; the difference is the abstraction (declarative SQL vs data flow vs configuration wiring) and the fact that none requires writing a Java `Mapper`.

### 5. Under the hood
- **Why ZooKeeper at all?** Coordination (leader election, membership, partition assignment) is hard because of *partial failure*: after a network error a sender cannot know whether the message arrived. ZooKeeper doesn't hide partial failure; it provides primitives — atomic operations, ephemeral nodes, ordered updates, notifications — with which applications can handle it safely. It runs on a fixed small ensemble (majority votes among 3–5 nodes) while serving many clients, effectively "outsourcing" consensus, ordering and failure detection. It stores small, slow-changing facts such as "node X leads partition 7", not high-rate application state. Higher-level helpers such as Apache Curator exist because correct use is still subtle. HDFS HA and YARN RM HA both use it for failover election.
- **Hive vs a database.** Hive keeps data in files on HDFS (schema-on-read), has no row-level transactions by default and high query latency; the metastore is the only "database" part and usually lives in an external RDBMS shared by clients.
- **Pig.** Client-only architecture; more flexible than SQL for multi-step transformations; legacy today.
- **Spark.** Not MapReduce-based; runs on YARN (client or cluster mode), caches data in memory between jobs and plans arbitrary DAGs; it carries its own libraries for SQL, ML, graphs and streaming.
- **The distributed-monolith problem.** One cluster hosts Hive, Pig and Spark with a single version of Hadoop libraries and Java, so upgrades are coordinated and risky. Remedies: ephemeral per-job clusters in the cloud (EMR, Dataproc) or containerised environments.
- **Storage/compute separation.** A common cloud pattern: data of record in object storage (S3), an ephemeral HDFS on SSD for intermediate results, results written back to S3 when the cluster ends.

### 6. Performance and design implications
- Hive/Pig performance is bounded by the engine below: MapReduce stages write to disk between steps; Tez and Spark avoid this.
- Partitioning and file format (ORC/Parquet) matter more for SQL-on-Hadoop speed than the SQL text.
- Flume's channel choice trades durability (file) against throughput (memory).
- ZooKeeper is for coordination, not data: keep znodes small and updates infrequent.
- Many small files hurt all tools: namenode memory, task count and listing time.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `hive -hiveconf k=v` / `--config dir` | Session or directory-level Hive config | Switching clusters |
| `hive-site.xml` | Metastore + cluster settings | Install |
| `pig -x local` / `-x mapreduce` | Pig execution mode | Test vs production |
| `flume-ng agent -n a1 -f agent.conf` | Start a Flume agent | Ingestion |
| `sqoop import --connect … --table …` | Import an RDBMS table | Bulk load |
| `zkCli.sh`, four-letter word `mntr` on port 2181 | Inspect ZooKeeper | Debugging |
| `spark-submit --master yarn --deploy-mode cluster` | Run Spark on YARN | Production jobs |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| Hive/Spark SQL for analyst access to HDFS data | Sub-second OLTP-style lookups (use HBase or a database) |
| Flume for event/log streams into HDFS | Heavy stream processing with exactly-once needs (use a log/stream platform) |
| Sqoop for periodic bulk transfers | Continuous replication (use CDC) |
| Pig only to maintain legacy scripts | New development |
| ZooKeeper for leader election and config | Storing large or fast-changing data |
| Ephemeral cloud clusters | Dependency conflicts on one shared on-prem cluster |

### 9. Common mistakes
- Running the Hive metastore on embedded Derby in a shared setup → only one session works → use an external RDBMS metastore.
- Using ZooKeeper as a database → write amplification and slowness → keep it for small coordination data.
- Single-node ZooKeeper or an even-sized ensemble → no fault tolerance → run 3 or 5.
- Treating Hive like an OLTP database → latency disappointments → batch/analytic use only.
- Forgetting that all tools share the cluster's library versions → upgrade breakage → isolate environments.
- Adding new Pig-based pipelines → maintenance burden → prefer Spark/SQL.

### 10. 🎯 Interview kit
**30-second answer:** The ecosystem layers tools over HDFS and YARN: Flume/Sqoop for ingestion, Hive/Pig/Spark for processing and SQL, HBase for random access, ZooKeeper for coordination. MapReduce itself is largely superseded by Spark/Tez, but HDFS and Hive's metastore remain common.

**Structure a longer answer:** layer diagram → one example per layer → why each exists (MapReduce friction) → current status and cloud alternatives.

**Likely follow-ups:**
- *Hive vs a relational database?* → files on HDFS, schema-on-read, batch latency, metastore for metadata.
- *Why ZooKeeper?* → consensus and failure detection as a service; leader election, membership.
- *Why is Hadoop "dead"?* → MapReduce replaced; the storage and ecosystem pieces live on.
- *How would you ingest web logs into HDFS?* → Flume agents with channels, or a log/stream platform.
- *What is the distributed monolith problem?* → shared dependencies across all jobs; ephemeral clusters help.

**Signals interviewers look for:** balanced, current view — you know the legacy tools but also what replaced them.

### 11. Check yourself
- [ ] Sketch the stack from HDFS up to Hive and name one tool per layer.
- [ ] Explain source/channel/sink in Flume.
- [ ] Describe the Hive query path including the metastore.
- [ ] Explain ZooKeeper ephemeral nodes in a leader election.
- [ ] Say why a small on-prem Hadoop cluster may be worse than a cloud alternative.
<!-- explainer:end -->

## In this section (study in order)

1. [Hadoop I/O Serialization and Compression](<Hadoop_IO/_index.md>) (L4 Advanced)
2. [Hive](<Hive/_index.md>) (L4 Advanced)
3. [HBase](<HBase/_index.md>) (L4 Advanced)
4. [ZooKeeper](<ZooKeeper/_index.md>) (L4 Advanced)
5. [Sqoop and Flume](<Sqoop_and_Flume/_index.md>) (L4 Advanced)
6. [Pig Crunch and Cascading](<Pig_and_Crunch/_index.md>) (L4 Advanced)

## References

- *Hadoop: The Definitive Guide* — Tom White — Flume › Chapter 14. Flume
- *Designing Data-Intensive Applications* — Martin Kleppmann — G › H
- *Hadoop: The Definitive Guide* — Tom White — Chapter 16. Pig › Installing and Running Pig
- *Hadoop: The Definitive Guide* — Tom White — Q › S
- *Hadoop: The Definitive Guide* — Tom White — Hive › Running Hive
- *Fundamentals of Data Engineering* — Joe Reis, Matt Housley — Storage › Hadoop is dead. Long live Hadoop!
- *Hadoop: The Definitive Guide* — Tom White — Setting Up a Hadoop Cluster › Important Hadoop Daemon Properties
- *Designing Data-Intensive Applications* — Martin Kleppmann — Membership and Coordination Services › Allocating work to nodes
- *Hadoop: The Definitive Guide* — Tom White — Cascading in Practice › Hadoop and Cascading at ShareThis
- *Hadoop: The Definitive Guide* — Tom White — Colophon › link

