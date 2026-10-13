---
id: "02_Data_Engineering/Modeling_and_Warehousing/OLTP_vs_OLAP/Separating_OLTP_and_Reporting"
level: 5
type: "deep-dive"
topic: "02_Data_Engineering/Modeling_and_Warehousing/OLTP_vs_OLAP"
tools: []
---

# Separating OLTP and Reporting

**L5 Expert** · [↑ OLTP vs OLAP](<../_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** Splitting reporting off the transactional system works only if you pick one reference architecture, define freshness and ownership, and make the boundary between the two systems a managed contract.

### 1. The big picture
The previous lesson explained *why* analytics must be separated from OLTP. This one is about doing it at **organizational scale**: when many teams, many sources and many dashboards are involved, the problem is no longer "add a replica"; it's "how do we avoid ending up with ten different, undocumented ways of getting data out of production?"

The answer in the book material is an **analytics reference architecture**: a documented guideline of which technologies, topologies and methodologies are allowed when a new data capability (for example streaming data into an OLAP cluster) is enabled. Without it, teams accumulate a zoo of one-off integrations; nobody knows where to look when one fails, and the team that built a capability is on the hook to keep it running. With a standard, every engineer knows the pattern, and failures are diagnosable.

Separation of reporting from OLTP is therefore a *platform decision* with four parts: the transport (batch, CDC, streaming), the target (warehouse, lakehouse, serving store), the guarantees (delivery semantics, freshness, quality), and the governance (lineage, contracts, access).

### 2. Core concepts and vocabulary
- **Reference architecture** — a standard blueprint that constrains tools and topology so new capabilities follow a known pattern.
- **Capability** — a reusable platform ability (e.g., "ingest a CDC stream into the lakehouse").
- **CDC (change data capture)** — reading the source's transaction log to emit inserts/updates/deletes in order.
- **Delivery guarantee** — at-most-once, at-least-once, exactly-once (usually "effectively once" via idempotent writes).
- **Dedup-at-read** — accepting duplicates in storage and removing them in queries/views using a key + version.
- **Data lineage** — recorded flow of data between datasets/jobs (e.g., OpenLineage's Dataset, Job, Run entities).
- **Data contract** — an agreed, enforced schema and semantic guarantee between producer and consumer.
- **Freshness SLA** — an explicit maximum staleness per dataset.
- **Blast radius / coupling** — impact scope of a failure; degree of dependence between producer and consumer.
- **Reverse ETL** — pushing derived analytic data back into operational systems (CRM, apps).

### 3. How it works, step by step
Establishing the separation at scale:

1. **Inventory consumers and needs.** Which reports/dashboards/ML jobs exist? What freshness, history, and accuracy does each need?
2. **Choose transport tiers.** Typically two: a nightly/hourly *batch* tier (incremental extract from replicas) and a *CDC/streaming* tier for datasets that need minutes of freshness. Fewer tiers = fewer things to operate.
3. **Define the landing contract.** Each source table has: owner, schema, primary key, change semantics (soft vs hard delete), PII classification, and a version.
4. **Land raw, immutably.** Append change events with source timestamps, LSN/offset, and operation type. This is your replay source.
5. **Build curated layers.** Deduplicate by key + latest version; apply deletes; conform dimensions; build facts.
6. **Make loads idempotent.** Merge by business key and version, or atomically swap partitions, so reruns are safe.
7. **Monitor.** Freshness, volume, schema drift, reconciliation (counts/sums vs source), lineage.
8. **Govern access.** Row/column security in the warehouse mirroring tenant isolation; workload isolation so one tenant's scan can't starve others.
9. **Document the standard** and require new integrations to use it (or justify exceptions).

### 4. Worked example
An idempotent incremental pattern over a CDC-landed staging table, correct under out-of-order and duplicate events:

```sql
-- stg.orders_cdc: one row per change event (OrderId, Op, SourceLsn, ChangedAt, payload...)
WITH ranked AS (
    SELECT *,
           ROW_NUMBER() OVER (PARTITION BY OrderId ORDER BY SourceLsn DESC) AS rn
    FROM   stg.orders_cdc
    WHERE  LoadBatchId = @batch
)
MERGE dw.orders AS t
USING (SELECT * FROM ranked WHERE rn = 1) AS s
   ON t.OrderId = s.OrderId
WHEN MATCHED AND s.Op = 'D' THEN DELETE            -- or set IsDeleted=1 (tombstone)
WHEN MATCHED AND s.SourceLsn > t.SourceLsn THEN
     UPDATE SET Status = s.Status, Total = s.Total, SourceLsn = s.SourceLsn
WHEN NOT MATCHED AND s.Op <> 'D' THEN
     INSERT (OrderId, Status, Total, SourceLsn) VALUES (s.OrderId, s.Status, s.Total, s.SourceLsn);
```

Why it's safe: `ROW_NUMBER` keeps only the latest event per key in the batch; the `SourceLsn >` guard ignores older events, so re-running the same batch or receiving events out of order doesn't regress state; deletes are explicit. Pair it with a reconciliation query (`COUNT(*)`, `SUM(Total)` per day vs source) that runs after each load and pages on mismatch.

### 5. Under the hood
- **Log-based CDC mechanics.** The source writes its transaction log; a connector (e.g., Debezium on Postgres logical decoding, SQL Server CDC) reads it using a replication slot/LSN position. An *unconsumed slot* forces the source to retain WAL, which can fill the disk — monitor slot lag and retention. Bulk updates (e.g., 200M-row backfill) flood the stream; mitigate with throttling, separate topics/queues, batch-mode catch-up, or excluding the table temporarily and reloading.
- **Delivery semantics.** Organization-wide standard is usually *at-least-once + idempotent apply*. True exactly-once across heterogenous systems is expensive; accept duplicates in raw and dedupe at the merge or at read for append-heavy event data.
- **Change-detection pitfalls.** `updated_at` watermarks miss writes that don't touch the column, and miss hard deletes. Use log-based CDC, row versions, or periodic reconciliation/full snapshots as a safety net.
- **Lakehouse corrections.** In Iceberg/Delta, time travel and snapshots mean corrections are new versions, not in-place overwrites; old snapshots are retained (cost) until vacuumed/expired. GDPR erasure then requires physical rewrite + snapshot expiry, not just a delete flag.
- **Event-sourced sources.** If the OLTP truth is an append-only event log, CDC is largely unnecessary — the events *are* the change stream; analytics models aggregate or project them.
- **Small-files problem.** Streaming CDC into a lake produces many tiny files; compaction cadence trades freshness against scan/metadata cost.
- **Lineage.** Open standards model a Job, its Run and the input/output Datasets with extensible facets so an incident can be traced from dashboard back to source column.

### 6. Performance and design implications
- Coupling: warehouses pulling directly from OLTP replicas nightly are simple and loosely coupled but spiky and stale; a streaming backbone is fresh but increases dependencies, so the platform team owns more.
- DR asymmetry: OLTP may require RPO≈0 across regions; the OLAP tier can often be rebuilt from raw/landed data, so it can use cheaper replication (or none) — that reduces cost.
- Tenant isolation: row-level security plus workload management (separate warehouses/queues per tenant tier) prevents noisy-neighbour scans.
- Measure: end-to-end latency (source commit → queryable), consumer lag, files per partition, reconciliation deltas, and cost per dataset.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| Logical decoding slot / replica identity | Source-side CDC configuration | Postgres CDC; monitor slot lag |
| `CDC`/Change Tracking enable | SQL Server change capture | Incremental extraction |
| `MERGE` with version guard | Idempotent upsert | Applying change batches |
| Compaction / `OPTIMIZE` / `VACUUM` | Merge small files, expire snapshots | Lakehouse maintenance |
| Row-level security policy | Per-tenant filter | Multi-tenant reporting |
| OpenLineage events | Emit job/dataset lineage | Platform observability |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| Nightly batch from replica | Business needs minute-level freshness |
| Log-based CDC | Team can't operate connectors, slots, schema drift |
| Dedup-at-read | Query cost/complexity matters and writes can dedupe cheaply |
| Strict standard architecture | Genuinely different needs; allow documented exceptions |
| Full reload partition on late data | Table is huge; use targeted corrections |

### 9. Common mistakes
- No standard → many bespoke integrations → unsupported failures → define reference architecture.
- Silent freshness gaps (dashboard missing yesterday until noon) → add ETL SLAs, DAG dependency alerts, freshness tests.
- Ignoring schema drift → pipeline breaks or silently drops columns → contracts + schema registry + drift alerts.
- Unmonitored replication slots → source disk fills → alert on lag/retention.
- Hard deletes not propagated → aggregates wrong → tombstones or reverse facts.
- Treating GDPR erasure as a flag → data persists in old snapshots → rewrite + expire.

### 10. 🎯 Interview kit
**30-second answer:** Separating reporting from OLTP at scale means defining a standard path from source to analytics: incremental or CDC extraction, an immutable landing zone, idempotent merges into curated models, and monitoring for freshness and drift. You standardize tools and contracts so each new integration follows a known pattern, and you accept at-least-once delivery with idempotent apply instead of chasing exactly-once everywhere.

**Structure a longer answer:** requirements (freshness/volume/guarantees) → architecture tiers → correctness (idempotency, ordering, deletes) → operations (monitoring, lineage) → governance (contracts, security, isolation) → cost and DR → scenario.

**Likely follow-ups:**
- *CDC vs incremental key scan for append-heavy tables?* → Key scan is simpler/cheaper and fine for pure appends with a monotonic key; CDC for updates/deletes and low latency; consider WAL retention and operational load.
- *How handle OLTP hard deletes already aggregated?* → Tombstone and reverse the fact, or rebuild the affected partition; choose by volume and audit needs.
- *Where do ordering/exactly-once/schema drift bite?* → Ordering across partitions, retries after partial failure, and DDL changes mid-stream.
- *6-hour rebuild blocks reporting?* → Move to incremental models with partition-level rebuild and reconciliation to keep the full rebuild's guarantees.

**Signals interviewers look for:** idempotency, operational awareness (slots, lag, drift), governance, and honest trade-offs.

### 11. Check yourself
- [ ] Write the standard batch+CDC architecture for a company with 20 source systems.
- [ ] Explain why an unconsumed replication slot is dangerous.
- [ ] Make a load idempotent under out-of-order events.
- [ ] Decide between tombstone, reversal, and partition reload for deletes.
- [ ] Define freshness SLAs and alerts for three datasets.
<!-- explainer:end -->

## Practice questions

- [Intermediate] [How do you validate an OLTP-to-OLAP migration end-to-end — row counts, checksums, business-metric reconciliation — before cutting over reporting?](<../../../../PRACTICE/02_Data_Engineering/Modeling_and_Warehousing/OLTP_vs_OLAP/Q-34755_How_do_you_validate_an_OLTP-to-OLAP_migration_end-to-end_row.md>)
- [Intermediate] [How do you handle a dimension that exists only in an external SaaS (not the OLTP DB) but must join to OLTP-sourced facts in the warehouse — where does that data land and how fresh must it be?](<../../../../PRACTICE/02_Data_Engineering/Modeling_and_Warehousing/OLTP_vs_OLAP/Q-34795_How_do_you_handle_a_dimension_that_exists_only_in_an_externa.md>)
- [Deep] [How do you validate an OLTP-to-warehouse migration before cutover — describe your row-count, checksum, and business-metric reconciliation plan and what tolerance you'd accept.](<../../../../PRACTICE/02_Data_Engineering/Modeling_and_Warehousing/OLTP_vs_OLAP/Q-35041_How_do_you_validate_an_OLTP-to-warehouse_migration_before_cu.md>)
- [Deep] [When migrating reporting off an OLTP DB, how do you phase the cutover (dual-run, shadow reads, gradual traffic shift) so you can prove correctness and roll back safely?](<../../../../PRACTICE/02_Data_Engineering/Modeling_and_Warehousing/OLTP_vs_OLAP/Q-35090_When_migrating_reporting_off_an_OLTP_DB_how_do_you_phase_the.md>)
- [Deep] [Explain how you'd design the cutover strategy to move a business-critical dashboard from live OLTP to the warehouse with a shadow/dual-run period, and what gates green-light the switch.](<../../../../PRACTICE/02_Data_Engineering/Modeling_and_Warehousing/OLTP_vs_OLAP/Q-34876_Explain_how_you_d_design_the_cutover_strategy_to_move_a_busi.md>)
- [Expert] [As principal you inherit a warehouse with 12000 orphaned models and no clear OLTP-source lineage — how do you drive a rationalization program without breaking downstream dashboards?](<../../../../PRACTICE/02_Data_Engineering/Modeling_and_Warehousing/OLTP_vs_OLAP/Q-34814_As_principal_you_inherit_a_warehouse_with_12000_orphaned_mod.md>)
- [Expert] [Design the migration path from a legacy Oracle system that runs both OLTP and monster reporting on one box, to a split architecture — how do you sequence it to avoid a big-bang cutover risk?](<../../../../PRACTICE/02_Data_Engineering/Modeling_and_Warehousing/OLTP_vs_OLAP/Q-34818_Design_the_migration_path_from_a_legacy_Oracle_system_that_r.md>)
- [Expert] [As principal, how do you build a migration off a vendor OLAP appliance (Teradata/Exadata) to open-format lakehouse without a multi-year stall, and how do you keep OLTP feeds stable through it?](<../../../../PRACTICE/02_Data_Engineering/Modeling_and_Warehousing/OLTP_vs_OLAP/Q-34874_As_principal_how_do_you_build_a_migration_off_a_vendor_OLAP_.md>)

## Related concepts

- [Coordinated Multi-store Schema Changes](<../../../../04_Data_Architecture_and_Strategy/Platform_Strategy/Schema_Evolution/Coordinated_Multi-store_Schema_Changes.md>) *(Schema Evolution)*
- [Schema Migration Strategy](<../../../../01_Databases_and_SQL/Database_Operations/Migrations/Concurrent_Index_Builds/Schema_Migration_Strategy.md>) *(Migrations)*
- [Multi-tenant Migrations](<../../../../07_Distributed_Systems_and_Design/System_Design/System_Design_Core/Read-through_and_Write-behind_Caching/Multi-tenant_Migrations.md>) *(System Design Core)*

## References

- *Data Engineering Best Practices* — Luiz Fernando F. Dos Santos — Reference Architectures › Analytics reference architecture

