---
id: "TOOLS/Google_Cloud/BigQuery/BigQuery_Cost_Control"
level: 4
type: "concept"
topic: "TOOLS/Google_Cloud/BigQuery"
tools: ["Google_Cloud"]
---

# BigQuery Cost Control

**L4 Advanced** · [↑ BigQuery](<_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** In BigQuery you pay for bytes scanned (on-demand) or for reserved slot capacity (editions), so cost control means reading fewer bytes — fewer columns, pruned partitions, clustered blocks — and putting hard limits and visibility around who can scan how much.

### 1. The big picture
BigQuery separates storage from compute. Storage is billed per GB-month (active vs long-term, logical vs physical). Query compute is billed in one of two ways: **on-demand** (per TiB of data *processed*, roughly $5–6.25/TiB depending on region and date — check current pricing — with the first TiB per month free) or **capacity** (BigQuery *Editions*: Standard, Enterprise, Enterprise Plus — you buy slots, optionally autoscaled, and pay per slot-hour regardless of bytes). Under on-demand, a careless `SELECT *` across a multi-TB table is a direct money event; under capacity, the same query consumes slots and delays other jobs.

### 2. Core concepts and vocabulary
- **Bytes processed / billed** — data scanned by columns referenced (after partition pruning); minimum 10 MB per table referenced.
- **Dry run** — a validation-only request that returns estimated bytes without running or charging.
- **Slot** — a unit of BigQuery compute; a reservation assigns slots to projects.
- **Partitioning** — splitting a table by ingestion time, a date/timestamp column, or integer range so queries can skip partitions.
- **Clustering** — sorting data within partitions by up to four columns so block pruning applies to filters.
- **`maximum_bytes_billed`** — per-query cap; the job fails if it would exceed.
- **Custom quotas** — per-project/per-user daily bytes-scanned limits.
- **Storage billing model** — logical (uncompressed) vs physical (compressed) per dataset.
- **Time travel / fail-safe** — 2–7 days of recoverable history (time travel; configurable), plus a 7-day fail-safe.
- **BigQuery ML** — `CREATE MODEL` in SQL to train and predict inside BigQuery.

### 3. How it works, step by step
1. Before executing, BigQuery computes the bytes each referenced table contributes: only the columns named in the query, restricted to the partitions that survive pruning.
2. On-demand: the total (rounded up per table to 10 MB) is the billable amount.
3. If clustered, block pruning reduces the *actual* bytes read, and billing reflects the reduced amount (for on-demand it is estimated up front and then adjusted to what was scanned).
4. `LIMIT` is applied after scanning, so on a non-clustered table it does **not** reduce bytes billed; a `LIMIT` on a clustered table with a matching filter can reduce reads, and `WHERE` on the partition column always does.
5. Capacity: the job takes slots from the reservation (or baseline + autoscale); cost is the slot-hours.
6. Results are cached for 24 h per user per identical query (if tables unchanged and no non-deterministic functions); cache hits are free.

### 4. Worked example
```sql
-- Bad: scans every column of every partition
SELECT * FROM `proj.sales.events`;

-- Better: columns + partition filter + clustering filter
SELECT user_id, event_name, ts
FROM `proj.sales.events`
WHERE DATE(ts) BETWEEN '2026-09-01' AND '2026-09-07'   -- partition pruning
  AND country = 'DE';                                  -- clustered column
```
```sql
-- Table designed to be cheap to query
CREATE TABLE `proj.sales.events`
(
  ts TIMESTAMP, user_id STRING, country STRING, event_name STRING, payload JSON
)
PARTITION BY DATE(ts)
CLUSTER BY country, event_name
OPTIONS (require_partition_filter = TRUE, partition_expiration_days = 400);
```
```bash
# Estimate cost without running
bq query --use_legacy_sql=false --dry_run 'SELECT user_id FROM `proj.sales.events` WHERE DATE(ts)="2026-09-01"'

# Cap spend per query
bq query --maximum_bytes_billed=100000000000 --use_legacy_sql=false '...'   # 100 GB
```
```sql
-- Who spent what in the last 7 days (on-demand)
SELECT user_email,
       SUM(total_bytes_billed) / POW(1024,4) AS tib_billed,
       COUNT(*) AS jobs
FROM `region-eu`.INFORMATION_SCHEMA.JOBS_BY_PROJECT
WHERE creation_time > TIMESTAMP_SUB(CURRENT_TIMESTAMP(), INTERVAL 7 DAY)
  AND job_type = 'QUERY'
GROUP BY user_email
ORDER BY tib_billed DESC;
```
Read the result as: bytes billed per user; multiply by the per-TiB rate to get cost; combine with `query` text to find repeat offenders.

### 5. Under the hood
- **Columnar storage (Capacitor).** Each column is stored and compressed separately, so naming fewer columns means reading fewer bytes — the root reason `SELECT *` is expensive and `SELECT *` + `LIMIT` is not cheap.
- **Partition vs cluster pruning.** Partition pruning is exact and known before execution, so the dry-run estimate is accurate; clustering prunes blocks dynamically, so the estimate is an upper bound.
- **Materialized views and BI Engine** cache/pre-aggregate; the optimizer can rewrite eligible queries automatically against a materialized view and read only incremental changes.
- **Slots and fairness.** In a reservation, queries share slots fairly across projects/jobs; idle slots can be borrowed across reservations. Autoscaling adds slots in increments (billed per second with a minimum); baseline slots are always on.
- **Storage choices.** Tables untouched for 90 days switch to long-term storage rate automatically (about half price). Physical billing can be cheaper for highly compressible data but counts time-travel and fail-safe bytes.
- **Streaming/Storage Write API** and load jobs: load jobs are free (shared pool), streaming inserts and the Storage Write API are charged per volume — prefer batch loads where latency allows.

### 6. Performance and design implications
- **Name columns**; avoid `SELECT *`; use `SELECT * EXCEPT(...)` when you truly need most.
- **Partition on the time column** you filter by; set `require_partition_filter`; cluster on high-cardinality filter/join columns in order of filter selectivity.
- **Pre-aggregate** repeatedly queried data into summary tables or materialized views; point dashboards at them.
- **Preview data** with the table preview or `bq head` / `TABLESAMPLE SYSTEM (1 PERCENT)` instead of `SELECT * LIMIT 10`.
- **Use approximate functions** (`APPROX_COUNT_DISTINCT`) where exactness isn't needed — CPU/slot saving.
- **Governance**: custom quota per user/project, `maximum_bytes_billed` in scheduled queries and BI tools, budgets and alerts in Cloud Billing, and a monthly review of top queries from `INFORMATION_SCHEMA.JOBS`.
- **Choosing a model**: steady, high, predictable usage → editions with baseline + autoscale; spiky or low usage → on-demand; mixed → on-demand for ad hoc, reservation for scheduled ETL (assign projects accordingly).

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| Dry run (`bq --dry_run`, API `dryRun`, console estimate) | Reports bytes without running | Before expensive queries |
| `maximum_bytes_billed` | Fails queries over a cap | Guard rails on schedules/tools |
| Custom quotas (IAM & Admin → Quotas) | Daily bytes per user/project | Stop runaway users |
| `require_partition_filter` | Rejects queries without a partition filter | Large partitioned tables |
| `INFORMATION_SCHEMA.JOBS*` | Job history, bytes, slot-ms | Cost attribution |
| Reservations/assignments | Slot capacity | Editions pricing |
| `CREATE MODEL` (BigQuery ML) | Train models in SQL | Analytics ML without exporting data |
| Table `labels` | Cost allocation tags | Chargeback |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| On-demand | Usage is low/spiky, or you can enforce byte caps |
| Editions with autoscaling | Spend is steady and large; need workload isolation |
| Partition + cluster | Table is large and queries filter on those columns |
| Materialized view | Same aggregation is queried repeatedly and base tables append mostly |
| BigQuery ML | Models are standard and data is already in BigQuery |

### 9. Common mistakes
- `LIMIT` to cut cost → still scans everything → filter on partitions, preview with table preview.
- No partition filter requirement → analysts scan years → set `require_partition_filter`.
- Over-partitioning (daily partitions of tiny data) → metadata overhead and no savings → coarser partitions or clustering only.
- Using on-demand in an automated tool without caps → a loop or bad join bills thousands → `maximum_bytes_billed`.
- Ignoring storage → forgotten copies and wide tables → table expiration, physical billing evaluation, `INFORMATION_SCHEMA.TABLE_STORAGE`.
- Scheduled queries re-scanning full history → incremental/MERGE patterns.

### 10. 🎯 Interview kit
**30-second answer:** On-demand BigQuery bills bytes scanned from referenced columns after partition pruning, so I cut cost with column selection, partitioning and clustering, summary tables, and guard rails (dry run, `maximum_bytes_billed`, quotas). For steady heavy usage I move to slot-based editions.

**Structure a longer answer:** pricing model → what drives bytes → table design → guard rails → monitoring → on-demand vs editions.

**Likely follow-ups:**
- *Why doesn't LIMIT save money?* → limit is applied after the scan; columnar billing is by columns scanned.
- *How to stop a 50 TB accident?* → `maximum_bytes_billed`, custom quotas, `require_partition_filter`, review access.
- *When editions?* → predictable high throughput, isolation of ETL vs ad hoc.
- *Dry run?* → estimates bytes, free.

**Signals interviewers look for:** you tie cost to storage format; you know cache and clustering behaviours; you mention governance and measurement.

### 11. Check yourself
- [ ] Estimate cost of a query given columns and partitions touched.
- [ ] Redesign a table to make the top three queries cheap.
- [ ] Write the `INFORMATION_SCHEMA` query for top spenders.
- [ ] Explain on-demand vs edition break-even reasoning.
- [ ] Explain why `SELECT *` with `LIMIT` isn't cheaper.
<!-- explainer:end -->

## Implementations (tools)

- [Google Cloud](<../_index.md>)

## Practice questions

- [Basic] [What is BigQuery ML and how can you train a model using SQL?](<../../../PRACTICE/TOOLS/Google_Cloud/BigQuery/Q-05169_What_is_BigQuery_ML_and_how_can_you_train_a_model_using_SQL.md>)
- [Intermediate] [Why does SELECT * cost more than selecting specific columns in BigQuery?](<../../../PRACTICE/TOOLS/Google_Cloud/BigQuery/Q-04996_Why_does_SELECT_cost_more_than_selecting_specific_columns_in.md>)
- [Intermediate] [How do you estimate the cost of a query before running it in BigQuery?](<../../../PRACTICE/TOOLS/Google_Cloud/BigQuery/Q-05115_How_do_you_estimate_the_cost_of_a_query_before_running_it_in.md>)
- [Intermediate] [How does BigQuery handle active storage versus long-term storage pricing?](<../../../PRACTICE/TOOLS/Google_Cloud/BigQuery/Q-05168_How_does_BigQuery_handle_active_storage_versus_long-term_sto.md>)
- [Intermediate] [How can you set up a maximum bytes billed limit to prevent runaway query costs?](<../../../PRACTICE/TOOLS/Google_Cloud/BigQuery/Q-05194_How_can_you_set_up_a_maximum_bytes_billed_limit_to_prevent_r.md>)
- [Deep] [What are some common reasons a BigQuery query becomes expensive and how do you reduce cost?](<../../../PRACTICE/TOOLS/Google_Cloud/BigQuery/Q-05191_What_are_some_common_reasons_a_BigQuery_query_becomes_expens.md>)
- [Deep] [How would you build a scan-cost budget alert and automatic query kill for runaway analyst queries using Cloud Monitoring and custom quotas?](<../../../PRACTICE/TOOLS/Google_Cloud/BigQuery/Q-05026_How_would_you_build_a_scan-cost_budget_alert_and_automatic_q.md>)
- [Expert] [Explain how you would size and choose between on-demand billing and BigQuery Editions (Standard/Enterprise) slot reservations for a mixed batch-plus-interactive workload.](<../../../PRACTICE/TOOLS/Google_Cloud/BigQuery/Q-05003_Explain_how_you_would_size_and_choose_between_on-demand_bill.md>)

## Related concepts

- [Database Cost Optimization](<../../../01_Databases_and_SQL/Database_Strategy/Cost_Optimization/Database_Cost_Optimization.md>) *(Cost Optimization)*
- [Statistics and Sargability](<../../../01_Databases_and_SQL/Query_Performance/Execution_Plans/Statistics_and_Sargability.md>) *(Execution Plans)*
- [Data Warehouse Fundamentals](<../../../02_Data_Engineering/Modeling_and_Warehousing/Data_Warehousing/Data_Warehouse_Fundamentals.md>) *(Data Warehousing)*

