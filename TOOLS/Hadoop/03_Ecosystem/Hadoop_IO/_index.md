---
id: "TOOLS/Hadoop/03_Ecosystem/Hadoop_IO"
level: 4
type: "curriculum"
curriculum: "data_engineering"
---

# Hadoop I/O Serialization and Compression

**L4 Advanced** · [↑ Hadoop Ecosystem](<../_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** Hadoop I/O is the set of primitives that make big data safe and cheap to move — checksums for integrity, compression codecs (with a *splittable* requirement), and serialization formats (Writable, Avro, SequenceFile) that turn objects into bytes for RPC and storage.

### 1. The big picture
At multi-terabyte scale, rare events become routine: a disk flips a bit, a network link corrupts a packet, a file is too big to ship uncompressed. Hadoop therefore builds three concerns into its I/O layer: **integrity** (detect corruption), **compression** (save space and transfer time) and **serialization** (represent structured records as bytes, for both inter-process RPC and persistent storage). Choices made here — file format, codec, schema — echo through every later job: they decide how much data is read, whether a file can be processed in parallel, and whether a pipeline survives schema change.

This lesson connects to the storage/format topics earlier in the curriculum (row vs columnar formats) and to MapReduce input splits.

### 2. Core concepts and vocabulary
- **Checksum** — a small value (e.g. CRC-32, 32 bits) computed from data; a mismatch signals corruption. It detects but cannot repair.
- **Codec** — implementation of a compression algorithm, represented by `CompressionCodec` in Hadoop (`GzipCodec`, `BZip2Codec`, `SnappyCodec`, `Lz4Codec`, `LzopCodec`).
- **Splittable** — a compressed file can be cut at arbitrary points so several mappers read it in parallel.
- **Serialization / deserialization** — objects ↔ byte streams.
- **Writable** — Hadoop's original compact, fast serialization interface (`IntWritable`, `Text`, `LongWritable`…).
- **Avro** — language-neutral, schema-based binary serialization with schema resolution and a self-describing container file.
- **SequenceFile** — binary file of key-value pairs, with sync markers, splittable, optionally record/block compressed.
- **File-based data structures** — SequenceFile, MapFile (sorted, indexed), Avro data files, plus columnar ORC/Parquet.

### 3. How it works, step by step
**Data integrity.** HDFS computes a CRC-32C checksum for every 512 bytes (`io.bytes.per.checksum`) and stores it alongside the block. DataNodes verify when they receive data; clients verify when they read; each DataNode also runs a background **block scanner**. If a replica is corrupt, the client reports it, reads another replica, and the NameNode re-replicates a good copy. `LocalFileSystem` uses hidden `.crc` files for the same purpose.

**Choosing compression.**

| Format | Codec speed | Ratio | Splittable? | Notes |
|---|---|---|---|---|
| gzip (DEFLATE) | medium | good | **No** | Ubiquitous; one mapper per file |
| bzip2 | slow | best | Yes | Heavy CPU |
| LZO | fast | fair | Only with index | Licence (GPL) means separate install |
| LZ4 | very fast | fair | No (as raw) | Speed focus |
| Snappy | very fast | fair | No (as raw) | Common inside containers |

The usual rule: use a **splittable container** (SequenceFile, Avro, ORC, Parquet) with a fast codec (Snappy/LZ4) *inside* it, instead of compressing a whole text file with gzip.

**Serialization properties wanted in Hadoop:** compact, fast, extensible, interoperable. Writables are compact and fast but Java-only and hard to evolve; Avro adds language neutrality and schema evolution.

### 4. Worked example
```bash
# compress map output and final output with Snappy
hadoop jar job.jar MaxTemperature \
  -D mapreduce.map.output.compress=true \
  -D mapreduce.map.output.compress.codec=org.apache.hadoop.io.compress.SnappyCodec \
  -D mapreduce.output.fileoutputformat.compress=true \
  -D mapreduce.output.fileoutputformat.compress.type=BLOCK \
  input output

# verify a file's checksum and block health
hadoop fs -checksum /data/events/part-00000
hdfs fsck /data/events -files -blocks
```
Reading it: a 1 GB gzip text file gives **one** split — one mapper reads it all, the rest of the cluster idles. The same data in a block-compressed SequenceFile or Parquet yields one split per HDFS block. Compressing *map output* is almost always a win because the shuffle crosses the network.

### 5. Under the hood
- **Why gzip can't be split:** DEFLATE streams have no sync points; to decode byte N you need all bytes before it. SequenceFile writes a 16-byte **sync marker** about every 2 KB so a reader can start at any offset and resynchronise; bzip2 has block boundaries it can find.
- **Writable wire format:** fields are written back-to-back with no tags or schema — fast but brittle. `VIntWritable`/`VLongWritable` use variable-length encodings for small numbers.
- **Avro:** the writer's schema is stored in the file header; a reader supplies its own schema and Avro *resolves* differences (added fields with defaults, removed fields, renamed via aliases). That is what makes it good for evolving pipelines.
- **Compression type in SequenceFile:** NONE, RECORD (each value alone — poor ratio), BLOCK (batches of records — preferred).
- **Cost model:** compression trades CPU for I/O. Cluster disks and network are usually the bottleneck, so fast codecs usually reduce job time; slow codecs (bzip2) only when storage cost dominates.

### 6. Performance and design implications
- File-format and compression choice often matters more than any job tuning — switching CSV to Parquet can speed jobs by orders of magnitude.
- Always check splittability before choosing a codec for raw files.
- Compress intermediate (map) output; compress final output for archival.
- Keep checksums on; run on ECC-memory hardware; monitor corrupt-block counts via `fsck` and NameNode metrics.
- Prefer self-describing, schema-versioned formats for data shared across teams.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `hadoop fs -checksum path` | Show file checksum | Compare copies |
| `hdfs fsck path -files -blocks -locations` | Check health, corrupt/missing blocks | Integrity audit |
| `io.bytes.per.checksum` | Bytes per checksum chunk (512) | Rarely changed |
| `mapreduce.map.output.compress(.codec)` | Compress shuffle data | Almost always |
| `mapreduce.output.fileoutputformat.compress(.codec/.type)` | Compress job output | Archival/storage |
| `io.compression.codecs` | Registered codecs | Adding LZO etc. |
| `hadoop fs -text file` | Read compressed/SequenceFile as text | Debugging |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| Snappy/LZ4 inside a container format | Whole-file gzip on large text for parallel jobs |
| bzip2 for cold, rarely read archives | bzip2 on hot pipelines (CPU) |
| Avro for row-oriented, evolving ingestion data | Writables for cross-language data |
| Parquet/ORC for analytics | Row formats for column-selective scans |

### 9. Common mistakes
- gzip a huge CSV and run MapReduce/Spark on it → single task → switch to splittable or chunked files.
- Ignoring corruption reports → silent data loss risk → schedule `fsck` and alert on corrupt blocks.
- Using Java serialization or Writables for shared data → lock-in and breakage on change → use Avro/Parquet with schemas.
- Block compression disabled in SequenceFiles → poor ratio → set `BLOCK`.

### 10. 🎯 Interview kit
**30-second answer:** Hadoop detects corruption with per-chunk CRC checksums verified on read, write and by background scanners, then heals from other replicas. For efficiency it compresses with codecs — preferring splittable containers with Snappy/LZ4 — and serializes with Writables or, for interoperability and evolution, Avro.

**Structure a longer answer:** clarify → core mechanism → trade-offs → close with a real scenario.

**Likely follow-ups:**
- *Why isn't gzip splittable and why does it matter?* → no sync points; one mapper per file.
- *Which codec for the shuffle?* → fast one (Snappy/LZ4).
- *Avro vs Parquet?* → row vs columnar; ingestion vs analytics.
- *What does HDFS do on a checksum failure?* → read another replica and re-replicate.

**Signals interviewers look for:** connecting format and codec decisions to parallelism and I/O cost.

### 11. Check yourself
- [ ] Explain how HDFS detects and repairs a corrupt block.
- [ ] Pick a codec/format combination for a 10 TB daily log feed and justify it.
- [ ] Describe what a sync marker is and why it enables splitting.
- [ ] Explain Avro schema resolution with an example of an added field.
<!-- explainer:end -->

## References

- *Hadoop: The Definitive Guide* — Tom White — Hadoop IO › Chapter 5. Hadoop I/O
- *Hadoop: The Definitive Guide* — Tom White — Hadoop IO › Codecs
- *Hadoop: The Definitive Guide* — Tom White — Hadoop IO › File-Based Data Structures
- *Hadoop: The Definitive Guide* — Tom White — Hadoop IO › Serialization Frameworks
- *Hadoop: The Definitive Guide* — Tom White — Hadoop IO › Implementing a Custom Writable
- *Hadoop: The Definitive Guide* — Tom White — Hadoop IO › Writable collections
- *Fundamentals of Data Engineering* — Joe Reis, Matt Housley — The Future of Data Engineering › Appendix A. Serialization and Compression Technical Details
- *Hadoop: The Definitive Guide* — Tom White — MapReduce Types and Formats › Binary Input
- *Hadoop: The Definitive Guide* — Tom White — I
- *Hadoop: The Definitive Guide* — Tom White — Hadoop IO › Serialization

