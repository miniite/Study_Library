---
id: "02_Data_Engineering/02_Storage_and_Formats/Parquet_Format"
level: 4
type: "curriculum"
curriculum: "data_engineering"
---

# Parquet Columnar Format

**L4 Advanced** · [↑ Storage Systems and File Formats](<../_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** Parquet is an open, self-describing columnar file format that stores each column's values together (with efficient encodings and compression) and supports nested data, so analytical engines read only the columns and rows they need.

### 1. The big picture

Analytical queries typically touch a few columns of very wide tables. A row-oriented file forces you to read every column of every row; a **columnar** file keeps each column's values adjacent, so unneeded columns are skipped entirely and the similar neighbouring values compress extremely well.

Apache Parquet, created by engineers at Twitter and Cloudera, became the de facto standard for data lakes. It is portable across tools (MapReduce, Pig, Hive, Impala, Spark, Presto, and cloud warehouses), carries its own schema, handles deeply nested structures using the Dremel encoding, and is the default file format in Spark.

This lesson covers *what* Parquet is and how to use it. The child lesson, *Parquet Internals*, dissects the byte layout (footer, row groups, column chunks, pages).

### 2. Core concepts and vocabulary

- **Columnar format** — values of one column are stored contiguously; contrasts with row-oriented formats (Avro, SequenceFile, CSV) where a record's fields are stored together.
- **Row group** (Parquet "block") — a horizontal slice of rows, buffered in memory while writing; default 128 MB.
- **Column chunk** — the data of one column within a row group.
- **Page** — the smallest unit of storage within a column chunk; the unit that is decompressed and decoded; default 1 MB.
- **Schema / message** — the root `message` contains fields, each with a repetition (`required`, `optional`, `repeated`), a type and a name.
- **Primitive types** — `boolean`, `int32`, `int64`, `int96`, `float`, `double`, `binary`, `fixed_len_byte_array`.
- **Logical type** — an annotation that tells readers how to interpret a primitive (e.g., a string is `binary` with `UTF8`; also dates, decimals, JSON/BSON, lists, maps).
- **Definition level / repetition level** — two small integers per value that encode nesting and nulls, letting nested columns be stored flat.
- **Projection** — reading only chosen columns.
- **Predicate pushdown** — filtering using stored metadata so non-matching blocks are skipped.
- **ORC** — Optimized Row Columnar, a similar format popular with Hive; less supported in modern cloud tools (e.g., Snowflake and BigQuery can import it but not export it).

### 3. How it works, step by step

1. **Define a schema.** For example:
   ```text
   message WeatherRecord {
     required int32 year;
     required int32 temperature;
     required binary stationId (UTF8);
   }
   ```
2. **Buffer rows** into a row group in memory (default 128 MB).
3. **Split by column:** each primitive field becomes its own column chunk; for nested data, definition and repetition levels are stored alongside values.
4. **Encode** values per page: plain, delta, run-length, dictionary, plus bit packing.
5. **Compress** each page (Snappy, Gzip, LZO, etc.).
6. **Write the footer** with schema, key-value metadata and metadata about every row group, then the footer length and the magic number `PAR1`.
7. **Read:** seek to the end, read the footer, choose the row groups and column chunks you need (projection, pushdown), and decode only those pages.

Why this helps:

| Effect | Mechanism |
|---|---|
| Smaller files | Similar adjacent values encode well (e.g., timestamps as first value + small deltas) |
| Faster queries | Skip unneeded columns (projection) and blocks (pushdown) |
| Nested reads | Each leaf field is its own column, so you can read map keys without values |
| Portability | Language-neutral spec (`parquet-format`) with Java and C++ implementations |

### 4. Worked example

Writing and querying Parquet with Spark:

```python
# Write: Parquet is Spark's default format; snappy is a speed-oriented codec
(df.write.format("parquet")
   .mode("overwrite")
   .option("compression", "snappy")
   .save("/tmp/data/parquet/flights"))

# Typical output files
# _SUCCESS
# part-00000-<...>-c000.snappy.parquet

# Read only two columns and filter - projection + predicate pushdown
spark.read.parquet("/tmp/data/parquet/flights") \
     .select("origin", "delay") \
     .where("delay > 60") \
     .show()
```

Or from Hive:

```sql
CREATE TABLE users_parquet STORED AS PARQUET AS SELECT * FROM users;
```

Inspect with the command-line tool:

```bash
parquet-tools dump output/part-m-00000.parquet
# INT64 offset
# *** row group 1 of 1, values 1 to 4 ***
# value 1: R:0 D:0 V:0
```

`V` is the value, `R` the repetition level and `D` the definition level; both are zero for a flat schema with no nulls or nesting. The grouping shows that all values of a column are stored together inside the row group.

### 5. Under the hood

**Dremel-style nested encoding.** Some columnar formats flatten nested structures so only top-level columns are columnar; reading a map's keys then drags in the values too. Parquet stores *every* primitive leaf as a separate column and records structure with two integers per value. You can think of this as a generalisation of a bit field that flags nulls in a flat record, with only non-null values written one after another. The result: any column, even a deeply nested one, is readable independently.

**Logical vs physical types.** There is no string primitive; strings are `binary` annotated `UTF8`. This separates serialised representation from semantics, and means a file written by one tool is interpreted consistently by another.

**Encodings.** Plain encoding writes values in full (an `int32` as 4 little-endian bytes). More compact ones are delta, run-length, dictionary (values replaced by integer indexes into an encoded dictionary) and bit packing. Parquet picks an encoding automatically by column type — Booleans use run-length + bit-packing, most types default to dictionary encoding, with a fall-back to plain when the dictionary grows beyond `parquet.dictionary.page.size`.

**Memory behaviour.** Because rows are buffered per row group, a writer needs memory for at least one full row group. Out-of-memory failures in MapReduce/Spark writers are fixed by shrinking `parquet.block.size` or giving tasks more memory.

**Row group vs HDFS block.** The row group should be no larger than the HDFS block so each is read from a single block (a single datanode); the common choice is both at 128 MB.

**Pluggable object model.** Parquet separates the file format from in-memory models: `ReadSupport`/`WriteSupport` convert between tool objects (Avro, Thrift, Protocol Buffers, Hive rows) and Parquet types. The bundled `Group` example model is for demos; production code uses a supported framework.

### 6. Performance and design implications

- **Block size trade-off:** bigger row groups give more efficient sequential scans (less per-chunk overhead) but need more memory to buffer.
- **Page size trade-off:** small pages speed single-row lookups (less to decode) but increase metadata overhead; large pages give better compression and scan efficiency. Default 1 MB.
- **Splittable:** block boundaries are stored in the footer, so no sync markers are needed and engines can process row groups in parallel.
- **Metadata at the end:** reading requires an initial seek to the end of the file (footer length) and a second backward seek — cheap on HDFS, but each small file adds this fixed cost, so avoid millions of tiny files.
- **Compression choice:** Snappy is popular for its speed; Gzip gives better ratios at higher CPU. The default for `parquet.compression` in MapReduce output is `UNCOMPRESSED`, so set it explicitly.
- **Use row formats when needed:** queries that need most columns of a row, or write-heavy record-at-a-time workloads, favour Avro/row formats. Parquet shines for analytical scans.
- **Interoperability:** proprietary warehouse formats are fast internally but force deserialise/reserialise to share data; Parquet files in a lake avoid that in a polyglot toolset.

### 7. Commands and settings you should know

| Command / setting | What it does | When to use |
|---|---|---|
| `parquet.block.size` (default 128 MB) | Row group size | Tune scan efficiency vs writer memory |
| `parquet.page.size` (default 1 MB) | Page size | Smaller for point lookups; larger for scans |
| `parquet.dictionary.page.size` | Max dictionary before falling back to plain | High-cardinality columns |
| `parquet.enable.dictionary` | Toggles dictionary encoding | Disable if it hurts or for very unique values |
| `parquet.compression` | `UNCOMPRESSED`, `SNAPPY`, `GZIP`, `LZO` | Always set deliberately |
| `df.write.format("parquet").option("compression", …)` | Spark write | Standard lake writes |
| `STORED AS PARQUET` | Hive table format | Converting Hive tables |
| `parquet-tools dump` / schema | Inspect file contents and schema | Debugging and validation |

### 8. Trade-offs and decisions

| Choose this when | Avoid / watch out when |
|---|---|
| Analytical scans touching a subset of columns | Reading whole rows one at a time or streaming single records |
| Wide tables, nested data, many consumers/tools | Frequent row-level updates (use a table format such as Delta/Hudi/Iceberg on top) |
| Data lake on object/HDFS storage | Tiny files or tiny writes — heavy footer overhead |
| Parquet over ORC for cloud ecosystem support | Hive-centric stack already standardised on ORC |
| Row formats (Avro) for ingestion, schema evolution, messaging | Heavy columnar analytics on those same files |

### 9. Common mistakes

- Writing many small Parquet files → slow listing, footer overhead, poor compression → compact/coalesce to target file sizes.
- Leaving compression as `UNCOMPRESSED` in MapReduce → large files and I/O → set `parquet.compression`.
- Selecting `*` → defeats projection → select only needed columns.
- Setting row groups larger than task memory → OOM → lower `parquet.block.size` or raise memory.
- Setting row group size larger than the HDFS block → reads span datanodes → align them.
- Using the example `Group` model in production → fragile → use Avro, Thrift or Protobuf support, or Spark/Hive.

### 10. 🎯 Interview kit

**30-second answer:** Parquet is a columnar, self-describing, splittable file format with per-column encodings and compression, nested-data support via definition/repetition levels, and metadata that enables projection and predicate pushdown. It's the default for lakes because it is compact, fast for analytics and readable by almost every engine.

**Structure a longer answer:** clarify row vs column → file layout (row groups, column chunks, pages, footer) → encodings/compression → pushdown → trade-offs vs Avro/ORC and the update limitation.

**Likely follow-ups:**
- *Why is Parquet faster for analytics?* → Skips unneeded columns, encodes similar values compactly, and prunes blocks via footer statistics.
- *How does it store nested data?* → Each leaf is a column; definition and repetition levels encode structure.
- *Parquet vs ORC vs Avro?* → Parquet/ORC are columnar (analytics); Avro is row-oriented with strong schema evolution (ingestion/streams). Parquet has broadest cloud support.
- *Why is the footer at the end?* → The writer knows block positions only after writing; it also makes files splittable without sync markers.

**Signals interviewers look for:** you can explain benefit with mechanics (projection, encoding, pushdown), name sizing parameters, and recognise when not to use it.

### 11. Check yourself

- [ ] Explain why a column of timestamps compresses well in a columnar layout.
- [ ] Define row group, column chunk, page and the default sizes.
- [ ] Write a Spark snippet that reads two columns with a filter and explain what is skipped.
- [ ] Describe how `parquet.block.size` and HDFS block size interact.
- [ ] Say when you'd pick Avro or a table format over plain Parquet files.
<!-- explainer:end -->

## In this section (study in order)

1. [Parquet File Internals](<Parquet_Internals/_index.md>) (L6 Internals)

## References

- *Hadoop: The Definitive Guide* — Tom White — Parquet › Chapter 13. Parquet
- *Fundamentals of Data Engineering* — Joe Reis, Matt Housley — The Future of Data Engineering › Parquet
- *Hadoop: The Definitive Guide* — Tom White — Parquet › Warning
- *Hadoop: The Definitive Guide* — Tom White — Parquet › Nested Encoding
- *Hadoop: The Definitive Guide* — Tom White — Biological Data Science Saving Lives with Software › Column-oriented access with Parquet
- *Learning Spark (2nd ed.)* — Jules S. Damji, Brooke Wenig, Tathagata Das, Denny Lee — Spark SQL and DataFrames Introduction to Built-in Data Sources › In Python
- *Hadoop: The Definitive Guide* — Tom White — Parquet › Data Model
- *Hadoop: The Definitive Guide* — Tom White — Hive › Binary storage formats: Sequence files, Avro datafiles, Parquet files, RCFiles, and ORCFiles
- *Hadoop: The Definitive Guide* — Tom White — Parquet › Note
- *Hadoop: The Definitive Guide* — Tom White — Parquet › Parquet Configuration

