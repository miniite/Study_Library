---
id: "02_Data_Engineering/Modeling_and_Warehousing/OLTP_vs_OLAP/Feeding_OLAP_from_OLTP"
level: 5
type: "deep-dive"
topic: "02_Data_Engineering/Modeling_and_Warehousing/OLTP_vs_OLAP"
tools: ["Apache_Kafka"]
---

# Feeding OLAP from OLTP

**L5 Expert** · [↑ OLTP vs OLAP](<../_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** Moving data from the transactional database into the analytical store is a repeatable, idempotent, monitored pipeline — snapshot or incremental or CDC — whose hard parts are deletes, ordering, duplicates and schema change.

### 1. The big picture
Once analytics lives apart from OLTP, something must carry the data across. That carrier is the extraction-and-load pipeline. It sounds simple ("copy new rows"), but each choice has a failure mode that quietly corrupts numbers: a watermark that misses edits, a retry that double-counts, a delete that never arrives, a column added upstream that breaks the load.

This lesson covers the three ways to extract (full snapshot, incremental watermark, log-based CDC), how to load idempotently, how to handle deletes and corrections, and how to run the pipeline so that problems are caught by monitoring, not by a VP noticing yesterday's numbers are missing.

Cloud tooling (Azure Data Factory's Copy Data, for example) automates connection and movement — connecting to many source types with a service principal or managed identity and pushing into a warehouse such as Synapse — and for large loads you enable **staging**: data lands in intermediate blob storage, is bulk-loaded into the warehouse, and the temporary files are cleaned up automatically. The tool moves bytes; the correctness design is still yours.

### 2. Core concepts and vocabulary
- **Full (snapshot) extraction** — read the entire source table every run and replace the target.
- **Incremental / watermark extraction** — read only rows whose `updated_at` (or ID/version) exceeds the last stored high-water mark.
- **CDC (change data capture)** — read the source's transaction log to get every insert, update and delete in commit order.
- **Soft delete** — a flag (`is_deleted`, `deleted_at`) instead of removing the row; **hard delete** — the row is physically removed.
- **Tombstone** — a record marking a key as deleted in the target.
- **Idempotent load** — running it twice produces the same result as once.
- **Staging table** — transient target used to validate, dedupe and then merge.
- **Late / out-of-order event** — a change arriving after a newer one for the same key.
- **Replication slot / WAL retention** — source-side bookmark and log retention required for log-based CDC.
- **Schema drift** — upstream schema changes (new/renamed/retyped columns).
- **Reconciliation** — comparing counts/sums between source and target.

### 3. How it works, step by step
**Choosing an extraction method**

| Method | How | Catches updates | Catches hard deletes | Source load | Typical use |
|---|---|---|---|---|---|
| Snapshot | Select all, replace | Yes | Yes (implicitly) | High, scales poorly | Small tables, reference data |
| Watermark | `WHERE updated_at > @last` | Only if column always bumped | No | Low (indexed) | Large tables with reliable `updated_at` |
| Key scan | `WHERE id > @max_id` | No (append-only) | No | Very low | Append-only logs/events |
| Log-based CDC | Read transaction log | Yes | Yes | Low on tables, some log overhead | Updates/deletes, low latency |

**A robust incremental load**
1. Read the stored watermark (previous successful max version).
2. Extract rows changed since then, with a small *overlap window* (e.g., last 5 minutes) to catch late commits.
3. Write them to a staging table tagged with a batch ID.
4. Validate (keys not null, types, row counts within expected range).
5. Deduplicate within the batch: keep the latest version per business key.
6. `MERGE` into the target using key + version guard (insert new, update if newer, delete/tombstone if op = delete).
7. In the same transaction (or atomically after), advance the watermark.
8. Run reconciliation and update a freshness marker.

**Why a failed run is safe to repeat:** step 6's merge depends only on keys and versions, not on "how many times have I run"; step 7 advances the watermark only after success.

### 4. Worked example
Soft deletes break watermarks silently. Source:

```sql
-- Source (OLTP)
CREATE TABLE sales.Customer (
    CustomerId INT PRIMARY KEY,
    Name NVARCHAR(100),
    Email NVARCHAR(200),
    IsDeleted BIT NOT NULL DEFAULT 0,
    UpdatedAt DATETIME2 NOT NULL DEFAULT SYSUTCDATETIME()
);
```

A soft delete *updates* the row (`IsDeleted = 1`, `UpdatedAt` bumps) so a watermark extract sees it — as long as `UpdatedAt` is maintained. A **hard** delete leaves nothing to find. And any code path that updates a row without bumping `UpdatedAt` (a trigger-less bulk fix, a manual repair) is missed.

Target load with tombstone handling:

```sql
MERGE dw.Customer AS t
USING (
    SELECT CustomerId, Name, Email, IsDeleted, UpdatedAt
    FROM (
        SELECT *, ROW_NUMBER() OVER (PARTITION BY CustomerId ORDER BY UpdatedAt DESC) rn
        FROM stg.Customer WHERE BatchId = @BatchId
    ) x WHERE rn = 1
) s ON t.CustomerId = s.CustomerId
WHEN MATCHED AND s.UpdatedAt >= t.UpdatedAt
     THEN UPDATE SET Name = s.Name, Email = s.Email, IsDeleted = s.IsDeleted, UpdatedAt = s.UpdatedAt
WHEN NOT MATCHED THEN
     INSERT (CustomerId, Name, Email, IsDeleted, UpdatedAt)
     VALUES (s.CustomerId, s.Name, s.Email, s.IsDeleted, s.UpdatedAt);
```

For **facts**, double counting is the risk: load by natural key (`OrderLineId`) with `MERGE`, or load into a staging table and *swap the partition* for the affected date atomically. Both make re-runs safe.

To catch hard deletes without CDC, add a periodic **key reconciliation**: pull the full list of source keys (cheap, narrow) and compare to target; mark missing as deleted.

### 5. Under the hood
- **Watermark correctness.** Commit order is not timestamp order: a transaction that started earlier can commit later with an older `updated_at`, falling behind a watermark that already advanced. The overlap window and idempotent merge fix this; long-open transactions are the enemy. Use rowversion/LSN-based watermarks where available.
- **CDC and log mechanics.** The connector reads the log from a stored position. The source must *retain* log until the connector acknowledges it — an unconsumed slot makes WAL accumulate until the disk fills. Monitor slot lag; set retention limits and alerts. Bulk UPDATEs create huge bursts; catch-up may need parallelism or a snapshot-and-resume.
- **Ordering and exactly-once.** Ordering is guaranteed per key/partition at best. End-to-end exactly-once across systems is really at-least-once delivery + idempotent application. Where a store can't dedupe on write (append-only lake), dedupe at read (latest version per key view) and compact later.
- **Lakehouse targets.** Streaming CDC creates many small files; schedule compaction/`OPTIMIZE` — frequent compaction gives fresher, bigger-file layouts but costs compute and can conflict with writers; infrequent compaction hurts query planning and scan time. Time travel (Iceberg/Delta) turns corrections into new snapshots; retention costs storage and must be balanced against audit needs.
- **Append-only logs.** For event/log tables a monotonic key scan is far cheaper than CDC and has no WAL cost; CDC only pays when updates/deletes matter.
- **Event-sourced sources.** If the source of truth is the event log, you consume events directly; model facts as projections/aggregations and treat the stream as the change feed.

### 6. Performance and design implications
- Extraction needs an **index on the watermark column**, or the incremental query becomes a table scan every run.
- Extract from a replica, not the primary, for large pulls.
- Staging + bulk load beats row-by-row insert by orders of magnitude.
- Track: extract duration, rows/sec, lag, rejected rows, reconciliation delta, files per partition.
- Cost: CDC costs operations (connectors, slots, schema registry); snapshot costs compute and source I/O; choose by table behaviour rather than one-size-fits-all.
- Freshness alerting: define cadence per dataset, alert on `now - last_success > cadence + tolerance`, and use dependency awareness so a failed upstream alert is one page, not twenty.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| Watermark table (`etl.Watermark`) | Stores last processed version per table | Incremental loads |
| `ROW_NUMBER() … rn = 1` | Latest event per key | Dedup within a batch |
| `MERGE` / partition swap | Idempotent upsert / atomic replace | Facts and dimensions |
| Copy activity with staging (ADF) | Bulk load through blob staging | Large loads to a warehouse |
| Replication slot lag / WAL size | Source-side CDC health | Postgres/log CDC |
| `OPTIMIZE` / compaction | Merge small files | Streaming into lakehouse |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| Snapshot: table small, deletes common | Table large; source can't handle full scans |
| Watermark: reliable `updated_at`, few hard deletes | Writes bypass the column; hard deletes matter |
| CDC: need deletes, order, low latency | No capacity to operate log connectors |
| Dedup at read: append-only storage, high volume | Consumers are many and can't all dedupe correctly |
| Partition reload: fix late data | Partition huge / expensive |

### 9. Common mistakes
- Retrying a load that inserts blindly → duplicates / double-counted facts → key-based merge or partition swap.
- Watermark advanced before the load commits → lost data on failure → advance after success.
- No overlap window → late commits missed → overlap + idempotence.
- Hard deletes ignored → inflated totals → CDC or key reconciliation.
- No unique constraint on staging → silent duplicates → enforce and validate.
- Ignoring schema drift → silent column loss → drift detection and contracts.
- Unbounded WAL retention → source outage → slot monitoring.

### 10. 🎯 Interview kit
**30-second answer:** I pick extraction per table: snapshot for small reference data, watermark or key scan for large append-mostly tables, log-based CDC where updates and deletes matter. I land into staging, dedupe to the latest version per key, merge idempotently, advance the watermark only after success, and reconcile and monitor freshness. At-least-once delivery plus idempotent apply gives effectively-once results.

**Structure a longer answer:** table behaviour (update/delete/volume) → method → idempotent load design → delete and correction strategy → ops (monitoring, lag, drift) → trade-offs → scenario.

**Likely follow-ups:**
- *Snapshot vs watermark with soft deletes?* → Snapshot sees current truth (and hard deletes) at high cost; watermark is cheap but misses untouched-`updated_at` writes and hard deletes; soft deletes work if the flag update bumps the timestamp.
- *Design idempotent ETL so re-runs don't double-count?* → Natural/business key, staging with uniqueness, merge with version guard or atomic partition swap, load markers/batch IDs.
- *Incremental extract missed updates because `updated_at` not bumped?* → Use CDC/rowversion, or periodic checksum/key reconciliation, and fix writers.
- *200M-row bulk update floods CDC?* → Throttle/partition, isolate the table, catch up in bulk, or reload from snapshot; protect source disk.
- *Small-files compaction strategy?* → Time-based plus size thresholds; compact hot partitions off-peak; target ~128–512 MB files.

**Signals interviewers look for:** correctness first (idempotency, ordering, deletes), operational awareness (slots, lag, drift), and cost-aware trade-offs.

### 11. Check yourself
- [ ] Compare the four extraction methods on updates, deletes and source load.
- [ ] Write a merge that is safe under duplicates and out-of-order events.
- [ ] Explain why advancing the watermark last matters.
- [ ] Describe how to detect hard deletes without CDC.
- [ ] State what to monitor for log-based CDC on the source.
<!-- explainer:end -->

## Implementations (tools)

- [Apache Kafka](<../../../../TOOLS/Apache_Kafka/_index.md>)

## Practice questions

- [Intermediate] [Explain the difference between snapshot-based extraction and incremental (watermark) extraction from an OLTP source, and the failure modes of each on a table with soft deletes.](<../../../../PRACTICE/02_Data_Engineering/Modeling_and_Warehousing/OLTP_vs_OLAP/Q-35005_Explain_the_difference_between_snapshot-based_extraction_and.md>)
- [Deep] [A dbt incremental model reads from a CDC-landed staging table but occasionally drops updates — walk me through how you'd design the merge/dedup logic to be correct under out-of-order events.](<../../../../PRACTICE/02_Data_Engineering/Modeling_and_Warehousing/OLTP_vs_OLAP/Q-35008_A_dbt_incremental_model_reads_from_a_CDC-landed_staging_tabl.md>)
- [Deep] [How do you handle an OLTP hard delete that CDC captures but the OLAP fact table already aggregated — do you tombstone, reverse the fact, or reload the partition, and why?](<../../../../PRACTICE/02_Data_Engineering/Modeling_and_Warehousing/OLTP_vs_OLAP/Q-35071_How_do_you_handle_an_OLTP_hard_delete_that_CDC_captures_but_.md>)
- [Deep] [Walk me through a CDC pipeline from an OLTP Postgres to an OLAP warehouse — where do ordering, exactly-once and schema-drift problems bite you?](<../../../../PRACTICE/02_Data_Engineering/Modeling_and_Warehousing/OLTP_vs_OLAP/Q-34704_Walk_me_through_a_CDC_pipeline_from_an_OLTP_Postgres_to_an_O.md>)
- [Deep] [Design a DR strategy where the OLTP tier requires RPO near zero across regions but the OLAP tier can be fully reconstructed from source — how does that asymmetry shape your replication topology and cost?](<../../../../PRACTICE/02_Data_Engineering/Modeling_and_Warehousing/OLTP_vs_OLAP/Q-34800_Design_a_DR_strategy_where_the_OLTP_tier_requires_RPO_near_z.md>)
- [Expert] [Your streaming pipeline claims exactly-once from OLTP CDC to the OLAP sink, but you see duplicate facts after a broker failover — how do you locate where the guarantee actually broke?](<../../../../PRACTICE/02_Data_Engineering/Modeling_and_Warehousing/OLTP_vs_OLAP/Q-35019_Your_streaming_pipeline_claims_exactly-once_from_OLTP_CDC_to.md>)
- [Expert] [How do you tune the OLTP source for CDC — logical decoding slots, WAL retention, replica identity — without letting an unconsumed slot fill the disk?](<../../../../PRACTICE/02_Data_Engineering/Modeling_and_Warehousing/OLTP_vs_OLAP/Q-34749_How_do_you_tune_the_OLTP_source_for_CDC_logical_decoding_slo.md>)
- [Expert] [How do you evaluate whether a 'zero-ETL' native integration (e.g. Aurora-to-Redshift) is the right platform bet versus owning your own CDC pipeline, considering flexibility, lock-in, and failure-mode control?](<../../../../PRACTICE/02_Data_Engineering/Modeling_and_Warehousing/OLTP_vs_OLAP/Q-34810_How_do_you_evaluate_whether_a_zero-ETL_native_integration_e_.md>)

## Related concepts

- [Data Warehouse Fundamentals](<../../Data_Warehousing/Data_Warehouse_Fundamentals.md>) *(Data Warehousing)*
- [Handling Source Changes and Spikes](<../../../Pipelines_and_Integration/ETL_ELT/Handling_Source_Changes_and_Spikes.md>) *(ETL ELT)*
- [CDC Fundamentals and Ordering](<../../../Pipelines_and_Integration/Change_Data_Capture/CDC_Fundamentals_and_Ordering.md>) *(Change Data Capture)*
- [Real-time OLAP Serving Layer](<../../../../04_Data_Architecture_and_Strategy/Architecture_Patterns/Data_Architecture/Tiered_Lake_Storage/Real-time_OLAP_Serving_Layer.md>) *(Data Architecture)*

## References

- *Azure Data Factory Cookbook* — Xenia Ireton et al. — Orchestration and Control Flow › Using the ForEach and Filter activities

