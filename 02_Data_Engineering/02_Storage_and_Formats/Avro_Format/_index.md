---
id: "02_Data_Engineering/02_Storage_and_Formats/Avro_Format"
level: 5
type: "curriculum"
curriculum: "data_engineering"
---

# Avro Data Files and Schema Resolution

**L5 Expert** · [↑ Storage Systems and File Formats](<../_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** Avro is a compact, schema-driven, row-oriented binary format whose files carry their own schema and whose readers resolve differences between the schema the data was written with and the schema the reader expects — which is what makes it the workhorse for ingestion and streaming with evolving data.

### 1. The big picture

Text formats (JSON, CSV) repeat field names in every record, are loosely typed and are slow to parse. Avro solves this by separating the *schema* (a JSON document describing the record) from the *data* (a dense binary encoding with no field names or tags). To decode a record you need the schema; with it, the data is small, typed and fast.

Avro's distinctive idea is **schema resolution**: the *writer's schema* (used to encode) and the *reader's schema* (what the consuming code expects) need not be identical, only compatible. This makes it ideal where producers and consumers evolve independently: Hadoop datasets, Kafka topics with a schema registry, database exports. It sits at L5 because understanding it means understanding how schemas travel with data and how resolution rules work.

### 2. Core concepts and vocabulary

- **Schema** — JSON definition of types; for records: a name, optional namespace and a list of fields with types and defaults.
- **Primitive types** — `null`, `boolean`, `int` (32-bit), `long` (64-bit), `float`, `double`, `bytes`, `string`.
- **Complex types** — `record` (named fields), `array` (items of one schema), `map` (string keys → values of one schema), `enum`, `fixed`, `union` (value matches one listed schema).
- **Union** — expresses optionality: `["null", "string"]`. A default of `null` is only legal if `null` is a branch of the union.
- **Writer's schema** — schema used to encode the data.
- **Reader's schema** — schema the application expects when decoding.
- **Schema resolution** — Avro library matches writer and reader schemas side by side to translate data.
- **Object container file (datafile)** — header with schema and metadata, then blocks of serialised records separated by a sync marker.
- **Sync marker** — 16-byte random marker unique to a file; allows resynchronisation after a seek, making files splittable.
- **Generic / Specific / Reflect mappings** — (Java) dynamic records, generated classes, or mapping existing classes by reflection.
- **Aliases** — alternative names in the reader schema allowing field renames.
- **Schema registry** — external store of schema versions, referenced by ID or version in each message.

### 3. How it works, step by step

**Encoding.** The encoder walks the schema in field order and writes values with no field names or tags: variable-length integers (zig-zag), length-prefixed strings and bytes, blocks for arrays and maps, an index for union branches. The bytes alone are meaningless without the schema.

**Decoding with resolution.**

1. The reader obtains the *writer's schema* (from the file header, a version number, or a negotiated connection).
2. It already has its own *reader's schema*.
3. The library matches fields **by name**, not position:
   - field in writer but not reader → skipped,
   - field in reader but not writer → filled with the reader's declared default (error if no default),
   - field order differences → irrelevant,
   - compatible type changes (e.g., `int` → `long`) → promoted.
4. The resulting record conforms to the reader's schema.

**How the reader learns the writer's schema** depends on context:

| Context | Mechanism |
|---|---|
| Large file with millions of records | Schema stored once in the container file header |
| Database with individually written records | Version number (or hash) at the start of each record, with a table of schema versions |
| Network connection / RPC | Schema negotiated at connection setup |
| Message streams | Schema ID in each message, resolved via a schema registry |

### 4. Worked example

A schema evolves from v1 to v2; old and new programs must keep working.

```json
// v1 (writer's schema for historical files)
{
  "type": "record", "name": "Order", "namespace": "shop",
  "fields": [
    {"name": "order_id", "type": "long"},
    {"name": "customer", "type": "string"},
    {"name": "amount",   "type": "double"}
  ]
}
```

```json
// v2 (new writer / new reader)
{
  "type": "record", "name": "Order", "namespace": "shop",
  "fields": [
    {"name": "order_id", "type": "long"},
    {"name": "customer", "type": "string"},
    {"name": "amount",   "type": "double"},
    {"name": "coupon",   "type": ["null", "string"], "default": null}
  ]
}
```

```text
Case 1  new reader (v2) reads old file (v1 writer)
        'coupon' missing in writer -> filled with default null        -> OK (backward compatible)
Case 2  old reader (v1) reads new data (v2 writer)
        'coupon' unknown to reader -> skipped                         -> OK (forward compatible)
Case 3  v3 adds field "region" of type string with NO default
        new reader on old data: no value, no default                  -> ERROR (breaks backward compat)
```

Datafile check from the command line (Avro tools):

```text
% java -jar avro-tools.jar getschema orders.avro        # prints embedded writer's schema
% java -jar avro-tools.jar tojson orders.avro | head    # decode records as JSON
```

### 5. Under the hood

**Why no tag numbers.** Protocol Buffers and Thrift identify each encoded field by a numeric tag, so evolution means assigning and never reusing tags. Avro identifies nothing in the data; position is dictated entirely by the schema, and name-based matching happens at resolution time. The consequence is that Avro handles **dynamically generated schemas** gracefully: dump a relational table by generating a record schema from its columns (column → field). When the table gains or loses a column, regenerate the schema and export again — the exporter needs no manual tag bookkeeping, and readers match fields by name. With tagged formats someone would have to maintain the column→tag mapping by hand and ensure tags are never reused.

**Container file layout.** Header: magic bytes, metadata map (including `avro.schema` and codec), 16-byte sync marker. Then repeated blocks: record count, byte size, (compressed) serialised records, sync marker. Because each block is delimited by the marker, a reader dropped at an arbitrary HDFS split boundary scans forward to the next marker, which makes the file *splittable* even with block-level compression. Writers produce files via a `DataFileWriter` bound to the schema; appending an object that does not conform to the schema throws an error.

**Row-oriented trade-off.** Because records are stored contiguously and written as streams, Avro files can be read up to the last sync point after a writer crash — unlike columnar formats, which must buffer a whole row group. That is why Avro suits streaming and ingestion, and columnar formats suit analytics.

**Rules recap.** Add or remove only fields that have defaults. Renaming a field via aliases is backward compatible but not forward compatible. Adding a branch to a union is backward but not forward compatible. Type changes are allowed only when Avro can convert the type (e.g., widening numeric types).

**Naming trap in tools.** In Flume, "Avro source/sink" only use Avro RPC to move events between agents; they do not write Avro files. To produce Avro datafiles, use the file sink with the Avro serialiser.

### 6. Performance and design implications

- **Compact**: no field names or tags in the data; typically much smaller than JSON.
- **Fast ingestion**: row-based and append-friendly; schema validation at write time prevents bad data landing.
- **Splittable and compressible**: block-level codecs (snappy, deflate) preserve parallelism.
- **Analytics cost**: reading one column still reads whole rows; convert to Parquet/ORC for heavy analytical use.
- **Schema registry round trip**: cache schemas locally; resolution needs both schemas parsed once per pair.
- **Compatibility policy**: choose backward, forward or full compatibility per topic or dataset and enforce it in CI before deploying schema changes.
- **Measure**: bytes per record vs JSON, decode throughput, number of schema versions in circulation.

### 7. Commands and settings you should know

| Command / setting | What it does | When to use |
|---|---|---|
| `avro-tools getschema / getmeta / tojson / fromjson` | Inspect or create Avro files | Debugging datafiles |
| `DataFileWriter.setCodec(...)` / `avro.output.codec` | Chooses block compression | Balancing size and speed |
| Default values on every added field | Maintains backward compatibility | Any schema change |
| Union `["null", T]` with `"default": null` | Optional field | Nullable columns |
| Aliases in reader schema | Maps old names to new | Field renames |
| Schema registry compatibility mode | Rejects incompatible schema versions | CI/CD for schemas |
| Sqoop `--as-avrodatafile` / Hive `STORED AS AVRO` | Writes/reads Avro in Hadoop tools | Hadoop ingestion |

### 8. Trade-offs and decisions

| Choose this when | Avoid / watch out when |
|---|---|
| Avro — streaming/ingestion, evolving schemas, cross-language | Analytical scans of a few columns on huge tables |
| Avro datafiles — you need a portable, splittable, self-describing file | You need columnar skipping or predicate pushdown (use Parquet/ORC) |
| Schema from database metadata (dynamic) | You need hand-curated, documented domain schemas |
| Protobuf/Thrift — code-generated types and RPC | Frequently regenerated schemas that would require manual tag management |
| JSON — human readability, quick prototypes | High volume, strict typing, cost-sensitive storage |

### 9. Common mistakes

- **Adding a field with no default** → new readers can't read old data → always provide a default (and make it a union with null if optional).
- **Default `null` without a null branch** → invalid schema → use `["null", "string"]` with `"default": null` (null first).
- **Assuming the reader's schema equals the file's schema** → silent surprises → inspect the embedded writer schema.
- **Renaming fields without aliases** → data loss or errors → use aliases and plan rollouts.
- **Storing the full schema with every message** → bloat → store an ID/version and use a registry.
- **Using Avro for heavy analytics** → full-row reads → convert to a columnar format downstream.
- **Mixing up Flume Avro sinks with Avro files** → no files produced → use the file sink with Avro serialiser.

### 10. 🎯 Interview kit

**30-second answer:** "Avro is a row-based binary format with a JSON schema. Data is encoded without field names, so you need the writer's schema to decode; the reader supplies its own schema and the library resolves differences by field name — ignoring extra fields and filling defaults for missing ones. Files embed the schema and use sync markers so they're splittable; in Kafka a schema registry holds the schemas."

**Structure a longer answer:** clarify (streaming vs analytics, schema change frequency) → encoding and resolution → evolution rules → trade-offs vs Parquet/Protobuf → scenario.

**Likely follow-ups:**
- *Backward vs forward compatibility in Avro?* → Backward: new reader, old writer; forward: old reader, new writer. Adding/removing fields with defaults keeps both.
- *How does a reader get the writer's schema?* → File header, per-record version, connection negotiation or registry ID.
- *Why Avro rather than Protobuf for DB dumps?* → No tag numbers; schemas can be generated automatically from table definitions.
- *Is Avro splittable?* → Yes, via sync markers between blocks.

**Signals interviewers look for:** precise compatibility vocabulary, awareness of registries, knowing where Avro fits relative to columnar formats.

### 11. Check yourself

- [ ] Explain writer's schema vs reader's schema with an example of each resolution case.
- [ ] Write a schema with an optional field and a default.
- [ ] State which schema changes are backward, forward, or both.
- [ ] Describe the layout of an Avro object container file.
- [ ] Explain why Avro is friendlier than Protobuf to generated schemas.
- [ ] Choose Avro vs Parquet for (a) a Kafka topic and (b) a BI fact table.
<!-- explainer:end -->

## References

- *Designing Data-Intensive Applications* — Martin Kleppmann — Encoding and Evolution › The writer's schema and the reader's schema
- *Designing Data-Intensive Applications* — Martin Kleppmann — Encoding and Evolution › But what is the writer's schema?
- *Hadoop: The Definitive Guide* — Tom White — Avro › Avro Datafiles
- *Hadoop: The Definitive Guide* — Tom White — Avro › Avro Data Types and Schemas
- *Hadoop: The Definitive Guide* — Tom White — Avro › Avro in Other Languages
- *Hadoop: The Definitive Guide* — Tom White — Flume › Warning
- *Hadoop: The Definitive Guide* — Tom White — Crunch › Note
- *Hadoop: The Definitive Guide* — Tom White — Biological Data Science Saving Lives with Software › Literate programming with the Avro interface description language (IDL)
- *Hadoop: The Definitive Guide* — Tom White — Avro › Interoperability
- *Hadoop: The Definitive Guide* — Tom White — Avro › The Specific API

