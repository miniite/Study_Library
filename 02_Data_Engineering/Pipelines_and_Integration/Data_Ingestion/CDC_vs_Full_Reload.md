---
id: "02_Data_Engineering/Pipelines_and_Integration/Data_Ingestion/CDC_vs_Full_Reload"
level: 4
type: "concept"
topic: "02_Data_Engineering/Pipelines_and_Integration/Data_Ingestion"
tools: []
---

# CDC vs Full Reload

**L4 Advanced** · [↑ Data Ingestion](<_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** Ingestion brings data from sources into your platform; the core choice is between re-copying everything (full reload) and moving only what changed (incremental or CDC), and it is shaped by latency, volume, source impact and failure recovery.

### 1. The big picture
Data ingestion is the first stage of the pipeline: getting data out of operational databases, SaaS APIs, files, queues and devices and into a landing area (lake, warehouse, stream). Everything downstream depends on it being complete, timely and repeatable.

Three decisions define an ingestion design:
- **Batch vs streaming** — do you move data in scheduled groups or continuously? Batch gives high throughput and lower cost for analytical workloads; streaming gives near real-time freshness for operational use cases.
- **Push vs pull** — does the source write data out, or does your system fetch it? Timestamp-based extraction is a pull; log-based CDC is effectively a push into a log you read; IoT events are pushed to an endpoint.
- **Full vs incremental** — do you move the whole dataset each time or only changes?

This lesson focuses on the third decision and how it relates to CDC.

### 2. Core concepts and vocabulary
- **Full reload (full refresh, truncate-and-load)** — read the entire source table and replace the target.
- **Incremental load** — extract only rows new or changed since the last run, using a cursor (timestamp, ID, version).
- **CDC** — capture each change event from the source's log, triggers or tracking columns.
- **Cursor / high-water mark** — the stored value (max `updated_at` or ID) marking where the last run ended.
- **Staging (landing) table** — a temporary or raw table where extracted data first lands, with minimal change, before being merged into the final table.
- **Upsert / merge** — insert new rows and update existing ones by key.
- **Backfill** — re-ingesting a historical range, often after a bug or new source.
- **Data freshness** — the age of the newest data visible to consumers (now minus latest source event time reflected downstream).
- **Idempotent** — running the same load twice gives the same result.

### 3. How it works, step by step
**Full reload**
1. Read the entire source table (or file set).
2. Write to a staging table.
3. Swap or truncate-and-insert into the final table (ideally atomically, e.g. rename/exchange partition or a transaction).
4. Validate counts.

**Incremental (cursor-based)**
1. Read the stored cursor.
2. Query `WHERE cursor_col > :cursor` (often with a small overlap window to tolerate late commits).
3. Land to staging.
4. MERGE into target by primary key; deduplicate within the batch.
5. Advance the cursor only after the merge succeeds.

**CDC (log-based)**
1. Initial snapshot with a recorded log position.
2. Stream change events from that position.
3. Land events as an append-only change table (with op type and position).
4. Merge into current-state tables, applying deletes.
5. Periodically reconcile.

### 4. Worked example
A 500 million-row `events` table grows by 2 million rows a day.

- *Full reload nightly:* extracts 500M rows, ~hours of source load, and moves 250× more data than necessary. Simple and self-healing (every run restores truth), but cost and window grow with the table.
- *Incremental on `created_at`:* extracts ~2M rows in minutes. Works for append-only tables. For a mutable table, it misses rows updated without touching the cursor and misses deletes.
- *CDC:* near real-time; handles updates and deletes; operationally heavier.

```sql
-- incremental merge into target
MERGE INTO dw.customers t
USING (
  SELECT * FROM (
    SELECT s.*, ROW_NUMBER() OVER (PARTITION BY customer_id ORDER BY updated_at DESC) rn
    FROM stg.customers s
  ) WHERE rn = 1
) s
ON t.customer_id = s.customer_id
WHEN MATCHED AND s.updated_at > t.updated_at THEN UPDATE SET ...
WHEN NOT MATCHED THEN INSERT ...;
```

Practical hybrid: incremental or CDC daily, plus a **weekly full reload** or a keyed reconciliation to heal drift.

### 5. Under the hood
- **API ingestion** adds its own mechanics: authentication (tokens, OAuth), pagination (cursors or offsets), rate limiting (throttle, exponential backoff), incremental filters (`updated_since`), and error handling with retries and alerting. Incremental fetching with a server-side filter avoids re-pulling everything.
- **Why timestamps lie.** `updated_at` can be set by application code (so bypassed by direct SQL), set at statement start while commit happens later (a row committed after your extract with an earlier timestamp is missed), or in a different timezone. Mitigations: re-read an overlap window (e.g. last 10 minutes) and deduplicate, use a database-assigned commit sequence, or use log-based CDC.
- **Deletes.** A full reload sees deletes implicitly (the row is gone). Incremental and query-based CDC need soft deletes, tombstone tables or periodic key-set comparison to detect them.
- **Landing semantics.** Land raw data first, with minimal transformation. A staging layer lets you re-run transformations without touching the source again and gives an audit trail.
- **Streaming vs batch trade-offs.** Streaming brings out-of-order and late events, state management and watermarks; batch is simpler and cheaper. Many designs use streaming ingestion into a raw layer and batch curation after.

### 6. Performance and design implications
- Full reload cost grows linearly with table size, regardless of change rate; it also loads the source heavily.
- Incremental and CDC cost scales with change volume, and CDC latency can be seconds.
- Run extracts against a read replica or in off-hours windows where possible.
- Choose batch CDC (hourly) when business freshness tolerance is hours and cost matters; choose streaming where minutes matter (fraud, inventory).
- Measure freshness (source event time vs warehouse availability), completeness (row counts, checksums) and pipeline lag; alert on SLA breaches.
- Validate ingestion with row-count comparison, checksums by partition, and min/max/sum aggregates against the source.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `MERGE` / `INSERT ... ON CONFLICT` | Upsert into target by key | Applying incremental/CDC batches |
| `TRUNCATE` + `INSERT` or table swap | Full replace | Small dimensions, reference data |
| Overlap window on cursor | Re-reads recent data | Guarding against late-committing rows |
| Read replica endpoint | Offloads extraction | Heavy extracts from production |
| Reconciliation query (`COUNT`, `SUM`, hash by partition) | Detects drift | After loads, scheduled audits |
| API `updated_since` / pagination cursor | Incremental API fetch | SaaS sources |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| Full reload: small tables, no reliable change tracking, need self-healing simplicity | Tables large enough that the window or source load is unacceptable |
| Incremental by timestamp/ID: append-mostly, trustworthy cursor | Mutable rows without reliable `updated_at`, need for deletes |
| Log-based CDC: large mutable tables, need low latency and deletes | Team cannot operate it or source forbids log access |
| Streaming ingestion: operational freshness needs | Only daily reporting needs; added complexity and cost |
| Hybrid (CDC + periodic reconcile) | Never reconciling; drift accumulates unseen |

### 9. Common mistakes
- Using `>` the last cursor with no overlap → missed late commits → use an overlap and dedupe.
- Advancing the cursor before the load commits → permanent data loss on failure → advance after success, transactionally.
- Over-processing in staging → hard to rerun and audit → keep staging raw.
- Assuming incremental sees deletes → ghost rows → soft delete, CDC, or key comparison.
- Running big extracts on the primary at peak → production slowdowns → replicas or off-hours.
- No freshness/volume monitoring → silent staleness → measure and alert.

### 10. 🎯 Interview kit
**30-second answer:** Ingestion moves data from sources to the platform. A full reload is simple and self-correcting but scales with table size; incremental loads move only changes using a cursor; CDC reads the source log to capture every insert, update and delete with low latency. I choose by table size, mutability, delete handling, freshness need and source impact, and I add periodic reconciliation.

**Structure a longer answer:** requirements (freshness, volume, mutability) → options (full, incremental, CDC) → failure modes (late commits, deletes, cursor bugs) → idempotency and staging → monitoring.

**Likely follow-ups:**
- *Staging vs production table?* → staging holds raw landed data temporarily; production holds curated, deduplicated data.
- *Why CDC over polling?* → less source load, sees deletes and intermediate changes, lower latency.
- *How measure freshness?* → latest source event time versus time visible in the target, tracked per table.
- *Batch vs streaming?* → latency and cost versus complexity and consistency handling.

**Signals interviewers look for:** you tie the choice to requirements, mention idempotency and deletes, and have a validation story.

### 11. Check yourself
- [ ] Explain why a timestamp cursor can miss rows and two fixes.
- [ ] Write the steps of an incremental load that is safe to rerun.
- [ ] Compare full reload, incremental and CDC on cost, latency and delete handling.
- [ ] Describe what you would reconcile after a load and how.
- [ ] Explain push versus pull using three ingestion examples.
<!-- explainer:end -->

## Practice questions

- [Basic] [What is data ingestion and where does it fit in a typical data pipeline?](<../../../PRACTICE/02_Data_Engineering/Pipelines_and_Integration/Data_Ingestion/Q-11642_What_is_data_ingestion_and_where_does_it_fit_in_a_typical_da.md>)
- [Basic] [How is batch ingestion different from streaming ingestion?](<../../../PRACTICE/02_Data_Engineering/Pipelines_and_Integration/Data_Ingestion/Q-11648_How_is_batch_ingestion_different_from_streaming_ingestion.md>)
- [Basic] [What is the difference between a staging table and a final production table in an ingestion flow?](<../../../PRACTICE/02_Data_Engineering/Pipelines_and_Integration/Data_Ingestion/Q-11740_What_is_the_difference_between_a_staging_table_and_a_final_p.md>)
- [Intermediate] [What is Change Data Capture (CDC) and why is it useful for ingestion?](<../../../PRACTICE/02_Data_Engineering/Pipelines_and_Integration/Data_Ingestion/Q-11654_What_is_Change_Data_Capture_CDC_and_why_is_it_useful_for_ing.md>)
- [Intermediate] [How does CDC-based ingestion differ from a full table reload?](<../../../PRACTICE/02_Data_Engineering/Pipelines_and_Integration/Data_Ingestion/Q-11655_How_does_CDC-based_ingestion_differ_from_a_full_table_reload.md>)
- [Intermediate] [What is data freshness and how do you measure it for an ingestion pipeline?](<../../../PRACTICE/02_Data_Engineering/Pipelines_and_Integration/Data_Ingestion/Q-11693_What_is_data_freshness_and_how_do_you_measure_it_for_an_inge.md>)
- [Intermediate] [What is a webhook-based ingestion source and how does it differ from polling?](<../../../PRACTICE/02_Data_Engineering/Pipelines_and_Integration/Data_Ingestion/Q-11716_What_is_a_webhook-based_ingestion_source_and_how_does_it_dif.md>)
- [Intermediate] [What is the difference between push-based and pull-based ingestion?](<../../../PRACTICE/02_Data_Engineering/Pipelines_and_Integration/Data_Ingestion/Q-11731_What_is_the_difference_between_push-based_and_pull-based_ing.md>)

## Related concepts

- [CDC Fundamentals and Ordering](<../Change_Data_Capture/CDC_Fundamentals_and_Ordering.md>) *(Change Data Capture)*
- [AWS DMS vs Debezium](<../Change_Data_Capture/AWS_DMS_vs_Debezium.md>) *(Change Data Capture)*

## References

- *Ultimate Data Engineering Design Patterns* — Bragadeesh Sundararajan — Data Ingestion Patterns in Data Engineering › 4. Data Ingestion Patterns in Data Engineering
- *Data Engineering with Medallion Architecture* — Miki Eto — Building Scalable Data Pipelines › Streaming vs. batch ingestion
- *Building Medallion Architectures* — Piethein Strengholt — Laying the Groundwork › Real-Time Data and Streaming Ingestion Takeaway
- *Fundamentals of Data Engineering* — Joe Reis, Matt Housley — The Data Engineering Lifecycle › Batch versus streaming
- *Fundamentals of Data Engineering* — Joe Reis, Matt Housley — The Data Engineering Lifecycle › Key considerations for batch versus stream ingestion
- *Fundamentals of Data Engineering* — Joe Reis, Matt Housley — The Data Engineering Lifecycle › Push versus pull
- *Fundamentals of Data Engineering* — Joe Reis, Matt Housley — Ingestion › Chapter 7. Ingestion
- *Data Engineering Fundamentals* — Zhaolong Liu — CHAPTER 2Data Ingestion and Acquisition › CHAPTER 2Data Ingestion and Acquisition(#contents.xhtmlch02a)
- *AWS Certified Data Engineer Associate Study Guide* — Sakti Mishra et al. — Chapter 4. Data Ingestion and Transformation › Data Ingestion
- *AWS Certified Data Engineer Associate Study Guide* — Sakti Mishra et al. — Chapter 4. Data Ingestion and Transformation › Real-Time Streaming Data Ingestion
- *AWS Certified Data Engineer Associate Study Guide* — Sakti Mishra et al. — Kinesis Data Streams Versus Amazon MSK › Sample Streaming Ingestion Use Cases
- *AWS Certified Data Engineer Associate Study Guide* — Sakti Mishra et al. — Ingesting Data Using Zero-ETL Integrations › Ingesting Data from Databases with CDC Using AWS Data Migration Service

