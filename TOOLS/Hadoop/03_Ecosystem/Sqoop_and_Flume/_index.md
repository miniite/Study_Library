---
id: "TOOLS/Hadoop/03_Ecosystem/Sqoop_and_Flume"
level: 4
type: "curriculum"
curriculum: "data_engineering"
---

# Sqoop and Flume

**L4 Advanced** · [↑ Hadoop Ecosystem](<../_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** Sqoop moves *tables* between relational databases and Hadoop in parallel batch jobs, while Flume continuously collects *event streams* (logs, clickstream) from many machines into HDFS through agents built from sources, channels and sinks.

### 1. The big picture
Data needs to get into Hadoop before anything else is possible. Two very different shapes exist: **structured data sitting in a database** (customers, orders) and **a never-ending flow of events** (web server logs). Sqoop and Flume are the classic Hadoop answers to each. They are the ingestion layer in the lifecycle you saw earlier: batch extraction vs streaming/event ingestion.

Both are older tools; today CDC pipelines (Debezium), Kafka Connect, Spark/Flink and managed cloud services often fill these roles, but the patterns — parallel split-by-key import, transactional channel hand-off, tiered aggregation — remain the vocabulary of ingestion.

### 2. Core concepts and vocabulary
- **Sqoop import / export** — database → HDFS/Hive/HBase, and back.
- **Split column (`--split-by`)** — column used to divide a table among parallel mappers.
- **Connector / JDBC driver** — how Sqoop talks to the database.
- **Incremental import** — `append` (by increasing key) or `lastmodified` (by timestamp).
- **Flume agent** — a JVM process hosting sources, channels, sinks.
- **Source** — receives events (spooling directory, exec/tail, Avro, syslog, HTTP, Kafka).
- **Channel** — buffer between source and sink: *memory* (fast, lost on crash) or *file* (durable).
- **Sink** — writes events out (HDFS, logger, Avro for next tier, HBase, Solr, Kafka).
- **Event** — a byte payload plus string headers.
- **Fan-out, selector, tiers** — replicate/route events to multiple channels; layer agents to aggregate.

### 3. How it works, step by step
**Sqoop import**
1. Sqoop queries the database for metadata and the min/max of the split column.
2. It divides the range into N partitions (default 4 mappers).
3. Each mapper runs `SELECT … WHERE split BETWEEN a AND b` and writes text/Avro/SequenceFile/Parquet files to HDFS in parallel (a map-only job).
4. Optionally it generates a Java class for the record and creates/loads a Hive table (`--hive-import`), inferring the Hive schema (sometimes with a precision warning).
5. Export runs the reverse: mappers read HDFS files and issue batched `INSERT`s.

**Flume flow (one agent)**
1. Source takes a batch of events and starts a **transaction** to put them in the channel.
2. Channel stores them durably (file) or in RAM.
3. Sink takes a batch in its own transaction, writes it to the destination, and commits (removing from the channel).
4. If delivery fails, the transaction rolls back and events are retried — **at-least-once** delivery (duplicates possible after failures).

```
 web server → [source] → (channel) → [sink: avro] ──► tier-2 agent ──► [sink: hdfs]
```

### 4. Worked example
```bash
# Sqoop: import MySQL table 'widgets' with 4 mappers into Hive
sqoop import --connect jdbc:mysql://dbhost/hadoopguide \
  --username etl -P --table widgets \
  --split-by id --num-mappers 4 \
  --target-dir /data/raw/widgets --as-avrodatafile --hive-import

# incremental append (only new rows)
sqoop import ... --incremental append --check-column id --last-value 10000

# export aggregated results back
sqoop export --connect jdbc:mysql://dbhost/reports --table zip_profits \
  --export-dir /user/hive/warehouse/zip_profits --input-fields-terminated-by '\001'
```
```properties
# Flume: spool directory -> HDFS (agent1.conf)
agent1.sources  = src1
agent1.channels = ch1
agent1.sinks    = snk1
agent1.sources.src1.type = spooldir
agent1.sources.src1.spoolDir = /var/log/incoming
agent1.sources.src1.channels = ch1
agent1.channels.ch1.type = file
agent1.sinks.snk1.type = hdfs
agent1.sinks.snk1.hdfs.path = /data/events/%Y-%m-%d
agent1.sinks.snk1.hdfs.fileType = DataStream
agent1.sinks.snk1.hdfs.useLocalTimeStamp = true
agent1.sinks.snk1.channel = ch1
```
Start with `flume-ng agent --conf conf --conf-file agent1.conf --name agent1`. Drop a file in `/var/log/incoming`: it is renamed `.COMPLETED` and its lines appear as events in HDFS.

### 5. Under the hood
- Sqoop's parallelism depends on a well-distributed numeric split key; a skewed or non-indexed key makes one mapper do most work and hammers the source DB. Use `--boundary-query` or `--split-by` on a different column, or `-m 1` for tables without a usable key.
- Sqoop runs mappers *as the database's clients*: too many mappers can overload an OLTP system; use read replicas or off-peak windows.
- **Flume fan-out**: one source can feed several channels. Each channel gets its own transaction. Mark a channel *optional* (`selector.optional`) so its failure doesn't block delivery to a required channel (HDFS). A *multiplexing selector* routes by header value instead of replicating.
- **Agent tiers**: first-tier agents on each web server forward to a second tier that aggregates into fewer, larger HDFS files — fewer small files, more frequent rolling.
- If the agent crashes before both channel transactions commit, events are redelivered — even for optional channels.
- Memory channel = throughput, file channel = durability; pick consciously.

### 6. Performance and design implications
- Sqoop: tune mappers, fetch size, use `--direct` where the database offers a fast native path, import into columnar formats, and avoid peak hours.
- Flume: batch size, channel capacity and `hdfs.rollInterval/rollSize/rollCount` control the size of files you create; avoid thousands of tiny files.
- Monitor channel fill percentage (backpressure sign) and sink failure counters.
- Use at-least-once plus idempotent downstream processing or dedupe keys.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `sqoop import / export / list-tables` | Core transfers | Batch ETL |
| `--split-by`, `-m` | Partition column, mapper count | Parallelism |
| `--incremental append|lastmodified` | Load only new/changed rows | Daily refresh |
| `--hive-import`, `--as-parquetfile` | Target Hive / format | Land in warehouse |
| `flume-ng agent -n name -f file` | Start agent | Run ingestion |
| `hdfs.rollInterval/Size/Count` | When to close an HDFS file | Control file size |
| `selector.optional` | Mark channels as best-effort | Non-critical fan-out |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| Sqoop for periodic table dumps / small to medium incremental loads | Real-time change capture (use CDC tools) |
| Flume for log/event collection into HDFS | Very high-throughput, replayable streams (Kafka is the modern fit) |
| File channel for durability | Memory channel for data you cannot lose |
| Tiered agents for many sources | Tiny cluster where tiers add complexity |

### 9. Common mistakes
- Splitting on a skewed or string column → one slow mapper → pick uniform numeric key.
- Too many parallel Sqoop mappers against production OLTP → outages → cap and schedule.
- Memory channel for must-not-lose logs → data loss on crash → file channel.
- HDFS sink rolling every few seconds → millions of small files → increase roll size/time, add a tier.
- Assuming exactly-once → duplicates after retry → dedupe downstream.

### 10. 🎯 Interview kit
**30-second answer:** Sqoop runs parallel map-only jobs that split a table by key to import/export between an RDBMS and Hadoop. Flume is a streaming collector: agents with source → channel → sink, transactional hand-off for at-least-once delivery, fan-out and tiers for aggregation.

**Structure a longer answer:** clarify → core mechanism → trade-offs → close with a real scenario.

**Likely follow-ups:**
- *How does Sqoop parallelise?* → `--split-by` min/max ranges across mappers.
- *Delivery guarantee in Flume?* → at-least-once via channel transactions.
- *Memory vs file channel?* → speed vs durability.
- *How do you do incremental loads?* → `append` or `lastmodified`, store last value.

**Signals interviewers look for:** you reason about load on the source system and about delivery semantics.

### 11. Check yourself
- [ ] Describe what each Sqoop mapper executes.
- [ ] Draw a two-tier Flume topology and explain why you need the second tier.
- [ ] Explain optional vs required channels.
- [ ] Choose between Sqoop, Flume and a CDC/Kafka pipeline for three scenarios.
<!-- explainer:end -->

## References

- *Hadoop: The Definitive Guide* — Tom White — Flume › Integrating Flume with Applications
- *Hadoop: The Definitive Guide* — Tom White — Colophon › link
- *Hadoop: The Definitive Guide* — Tom White — Sqoop › Performing an Export
- *Hadoop: The Definitive Guide* — Tom White — Sqoop › Sqoop Connectors
- *Hadoop: The Definitive Guide* — Tom White — Flume › Delivery Guarantees
- *Hadoop: The Definitive Guide* — Tom White — Flume › Note
- *Hadoop: The Definitive Guide* — Tom White — Sqoop › Getting Sqoop
- *Hadoop: The Definitive Guide* — Tom White — Flume › Fan Out
- *Hadoop: The Definitive Guide* — Tom White — Flume › Installing Flume
- *Hadoop: The Definitive Guide* — Tom White — Sqoop › A Sample Import

