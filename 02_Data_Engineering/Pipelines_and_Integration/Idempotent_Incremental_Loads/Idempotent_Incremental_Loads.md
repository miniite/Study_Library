---
id: "02_Data_Engineering/Pipelines_and_Integration/Idempotent_Incremental_Loads/Idempotent_Incremental_Loads"
level: 4
type: "concept"
topic: "02_Data_Engineering/Pipelines_and_Integration/Idempotent_Incremental_Loads"
tools: ["Apache_Spark"]
---

# Idempotent Incremental Loads

**L4 Advanced** · [↑ Idempotent Incremental Loads](<_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** An idempotent incremental load moves only new or changed data but is written so that repeating any run, in any order or overlap, leaves the target in the same correct state.

### 1. The big picture
Incremental loading is what makes pipelines affordable: process the delta, not the whole table. But "only new data" plus retries, overlapping windows, late arrivals and backfills is exactly the recipe for duplicates and gaps. Idempotency is the property that tames it: whether a window is processed once, twice or five times, the target ends up identical.

The pattern combines a **change detector** (cursor, partition, CDC position), a **stable key**, an **upsert or overwrite** write, and **state management** that advances only after success. Variations differ in how they identify changes and how they handle deletes and late data.

### 2. Core concepts and vocabulary
- **Watermark / high-water mark** — stored value marking progress (max timestamp, ID, log position).
- **Lookback / overlap window** — deliberately re-reading a recent range to catch late commits.
- **Natural key / business key** — the identifier that defines a unique entity.
- **Surrogate key** — a generated key in the target; mapping from natural key is needed when merging.
- **Upsert (merge)** — insert if the key is absent, update if present.
- **Deduplication** — selecting one row per key (usually latest version) within a batch.
- **Tombstone / soft delete** — marker indicating a row was deleted at the source.
- **Partition overwrite** — replace an entire time or key partition atomically.
- **Slowly changing dimension (SCD)** — patterns for preserving history (type 1 overwrite, type 2 versioned rows).
- **Exactly-once effect** — end state equals processing each change once, even with repeats.

### 3. How it works, step by step
**Pattern A: watermark + merge**
1. Read watermark `W` (initially the beginning of time).
2. Extract rows with `cursor > W - lookback` into staging.
3. Deduplicate staging by key, keeping the latest version by cursor (and a tie-breaker).
4. `MERGE` into the target: update when matched and the incoming version is newer; insert when new; apply deletes if flagged.
5. Validate counts/keys.
6. Set the new watermark `W' = max(cursor in staging)`, in the same transaction as the merge if possible, or after the merge succeeds.

**Pattern B: partition overwrite**
1. For a window (hour/day), extract all rows belonging to that partition from the source.
2. Replace the target partition in one atomic operation.
3. Late data for old partitions is handled by re-running those partitions (lookback).

**Pattern C: append + dedupe view**
1. Always append incoming rows with load time and batch ID.
2. Expose a view or model that selects the latest row per key.
3. Periodically compact.

### 4. Worked example
Customers table with `customer_id`, `email`, `updated_at`, `is_deleted`.

```sql
-- 1. staging holds last extract (with 10-minute lookback)
-- 2. dedupe
CREATE OR REPLACE TEMP VIEW stg_latest AS
SELECT * EXCEPT (rn) FROM (
  SELECT s.*,
         ROW_NUMBER() OVER (PARTITION BY customer_id
                            ORDER BY updated_at DESC, _ingested_at DESC) AS rn
  FROM stg.customers s
) WHERE rn = 1;

-- 3. idempotent merge
MERGE INTO dw.customers t
USING stg_latest s
ON t.customer_id = s.customer_id
WHEN MATCHED AND s.is_deleted THEN DELETE
WHEN MATCHED AND s.updated_at >= t.updated_at THEN
  UPDATE SET email = s.email, updated_at = s.updated_at
WHEN NOT MATCHED AND NOT s.is_deleted THEN
  INSERT (customer_id, email, updated_at) VALUES (s.customer_id, s.email, s.updated_at);
```

Run it twice with the same staging: the second run finds `updated_at` equal, performs a no-op update, and the table is unchanged. Run it with an overlapping window: older rows are ignored by the `updated_at >=` guard. Run an out-of-order batch (older data arriving after newer): the guard prevents regressions.

State: a small control table `etl_state(table_name, watermark, run_id, status)` updated after validation.

### 5. Under the hood
- **Why the guard on version.** Without `s.updated_at >= t.updated_at`, replaying an older batch overwrites newer values. A monotonic version (timestamp, sequence, log position) makes the merge order-insensitive.
- **Ties and clocks.** Equal timestamps need a deterministic tie-breaker (an ingestion sequence). Source clocks can skew; prefer database-assigned versions or log positions where available.
- **Lookback sizing.** Choose it larger than the maximum commit delay between "row timestamp assigned" and "row visible" plus extraction delay; it costs extra reads but buys correctness.
- **Merge cost.** `MERGE` scans/joins the target on the key; with big tables, cluster/partition on the key or restrict the target scan with a partition predicate (for example only recent partitions) to avoid full scans.
- **Deletes.** Hard deletes are invisible to cursors; options are CDC delete events, periodic key-set diff (anti-join between source key list and target), or soft delete flags. If you can't see deletes, state the limitation and schedule full reconciliation.
- **Atomic state update.** If the watermark advances before the merge commits and the merge fails, data is skipped forever. If the merge commits but the watermark update fails, the next run re-reads a window; with idempotent merge this is harmless — the reason to prefer "write then advance" with idempotent writes.
- **Backfills.** Because the logic is idempotent, a backfill is just running the same job over past windows (ideally with limited concurrency), without special cleanup.
- **Table formats.** Delta, Iceberg and Hudi provide ACID `MERGE`, partition overwrite and time travel, making Patterns A and B safe on object storage.

### 6. Performance and design implications
- Cost scales with delta size plus merge lookup cost; keep staging small and partition-prune merges.
- Dedupe window functions are memory-heavy on large batches; partition work by key range or hour.
- Small files accumulate from frequent micro-batches; compact regularly.
- Monitor: rows extracted, rows merged (inserted/updated/deleted), watermark lag, duplicate key count in target (should be zero), and reconciliation diffs.
- Test: replay a batch twice, replay out of order, drop and re-run a window, and assert the target matches a full-reload baseline.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `MERGE INTO ... WHEN MATCHED/NOT MATCHED` | Atomic upsert/delete | Pattern A |
| `INSERT ... ON CONFLICT (key) DO UPDATE` (PostgreSQL) | Upsert | Operational DBs |
| `INSERT OVERWRITE PARTITION` / `REPLACE WHERE` | Atomic partition replace | Pattern B |
| `ROW_NUMBER() OVER (PARTITION BY key ORDER BY version DESC)` | Dedupe latest | Staging cleanup |
| Control/state table | Persist watermark | Every incremental pipeline |
| Anti-join on keys | Detect hard deletes | Reconciliation |
| Compaction (`OPTIMIZE`, `VACUUM`, rewrite files) | Reduce small files | Frequent micro-batches |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| Merge by key: mutable entities, late updates | Huge tables with no partition pruning on the key |
| Partition overwrite: event/fact data aligned to time windows | Updates to arbitrary old partitions without re-running them |
| Append + latest view: audit trail, simple writes | Heavy read cost without compaction/materialization |
| Lookback window: unreliable commit visibility | Very large lookbacks increasing cost without need |
| SCD2 history | Consumers only need current state (type 1 is simpler) |

### 9. Common mistakes
- Using `>` with the last max timestamp and no lookback → rows committed late are lost → add overlap and dedupe.
- Merging without a version guard → stale replay overwrites newer data → compare versions.
- Advancing watermark before commit → gaps → advance after success.
- Dedupe by timestamp only → nondeterministic ties → add tie-breaker.
- Ignoring deletes → ghost rows → CDC, soft delete or key diff.
- Not testing replays → duplicate bugs found in production → CI idempotency tests.

### 10. 🎯 Interview kit
**30-second answer:** An idempotent incremental load extracts a delta with a watermark and lookback, dedupes by key keeping the latest version, and merges into the target with a version guard, then advances the watermark after commit. Re-running any window produces the same result, so retries and backfills are safe.

**Structure a longer answer:** change detection → staging/dedupe → upsert with guard → deletes → state handling → testing and monitoring.

**Likely follow-ups:**
- *Late-arriving rows?* → lookback or reprocess partitions; merge by key.
- *Deletes?* → CDC events, soft deletes, key diff.
- *Why watermark after commit?* → avoids skipping; replays are harmless due to idempotency.
- *Merge slow?* → partition pruning, clustering on key, smaller staging.
- *Full reload vs incremental?* → size/mutability/latency; periodic full reconciliation.

**Signals interviewers look for:** the version guard, tie-breakers, delete handling, and a test strategy.

### 11. Check yourself
- [ ] Write a MERGE that is safe under replay and out-of-order batches.
- [ ] Explain how to size a lookback window.
- [ ] Show two ways to detect hard deletes without CDC.
- [ ] Describe what happens if the watermark update fails after a successful merge.
- [ ] Design a CI test proving a load is idempotent.
<!-- explainer:end -->

## Implementations (tools)

- [Apache Spark](<../../../TOOLS/Apache_Spark/_index.md>)

## Practice questions

- [Basic] [An interviewer asks: what's the difference between idempotent and deterministic in the context of a pipeline task, and why does it matter?](<../../../PRACTICE/02_Data_Engineering/Pipelines_and_Integration/Idempotent_Incremental_Loads/Q-30014_An_interviewer_asks_what_s_the_difference_between_idempotent.md>)
- [Deep] [Explain how you'd size and store a dedup lookback so it's bounded but still catches realistic duplicate windows.](<../../../PRACTICE/02_Data_Engineering/Pipelines_and_Integration/Idempotent_Incremental_Loads/Q-29986_Explain_how_you_d_size_and_store_a_dedup_lookback_so_it_s_bo.md>)
- [Deep] [How do you design a watermark or high-water-mark based incremental pull from a source that only exposes an updated_at column that can go backwards?](<../../../PRACTICE/02_Data_Engineering/Pipelines_and_Integration/Idempotent_Incremental_Loads/Q-30000_How_do_you_design_a_watermark_or_high-water-mark_based_incre.md>)
- [Deep] [How would you design an incremental aggregation pipeline for daily active users that must be correct after a partial-day reprocess?](<../../../PRACTICE/02_Data_Engineering/Pipelines_and_Integration/Idempotent_Incremental_Loads/Q-30013_How_would_you_design_an_incremental_aggregation_pipeline_for.md>)
- [Deep] [If you store the cursor in the pipeline but the write to the sink succeeds and the cursor update fails, what happens on retry and how do you make it safe?](<../../../PRACTICE/02_Data_Engineering/Pipelines_and_Integration/Idempotent_Incremental_Loads/Q-30030_If_you_store_the_cursor_in_the_pipeline_but_the_write_to_the.md>)
- [Deep] [Why is the combination of unique constraint plus INSERT ... ON CONFLICT a common idempotency pattern and where does it fail at scale?](<../../../PRACTICE/02_Data_Engineering/Pipelines_and_Integration/Idempotent_Incremental_Loads/Q-30045_Why_is_the_combination_of_unique_constraint_plus_INSERT_ON_C.md>)
- [Deep] [Walk me through the trade-offs of computing incremental deltas in the source database versus in Spark.](<../../../PRACTICE/02_Data_Engineering/Pipelines_and_Integration/Idempotent_Incremental_Loads/Q-30060_Walk_me_through_the_trade-offs_of_computing_incremental_delt.md>)
- [Deep] [How do you keep an incremental pipeline's cost predictable when a large historical backfill can suddenly balloon compute?](<../../../PRACTICE/02_Data_Engineering/Pipelines_and_Integration/Idempotent_Incremental_Loads/Q-30078_How_do_you_keep_an_incremental_pipeline_s_cost_predictable_w.md>)

## Related concepts

- [Handling Source Changes and Spikes](<../ETL_ELT/Handling_Source_Changes_and_Spikes.md>) *(ETL ELT)*
- [Pipeline Design Patterns](<../Pipeline_Design/Pipeline_Design_Patterns.md>) *(Pipeline Design)*
- [Incremental Rollups](<../../../01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Incremental_Rollups.md>) *(SQL Fundamentals)*
- [Exactly Once Spark Writes](<../Pipeline_Orchestration/Exactly_Once_Spark_Writes.md>) *(Pipeline Orchestration)*
- [Optimistic Concurrency and Clocks](<../../../07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Distributed_Joins_in_MPP_Systems/Optimistic_Concurrency_and_Clocks.md>) *(Distributed Systems Core)*
- [Idempotent Ingestion](<../../../07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Idempotent_Ingestion/_index.md>) *(Distributed Systems Core)*

## References

- *Data Pipelines with Apache Airflow* — Bas P. Harenslak, Julian de Ruiter — Project Finding the fastest way to get around NYC › Project: Finding the fastest way to get around NYC
- *Building Medallion Architectures* — Piethein Strengholt — Streamline the Gold Layer › Idempotency as a Key Design Principle
- *Data Engineering Design Patterns* — Bartosz Konieczny — Error Management Design Patterns › Idempotent producer
- *Data Engineering Design Patterns* — Bartosz Konieczny — Idempotency Design Patterns › Problem
- *Data Engineering Design Patterns* — Bartosz Konieczny — Idempotency Design Patterns › Mutable data source
- *Data Engineering Design Patterns* — Bartosz Konieczny — Chapter 5. Data Value Design Patterns › Idempotency
- *Data Engineering with Medallion Architecture* — Miki Eto — Building Scalable Data Pipelines › Process or steps
- *Data Engineering with Medallion Architecture* — Miki Eto — Building Scalable Data Pipelines › Cautions and limits
- *Data Engineering with Medallion Architecture* — Miki Eto — Building Scalable Data Pipelines › Expert notes
- *Delta Lake: The Definitive Guide* — Denny Lee et al. — Streaming In and Out of Your Delta Lake › Idempotent writes
- *Databricks Lakehouse Platform Cookbook* — Alan L. Dennis — CHAPTER 6Extracting from Source and Loading to Bronze(#contents.xhtml_ch06a) (cont.)

