---
id: "01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Schema_Evolution_in_Lakehouse_Tables"
level: 4
type: "concept"
topic: "01_Databases_and_SQL/SQL_Language/SQL_Fundamentals"
tools: ["PostgreSQL"]
---

# Schema Evolution in Lakehouse Tables

**L4 Advanced** · [↑ SQL Fundamentals](<_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** Lakehouse table formats (Delta Lake, Apache Iceberg, Hudi) keep the schema in a transaction log or metadata layer separate from the Parquet files, so adding columns is a cheap metadata change, while renames, drops and type changes are safe only when the format tracks columns by ID or mapping, not by name or position.

### 1. The big picture
Source systems change: a new column appears, a field is renamed, an integer becomes a bigint, a string becomes a struct. A pipeline must absorb that without corrupting history or breaking downstream readers. In a classic relational database you run `ALTER TABLE` and the engine updates catalog metadata (sometimes rewriting the table). In a data lake the data lives in immutable files (Parquet/ORC/Avro) scattered across object storage, and millions of files written under older schemas must remain readable.

Lakehouse table formats solve this by adding a **metadata layer**: a transaction log (Delta), or manifests and snapshots (Iceberg), that records the table schema *per commit* and tells readers how to reconcile the schema of each data file with the current table schema. This gives you **schema enforcement** (reject bad writes) and **schema evolution** (allow controlled changes), plus time travel to see old schema versions.

The key question for every change is: *will old files, new files, running readers and downstream consumers all still agree on what each column means?*

### 2. Core concepts and vocabulary
- **Schema enforcement (schema-on-write)** — a write whose columns/types do not match the table is rejected.
- **Schema evolution** — an allowed change to the table schema (add, widen, rename, drop, reorder).
- **Schema drift** — upstream changes the shape without telling you.
- **Additive change** — adding a nullable column; backward compatible.
- **Breaking change** — removing/renaming a column or narrowing/changing a type.
- **Column mapping / column IDs** — physical identity for a column independent of its name (Iceberg always; Delta via `columnMapping.mode = 'name'` or `'id'`).
- **Backward compatible** — new readers can read old data. **Forward compatible** — old readers can read new data.
- **Data contract** — an agreed, tested definition of a dataset's schema and semantics between producer and consumer.
- **Expand/contract (parallel change)** — add new alongside old, migrate readers, then remove old.

### 3. How it works, step by step
**Adding a nullable column (Delta or Iceberg).**
1. `ALTER TABLE t ADD COLUMNS (discount_pct DECIMAL(5,2))`.
2. The format writes a new metadata version (Delta: a JSON commit with `metaData` action; Iceberg: a new metadata file with a new schema ID).
3. No data file is rewritten.
4. Readers scanning an old file find the column missing and return `NULL` for it.
5. New writes include the column.

**Automatic evolution during a write.**
- Delta: `df.write.option("mergeSchema","true")` or `SET spark.databricks.delta.schema.autoMerge.enabled = true` for `MERGE`; new columns in the source are added to the target.
- Iceberg (Spark): `ALTER TABLE ... ADD COLUMN`; or write option `mergeSchema` / `spark.sql.iceberg.check-nullability` settings depending on engine.
- Without it, the write fails with an *incompatible schema* error: that is enforcement doing its job.

**Rename or drop.**
- With name-based Parquet column resolution (default in Delta without column mapping), renaming a column means old files still contain the old name; the new name finds nothing → `NULL`s → silent data loss. So renames need *column mapping mode* so the log maps logical name → physical column.
- Iceberg identifies columns by field ID stored in each file, so rename/drop/reorder are metadata-only and safe.

**Type changes.**
- Widening (`int → bigint`, `float → double`, `decimal(10,2) → decimal(12,2)`) can be metadata-only where the format supports *type widening* (Delta 3.x, Iceberg permits int→long, float→double, decimal precision increase).
- Anything else (string → int, int → string) requires rewriting the data or adding a new column.

### 4. Worked example
```sql
-- Delta table of orders
CREATE TABLE sales.orders (
  order_id BIGINT, customer_id BIGINT, amount DECIMAL(10,2), order_ts TIMESTAMP
) USING DELTA
TBLPROPERTIES ('delta.columnMapping.mode' = 'name',
               'delta.minReaderVersion'   = '2',
               'delta.minWriterVersion'   = '5');

-- 1) Additive: safe, metadata-only
ALTER TABLE sales.orders ADD COLUMNS (channel STRING AFTER amount);

-- 2) Rename: safe only because column mapping is on
ALTER TABLE sales.orders RENAME COLUMN customer_id TO cust_id;

-- 3) Type widening: amount DECIMAL(10,2) -> DECIMAL(14,2) (precision increase)
ALTER TABLE sales.orders ALTER COLUMN amount TYPE DECIMAL(14,2);

-- 4) Inspect history and schema over time
DESCRIBE HISTORY sales.orders;
SELECT * FROM sales.orders VERSION AS OF 3;          -- time travel to an older schema
```

Compare what a nightly audit sees across two snapshots (Delta):

```sql
SELECT 'added' AS change, n.* FROM sales.orders VERSION AS OF 12 n
LEFT ANTI JOIN sales.orders VERSION AS OF 11 o USING (order_id)
UNION ALL
SELECT 'removed', o.* FROM sales.orders VERSION AS OF 11 o
LEFT ANTI JOIN sales.orders VERSION AS OF 12 n USING (order_id);
```

For a contract check before loading from a source whose types might drift (portable SQL):

```sql
SELECT s.column_name, s.data_type AS source_type, t.data_type AS target_type
FROM   information_schema.columns s
JOIN   information_schema.columns t
       ON t.column_name = s.column_name
WHERE  s.table_name = 'src_orders' AND t.table_name = 'orders'
  AND  s.data_type <> t.data_type;      -- any row = fail the load before it corrupts a mart
```

### 5. Under the hood
- **Delta Lake.** The `_delta_log/` folder holds ordered JSON commits plus periodic Parquet checkpoints. A `metaData` action carries the schema as JSON; `add`/`remove` actions list files. Readers replay the log to get the schema and file list at a version. Parquet files carry their own schema; Delta's reader projects each file to the table schema (missing → `NULL`, widening → cast). Column mapping adds a `delta.columnMapping.physicalName` per field so renames don't touch files.
- **Apache Iceberg.** A table points to a metadata JSON that lists *all schema versions* with `schema-id`, and each field has a stable integer `id`. Parquet files store the field IDs. Manifests reference data files. Evolving a schema creates a new snapshot lineage entry, never touches data files, and guarantees "no zombie columns": dropping `x` and later adding a different `x` gets a new ID so old data is not resurrected.
- **Partition evolution** (Iceberg) is similar: partition specs are versioned, old files keep old layout.
- **Streaming.** A Structured Streaming job restarted after the source adds a column needs `mergeSchema` or it fails fast. For Kafka with a schema registry, compatibility modes (BACKWARD, FORWARD, FULL) gate which changes producers may publish.
- **Rename vs add-then-drop.** A rename is an identity-preserving change (same column, new name); add-then-drop creates a new column with no history, so downstream queries referencing the old name break and old rows are `NULL` in the new one.

### 6. Performance and design implications
- Additive changes are free; type changes that need rewrites are expensive on large tables (full table rewrite = proportional to bytes).
- Auto-evolution is convenient for bronze/raw layers (you want to capture everything) and dangerous for silver/gold (you want contracts); use enforcement downstream.
- Wide tables with hundreds of evolving columns slow metadata operations; flatten rarely-used attributes into a `MAP` or `VARIANT`/JSON column on purpose.
- Document and version the contract; announce breaking changes with a deprecation window.
- Test with a prod-shaped copy: confirm old files, new files and a time-travel read all return expected values.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `ALTER TABLE … ADD COLUMNS` | Add column (metadata only) | New source attribute |
| `mergeSchema = true` / `autoMerge.enabled` | Allow write/MERGE to add columns | Bronze ingestion |
| `delta.columnMapping.mode = 'name'` | Rename/drop without rewriting | Before any rename |
| `ALTER COLUMN … TYPE` (widening) | Increase numeric width | Overflow risk |
| `overwriteSchema = true` | Replace table schema on overwrite | Deliberate rebuild only |
| `DESCRIBE HISTORY`, `VERSION AS OF` | Audit schema/data over time | Debug, compare loads |
| Schema registry compatibility mode | Gate producer changes | Kafka / Avro / Protobuf |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| Auto-merge schema on raw/bronze ingestion | Curated layers consumed by BI — unexpected columns/types break reports |
| Column mapping/IDs for rename/drop | Old engines/readers that don't support the protocol version |
| Expand/contract for breaking changes | You can coordinate a single atomic cutover and rewrites are cheap |
| Rewrite the table for type changes | Table is petabyte-scale — add a new column and backfill gradually |
| Strict enforcement | Highly variable sources where you'd rather quarantine bad rows |

### 9. Common mistakes
- Renaming a column in a name-resolved table → old files read as `NULL` → enable column mapping first.
- Turning on auto-merge everywhere → upstream typos become permanent columns → restrict to bronze, review additions.
- Changing `int` to `string` in place → failed writes or silently cast data → add a new column, backfill, switch readers.
- Forgetting to bump reader/writer protocol → old clients fail → check `minReaderVersion` compatibility.
- Dropping a column that downstream views select → broken pipelines → check lineage first, deprecate before dropping.
- Letting a changed source type flow into a mart → wrong aggregates → add pre-load contract checks.

### 10. 🎯 Interview kit
**30-second answer:** Lakehouse formats store the schema in a versioned log/metadata layer, so adding a nullable column is metadata-only and old files read as `NULL`. Renames and drops are safe only with column IDs or column mapping; most type changes require a rewrite or a new column. I use auto-merge in raw layers, strict enforcement downstream, and expand/contract for breaking changes.

**Structure a longer answer:** why files + metadata → additive vs breaking changes → how the format resolves columns (name vs ID) → enforcement vs evolution by layer → rollout (contract checks, parallel change, time travel for audits).

**Likely follow-ups:**
- *Why can a rename lose data in Parquet-based tables?* → resolution by name.
- *How does Iceberg avoid that?* → stable field IDs.
- *How do you add NOT NULL to a huge table?* → add nullable, backfill in batches, then constrain (and on PostgreSQL 11+ a constant default is metadata-only).
- *How do you zero-downtime change a type?* → add column, dual-write, backfill, swap readers, drop.

**Signals interviewers look for:** compatibility vocabulary, knowing which changes are metadata-only, layered strategy, contract tests, rollback via time travel.

### 11. Check yourself
- [ ] Classify five schema changes as backward compatible, breaking or rewrite-required.
- [ ] Explain how Iceberg field IDs prevent resurrecting a dropped column.
- [ ] Write the steps of an expand/contract migration for `int → bigint`.
- [ ] State when `mergeSchema` is appropriate and when not.
- [ ] Show how to audit what a load changed with time travel.
<!-- explainer:end -->

## Implementations (tools)

- [PostgreSQL](<../../../TOOLS/PostgreSQL/_index.md>)

## Practice questions

- [Basic] [What does the ALTER TABLE statement do and give three things you can change with it?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Q-39759_What_does_the_ALTER_TABLE_statement_do_and_give_three_things.md>)
- [Intermediate] [In an ETL job, why would you truncate-and-reload a dimension table versus doing an incremental merge, and what are the trade-offs?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Q-39982_In_an_ETL_job_why_would_you_truncate-and-reload_a_dimension_.md>)
- [Intermediate] [How do you design a schema-versioned migration workflow across a team of many engineers so migrations are ordered, reversible, and safe to run against production with automated CI?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Q-40330_How_do_you_design_a_schema-versioned_migration_workflow_acro.md>)
- [Deep] [How would you design a schema migration workflow across a team so migrations are ordered, reversible, and safe to run in CI against a prod-shaped dataset?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Q-39963_How_would_you_design_a_schema_migration_workflow_across_a_te.md>)
- [Deep] [Your streaming SQL pipeline into Kafka topics produces a changelog stream. Explain the difference between a KTable and a KStream and how a GROUP BY over a stream becomes a compacted changelog table.](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Q-40143_Your_streaming_SQL_pipeline_into_Kafka_topics_produces_a_cha.md>)
- [Deep] [Explain the difference between a full-refresh and a CDC-based incremental strategy for a large dimension, and how you'd decide the crossover point where incremental complexity pays off.](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Q-40170_Explain_the_difference_between_a_full-refresh_and_a_CDC-base.md>)
- [Deep] [How do you safely add a NOT NULL column with a computed default to a large table on Postgres 11+ vs. older versions, and why did older versions rewrite the whole table?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Q-40312_How_do_you_safely_add_a_NOT_NULL_column_with_a_computed_defa.md>)
- [Expert] [Explain how you would design a schema migration that changes a column's type on a 500M-row table with zero downtime, including the online-backfill and dual-write phases.](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Q-40279_Explain_how_you_would_design_a_schema_migration_that_changes.md>)

## Related concepts

- [Schema Migration Risks](<../../Database_Internals/DBMS_Concepts/Schema_Migration_Risks.md>) *(DBMS Concepts)*
- [Migrations with Read Your Writes](<../../../04_Data_Architecture_and_Strategy/Platform_Strategy/Schema_Evolution/Migrations_with_Read_Your_Writes.md>) *(Schema Evolution)*
- [Online Schema Changes](<../SQL_Command_Types/Online_Schema_Changes.md>) *(SQL Command Types)*

## References

- *Building Medallion Architectures* — Piethein Strengholt — Construct the Bronze Layer › Updating Tables with MERGE Operations
- *Building Medallion Architectures* — Piethein Strengholt — Construct the Bronze Layer › Schema Management
- *Building Medallion Architectures* — Piethein Strengholt — Construct the Bronze Layer › Handling Schema Evolution
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 6: CTEs, MERGE, Pivoting, JSON, and XML › Real-World Use Cases
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 9: Execution Plans and Query Store › Factors That Prevent Plan Reuse
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 14: Unit Testing T-SQL with tSQLt › Mocking Tables with tSQLt.FakeTable
- *Delta Lake: The Definitive Guide* — Denny Lee et al. — Essential Delta Lake Operations › Create
- *Delta Lake: The Definitive Guide* — Denny Lee et al. — Essential Delta Lake Operations › Update
- *Delta Lake: The Definitive Guide* — Denny Lee et al. — Essential Delta Lake Operations › The replace method
- *Delta Lake: The Definitive Guide* — Denny Lee et al. — Diving into the Delta Lake Ecosystem › Builder options
- *Fundamentals of Data Engineering* — Joe Reis, Matt Housley — Modeling Streaming Data › Schema updates
- *Delta Lake: Up and Running* — Bennie Haelen, Dan Davis — Schema Handling › Schema Enforcement Example
- *Delta Lake: Up and Running* — Bennie Haelen, Dan Davis — Schema Handling › Schema Evolution
- *Delta Lake: Up and Running* — Bennie Haelen, Dan Davis — Schema Handling › Adding a Column
- *Delta Lake: Up and Running* — Bennie Haelen, Dan Davis — Schema Handling › Missing Data Column in Source DataFrame
- *Learning Spark (2nd ed.)* — Jules S. Damji, Brooke Wenig, Tathagata Das, Denny Lee — Building Reliable Data Lakes with Apache Spark › Evolving Schemas to Accommodate Changing Data

