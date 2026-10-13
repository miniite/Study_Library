---
id: "02_Data_Engineering/02_Storage_and_Formats"
level: 3
type: "curriculum"
curriculum: "data_engineering"
---

# Storage Systems and File Formats

**L3 Practitioner** · [↑ Data Engineering](<../_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** Storage choices for analytics come down to how bytes are laid out — row vs column, serialised in which format, on one machine or many — and each layout is a trade between write convenience, read speed, compression and interoperability.

### 1. The big picture

Every pipeline eventually writes bytes somewhere and reads them back. The layout of those bytes decides how fast queries run, how much storage costs, how easily other tools can read the data and how painful schema changes are. Two decisions dominate:

1. **Orientation** — are values stored row by row (all fields of a record together) or column by column (all values of one field together)?
2. **Serialisation** — in what format are records or columns encoded (CSV, JSON, Avro, Parquet, ORC, in-memory Arrow)?

A third dimension is **where** the data lives: a single machine or a distributed storage system (object storage, HDFS, a cloud warehouse) that spreads and replicates data across servers. This lesson is the area-level map for the *Storage and Formats* part of the curriculum; the child lessons go deep on specific encodings, Avro and schema evolution, and data models.

### 2. Core concepts and vocabulary

- **File** — a sequence of bytes on disk; also the universal medium for exchanging data between organisations (CSV, Excel, JSON, XML, text).
- **Serialisation** — flattening in-memory data into a byte format that another process can decode; also the idea behind low-level database storage.
- **Row-oriented storage** — each record's fields are contiguous; good for point lookups, updates and reading whole records.
- **Column-oriented storage** — each column's values are contiguous (often in separate files); good for scans and aggregates over a few columns.
- **Compression / encoding** — dictionary, run-length and bitmap encoding exploit similarity of values in a column.
- **Vectorised execution** — processing many values of a column per CPU instruction.
- **Splittable** — a file format that can be divided at block boundaries so many workers read it in parallel.
- **Sync marker** — a unique marker between blocks that lets a reader resynchronise after seeking to an arbitrary byte offset.
- **Single-machine vs distributed storage** — one server vs many servers coordinated for scale and redundancy.
- **Sector / page** — the smallest unit a device or storage engine transfers; the reason random I/O is expensive on spinning disks.

### 3. How it works, step by step

How a column-oriented file is read for `SELECT SUM(amount) FROM sales WHERE country = 'DE'`:

1. The engine consults file/footers metadata (schema, row-group min/max, column offsets).
2. It skips row groups whose statistics rule out `country = 'DE'`.
3. It reads only the `country` and `amount` column chunks, not the other columns.
4. Compressed column data is decoded in blocks; filters and sums run on vectors of values.
5. Partial sums from parallel workers are combined.

The layouts compared:

| | Row-oriented | Column-oriented |
|---|---|---|
| Layout | `[id,name,amount][id,name,amount]…` | `[id,id,…][name,name,…][amount,amount,…]` |
| Best for | Point reads, small updates, full-record access, transactional work | Large scans, aggregates over few columns, analytics |
| Compression | Moderate (mixed types per row) | High (same-type neighbours, repeated values) |
| Writes | Easy, in place or append | Harder: many column files; usually batched or LSM-style |
| Streaming writes | Recoverable up to last sync | Writer buffers a row group; a crash can lose the in-progress file |
| Typical formats | CSV, JSON, Avro, SequenceFile | Parquet, ORC |

![Row-oriented versus column-oriented storage](<../../_assets/Hadoop_The_Definitive_Guide_Tom_White/hddg_0504.png>)

### 4. Worked example

Same 100-million-row `events` table, three storage choices, and what a typical analytics query costs.

```sql
-- Query used for the comparison
SELECT event_date, COUNT(*) AS n, SUM(revenue) AS rev
FROM   events
WHERE  event_date BETWEEN '2026-10-01' AND '2026-10-07'
GROUP  BY event_date;
```

```text
Table: 40 columns, ~1 KB per row, 100 M rows (~100 GB raw)

Option A  gzip'd CSV in object storage
  reads all bytes (cannot skip columns or row groups), plus a text parse
  gzip is not splittable -> few parallel readers
  scan ≈ 30 GB compressed, slow, error-prone typing

Option B  Avro (row-oriented, snappy)
  splittable, typed, schema embedded
  still reads whole rows -> scan ≈ 40 GB

Option C  Parquet (columnar, snappy) partitioned by event_date
  prunes 358 of 365 date partitions
  reads 2 of 40 columns (event_date, revenue)
  scan ≈ 0.3 GB
```

The numbers are illustrative orders of magnitude, but the pattern is real: the same logical query moves by two orders of magnitude depending on format and layout. What to look at in practice: bytes scanned in the query profile, number of files opened, and whether partition and column pruning appear in the plan.

### 5. Under the hood

**Why columns compress well.** Values in one column share a type and often a distribution, so run-length, dictionary and bitmap encodings work. A bitmap index turns a column with *n* distinct values into *n* bitmaps with one bit per row; predicates like `product IN (30, 68, 69)` become bitwise ORs and `product = 31 AND store = 3` becomes a bitwise AND, which is valid because every column file keeps rows in the same order. When *n* is large the bitmaps are sparse and are run-length encoded.

**Why writes are hard.** Compressed, sorted column files cannot be updated in place: inserting a row in the middle would rewrite every column file consistently. The practical solution is the LSM idea: buffer writes in a sorted in-memory structure, flush to new immutable files in bulk, and merge at query time and in the background. The query layer hides the split between recent in-memory data and on-disk columns.

**CPU and cache effects.** Reading many values of one column in a run improves cache use and enables vectorised instructions; choosing a layout is just one of several optimisations a columnar engine uses together.

**Hardware lineage.** Many classical structures were designed for spinning disks, where seek and rotation dominate cost and sequential access is cheap; the minimum transfer is a sector (traditionally 512 B to 4 KB). SSDs and byte-addressable memory changed the trade-offs, but the idea that bulk sequential I/O beats random I/O still shapes file formats.

**Serialisation as a design decision.** It influences network cost, CPU overhead and latency. Lakes pair a base store (object storage) with serialisation standards balancing interoperability and performance. Worth knowing: Parquet (columnar files), hybrid table formats (Hudi-style), and Arrow (in-memory columnar).

### 6. Performance and design implications

- **Match layout to access pattern**: scans/aggregates → columnar; record lookups and frequent updates → row.
- **Prune early**: partition by commonly filtered columns (usually date); keep file sizes in the hundreds of MB.
- **Avoid text formats in pipelines**: CSV is error-prone (quoting, delimiters, types) and slow; if you must keep it for archival, document the exact dialect.
- **Use JSON Lines for raw landing** of API/application data, then convert to a columnar format for intermediate and serving layers.
- **Splittability matters**: non-splittable compressed text caps parallelism.
- **Distributed storage** gives scale and redundancy but means consistency semantics and network cost matter.
- **Measure**: bytes scanned vs bytes returned, compression ratio, files per partition, skew in file sizes.

### 7. Commands and settings you should know

| Command / setting | What it does | When to use |
|---|---|---|
| `parquet-tools meta/schema <file>` | Prints Parquet footer, row groups, schema | Inspecting file layout |
| `COPY … TO … (FORMAT PARQUET)` / `df.write.parquet()` | Writes columnar files | Converting landing data |
| `PARTITIONED BY` / partition columns | Directory-level pruning | Large time-based tables |
| Compression codec (snappy, zstd, gzip) | Trades CPU for size | Balancing scan speed and storage |
| Row-group / block size | Unit of column chunking and skipping | Tuning read parallelism |
| `avro-tools` / `jq` on JSONL | Inspecting row-based files | Debugging ingestion output |

### 8. Trade-offs and decisions

| Choose this when | Avoid / watch out when |
|---|---|
| Columnar (Parquet/ORC) — analytics, wide tables, selective columns | Streaming writes needing durability per record; frequent single-row updates |
| Row-based binary (Avro) — ingestion, streaming messages, schema evolution | Aggregation-heavy queries over few columns |
| JSON/JSONL — raw API landing, flexible schema | Large-scale analytics, strict typing |
| CSV — exchange with systems you do not control | Pipeline-internal storage (error-prone, slow) |
| XML — only when a legacy partner requires it | New designs; slow to parse |
| Distributed storage — large volume, redundancy | Tiny datasets where one machine is simpler |

### 9. Common mistakes

- **Using CSV between pipeline stages** → silent type and quoting bugs → typed binary formats.
- **Gzip-compressed single big file** → one reader does all the work → use splittable codecs and many files.
- **Tiny files** → metadata and open-file overhead → compact to 128 MB–1 GB files.
- **Columnar format for single-row updates** → rewrite storms → use an OLTP store or a table format with merge support.
- **Choosing format by fashion** → mismatch with access pattern → profile real queries.
- **No schema recorded with CSV archives** → future readers can't decode → archive the dialect and schema with the data.

### 10. 🎯 Interview kit

**30-second answer:** "Row stores keep whole records together and suit transactional lookups and updates; column stores keep each column together, so analytics reads only needed columns and compresses very well, at the cost of harder writes. I land raw data as JSONL or Avro, convert to Parquet partitioned by date for analytics, and avoid CSV inside pipelines."

**Structure a longer answer:** clarify access pattern and write pattern → layout choice → format and partitioning → trade-offs (small files, updates, streaming) → scenario.

**Likely follow-ups:**
- *Why is columnar better for aggregates?* → Column pruning, better compression, vectorised processing.
- *Why are updates hard in columnar files?* → Compressed immutable column files; must rewrite or use LSM/merge.
- *What does splittable mean?* → Readers can start at an arbitrary block boundary using sync markers.
- *Parquet vs Avro?* → Parquet columnar for analytics; Avro row-based for ingestion/streaming and schema evolution.

**Signals interviewers look for:** reasoning from access patterns, awareness of file-size and partitioning effects.

### 11. Check yourself

- [ ] Sketch row vs column layout for a three-column table and state which query favours which.
- [ ] Explain bitmap encoding and how `AND`/`OR` predicates use it.
- [ ] Explain why compressed column files can't be updated in place.
- [ ] Justify a format for raw landing vs analytics serving.
- [ ] Describe what makes a file format splittable.
- [ ] Explain why CSV is discouraged inside pipelines.
<!-- explainer:end -->

## In this section (study in order)

1. [Storage Media and Hardware](<Storage_Media_and_Hardware/_index.md>) (L4 Advanced)
2. [File Block and Object Storage](<File_Block_and_Object_Storage/_index.md>) (L4 Advanced)
3. [Data Models and Query Languages](<Data_Models_and_Query_Languages/_index.md>) (L4 Advanced)
4. [Encoding and Schema Evolution](<Encoding_and_Schema_Evolution/_index.md>) (L4 Advanced)
5. [Avro Data Files and Schema Resolution](<Avro_Format/_index.md>) (L5 Expert)
6. [Parquet Columnar Format](<Parquet_Format/_index.md>) (L4 Advanced)

## References

- *Fundamentals of Data Engineering* — Joe Reis, Matt Housley — The Future of Data Engineering › Avro
- *Fundamentals of Data Engineering* — Joe Reis, Matt Housley — Storage › Serialization
- *Fundamentals of Data Engineering* — Joe Reis, Matt Housley — The Future of Data Engineering › Row-Based Serialization
- *Hadoop: The Definitive Guide* — Tom White — Hadoop IO › Other File Formats and Column-Oriented Formats
- *Database Internals* — Alex Petrov — Introduction and Overview › Distinctions and Optimizations
- *Database Internals* — Alex Petrov — File Formats › Chapter 3. File Formats
- *Designing Data-Intensive Applications* — Martin Kleppmann — Storage and Retrieval › Writing to Column-Oriented Storage
- *Fundamentals of Data Engineering* — Joe Reis, Matt Housley — Storage › Data Storage Systems
- *Designing Data-Intensive Applications* — Martin Kleppmann — Storage and Retrieval › Column Compression
- *Fundamentals of Data Engineering* — Joe Reis, Matt Housley — Data Generation in Source Systems › Source Systems: Main Ideas

