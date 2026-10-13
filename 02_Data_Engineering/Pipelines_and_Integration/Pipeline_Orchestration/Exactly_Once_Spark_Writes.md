---
id: "02_Data_Engineering/Pipelines_and_Integration/Pipeline_Orchestration/Exactly_Once_Spark_Writes"
level: 4
type: "concept"
topic: "02_Data_Engineering/Pipelines_and_Integration/Pipeline_Orchestration"
tools: ["Apache_Spark", "Apache_Airflow", "Apache_Kafka", "Databricks"]
---

# Exactly Once Spark Writes

**L4 Advanced** · [↑ Pipeline Orchestration](<_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** "Exactly-once" in Spark is not magic delivery — it is replayable sources, checkpointed progress, and an idempotent or transactional sink combined, so that retries and restarts produce the same final table.

### 1. The big picture
Distributed jobs fail midway: an executor dies, a task is retried, a stream restarts. Each retry may re-read input and re-write output. If the write is a naive append, the same rows land twice (or half a batch is visible). Exactly-once *processing* means the final result is as if every input record had affected the output exactly once, despite failures.

No system can guarantee that a message is physically delivered once over an unreliable network. What you build instead is **at-least-once delivery + idempotent effect**: the system may process a record twice, but applying it twice changes nothing. Spark gives you building blocks: deterministic recomputation, offset/progress tracking in a checkpoint, and — through table formats like Delta — atomic commits that make a write all-or-nothing.

### 2. Core concepts and vocabulary
- **At-most-once** — never duplicates, may lose data.
- **At-least-once** — never loses, may duplicate.
- **Exactly-once (effectively-once)** — each input affects output once; achieved by at-least-once plus idempotency/transactions.
- **Idempotent write** — repeating it yields the same state (overwrite a partition, upsert by key).
- **Transactional (atomic) commit** — a write becomes visible all at once or not at all (Delta transaction log, Iceberg snapshots).
- **Checkpoint** — Structured Streaming's saved offsets and state, enabling restart without gaps or repeats.
- **Write-ahead log / offset log** — records which offsets a micro-batch will process *before* it runs, so a retry replays the same range.
- **Replayable source** — a source that can re-deliver the same data (Kafka offsets, files).
- **Deduplication** — removing repeated events by a unique id.
- **`foreachBatch`** — hook to run custom (e.g. `MERGE`) logic per micro-batch with a batch id.

### 3. How it works, step by step
**Structured Streaming micro-batch, with an idempotent sink**
1. The engine determines the offset range for batch *N* and writes it to the offset log in the checkpoint.
2. It reads exactly that range from the source and runs the transformations deterministically.
3. It writes output to the sink, tagged with the batch id *N*.
4. On success it records batch *N* as committed in the commit log.
5. If it crashes after step 3 but before step 4, on restart it re-plans the *same* range for batch *N* and re-runs it.
6. The sink must therefore recognize "batch *N* was already applied" (transaction log with batch id, `MERGE` by key, partition overwrite), so the second write has no extra effect.

**Batch job**
1. Write results to a staging location or a new transaction.
2. Commit atomically (rename/commit protocol, table commit).
3. Make reruns overwrite the same logical partition, not append.

### 4. Worked example
Deduplicate and upsert Kafka events into a Delta table safely:

```python
from delta.tables import DeltaTable

def upsert(batch_df, batch_id):
    # dedupe inside the micro-batch on business key (latest wins)
    deduped = (batch_df
        .withColumn("rn", F.row_number().over(
            Window.partitionBy("event_id").orderBy(F.col("event_ts").desc())))
        .filter("rn = 1").drop("rn"))

    target = DeltaTable.forName(spark, "silver.events")
    (target.alias("t")
        .merge(deduped.alias("s"), "t.event_id = s.event_id")
        .whenMatchedUpdateAll()
        .whenNotMatchedInsertAll()
        .execute())

(stream.writeStream
   .foreachBatch(upsert)
   .option("checkpointLocation", "s3://chk/silver_events")
   .start())
```
What happens on a replay: the same micro-batch is re-run, `MERGE` matches existing `event_id`s and updates them to the same values — table state is unchanged. Without the merge (plain append) the replay would insert every row again. Reading concurrently: Delta readers see a consistent snapshot, never a half-written batch, because the commit is a single atomic log entry.

### 5. Under the hood   *(optional at L4)*
- **Why replay is deterministic:** the offset log fixes the input range before processing, and transformations are expected to be deterministic (no `rand()` or "now" that changes results between attempts).
- **Delta/Iceberg commits:** data files are written first (invisible), then one atomic metadata commit makes them visible; concurrent writers use optimistic concurrency and conflict detection. Streaming sinks also store `(queryId, batchId)` so a repeated batch id is skipped.
- **Kafka as sink:** the Kafka connector is at-least-once; exactly-once to Kafka requires idempotent producers/transactions (Kafka's own EOS) and downstream deduplication by key.
- **Plain file sinks:** the file sink uses a metadata log to ignore half-written files; overwriting the same path from a failed attempt is guarded by this protocol.
- **Reading and overwriting the same table in one job** is dangerous: a retry may read already-overwritten data, producing different results — read from a snapshot/stage, write to a new target.

### 6. Performance and design implications
- Idempotent merges cost more than appends (lookup by key); limit the scanned range with partition predicates and clustering.
- Deduplication needs state or a key lookup; bound state with watermarks so you do not keep every id forever.
- Smaller micro-batches shorten replays but multiply commits and small files; compact periodically.
- Atomic table formats are the simplest route to safe writes; avoid hand-rolled "append and hope".

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `checkpointLocation` | Stores offsets/commits/state | Every streaming query |
| `foreachBatch` + `MERGE INTO` | Idempotent upsert per batch | CDC/dedup into Delta |
| `dropDuplicates(["id"])` + `withWatermark` | Streaming dedup with bounded state | At-least-once sources |
| `replaceWhere` / dynamic partition overwrite | Overwrite only affected partitions | Idempotent batch loads |
| `txnAppId` / `txnVersion` (Delta writer options) | Idempotent batch writes by app id | Repeated batch writes that must apply once |
| Kafka idempotent producer / transactions | Avoid duplicate sends | Spark → Kafka pipelines |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| MERGE/upsert by key — natural business key exists | No stable key; fall back to partition overwrite |
| Partition overwrite — data arrives in clean date slices | Late data touches many old partitions |
| Streaming dropDuplicates — bounded duplicate window | Duplicates can arrive after the watermark |
| Transactional table format — concurrent readers/writers | Sink is a system with no atomic commit |
| Accept at-least-once + downstream dedup | Strict downstream consumers cannot tolerate duplicates |

### 9. Common mistakes
- Default append writes with retries enabled → duplicates → use merge or overwrite.
- Non-deterministic logic (random, current time) in transforms → replays differ → derive from event data.
- Deleting or moving the checkpoint → duplicates or gaps on restart → treat it as part of the pipeline state.
- Dedup without a watermark → state grows unbounded → set watermark.
- Expecting Kafka sink to be exactly-once automatically → duplicates downstream → idempotent producer and keys.
- Reading and overwriting the same table in a retried task → inconsistent data → read from staging or snapshot.

### 10. 🎯 Interview kit
**30-second answer:** Exactly-once is at-least-once delivery plus idempotent or transactional writes. Spark logs offsets and commits in a checkpoint so a failed batch is replayed on the same input, and the sink — a Delta MERGE, a partition overwrite, or a transactional commit — makes the replay harmless.

**Structure a longer answer:** define semantics → source replayability → checkpoint mechanics → sink idempotency → examples and failure windows → limits (Kafka sink, dedup state).

**Likely follow-ups:**
- *Where does idempotence come from for Kafka?* → idempotent producer/transactions and keys; otherwise dedup downstream.
- *How avoid reading partial data?* → read from a transactional table snapshot, not raw files being written.
- *How dedup duplicates across partitions?* → key by business id with watermark; avoid full-history shuffle with bounded windows.
- *Risk of overwrite on same table?* → retry sees changed input.

**Signals interviewers look for:** you don't claim magic, you identify the failure window, and you can name concrete sink mechanisms.

### 11. Check yourself
- [ ] Explain why exactly-once = at-least-once + idempotence.
- [ ] Walk through a crash between sink write and commit log update.
- [ ] Write a `foreachBatch` merge for a keyed table.
- [ ] Say why dedup needs a watermark in streaming.
<!-- explainer:end -->

## Implementations (tools)

- [Apache Spark](<../../../TOOLS/Apache_Spark/_index.md>)
- [Apache Airflow](<../../../TOOLS/Apache_Airflow/_index.md>)
- [Apache Kafka](<../../../TOOLS/Apache_Kafka/_index.md>)
- [Databricks](<../../../TOOLS/Databricks/_index.md>)

## Practice questions

- [Basic] [What is a primary key and why does a pipeline need to know it to avoid duplicate rows?](<../../../PRACTICE/02_Data_Engineering/Pipelines_and_Integration/Pipeline_Orchestration/Q-13612_What_is_a_primary_key_and_why_does_a_pipeline_need_to_know_i.md>)
- [Intermediate] [What's the trade-off between deduplicating alerts inside the Spark pipeline versus at the ingestion gateway before they hit your DAG?](<../../../PRACTICE/02_Data_Engineering/Pipelines_and_Integration/Pipeline_Orchestration/Q-13454_What_s_the_trade-off_between_deduplicating_alerts_inside_the.md>)
- [Deep] [How would you deduplicate records in a pipeline while keeping only the most recent record per key?](<../../../PRACTICE/02_Data_Engineering/Pipelines_and_Integration/Pipeline_Orchestration/Q-13599_How_would_you_deduplicate_records_in_a_pipeline_while_keepin.md>)
- [Deep] [Your Spark job reads from a Delta table being written concurrently by the streaming ingest; how do you avoid reading a partial or inconsistent snapshot?](<../../../PRACTICE/02_Data_Engineering/Pipelines_and_Integration/Pipeline_Orchestration/Q-13495_Your_Spark_job_reads_from_a_Delta_table_being_written_concur.md>)
- [Deep] [How do you make Spark writes to the enriched-alert table atomic so a mid-write failure never leaves the correlation reader seeing half-written partitions?](<../../../PRACTICE/02_Data_Engineering/Pipelines_and_Integration/Pipeline_Orchestration/Q-13504_How_do_you_make_Spark_writes_to_the_enriched-alert_table_ato.md>)
- [Deep] [How do you guarantee at-least-once versus exactly-once delivery from a Spark streaming job into Kafka and where does idempotence come from?](<../../../PRACTICE/02_Data_Engineering/Pipelines_and_Integration/Pipeline_Orchestration/Q-13684_How_do_you_guarantee_at-least-once_versus_exactly-once_deliv.md>)
- [Deep] [How do you handle a source that sends the same business event across multiple partitions so your Spark dedup must be correct without a full-history shuffle?](<../../../PRACTICE/02_Data_Engineering/Pipelines_and_Integration/Pipeline_Orchestration/Q-13716_How_do_you_handle_a_source_that_sends_the_same_business_even.md>)
- [Deep] [How would you catch and handle duplicate events in an upstream Kafka topic before they inflate your Spark-computed count features?](<../../../PRACTICE/02_Data_Engineering/Pipelines_and_Integration/Pipeline_Orchestration/Q-13198_How_would_you_catch_and_handle_duplicate_events_in_an_upstre.md>)

## Related concepts

- [Optimistic Concurrency and Clocks](<../../../07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Distributed_Joins_in_MPP_Systems/Optimistic_Concurrency_and_Clocks.md>) *(Distributed Systems Core)*
- [Streaming Recovery and Deduplication](<../../../03_Big_Data_and_Streaming/Distributed_Processing/Big_Data_Processing/Streaming_Recovery_and_Deduplication.md>) *(Big Data Processing)*
- [Idempotent Incremental Loads](<../Idempotent_Incremental_Loads/Idempotent_Incremental_Loads.md>) *(Idempotent Incremental Loads)*

## References

- *Ultimate Data Engineering Design Patterns* — Bragadeesh Sundararajan — Building End-to-End Data Pipelines › 13. Building End-to-End Data Pipelines
- *Databricks Certified Data Engineer Associate Study Guide* — Derar Alhussein — Processing Incremental Data › Exactly-once semantics
- *Learning Spark (2nd ed.)* — Jules S. Damji, Brooke Wenig, Tathagata Das, Denny Lee — Under the Hood of an Active Streaming Query › Recovering from Failures with Exactly-Once Guarantees
- *Data Engineering Design Patterns* — Bartosz Konieczny — Error Management Design Patterns › Consequences
- *Delta Lake: The Definitive Guide* — Denny Lee et al. — Building Native Applications with Delta Lake › Concurrent writes on AWS S3
- *Delta Lake: The Definitive Guide* — Denny Lee et al. — Streaming In and Out of Your Delta Lake › Python
- *Delta Lake: The Definitive Guide* — Denny Lee et al. — Advanced Features › Python

