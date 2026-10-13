---
id: "TOOLS/Snowflake/Snowflake_Platform/Snowflake_Compute_and_Caching"
level: 4
type: "concept"
topic: "TOOLS/Snowflake/Snowflake_Platform"
tools: ["Snowflake"]
---

# Snowflake Compute and Caching

**L4 Advanced** · [↑ Snowflake Platform](<_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** Snowflake separates cheap shared storage from elastic virtual warehouses you size, pause and scale out per workload, and three cache layers decide whether a query touches a warehouse at all.

### 1. The big picture
Snowflake is a managed cloud data platform with three layers: **storage** (compressed columnar data in micro-partitions on object storage), **compute** (virtual warehouses, clusters of nodes that run queries) and **cloud services** (metadata, optimizer, security, transaction management). Because compute and storage scale independently, different teams can run separate warehouses against the same data without contention, and you pay for compute only while a warehouse runs. Performance tuning therefore means choosing warehouse size/count, helping partition pruning, and understanding caches. Cost control means limiting how long and how large warehouses run.

### 2. Core concepts and vocabulary
- **Virtual warehouse** — named compute cluster; sizes XS, S, M, L, XL… each step doubles nodes and credits per hour (XS = 1, S = 2, M = 4, L = 8).
- **Credit** — billing unit for compute; billed per second with a 60-second minimum each time a warehouse starts.
- **Auto-suspend / auto-resume** — pause after N idle seconds; wake on the next query.
- **Multi-cluster warehouse** (Enterprise+) — min/max clusters of the same size that start and stop for concurrency; *Standard* policy favours starting clusters early, *Economy* favours keeping them full.
- **Micro-partition** — immutable storage unit holding roughly 50–500 MB uncompressed, with per-column min/max metadata used for **pruning**.
- **Clustering key** — hint that orders data by chosen columns so pruning stays effective on large tables.
- **Result cache**, **local disk cache**, **metadata cache** — see below.
- **Resource monitor** — credit quota object with threshold actions.

### 3. How it works, step by step
A query arrives and the cloud services layer parses and optimizes it using partition metadata. Then:
1. **Metadata cache** — some questions (`COUNT(*)`, `MIN`/`MAX` on a column) are answered from metadata with no warehouse.
2. **Result cache** — if the same query text ran in the last 24 hours, the referenced data hasn't changed, privileges allow it and it has no non-deterministic function, the stored result is returned in milliseconds, free of compute, from any warehouse. Each reuse extends it, up to 31 days.
3. Otherwise a warehouse runs the query: it scans only partitions that survive pruning; partitions are held in the warehouse's **local SSD cache**, so repeats on a warm warehouse are faster. Suspending a warehouse drops that cache.
4. If intermediate data exceeds memory it **spills** to local disk and then remote storage, which is slow.
5. Results are stored for the result cache.

### 4. Worked example
```sql
CREATE WAREHOUSE bi_wh WITH WAREHOUSE_SIZE='MEDIUM'
  AUTO_SUSPEND=60 AUTO_RESUME=TRUE
  MIN_CLUSTER_COUNT=1 MAX_CLUSTER_COUNT=3 SCALING_POLICY='STANDARD';

CREATE RESOURCE MONITOR monthly_cap WITH CREDIT_QUOTA=500
  TRIGGERS ON 80 PERCENT DO NOTIFY
           ON 100 PERCENT DO SUSPEND
           ON 110 PERCENT DO SUSPEND_IMMEDIATE;
ALTER WAREHOUSE bi_wh SET RESOURCE_MONITOR = monthly_cap;

-- diagnose a slow dashboard query
SELECT query_id, total_elapsed_time, bytes_scanned, partitions_scanned, partitions_total,
       bytes_spilled_to_local_storage, bytes_spilled_to_remote_storage, queued_overload_time
FROM snowflake.account_usage.query_history
WHERE warehouse_name='BI_WH' ORDER BY start_time DESC LIMIT 20;
```
Read it: `partitions_scanned ≈ partitions_total` means poor pruning (fix with clustering or a better filter). Non-zero spill means the warehouse is too small for that query (scale up). `queued_overload_time` means too many concurrent queries (scale out with multi-cluster).

### 5. Under the hood
Snowflake queries are executed in parallel across the warehouse's nodes; each node processes a subset of partitions. Doubling size doubles parallelism, so a query that already finishes in a few seconds on a small warehouse, or one dominated by a single serial step, a few huge files, or cloud-services time, gains nothing. A scan-heavy or spilling query can improve by more than 2x because more memory eliminates spill. Larger warehouses cost twice the credits per hour but often finish in half the time, so cost may stay flat while latency drops; the reverse happens when there is no speedup.

### 6. Performance and design implications
- **Scale up** (bigger size) for single heavy queries; **scale out** (more clusters) for many concurrent users.
- Set auto-suspend short (60 s is common) for bursty workloads, longer for ones that profit from warm local cache.
- Separate warehouses by workload (ELT, BI, data science) so one can't starve another and costs can be attributed.
- Result cache is per query text and role privileges; add `ALTER SESSION SET USE_CACHED_RESULT = FALSE` when benchmarking.
- Cached results cannot hide staleness of changed data: any change to underlying tables invalidates them. A "stale" dashboard usually means the upstream load didn't run, not that the cache lied; but a task rerunning identical SQL returns a cached result only while data is unchanged, so check `LAST_ALTERED` or load timestamps.
- Choose among fixes for a slow dashboard in this order: inspect Query Profile → rewrite or prune better → materialized view/pre-aggregation for repeated heavy aggregates → resize warehouse → rely on result cache for identical repeats.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `ALTER WAREHOUSE … SET WAREHOUSE_SIZE` | resize | slow single queries, spill |
| `AUTO_SUSPEND`, `AUTO_RESUME` | pause/wake | cost control |
| `MIN/MAX_CLUSTER_COUNT` | scale out | concurrency |
| `CREATE RESOURCE MONITOR` | credit quota with actions | budget guardrails |
| `STATEMENT_TIMEOUT_IN_SECONDS` | kills long statements | stop runaway queries |
| `ALTER TABLE … CLUSTER BY` | clustering key | large tables with selective filters |
| `QUERY_HISTORY`, `WAREHOUSE_METERING_HISTORY` | usage analysis | tuning and chargeback |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| Bigger warehouse: spill, huge scans | query is already short or not parallel |
| Multi-cluster: spikes of concurrent users | steady single-user loads (idle clusters cost) |
| Clustering key: multi-TB tables, selective range filters | small or constantly rewritten tables (reclustering credits) |
| Materialized view: repeated aggregates on slowly changing data | volatile base tables |

### 9. Common mistakes
- Thinking a resource monitor stops a single bad query → it limits *credit consumption* at account or warehouse level; use `STATEMENT_TIMEOUT_IN_SECONDS` for individual queries.
- Believing Snowflake has indexes → it prunes micro-partitions by metadata.
- Leaving warehouses on long auto-suspend → idle credits → set sensible values.
- Resizing to fix concurrency queueing → no help → add clusters.
- Benchmarking with warm caches → misleading → disable result cache and compare cold/warm.

### 10. 🎯 Interview kit
**30-second answer:** Storage, compute and cloud services are separate layers; I size warehouses per workload, auto-suspend them, use multi-cluster for concurrency, rely on pruning rather than indexes, and know the result, local and metadata caches.

**Structure a longer answer:** clarify (latency or concurrency or cost problem) → mechanism → trade-offs → scenario.

**Likely follow-ups:**
- *Why did Large not beat Medium?* → query not parallel, already fast, or bottleneck elsewhere; spilling queries do gain.
- *Result cache rules?* → same text, data unchanged, privileges, deterministic, 24 h (extendable).
- *What happens to running queries when a warehouse suspends?* → suspension waits for or follows running statements; manual immediate suspend aborts them.
- *Credits model?* → per-second compute credits, plus storage and serverless features billed separately.

**Signals interviewers look for:** diagnosing from Query Profile data, separating scale-up from scale-out, cost guardrails.

### 11. Check yourself
- [ ] Explain the three caches and what clears each.
- [ ] Decide size vs clusters for a queueing symptom and for a spilling symptom.
- [ ] Write a resource monitor with notify and suspend thresholds.
- [ ] Describe how pruning replaces indexes.
<!-- explainer:end -->

## Implementations (tools)

- [Snowflake](<../_index.md>)

## Practice questions

- [Basic] [What is Snowflake and how is it different from a traditional on-premise data warehouse?](<../../../PRACTICE/TOOLS/Snowflake/Snowflake_Platform/Q-39444_What_is_Snowflake_and_how_is_it_different_from_a_traditional.md>)
- [Intermediate] [What is a micro-partition and how does it help query pruning?](<../../../PRACTICE/TOOLS/Snowflake/Snowflake_Platform/Q-39451_What_is_a_micro-partition_and_how_does_it_help_query_pruning.md>)
- [Intermediate] [What is a resource monitor in Snowflake and how does it prevent runaway credit usage?](<../../../PRACTICE/TOOLS/Snowflake/Snowflake_Platform/Q-39529_What_is_a_resource_monitor_in_Snowflake_and_how_does_it_prev.md>)
- [Intermediate] [How would you catch and alert on a task that fails silently because it returns SUCCESS but produced zero rows?](<../../../PRACTICE/TOOLS/Snowflake/Snowflake_Platform/Q-39588_How_would_you_catch_and_alert_on_a_task_that_fails_silently_.md>)
- [Deep] [What is a multi-cluster warehouse and when would you use it to handle concurrency?](<../../../PRACTICE/TOOLS/Snowflake/Snowflake_Platform/Q-39486_What_is_a_multi-cluster_warehouse_and_when_would_you_use_it_.md>)
- [Deep] [Why might increasing warehouse size from Medium to Large give zero speedup for one query but 2x for another?](<../../../PRACTICE/TOOLS/Snowflake/Snowflake_Platform/Q-39546_Why_might_increasing_warehouse_size_from_Medium_to_Large_giv.md>)
- [Deep] [How does query result caching interact with a task that runs the same query hourly, and could caching mask a data-freshness bug?](<../../../PRACTICE/TOOLS/Snowflake/Snowflake_Platform/Q-39582_How_does_query_result_caching_interact_with_a_task_that_runs.md>)
- [Deep] [How do you decide whether a slow dashboard should be fixed by a materialized view, query rewrite, warehouse resize, or result caching?](<../../../PRACTICE/TOOLS/Snowflake/Snowflake_Platform/Q-39622_How_do_you_decide_whether_a_slow_dashboard_should_be_fixed_b.md>)

## Related concepts

- [Data Warehouse Fundamentals](<../../../02_Data_Engineering/Modeling_and_Warehousing/Data_Warehousing/Data_Warehouse_Fundamentals.md>) *(Data Warehousing)*
- [Star Schemas and Dimension Keys](<../../../02_Data_Engineering/Modeling_and_Warehousing/Dimensional_Modeling/Star_Schemas_and_Dimension_Keys.md>) *(Dimensional Modeling)*
- [Rollups vs On-demand Queries](<../../../04_Data_Architecture_and_Strategy/Architecture_Patterns/Data_Architecture/Tiered_Lake_Storage/Rollups_vs_On-demand_Queries.md>) *(Data Architecture)*
- [Micro-partitions vs Manual Partitions](<../../../02_Data_Engineering/Storage_and_Lakehouse/Partitioning_and_Bucketing/Micro-partitions_vs_Manual_Partitions.md>) *(Partitioning and Bucketing)*

## References

- *Data Engineering Best Practices* — Luiz Fernando F. Dos Santos — CHAPTER 12Popular Technologies › Snowflake
- *Data Quality Fundamentals* — Barr Moses, Lior Gavish, Molly Vorwerck — Assembling the Building Blocks of a Reliable Data System › Step 1: Map your inventory
- *Data Quality Fundamentals* — Barr Moses, Lior Gavish, Molly Vorwerck — Assembling the Building Blocks of a Reliable Data System › Step 2: Monitor for data freshness and volume
- *Data Quality Fundamentals* — Barr Moses, Lior Gavish, Molly Vorwerck — Assembling the Building Blocks of a Reliable Data System › Step 3: Build your query history
- *Practical Lakehouse Architecture* — Gaurav Ashok Thalpati — Compute Engines for Lakehouse Architectures › Snowflake

