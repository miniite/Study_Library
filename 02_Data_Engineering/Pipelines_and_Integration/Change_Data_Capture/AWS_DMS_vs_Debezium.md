---
id: "02_Data_Engineering/Pipelines_and_Integration/Change_Data_Capture/AWS_DMS_vs_Debezium"
level: 4
type: "concept"
topic: "02_Data_Engineering/Pipelines_and_Integration/Change_Data_Capture"
tools: ["AWS", "Apache_Kafka"]
---

# AWS DMS vs Debezium

**L4 Advanced** · [↑ Change Data Capture](<_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** AWS DMS is a managed replication service and Debezium is an open-source log reader you run on Kafka; both do log-based CDC, and the decision is mostly control and flexibility versus operational effort.

### 1. The big picture
Once you decide on log-based CDC, you still have to pick the machinery that reads the log, handles snapshots, tracks position, and delivers events. Two common choices on AWS are:
- **AWS Database Migration Service (DMS)** — a fully managed service originally built to migrate databases, which also supports ongoing replication (CDC) between many source and target engines.
- **Debezium** — an open-source set of connectors that run on Kafka Connect, turning database logs into Kafka topics of structured change events.

A third family is **SaaS ingestion tools** (Fivetran, Airbyte Cloud and similar) that bundle CDC connectors and loading into a warehouse. This lesson focuses on the DMS-versus-Debezium choice and the surrounding decisions: serialization format, landing format, and what CDC costs the source.

### 2. Core concepts and vocabulary
- **Replication instance (DMS)** — the managed compute that runs replication tasks. You size it; it reads the source and writes the target.
- **Full load + CDC task (DMS)** — a task that first copies existing rows, then keeps applying ongoing changes.
- **Kafka Connect** — the framework that runs source and sink connectors, handles offsets and scaling. Debezium connectors are Kafka Connect source connectors.
- **Change topic** — a Kafka topic holding events for one table (e.g. `server.schema.table`).
- **Schema registry** — a service storing Avro/Protobuf/JSON schemas and checking compatibility as they evolve.
- **Serialization format** — how events are encoded on the wire/disk: JSON (readable, large, no enforced schema), Avro or Protobuf (compact, schema-enforced).
- **Heterogeneous migration** — source and target are different engines (e.g. Oracle to PostgreSQL); DMS is designed for this.
- **DynamoDB Streams** — a native change feed of item-level changes for DynamoDB tables, retained for a limited time.
- **Managed streaming (Amazon MSK)** — hosted Kafka; it can carry CDC streams and host Kafka Connect (MSK Connect) with Debezium.

### 3. How it works, step by step
**AWS DMS CDC**
1. Create source and target endpoints (connection details, credentials).
2. Create a replication instance in your VPC with enough memory for in-flight transactions.
3. Create a task: choose "full load and CDC", define table mappings (what to include/exclude and rename), and optional transformation rules.
4. DMS performs the initial copy while buffering log changes, then applies the cached changes and continues streaming.
5. Targets can be another database, S3 (files for a lake), Kinesis, or Kafka/MSK. In S3, DMS writes Parquet or CSV with an `Op` column (I/U/D) and, optionally, a timestamp/commit column.

**Debezium CDC**
1. Run Kafka and Kafka Connect (self-managed, Kubernetes, or MSK + MSK Connect).
2. Enable and configure the database log (binlog row format, logical WAL, and so on) and create a user with replication privileges.
3. Deploy a Debezium connector configuration: connection, tables to capture, snapshot mode, and optional single message transforms.
4. The connector takes an initial snapshot (by default), then switches to streaming the log. Its offsets are stored in Kafka so it resumes after restart.
5. Events appear as Kafka messages with `before`, `after`, `op`, `source` and `ts_ms`. Sink connectors (S3, Snowflake, JDBC, Iceberg) or stream processors consume them.

### 4. Worked example
Scenario: an Aurora MySQL operational database must feed a data lake on S3 with row-level history.

**Option A: DMS to S3.** You configure a task with `full-load-and-cdc`, target S3, Parquet output, and `cdcPath` partitioning. Files land under paths such as `s3://lake/raw/orders/2026/10/12/...` with an `Op` column. A downstream job merges them into an Iceberg/Delta table. Operationally: one managed service plus S3; you watch task status and replication lag.

**Option B: Debezium on MSK Connect.** You enable `binlog_format=ROW` on Aurora (parameter group), deploy an MSK cluster, run the Debezium MySQL connector on MSK Connect, and an S3 sink connector writes Avro files. A schema registry guards compatibility. You get low-latency events usable by other consumers too (search indexer, fraud service), at the cost of running more components.

```json
{
  "name": "orders-cdc",
  "config": {
    "connector.class": "io.debezium.connector.mysql.MySqlConnector",
    "database.hostname": "aurora-writer.example",
    "database.server.id": "5701",
    "topic.prefix": "shop",
    "table.include.list": "shop.orders",
    "snapshot.mode": "initial",
    "key.converter": "io.confluent.connect.avro.AvroConverter",
    "value.converter": "io.confluent.connect.avro.AvroConverter"
  }
}
```

Reading the result: each Kafka message key is the row's primary key; the value has `before`, `after`, `op` (`c`,`u`,`d`,`r`), and `source.pos` or `source.lsn` giving log position.

### 5. Under the hood
- **Both read the same log.** DMS and Debezium decode MySQL binlog or PostgreSQL logical replication output. The difference is packaging and delivery, not the fundamental source of truth.
- **DMS internals.** The replication instance holds a transaction cache in memory and spills to disk; very large or long transactions can exhaust memory. It applies changes to relational targets in batches or one-by-one (transactional apply). It has data-type mapping rules and per-engine limits (LOB handling modes, unsupported types, limited DDL propagation). It is designed for migration cutover as much as steady-state replication.
- **Debezium internals.** The connector emulates a replica (MySQL) or uses a logical replication slot with a publication (PostgreSQL). Offsets (log position) are committed to Kafka Connect's offset topic. Schema history for MySQL is stored in a dedicated topic so past events can be interpreted after DDL. It emits tombstone messages (null value) after deletes to let Kafka log compaction remove the key.
- **Serialization.** JSON carries field names and types in every message and has no enforced contract. Avro and Protobuf are binary and compact; the writer's schema is registered once and a small schema ID travels with each message. Avro in particular supports explicit evolution rules (backward/forward compatibility), which matters because source tables change.
- **Landing format in the lake.** Avro is row-oriented and good for the raw change log (append-only writes, schema embedded). Parquet is columnar and best for analytical reads after you have compacted or merged. JSON is acceptable for debugging or low volume but costs space and scan time.
- **DynamoDB Streams.** A native change feed with per-shard iterators. You consume it via Lambda or the Kinesis adapter. Records are retained for 24 hours, so a consumer that falls behind loses data; shard iterators expire and must be managed.

### 6. Performance and design implications
- Log-reading is lighter on the source than triggers because it reads data the engine already writes, versus executing extra statements inside every transaction.
- DMS throughput depends on instance size, number of tasks, and target apply mode; large LOBs and wide rows slow it down.
- Debezium throughput depends on connector tasks (typically one per database for MySQL/PostgreSQL), Kafka partitioning, and downstream sink speed.
- Cost: DMS bills by instance hours and storage; Debezium on MSK bills for brokers and connect workers plus your engineering time. A pipeline that writes many tiny micro-batches into a warehouse can burn credits; batch more or increase flush intervals.
- Always benchmark with production-like write volume and large transactions, and test failure recovery.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| DMS task `MigrationType: full-load-and-cdc` | Initial copy then ongoing changes | Standard replication to a lake or DB |
| DMS `CdcStartPosition` / start time | Resume CDC from a given point | Restarting after changing target |
| DMS table mappings / transformation rules | Select, rename, filter columns/tables | Excluding PII or reshaping |
| Debezium `snapshot.mode` (`initial`, `never`, `when_needed`, `schema_only`) | Controls initial snapshot behavior | Adopting existing data or resuming after log loss |
| Debezium `table.include.list` | Limits captured tables | Reducing volume and risk |
| Kafka topic `cleanup.policy=compact` | Keeps latest value per key | State-like topics (current row state) |
| Schema registry compatibility mode | Rejects incompatible schema changes | Protecting consumers |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| DMS: heterogeneous sources/targets, AWS-centric shop, small team, migration plus replication | You need rich streaming semantics, custom transforms, many consumers of the same stream |
| Debezium + Kafka: many downstream consumers, replayable log, open-source requirement, rich event model | You have no one to operate Kafka and Connect; simple point-to-point replication is enough |
| Managed SaaS (Fivetran/Airbyte Cloud): fast time to value, many SaaS and DB sources | Cost grows with row volume; limited control over latency and semantics |
| Avro/Protobuf with a registry | Throwaway prototypes where JSON is fine |
| Parquet for curated lake tables | Raw change logs written one event at a time |

### 9. Common mistakes
- Treating DMS as a perfect long-term CDC platform without checking engine limits (unsupported types, LOBs, DDL) → silent data gaps → read the per-engine limitations and test each table type.
- Underestimating Debezium operations (Kafka, Connect, schema history, upgrades) → outages when the one person who knows it leaves → budget for on-call and monitoring, or use a managed offering.
- Using JSON everywhere because it is easy → schema drift breaks consumers and storage balloons → adopt a schema contract and a registry early.
- Enabling CDC without sizing source log retention → reader falls behind and loses its position → monitor lag and retained log and alert.
- Ignoring DynamoDB Stream retention → consumer outage over 24 hours loses changes → alarm on iterator age and size consumers.

### 10. 🎯 Interview kit
**30-second answer:** Both DMS and Debezium read the database log. DMS is a managed AWS service that is great for migrations and simple replication across engines. Debezium is an open-source Kafka-based log reader that gives you a durable, replayable event stream with a richer event model, but you operate Kafka and Connect. I choose by consumer count, replay needs, team skills and cost.

**Structure a longer answer:** clarify sources, targets, freshness and number of consumers → compare managed vs self-managed on control, effort, cost → choose serialization and landing format → talk about failure modes (log retention, schema change, large transactions) → close with a concrete recommendation.

**Likely follow-ups:**
- *Why Avro/Protobuf over JSON for CDC?* → compact, schema enforced, explicit compatibility rules for evolution, schema ID instead of repeating field names.
- *Why is log reading lighter than triggers?* → triggers run inside every transaction and write extra rows; log reading consumes data already written.
- *Build vs buy TCO?* → engineering and on-call time versus per-row/instance fees; include opportunity cost and risk.
- *DynamoDB Streams vs Kinesis Data Streams?* → Streams are built-in with 24-hour retention and ordered per item; Kinesis adapter or KDS integration offers longer retention and fan-out options.

**Signals interviewers look for:** you recognize both tools read logs, you articulate operational ownership, and you bring up schema evolution and retention.

### 11. Check yourself
- [ ] Explain what a replication instance does and what can exhaust its memory.
- [ ] List what a Debezium event contains and what a tombstone is for.
- [ ] Justify Avro for raw CDC and Parquet for curated tables.
- [ ] Give three reasons a team might pick DMS and three for Debezium.
- [ ] Describe what happens if a DynamoDB Streams consumer is down for two days.
<!-- explainer:end -->

## Implementations (tools)

- [AWS](<../../../TOOLS/AWS/_index.md>)
- [Apache Kafka](<../../../TOOLS/Apache_Kafka/_index.md>)

## Practice questions

- [Basic] [Why might a company prefer CDC over polling the source with periodic SELECT queries?](<../../../PRACTICE/02_Data_Engineering/Pipelines_and_Integration/Change_Data_Capture/Q-06012_Why_might_a_company_prefer_CDC_over_polling_the_source_with_.md>)
- [Basic] [When would batch CDC (say hourly) be good enough instead of real-time streaming?](<../../../PRACTICE/02_Data_Engineering/Pipelines_and_Integration/Change_Data_Capture/Q-06031_When_would_batch_CDC_say_hourly_be_good_enough_instead_of_re.md>)
- [Intermediate] [What is AWS DMS and how can it be used for CDC?](<../../../PRACTICE/02_Data_Engineering/Pipelines_and_Integration/Change_Data_Capture/Q-06053_What_is_AWS_DMS_and_how_can_it_be_used_for_CDC.md>)
- [Intermediate] [How is DynamoDB Streams used to capture item-level changes in AWS?](<../../../PRACTICE/02_Data_Engineering/Pipelines_and_Integration/Change_Data_Capture/Q-06083_How_is_DynamoDB_Streams_used_to_capture_item-level_changes_i.md>)
- [Intermediate] [What file formats (Parquet Avro JSON) would you use to store landed CDC data and why?](<../../../PRACTICE/02_Data_Engineering/Pipelines_and_Integration/Change_Data_Capture/Q-06101_What_file_formats_Parquet_Avro_JSON_would_you_use_to_store_l.md>)
- [Deep] [Why are Avro or Protobuf often used to serialize CDC messages instead of plain JSON?](<../../../PRACTICE/02_Data_Engineering/Pipelines_and_Integration/Change_Data_Capture/Q-06051_Why_are_Avro_or_Protobuf_often_used_to_serialize_CDC_message.md>)
- [Deep] [What is the trade-off between building your own CDC with Debezium versus buying a managed tool?](<../../../PRACTICE/02_Data_Engineering/Pipelines_and_Integration/Change_Data_Capture/Q-06055_What_is_the_trade-off_between_building_your_own_CDC_with_Deb.md>)
- [Deep] [Why is reading the transaction log generally lighter on the source than running trigger-based CDC?](<../../../PRACTICE/02_Data_Engineering/Pipelines_and_Integration/Change_Data_Capture/Q-06062_Why_is_reading_the_transaction_log_generally_lighter_on_the_.md>)

## Related concepts

- [CDC vs Full Reload](<../Data_Ingestion/CDC_vs_Full_Reload.md>) *(Data Ingestion)*
- [Data Warehouse Fundamentals](<../../Modeling_and_Warehousing/Data_Warehousing/Data_Warehouse_Fundamentals.md>) *(Data Warehousing)*
- [Schema Migration Risks](<../../../01_Databases_and_SQL/Database_Internals/DBMS_Concepts/Schema_Migration_Risks.md>) *(DBMS Concepts)*

## References

- *Fundamentals of Data Engineering* — Joe Reis, Matt Housley — Ingestion › CDC considerations
- *AWS Certified Data Engineer Associate Study Guide* — Sakti Mishra et al. — Overview of AWS Analytics and Auxiliary Services › Amazon Managed Streaming for Apache Kafka

