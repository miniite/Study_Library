---
id: "02_Data_Engineering/02_Storage_and_Formats/Encoding_and_Schema_Evolution"
level: 4
type: "curriculum"
curriculum: "data_engineering"
---

# Encoding and Schema Evolution

**L4 Advanced** · [↑ Storage Systems and File Formats](<../_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** Data outlives code, so how you encode it and how you let schemas change — keeping old readers and new writers (and the reverse) working — determines whether deployments and pipelines survive rolling upgrades.

### 1. The big picture

Applications change constantly, but the data they wrote last year is still in the database, the lake or a Kafka topic, and some processes still run old code. In any rolling deployment, old and new versions of code and data coexist. That produces two compatibility requirements:

- **Backward compatibility** — *new* code can read data written by *old* code.
- **Forward compatibility** — *old* code can read data written by *new* code.

Backward compatibility is usually easier (you know the old format); forward compatibility demands that old code ignore what it does not understand. Encoding formats (JSON, XML, CSV, Protocol Buffers, Thrift, Avro, Parquet) differ in how they support this. This lesson explains how each family handles evolution, why schemas are valuable even in a "flexible" world, and how compatibility applies across databases, services (RPC/REST) and message streams.

### 2. Core concepts and vocabulary

- **Encoding / serialisation** — converting in-memory structures to bytes (and decoding back).
- **Schema evolution** — changing a schema over time while keeping data readable.
- **Backward / forward compatibility** — see above.
- **Schema-on-write vs schema-on-read** — validate structure when storing vs interpret when reading ("schemaless" really means implicit schema on read).
- **Field tag** — numeric identifier of a field in Protocol Buffers/Thrift encodings.
- **Required / optional / repeated** — field cardinality markers (Protobuf/Thrift).
- **Default value** — value used when data lacks the field.
- **Union type** — Avro's way of expressing alternatives, including nullability.
- **Self-describing file** — embeds its schema (Avro container file, Parquet footer).
- **Schema registry** — central versioned schema store enabling pre-deployment compatibility checks.
- **Dead-letter queue** — place for records that fail to decode or validate.
- **Dataflow modes** — through databases, through services (REST/RPC), through asynchronous messages.

### 3. How it works, step by step

Rules by format family:

| Format | Field identity | Add a field | Remove a field | Rename a field | Change a type |
|---|---|---|---|---|---|
| **JSON/XML (no schema)** | Name in every record | Readers must tolerate unknown keys | Readers must tolerate missing keys | Breaking unless both names kept | Risky; numbers and big integers are ambiguous |
| **Protocol Buffers / Thrift** | Numeric tag | Use a new tag; must be optional or have default | Only optional fields; never reuse the tag | Safe (names aren't encoded) | Possible but may truncate (e.g., 32→64-bit int) |
| **Avro** | Position per schema + name matching at resolution | Needs a default | Needs a default | Via aliases (backward only) | If Avro can convert |
| **Parquet/ORC (columnar)** | Column name/ID in file schema | Add column; old files read as null | Drop column from reader schema | Needs mapping by position or ID | Limited promotions |

Why compatibility becomes a process:

1. **Define the schema** (IDL file, Avro schema, table DDL).
2. **Register it** in a repository or registry.
3. **Check compatibility** of each proposed change against the chosen policy (backward, forward, full) in CI.
4. **Deploy readers first** (for backward-compatible changes) or **writers first** (for forward-compatible), depending on which side moves first.
5. **Monitor** decoding failures and unknown-field rates.

### 4. Worked example

Evolving a Protocol Buffers message and an Avro record side by side.

```text
// Protobuf v1
message User { string name = 1; int32 age = 2; }

// v2 — safe changes
message User {
  string name = 1;
  int32  age  = 2;
  string email = 3;          // new tag, optional: old readers skip tag 3 (forward OK);
                             // new readers see missing email in old data (backward OK)
  repeated string phone = 4; // can later evolve from optional single to repeated
}

// v3 — UNSAFE changes
//   reuse tag 2 for "string country"  -> old data now decoded as wrong type
//   make email required               -> new readers choke on old data
//   age int32 -> int64                -> old readers truncate large values
```

```json
// Avro: add optional field with default and rename via alias
{"name": "contact_email", "type": ["null", "string"], "default": null,
 "aliases": ["email"]}
```

What to look at: in Protobuf the *tag number* is the contract (name is free to change); in Avro the contract is name + default + resolution rules. The same change ("add an optional email") is expressed differently but obeys the same principle: **new fields must be optional or defaulted; removed fields must have been optional; identifiers are never reused.**

### 5. Under the hood

**JSON, XML and CSV.** Textual formats are human-readable and language-independent but ambiguous: XML and CSV can't distinguish numbers from digit strings without a schema; JSON doesn't distinguish integers from floats and loses precision above 2^53 in many parsers; none support binary strings natively (Base64 workaround adds ~33% size); optional schemas (XML Schema, JSON Schema) exist but are rarely used consistently. CSV has no schema at all: adding a column or changing its meaning is a silent break, and escaping rules are your problem. They remain popular because other organisations produce them and because they are good enough for many exchanges.

**Binary schema-based formats.** Thrift and Protocol Buffers encode `(tag, type, value)` triples; unknown tags are skipped using the type annotation to know how many bytes to jump — that is how forward compatibility works. Avro omits tags entirely and relies on schema resolution. Thrift has a dedicated list type; Protobuf uses `repeated` (which allows optional→repeated evolution, with old readers seeing only the last element).

**Merits of schemas.** Binary schema formats can be far more compact than "binary JSON" because field names are omitted; the schema is always-current documentation because decoding requires it; a registry of versions enables compatibility checks before deployment; and statically typed languages benefit from generated code. The result approximates the flexibility of schemaless document stores while providing stronger guarantees and tooling. Schema languages here are intentionally simpler than XML Schema or JSON Schema (no regex or range validation), which helped wide language support. ASN.1, an older schema language with tag numbers (still used for X.509 certificates), shows the idea is old but is complex and poorly documented for new work.

**Dataflow modes.**
- *Through databases*: the writer and reader may be the same application at different versions; old rows exist indefinitely ("data outlives code"). Adding a column with null default is cheap; rewriting data is expensive, so preserve unknown fields when read-modify-write cycles pass through old code.
- *Through services (REST/RPC)*: servers are typically upgraded before clients, so requests need backward compatibility and responses forward compatibility. Providers often cannot force clients to upgrade, so compatibility must be maintained for a long time or versions run side by side (URL version or `Accept` header). Thrift, gRPC and Avro RPC inherit their encoding's rules; REST/JSON typically treats adding optional parameters and response fields as compatible.
- *Through messages*: producers and consumers deploy independently via a broker; schema registries and dead-letter queues are the standard safety nets.

**Dynamic schema and code generation.** Code generation helps statically typed languages (type checks, IDE support) but is pointless in dynamic languages and awkward for generated schemas; formats with self-describing files let you open and analyse data with no generated classes.

### 6. Performance and design implications

- **Size and speed**: tagged/binary encodings reduce size and parse time vs JSON; the savings matter at streaming volumes.
- **Evolution cost**: rewriting terabytes to apply a schema change is expensive, so prefer additive changes that old data satisfies by default.
- **Operational safety**: run compatibility checks in CI; reject changes automatically.
- **Unknown field handling**: keep unknown fields when forwarding data; dropping them silently loses information.
- **Contracts with upstream**: for third-party sources you cannot control, assume change and use a flexible landing schema with a validated, versioned layer behind it.
- **Measure**: decode errors per minute, DLQ volume, count of schema versions live in a topic or table, time to roll out schema changes.

### 7. Commands and settings you should know

| Command / setting | What it does | When to use |
|---|---|---|
| Schema registry compatibility level (`BACKWARD`, `FORWARD`, `FULL`, `*_TRANSITIVE`) | Rejects incompatible schema updates | Kafka topics, shared datasets |
| `protoc --check` / `buf breaking` | Detects breaking Protobuf changes | CI for IDL files |
| Parquet `mergeSchema` (Spark) | Reconciles column differences across files | Reading evolved Parquet directories |
| `ALTER TABLE … ADD COLUMN` (nullable/default) | Adds a column without rewrite (engine-dependent) | Warehouse and lakehouse tables |
| JSON schema validation in ingestion | Catches malformed payloads early | Semi-structured landing |
| Dead-letter topic | Isolates undecodable records | Any streaming consumer |
| API versioning (URL/header) | Runs multiple service versions | Breaking REST/RPC changes |

### 8. Trade-offs and decisions

| Choose this when | Avoid / watch out when |
|---|---|
| Schema-on-write with registry — many consumers, strict quality needs | Rapid prototyping with unknown shape |
| Schema-on-read / JSON — source structure changes outside your control | Data quality guarantees and cost efficiency matter |
| Protobuf/gRPC — internal services with strong typing | Public APIs needing human-readable debugging |
| Avro — data pipelines and dynamic schemas | Need generated, domain-rich types with stable tags |
| Full compatibility policy — long-lived data, many producers/consumers | Short-lived data where only one direction matters |
| Side-by-side API versions — external clients you can't upgrade | Internal systems that can be deployed in lockstep |

### 9. Common mistakes

- **Reusing or renumbering a Protobuf tag** → old data misread → retire tags forever (reserve them).
- **Adding a required field after launch** → old data fails validation → make it optional/defaulted.
- **Assuming all consumers upgrade together** → outages during rolling deploys → design for coexistence of versions.
- **Relying on JSON number types** → precision loss for large IDs → use strings or explicit types for 64-bit identifiers.
- **Dropping unknown fields in a pass-through service** → silent data loss → round-trip unknowns.
- **Changing a type without checking widening/narrowing** → truncation → check conversion rules; add a new field instead.
- **No schema history** → can't diagnose old data → keep a versioned schema store.

### 10. 🎯 Interview kit

**30-second answer:** "Data outlives code, so old and new versions coexist. Backward compatibility means new code reads old data; forward means old code reads new data. With Protobuf the field tag is the contract and you add optional fields with new tags; with Avro you add fields with defaults and resolve reader and writer schemas by name. A schema registry with compatibility checks in CI keeps rolling upgrades safe."

**Structure a longer answer:** clarify dataflow (DB, service, stream) → compatibility direction needed → format-specific rules → process (registry, CI, deploy order) → scenario.

**Likely follow-ups:**
- *Which side deploys first?* → For services, servers first → requests backward-compatible, responses forward-compatible.
- *Why can't you rename a Protobuf tag but can rename the field?* → Wire format uses tags, not names.
- *What does "schemaless" mean?* → Schema-on-read; the reader assumes structure.
- *How do you evolve a Parquet table?* → Add nullable columns; reconcile with schema merge; avoid in-place type changes.

**Signals interviewers look for:** correct directional vocabulary, practical process, awareness that most production incidents come from "one more field".

### 11. Check yourself

- [ ] Define backward and forward compatibility and give one example of each.
- [ ] Explain why Protobuf tags must never be reused.
- [ ] State the rule for adding a field in Avro and why it holds.
- [ ] Describe compatibility requirements for REST requests vs responses during a rollout.
- [ ] List three problems with CSV or JSON as an evolving interface.
- [ ] Propose a CI check pipeline for schema changes.
<!-- explainer:end -->

## References

- *Designing Data-Intensive Applications* — Martin Kleppmann — Encoding and Evolution › Field tags and schema evolution
- *Designing Data-Intensive Applications* — Martin Kleppmann — Encoding and Evolution › Dynamically generated schemas
- *Designing Data-Intensive Applications* — Martin Kleppmann — Encoding and Evolution › Schema evolution rules
- *Designing Data-Intensive Applications* — Martin Kleppmann — Encoding and Evolution › The Merits of Schemas
- *Designing Data-Intensive Applications* — Martin Kleppmann — Encoding and Evolution › Datatypes and schema evolution
- *Designing Data-Intensive Applications* — Martin Kleppmann — D
- *Designing Data-Intensive Applications* — Martin Kleppmann — Encoding and Evolution › Code generation and dynamically typed languages
- *Hadoop: The Definitive Guide* — Tom White — Parquet › Avro, Protocol Buffers, and Thrift
- *Fundamentals of Data Engineering* — Joe Reis, Matt Housley — The Future of Data Engineering › XML
- *Designing Data-Intensive Applications* — Martin Kleppmann — Data Models and Query Languages › Schema flexibility in the document model

