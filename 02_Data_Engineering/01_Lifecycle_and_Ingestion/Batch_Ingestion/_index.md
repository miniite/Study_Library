---
id: "02_Data_Engineering/01_Lifecycle_and_Ingestion/Batch_Ingestion"
level: 4
type: "curriculum"
curriculum: "data_engineering"
---

# Batch Ingestion

**L4 Advanced** · [↑ Data Engineering Lifecycle and Ingestion](<../_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** Batch ingestion moves data in discrete chunks — by time window or by size — and the main design choices are snapshot vs incremental extraction, file-based vs direct reads, ETL vs ELT, and batch sizes that suit the target system.

### 1. The big picture

Batch ingestion is the oldest and still the most common way to get data from a source system into storage for analytics. Instead of reacting to every event, you periodically take a *bounded* slice of data — everything since midnight, or the next 128 MB — and move it in one operation. It is simple to reason about, easy to retry, and efficient for systems like columnar warehouses and object storage, which prefer a few large writes over many tiny ones.

Within the *Lifecycle and Ingestion* area, this lesson is the "batch" half of the ingestion story; the streaming half is covered in the message and stream ingestion lesson. Understanding batch well matters even for streaming-heavy companies, because most downstream stages (model training, daily reporting, backfills) end up running in batches anyway.

### 2. Core concepts and vocabulary

- **Time-interval batch** — a batch is the data from a window (hourly, daily). Classic for warehouse ETL.
- **Size-based batch** — a batch is cut when accumulated data reaches a size or event count; typical when moving a stream into object storage.
- **Full snapshot** — read the whole current state of the source on every run.
- **Differential (incremental) extraction** — read only rows changed since the last run (by timestamp, version, or change log).
- **File-based export** — the source exports files (CSV, JSON, Parquet) and hands them over by object storage, SFTP, SCP or EDI; a *push* pattern because the source does the preparation.
- **Pull vs push** — pull: the ingestion system fetches from the source; push: the source writes to the target.
- **ETL vs ELT** — transform before loading vs load raw and transform inside the target.
- **Data wrangling** — turning messy, malformed data into clean, structured data; typically a batch transformation.
- **Data migration** — one-off bulk movement of a whole system's data into a new platform.

### 3. How it works, step by step

1. **Choose the batch boundary.** Time window (e.g., `00:00–01:00`) or size threshold.
2. **Choose extraction mode.** Snapshot if the table is small or has no reliable change marker; incremental if volume or load on the source matters.
3. **Extract.** Query the source database, call an API, or receive an exported file. Also read metadata and schema changes.
4. **Land the raw data** in durable storage (object store or staging table), partitioned by batch window.
5. **Load** into the target, in large batches, respecting the target's write model.
6. **Transform** either before loading (ETL) or after (ELT).
7. **Record watermark / manifest** (the last timestamp or file processed) so the next run knows where to start and a rerun is idempotent.

| Decision | Option A | Option B |
|---|---|---|
| Extraction | Full snapshot — simple, heavy | Differential — light, needs a change marker |
| Transfer | Direct DB connection — fresh, loads source | File export — decoupled, source controls the content |
| Transform | ETL — clean data lands, pipeline heavier | ELT — raw lands first, flexible, uses warehouse compute |

### 4. Worked example

Daily incremental load of an `orders` table from an application database into a warehouse, using a timestamp watermark and landing files in object storage.

```sql
-- 1. Read the watermark saved by the previous run
SELECT last_loaded_at FROM ingest_state WHERE table_name = 'orders';
-- suppose it returns '2026-10-12 00:00:00'

-- 2. Extract only changed rows (run against a read replica)
SELECT *
FROM   orders
WHERE  updated_at >= '2026-10-12 00:00:00'
  AND  updated_at <  '2026-10-13 00:00:00';

-- 3. After landing the file (e.g. orders/dt=2026-10-12/part-0001.parquet)
--    and loading it, advance the watermark in one transaction
UPDATE ingest_state
SET    last_loaded_at = '2026-10-13 00:00:00'
WHERE  table_name = 'orders';
```

What to look at: the window is half-open (`>=` start, `<` end) so adjacent batches neither overlap nor leave gaps. Because the load writes the whole partition `dt=2026-10-12`, rerunning overwrites that partition and the result is the same — the job is idempotent. Two gaps to be aware of: timestamp-based extraction misses hard deletes and rows whose `updated_at` is not maintained; a change log (CDC) closes those gaps.

### 5. Under the hood

**Why batches should be large.** Columnar and object-store-based systems write immutable files. One-row inserts create many tiny files and many "create object" calls; in-place single-row updates are worse, because the engine may scan each existing column file to apply them. Row-store OLTP databases are the opposite: single-row writes are normal. Systems built for high insert rates (some real-time OLAP engines, certain hybrid stores) behave differently, and some warehouses accept streaming inserts through a dedicated buffer while performing poorly on plain single-row SQL inserts. The rule: learn the *natural* write pattern of the target.

**Why file-based hand-off exists.** Direct access to a backend database is often undesirable for security. With file export, the source team runs the export on its side, decides which columns and rows leave, and drops files in an agreed location. The cost: another moving part and a freshness lag.

**Where batch boundaries really live.** Even pipelines that ingest in near real time usually introduce batch boundaries downstream (training jobs, daily marts). Once data meets a batch process, that batch's frequency bounds the freshness of everything after it.

### 6. Performance and design implications

- **Source load**: full snapshots hammer the source; schedule off-peak or read from a replica.
- **Target efficiency**: target file sizes of hundreds of MB are usually healthier than thousands of small files.
- **Failure cost**: the larger the batch, the more is redone on failure — partition batches so retries are partial.
- **Latency floor**: batch interval is the minimum staleness; "hourly" cannot be fresher than an hour.
- **Network and storage**: differential extraction minimises transfer and target storage, which is why it is ideal on paper; snapshots remain common because they are simple and robust.
- **Measure**: rows and bytes per run, run duration vs interval, lag between `max(updated_at)` at source and at target, count of small files per partition.

### 7. Commands and settings you should know

| Command / setting | What it does | When to use |
|---|---|---|
| `COPY INTO` / bulk load (warehouse) | Loads staged files in parallel | Loading file batches into a warehouse |
| `aws s3 cp` / `sync` | Moves exported files to object storage | Shell-scripted file ingestion |
| `scp` / SFTP | Secure file transfer from a partner | EDI-style hand-offs |
| SSH tunnel via bastion host | Reaches a private database securely | Pulling from DBs not exposed to the internet |
| Watermark / state table | Stores last processed boundary | Any incremental pull |
| Partition overwrite | Replaces a whole batch partition | Making reruns idempotent |
| Sqoop-style import (Hadoop era) | Bulk import from RDBMS to HDFS via parallel tasks | Legacy Hadoop ingestion |

### 8. Trade-offs and decisions

| Choose this when | Avoid / watch out when |
|---|---|
| Full snapshot — small tables, no reliable change marker | Large tables, expensive source, tight windows |
| Incremental — large tables with reliable `updated_at` or a log | Hard deletes matter and nothing tracks them |
| File-based export — security or decoupling needs | You need sub-minute freshness |
| ELT — warehouse compute is cheap and raw data should be preserved | Sensitive data that must be masked before it lands |
| ETL — target has limited compute or strict schema | Frequently changing business logic (reloads are painful) |
| Size-based batching — stream-to-lake landing | Business needs fixed daily cut-offs |

### 9. Common mistakes

- **Many tiny inserts into a columnar store** → thousands of small files, slow queries and load → buffer and load in large batches.
- **Overlapping or gapped time windows** → duplicates or missed rows → half-open windows and a persisted watermark.
- **Relying on `updated_at` that application code doesn't always set** → silent data loss → verify with periodic reconciliation counts, or use CDC.
- **Non-idempotent loads (append-only on retry)** → duplicates after reruns → overwrite the partition or merge on a key.
- **Analytical queries on the production database** → contention → read from a replica or use exports.
- **Ignoring schema changes** → loads fail or silently truncate → capture schema metadata with each extraction.

### 10. 🎯 Interview kit

**30-second answer:** "Batch ingestion pulls or receives a bounded slice of data — by time interval or size — and loads it in bulk. I choose full snapshots for small or untrackable tables and incremental extraction with a watermark for big ones, land raw files first, and make every load idempotent by overwriting the batch partition."

**Structure a longer answer:** clarify (volume, freshness, source constraints) → mechanism (boundary, extraction, landing, load, watermark) → trade-offs (snapshot vs incremental, ETL vs ELT, file vs direct) → scenario.

**Likely follow-ups:**
- *Why are small inserts bad in a warehouse?* → Immutable columnar files; each insert creates tiny files and object operations; updates scan column files.
- *How do you make a batch job rerunnable?* → Deterministic window, partition overwrite or merge, watermark advanced only after success.
- *How do you capture deletes?* → CDC from the log, soft-delete flags, or periodic full-key reconciliation.
- *ETL or ELT?* → ELT when the target has strong compute and you want raw data retained; ETL when you must filter or mask before landing.

**Signals interviewers look for:** idempotency, awareness of target write model, treatment of late data and deletes, minimal impact on the source.

### 11. Check yourself

- [ ] Explain time-interval vs size-based batching with an example of each.
- [ ] Write a half-open window extraction query and explain why the boundary is half-open.
- [ ] Describe why file-based export is a push pattern and its security advantage.
- [ ] Explain why single-row inserts are poor for columnar targets.
- [ ] Design an idempotent daily incremental load including the watermark update.
- [ ] Name two things incremental timestamp extraction cannot see.
<!-- explainer:end -->

## References

- *Fundamentals of Data Engineering* — Joe Reis, Matt Housley — Ingestion › Snapshot or Differential Extraction
- *Fundamentals of Data Engineering* — Joe Reis, Matt Housley — Ingestion › ETL Versus ELT
- *Fundamentals of Data Engineering* — Joe Reis, Matt Housley — Ingestion › Batch Ingestion Considerations
- *Fundamentals of Data Engineering* — Joe Reis, Matt Housley — The Data Engineering Lifecycle › Push versus pull
- *Fundamentals of Data Engineering* — Joe Reis, Matt Housley — Modeling Streaming Data › Data wrangling
- *Hadoop: The Definitive Guide* — Tom White — Flume › Batching
- *Database Internals* — Alex Petrov — Q › S
- *Fundamentals of Data Engineering* — Joe Reis, Matt Housley — Ingestion › Inserts, Updates, and Batch Size
- *Hadoop: The Definitive Guide* — Tom White — Q › S
- *Learning Spark (2nd ed.)* — Jules S. Damji, Brooke Wenig, Tathagata Das, Denny Lee — T › V

