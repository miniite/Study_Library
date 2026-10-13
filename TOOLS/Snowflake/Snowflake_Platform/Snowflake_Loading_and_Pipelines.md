---
id: "TOOLS/Snowflake/Snowflake_Platform/Snowflake_Loading_and_Pipelines"
level: 4
type: "concept"
topic: "TOOLS/Snowflake/Snowflake_Platform"
tools: ["Snowflake", "AWS"]
---

# Snowflake Loading and Pipelines

**L4 Advanced** · [↑ Snowflake Platform](<_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** Snowflake loads files from stages with `COPY INTO` (batch) or Snowpipe (continuous), and tracks changes with streams, processed by tasks or dynamic tables.

### 1. The big picture
Snowflake favours **ELT**: land raw files cheaply, load them as-is, and transform with SQL inside the warehouse, because compute is elastic. The ingestion pattern is the same as for other warehouses: extract to files in cloud storage, load, transform. Snowflake adds native objects for each step: *stages* for file locations, *file formats* for parsing, *COPY INTO* and *Snowpipe* for loading, *streams* for change tracking, *tasks* for scheduling, and *dynamic tables* for declarative pipelines.

### 2. Core concepts and vocabulary
- **Stage** — a named location for files: *internal* (user `@~`, table `@%t`, or named) or *external* (S3/Blob/GCS). External access is best set up with a **storage integration** (an IAM trust, no embedded keys); IAM role or user keys are the weaker alternatives.
- **File format** — reusable object describing CSV/JSON/Parquet parsing: delimiter, header, null strings, date formats, compression.
- **COPY INTO** — bulk load from a stage to a table.
- **Snowpipe** — serverless continuous loader, triggered by cloud event notifications (auto-ingest) or REST calls.
- **Stream** — a change-tracking object on a table exposing inserted, updated and deleted rows via `METADATA$ACTION`, `METADATA$ISUPDATE`, `METADATA$ROW_ID`.
- **Task** — scheduled (cron or interval) or triggered SQL; can form a DAG with `AFTER`, and run only `WHEN SYSTEM$STREAM_HAS_DATA('s')`.
- **Dynamic table** — a table defined by a query with a `TARGET_LAG`; Snowflake keeps it refreshed.
- **Transient vs permanent vs temporary tables** — differ in Time Travel and Fail-safe, hence cost.
- **Stored procedure vs UDF** — procedures run procedural logic and side-effecting statements (DDL/DML, transactions); UDFs compute a value inside a query.

### 3. How it works, step by step
**Batch load**
1. Write files (aim 100–250 MB compressed each so loading parallelizes).
2. Create stage and file format.
3. `COPY INTO tbl FROM @stage FILE_FORMAT=(FORMAT_NAME=…) ON_ERROR=…`.
4. Snowflake records loaded file names in table load metadata (about 64 days), so re-running does not load the same file twice unless `FORCE=TRUE`.
5. Validate: `VALIDATION_MODE`, the `COPY_HISTORY` view, row counts and timestamp reconciliation.

**Continuous load (Snowpipe)**: file lands → event notification → pipe runs a `COPY` using Snowflake-managed compute → rows appear within roughly a minute. Per-file overhead makes it poor for few, huge files; use `COPY INTO` there.

**Change processing**: stream on the landing table → task runs on a schedule → `MERGE` from the stream into the target → the stream's offset advances only when the DML commits.

### 4. Worked example
```sql
CREATE FILE FORMAT pipe_csv TYPE=CSV FIELD_DELIMITER='|' SKIP_HEADER=1
  NULL_IF=('','NULL') TIMESTAMP_FORMAT='YYYY-MM-DD HH24:MI:SS';
CREATE STAGE raw.orders_stage URL='s3://bucket/orders/'
  STORAGE_INTEGRATION=s3_int FILE_FORMAT=pipe_csv;

COPY INTO raw.orders FROM @raw.orders_stage ON_ERROR='ABORT_STATEMENT';

CREATE STREAM raw.orders_stream ON TABLE raw.orders;

CREATE TASK etl.merge_orders
  WAREHOUSE=etl_wh SCHEDULE='5 MINUTE'
  WHEN SYSTEM$STREAM_HAS_DATA('raw.orders_stream')
AS
MERGE INTO core.orders t
USING (
  SELECT * FROM raw.orders_stream
  QUALIFY ROW_NUMBER() OVER (PARTITION BY order_id ORDER BY updated_at DESC) = 1
) s
ON t.order_id = s.order_id
WHEN MATCHED AND s.METADATA$ACTION = 'INSERT' THEN UPDATE SET t.status = s.status, t.updated_at = s.updated_at
WHEN NOT MATCHED AND s.METADATA$ACTION = 'INSERT' THEN INSERT (order_id,status,updated_at) VALUES (s.order_id,s.status,s.updated_at);

ALTER TASK etl.merge_orders RESUME;   -- tasks are created suspended
```
Because the stream offset only moves when the transaction commits, a failed run leaves the same changes in the stream for the next run, and the keyed `MERGE` with a deduplicated source applies them once. That is the idempotency; no separate bookkeeping table is required (note that an update shows as a DELETE + INSERT pair with `METADATA$ISUPDATE = TRUE`, so filter on `'INSERT'` as above and handle real deletes separately).

### 5. Under the hood
A stream stores no data: it keeps an *offset* in the table's change history and computes the delta on read. That is why it depends on Time Travel retention: if the offset falls behind the retention (extended by `MAX_DATA_EXTENSION_TIME_IN_DAYS`), the stream goes **stale** and must be recreated, losing the unconsumed delta, so you re-backfill from the source. A stream is *consumed* only by a DML statement that commits; a plain `SELECT` doesn't move it. Standard streams track all DML; append-only streams track inserts only and are cheaper. Dynamic tables use incremental refresh where the query shape allows it, otherwise full refresh.

### 6. Performance and design implications
- File size and count drive load parallelism; thousands of tiny files multiply overhead.
- `ON_ERROR` is a data-quality policy: `ABORT_STATEMENT` for strict pipelines, `SKIP_FILE` / `CONTINUE` with error capture otherwise.
- A task that "succeeds" with zero rows is a silent failure: assert counts inside the task, use `WHEN SYSTEM$STREAM_HAS_DATA` deliberately, and alert on freshness (`MAX(loaded_at)`), not only on task failure.
- Dynamic tables remove stream/task plumbing for transformations, but you give up fine control over ordering, side effects, custom error handling and some SQL constructs; avoid them for sub-minute latency or procedural logic.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `PUT` / `GET` | upload to / download from internal stage (SnowSQL) | local files |
| `COPY INTO <table>` | batch load | bulk, scheduled loads |
| `COPY INTO <location>` | unload to files | exports |
| `CREATE PIPE … AUTO_INGEST=TRUE` | continuous load | event-driven files |
| `CREATE STREAM` | change tracking | CDC into marts |
| `CREATE TASK` / `ALTER TASK RESUME` | scheduling | orchestration inside Snowflake |
| `CREATE DYNAMIC TABLE … TARGET_LAG` | declarative pipeline | simple incremental transforms |
| `TRY_TO_DATE`, `TRY_TO_TIMESTAMP` | parse without erroring | dirty strings |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| `COPY INTO` for scheduled bulk files | need seconds-level arrival |
| Snowpipe for steady small files | few giant files |
| Streams + tasks for custom CDC logic | a dynamic table would do |
| Dynamic tables for declarative, simple lineage | complex branching, side effects |
| External stage with storage integration | embedding access keys |

### 9. Common mistakes
- Loading with `FORCE=TRUE` casually → duplicate rows → rely on load metadata, dedupe in the `MERGE`.
- Forgetting `RESUME` on tasks → nothing runs → resume leaf tasks before the root.
- Storing numbers in `VARCHAR` → slow, wrong ordering → use numeric types.
- Using `COUNT(column)` expecting row counts → NULLs excluded; `COUNT(*)` counts rows, `COUNT(DISTINCT col)` unique non-nulls.
- Mixing up `TRUNCATE` (removes all rows, DDL-like, keeps the table), `DELETE` (rows, recoverable via Time Travel) and `DROP` (object, recoverable with `UNDROP` within retention).
- A dbt incremental model doing full refreshes → `is_incremental()` false or no unique key → run with `--full-refresh` only on purpose, inspect compiled SQL and the merge key.

### 10. 🎯 Interview kit
**30-second answer:** Files go to a stage via a storage integration, `COPY INTO` bulk-loads them, Snowpipe does continuous loads, streams capture changes and tasks (or dynamic tables) apply them with an idempotent `MERGE`.

**Structure a longer answer:** clarify (latency, volume, file arrival) → load mechanism → CDC and idempotency → monitoring and failure handling.

**Likely follow-ups:**
- *How secure ingestion from S3?* → storage integration with an IAM role, no keys in SQL.
- *How rerun a failed task safely?* → stream offset advances only on commit; keyed `MERGE`.
- *Stream is STALE?* → retention exceeded; recreate and backfill; prevent by consuming regularly or extending retention.
- *Streams+tasks vs dynamic tables?* → control vs simplicity.

**Signals interviewers look for:** idempotency, observability beyond job success, correct file-size reasoning.

### 11. Check yourself
- [ ] List the objects needed for S3 → table with auto-ingest.
- [ ] Explain exactly when a stream's offset moves.
- [ ] Write a deduplicating `MERGE` from a stream.
- [ ] Say when you would not use a dynamic table.
<!-- explainer:end -->

## Implementations (tools)

- [Snowflake](<../_index.md>)
- [AWS](<../../AWS/_index.md>)

## Practice questions

- [Basic] [What is a file format object in Snowflake and why is it useful when loading data?](<../../../PRACTICE/TOOLS/Snowflake/Snowflake_Platform/Q-39463_What_is_a_file_format_object_in_Snowflake_and_why_is_it_usef.md>)
- [Basic] [What is the difference between COUNT(*), COUNT(column), and COUNT(DISTINCT column) in a Snowflake query?](<../../../PRACTICE/TOOLS/Snowflake/Snowflake_Platform/Q-39523_What_is_the_difference_between_COUNT_COUNT_column_and_COUNT_.md>)
- [Intermediate] [What is Change Data Capture (CDC) and how do Snowflake Streams support it?](<../../../PRACTICE/TOOLS/Snowflake/Snowflake_Platform/Q-39467_What_is_Change_Data_Capture_CDC_and_how_do_Snowflake_Streams.md>)
- [Intermediate] [How do transactions work in Snowflake and what does COMMIT versus ROLLBACK do?](<../../../PRACTICE/TOOLS/Snowflake/Snowflake_Platform/Q-39500_How_do_transactions_work_in_Snowflake_and_what_does_COMMIT_v.md>)
- [Intermediate] [What is a stored procedure in Snowflake and how does it differ from a UDF?](<../../../PRACTICE/TOOLS/Snowflake/Snowflake_Platform/Q-39521_What_is_a_stored_procedure_in_Snowflake_and_how_does_it_diff.md>)
- [Intermediate] [A dbt incremental model on Snowflake keeps doing full refreshes; how do you debug the is_incremental logic and merge behavior?](<../../../PRACTICE/TOOLS/Snowflake/Snowflake_Platform/Q-39578_A_dbt_incremental_model_on_Snowflake_keeps_doing_full_refres.md>)
- [Deep] [Explain how you'd build an idempotent MERGE-based upsert from a stream so that re-running a failed task never double-applies rows.](<../../../PRACTICE/TOOLS/Snowflake/Snowflake_Platform/Q-39552_Explain_how_you_d_build_an_idempotent_MERGE-based_upsert_fro.md>)
- [Deep] [How would you use dynamic tables to replace a hand-built Streams-and-Tasks pipeline, and when is that a bad idea?](<../../../PRACTICE/TOOLS/Snowflake/Snowflake_Platform/Q-39592_How_would_you_use_dynamic_tables_to_replace_a_hand-built_Str.md>)

## Related concepts

- [Star Schemas and Dimension Keys](<../../../02_Data_Engineering/Modeling_and_Warehousing/Dimensional_Modeling/Star_Schemas_and_Dimension_Keys.md>) *(Dimensional Modeling)*
- [Databases and Schemas](<../../../01_Databases_and_SQL/Database_Internals/DBMS_Concepts/Databases_and_Schemas.md>) *(DBMS Concepts)*
- [Large Values and TOAST](<../../../01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Large_Values_and_TOAST.md>) *(SQL Fundamentals)*
- [File and Table Formats for Features](<../../../02_Data_Engineering/Pipelines_and_Integration/Pipeline_Orchestration/File_and_Table_Formats_for_Features.md>) *(Pipeline Orchestration)*
- [Micro-partitions vs Manual Partitions](<../../../02_Data_Engineering/Storage_and_Lakehouse/Partitioning_and_Bucketing/Micro-partitions_vs_Manual_Partitions.md>) *(Partitioning and Bucketing)*
- [Table Format Internals](<../../../02_Data_Engineering/Storage_and_Lakehouse/Table_Formats/Table_Format_Internals.md>) *(Table Formats)*

## References

- *Data Quality Fundamentals* — Barr Moses, Lior Gavish, Molly Vorwerck — Assembling the Building Blocks of a Reliable Data System › Example: Pulling data quality metrics from Snowflake
- *Data Engineering Best Practices* — Luiz Fernando F. Dos Santos — ETL and ELT › Snowflake Snowpipe
- *Data Pipelines Pocket Reference* — James Densmore — Data Ingestion Loading Data › Loading Data into a Snowflake Data Warehouse
- *Data Pipelines Pocket Reference* — James Densmore — Measuring and Monitoring Pipeline Performance › F
- *Data Engineering Fundamentals* — Zhaolong Liu — CHAPTER 8Monitoring and Performance › CHAPTER 8Monitoring and Performance(#contents.xhtmlch08a)
- *Fundamentals of Data Engineering* — Joe Reis, Matt Housley — Storage › Zero copy cloning

