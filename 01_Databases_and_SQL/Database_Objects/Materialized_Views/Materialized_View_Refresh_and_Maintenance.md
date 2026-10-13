---
id: "01_Databases_and_SQL/Database_Objects/Materialized_Views/Materialized_View_Refresh_and_Maintenance"
level: 5
type: "concept"
topic: "01_Databases_and_SQL/Database_Objects/Materialized_Views"
tools: []
---

# Materialized View Refresh and Maintenance

**L5 Expert** · [↑ Materialized Views](<_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** A materialized view stores the *result* of a query as a physical table and trades freshness for speed; everything hard about it is the **refresh** — how, how often, how much it costs, what readers see while it runs, and how you notice when it silently goes stale.

### 1. The big picture
A normal view is a saved query: every read re-runs it. A **materialized view (MV)** is a saved query *plus its stored result*. Reads hit the precomputed rows (often with their own indexes, sort keys or partitions) so a dashboard query that joins five tables and aggregates millions of rows returns in seconds. The Data Engineering Design Patterns book calls this the *Dataset Materializer* pattern: you trade **storage and freshness** for **read latency**.

The result is a copy, so it can drift from the base tables. That makes the MV a *cache with a refresh contract*. A well-run MV answers five operational questions:
1. **When** is it refreshed? (on demand, on commit, on a schedule, automatically/incrementally)
2. **How** is it refreshed? (complete rebuild vs incremental/delta)
3. **What do readers see** during a refresh? (old data, blocked, or an inconsistent half-state)
4. **How stale** may it be, and how do consumers know?
5. **What does it cost** — compute, locks, I/O, storage — and is that cost still acceptable as data grows?

Engines differ a lot: Oracle has rich refresh modes (FAST/COMPLETE, ON COMMIT/ON DEMAND, partition change tracking); PostgreSQL has manual `REFRESH MATERIALIZED VIEW` (optionally `CONCURRENTLY`) and no incremental maintenance; SQL Server has **indexed views** (always synchronously maintained, with many restrictions); Redshift, BigQuery and Snowflake maintain MVs automatically/incrementally within limits; Databricks/DLT materialized views are recomputed (incrementally where possible) by pipeline runs.

### 2. Core concepts and vocabulary
- **View** — stored query text only; **materialized view** — stored query *and* its result set.
- **Refresh** — bringing the stored result in line with the base tables.
- **Complete (full) refresh** — truncate/replace and recompute everything.
- **Incremental / fast refresh / incremental view maintenance (IVM)** — apply only the changes since last refresh.
- **Materialized view log / change tracking** — structure capturing base-table changes for fast refresh (Oracle MV logs; CDC/stream offsets in others).
- **On demand vs on commit vs scheduled vs automatic** — the refresh trigger.
- **Concurrent refresh** — refresh without blocking readers (PostgreSQL `CONCURRENTLY` needs a unique index; builds a new copy and diffs).
- **Staleness / freshness / lag** — how far the MV is behind the sources; a **freshness SLA** is the promise.
- **Query rewrite** — optimizer transparently answers a query from an MV that covers it (Oracle, Redshift, BigQuery "smart tuning").
- **Self-maintainable aggregate** — can be updated from the delta alone (`SUM`, `COUNT`); **non-self-maintainable** needs the base data (`MIN`, `MAX`, `COUNT DISTINCT`, medians, with deletes).
- **PCT (partition change tracking)** — refresh only the MV partitions affected by changed base partitions.
- **Indexed view (SQL Server)** — view with a unique clustered index; the engine keeps it in sync inside every DML statement.
- **Projection / read model** — an application- or stream-maintained denormalised copy; the alternative to an MV.

### 3. How it works, step by step
**Creating and reading**
1. `CREATE MATERIALIZED VIEW mv AS SELECT …` runs the query once and stores the rows (plus metadata about its base objects and, on some engines, a log).
2. Queries on `mv` read the stored rows like a table — and can be indexed, partitioned, sorted.
3. Base tables change; the MV does **not** change (unless the engine maintains it).

**Complete refresh**
1. Engine re-executes the defining query.
2. It replaces the stored rows — by `TRUNCATE`+insert (Oracle atomic_refresh=false: fast, but empty in between), by `DELETE`+insert in one transaction (consistent but slow, lots of undo/redo), or by building a new heap and swapping (PostgreSQL non-concurrent: takes `ACCESS EXCLUSIVE`, so readers wait).
3. Statistics should be refreshed afterwards.

**Incremental refresh**
1. Changes since the last refresh are read from an MV log / change set / stream position.
2. The engine computes the delta result for the affected groups/rows (insert new aggregates, adjust existing ones).
3. It merges the delta into the MV and advances its refresh marker.
4. Fails over to complete refresh when the query isn't incrementally maintainable or the log is lost.

**Concurrent refresh (PostgreSQL)**
1. Build a temporary copy of the new result.
2. Compare to the existing MV by the required unique key; apply inserts/updates/deletes.
3. Readers continue to see the old data throughout, at the price of more I/O and a unique index requirement.

| Strategy | Freshness | Read blocking | Cost | Typical fit |
|---|---|---|---|---|
| Nightly complete refresh | up to 24 h stale | brief swap or none | high but predictable | daily reporting marts |
| Scheduled incremental | minutes–hours | none | proportional to change | dashboards on append-heavy facts |
| ON COMMIT (Oracle) | immediate | slows writers | write overhead on every commit | small, hot aggregates |
| Indexed view (SQL Server) | immediate | none (but lock contention on writers) | write amplification | simple aggregates over OLTP data |
| Auto-refresh by platform (BigQuery 15–30 min, Redshift auto) | near-real-time, **best effort** | none | managed | cloud warehouse |
| Stream-maintained projection | seconds | none | application complexity | real-time read models |

### 4. Worked example
```sql
-- PostgreSQL: daily sales summary
CREATE MATERIALIZED VIEW mv_daily_sales AS
SELECT date_trunc('day', sale_ts)::date AS sale_date,
       product_category, region,
       COUNT(*) AS txn_count, SUM(amount) AS total_sales, AVG(amount) AS avg_sale
FROM   sales_transactions
GROUP  BY 1, 2, 3
WITH DATA;

-- A unique index is REQUIRED for CONCURRENTLY and also speeds reads
CREATE UNIQUE INDEX ux_mv_daily_sales ON mv_daily_sales (sale_date, product_category, region);

-- Blocks readers while it runs (ACCESS EXCLUSIVE):
REFRESH MATERIALIZED VIEW mv_daily_sales;

-- Readers keep working; slower and needs the unique index:
REFRESH MATERIALIZED VIEW CONCURRENTLY mv_daily_sales;
```
What to check: during a plain refresh, a `SELECT` from another session shows as waiting in `pg_stat_activity` (`wait_event_type = Lock`). With `CONCURRENTLY` it doesn't, but the refresh does a full recompute plus diff, so it's *not* cheaper — only less disruptive.

Oracle equivalent with incremental maintenance:
```sql
CREATE MATERIALIZED VIEW LOG ON sales WITH ROWID, SEQUENCE (product_id, amount, sale_date) INCLUDING NEW VALUES;
CREATE MATERIALIZED VIEW mv_sales_by_product
  REFRESH FAST ON DEMAND AS
  SELECT product_id, COUNT(*) c, COUNT(amount) ca, SUM(amount) s FROM sales GROUP BY product_id;
EXEC DBMS_MVIEW.REFRESH('MV_SALES_BY_PRODUCT', method => 'F');   -- 'C' = complete
```
Note the extra `COUNT(*)`/`COUNT(amount)` columns: Oracle requires them so `SUM` can be maintained correctly under deletes — an example of self-maintainability driving the *schema* of the MV.

Freshness exposure (works in any engine):
```sql
CREATE VIEW dash_sales AS
SELECT s.*, (SELECT MAX(refreshed_at) FROM mv_refresh_log WHERE mv_name='mv_daily_sales') AS data_as_of
FROM   mv_daily_sales s;
```

### 5. Under the hood
- **Why incremental is hard.** For `SUM`/`COUNT` over an inner join the delta algebra is simple. `MIN`/`MAX` can't be fixed after deleting the current extreme without rescanning the group; `COUNT(DISTINCT)`, `AVG` over nullable data, outer joins, `HAVING`, windows and non-deterministic functions need special handling or are simply **not eligible** for fast refresh. Engines therefore publish restriction lists; if your query violates them the refresh silently degrades to complete (Oracle: `DBMS_MVIEW.EXPLAIN_MVIEW` tells you why).
- **Update-in-place vs append-only sources.** Append-only facts make incremental cheap (new partitions/files only). Heavy update/delete streams need change capture with before/after images, and correctness hinges on ordering and late data — with out-of-order events, watermarks and "retraction" semantics become part of the MV's definition.
- **PCT refresh.** If both base table and MV are partitioned on a related key (e.g. day), the engine records which base partitions changed (via `DBA_TAB_MODIFICATIONS`-like metadata) and refreshes only the corresponding MV partitions. Great for late-arriving corrections in a few recent days; useless if every partition is touched.
- **Cost model.** Refresh cost ≈ cost of defining query (complete) or ≈ delta size × maintenance cost (incremental). It *grows silently* with data volume: a refresh that took 4 min last year may take 40 min now, then overlap its own schedule. Overlapping or failed refreshes leave readers on stale data without any error visible to them.
- **Locks and writers.** PostgreSQL non-concurrent refresh takes `ACCESS EXCLUSIVE`; Oracle ON COMMIT refresh serialises commits on the MV log; SQL Server indexed views add per-DML maintenance and can cause blocking/deadlocks between writers touching the same aggregate row (hot groups).
- **Platform "automatic" refresh** (Redshift auto-refresh, BigQuery `enable_refresh`, `refresh_interval_minutes`) is **best-effort**: BigQuery aims within ~5 minutes of a base change but can lag if capacity is short, and may fall back to reading base tables for unrefreshed deltas (query-time merging). Don't equate it with transactional consistency.
- **Security and governance.** An MV combines several tables; denying access to a source table should still deny access to the MV (or apply row/column-level policies). Retention is awkward: you can't simply delete a base partition without deciding what the MV reflects.
- **DR.** An MV is derived data. Replicating it (storage/log shipping) gives immediate availability but doubles replication volume; rebuilding after failover costs RTO but keeps the replicated surface minimal. Choose per RPO/RTO: rebuild if refresh is fast and acceptable, replicate if the rebuild would blow RTO.

### 6. Performance and design implications
- **Measure benefit**: query latency/reads before vs after, read frequency × saved time, versus refresh cost + storage. An MV used twice a day over a 40-minute refresh is often a net loss.
- **Choose granularity** so many queries can use it (a daily-by-category MV serves monthly and yearly rollups too); avoid one MV per report.
- **Index/partition/sort the MV** like any table — it's your chance to design the physical layout for reads.
- **Idempotent, restartable refresh jobs**: refresh by partition/key range with delete-and-insert in one transaction (or `MERGE`), tracked by a high-water mark; rerunning must not double count. Build the new version in a staging table and swap atomically for zero-downtime.
- **Monitoring**: track refresh duration, rows changed, last success time, error count, lag vs SLA; alert on *trend* (duration up 20% over 4 weeks) not just failure. Sources: `pg_stat_user_tables`/job logs, `DBA_MVIEWS.LAST_REFRESH_DATE`, `STV_MV_INFO` (Redshift), `INFORMATION_SCHEMA.MATERIALIZED_VIEWS`, orchestration metadata.
- **Make freshness explicit**: expose `data_as_of`, publish a freshness contract per MV (tier: realtime / 15 min / daily), and agree what "real-time" means with stakeholders.
- **Schema change**: changing a base column typically invalidates the MV; use versioned MVs (blue/green: build `mv_v2`, switch consumers via a view, drop `mv_v1`).
- **Graceful degradation**: if refresh fails or the source is under load, keep serving the last good version with a staleness flag instead of erroring.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `CREATE MATERIALIZED VIEW … [WITH [NO] DATA]` | Create (and optionally populate) | Initial build |
| `REFRESH MATERIALIZED VIEW [CONCURRENTLY] mv` (PG) | Recompute; concurrent keeps reads working | Scheduled refresh; need unique index for concurrent |
| `DBMS_MVIEW.REFRESH(name, method => 'F'/'C'/'?')` (Oracle) | Fast/complete/force refresh | On-demand refresh |
| `CREATE MATERIALIZED VIEW LOG ON t` (Oracle) | Track changes for fast refresh | Before FAST refresh MVs |
| `DBMS_MVIEW.EXPLAIN_MVIEW` | Explain capabilities/limits | Why no fast refresh/rewrite |
| `CREATE MATERIALIZED VIEW … OPTIONS(enable_refresh, refresh_interval_minutes)` (BigQuery) | Auto refresh | Warehouse dashboards |
| `AUTO REFRESH YES` / `REFRESH MATERIALIZED VIEW` (Redshift) | Auto/manual refresh | Warehouse; streaming ingestion MVs |
| `CREATE UNIQUE CLUSTERED INDEX` on a `WITH SCHEMABINDING` view (SQL Server) | Indexed view | Synchronous aggregate over OLTP tables |
| `CREATE OR REPLACE MATERIALIZED VIEW` (Databricks/DLT) | Pipeline-managed MV | Lakehouse transformations |
| `pg_cron`, Airflow, dbt incremental models | Scheduling/orchestration | Engines without auto refresh |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| **Materialized view** — expensive repeated aggregation, tolerable staleness, many readers | Data must be transactionally current, or each query is unique |
| **Complete refresh** — small/medium data, complex SQL not eligible for fast refresh | Large data with a small change rate |
| **Incremental refresh** — big append-heavy sources, frequent refresh | Many updates/deletes, MIN/MAX, non-deterministic logic, or loss of change log risk |
| **ON COMMIT / indexed view** — small, hot, simple aggregate, freshness is mandatory | High write concurrency; wide groups; heavy OLTP |
| **Result cache** — identical repeated query, data rarely changes, no storage plan | Different parameters each time; need persistence/indexes |
| **Stream-processing read model** — seconds-level freshness, multiple consumers, complex joins | Small team; SQL-only requirement; operational overhead |
| **Summary table via ETL (dbt/Spark)** — need full control, versioning, testing | You want transparent optimizer rewrite |

### 9. Common mistakes
- **Running plain `REFRESH` in business hours on PostgreSQL** → readers blocked behind `ACCESS EXCLUSIVE` → use `CONCURRENTLY` (with unique index) or swap tables.
- **Assuming `CONCURRENTLY` is cheaper** → it's heavier, just non-blocking → plan capacity.
- **No freshness visibility** → stakeholders act on stale numbers → expose `data_as_of`, alert on lag.
- **Refresh schedule overlapping its own duration** → pile-ups, partial states → lock the job, size the interval above the p99 duration.
- **Defining a query that can't refresh incrementally** → silently complete refresh each time → check `EXPLAIN_MVIEW`/docs.
- **Forgetting statistics and indexes on the MV** → bad plans on a table that "should be fast" → analyze after refresh.
- **Treating automatic refresh as guaranteed** → missed SLA under capacity pressure → monitor and fall back.
- **MV on top of MV chains** → cascading staleness and refresh ordering bugs → keep depth shallow, define dependencies.
- **Giving broad access to an MV that joins sensitive tables** → data leak → align permissions or policies.
- **Mixing batch refresh with incremental updates on the same MV** → duplicates/inconsistency → one writer path.

### 10. 🎯 Interview kit
**30-second answer:** "A materialized view precomputes and stores a query's result so reads are cheap, at the cost of staleness and refresh work. The design is all in the refresh: complete vs incremental, scheduled vs on commit, concurrent so readers aren't blocked, and monitoring so that staleness and rising refresh time are visible. I'd use one for expensive, frequently-read aggregates where minutes-to-hours of lag is acceptable."

**Structure a longer answer:** purpose → freshness requirement → refresh mechanism and its cost → what readers see during refresh → monitoring/SLA → alternatives (cache, summary table, streaming read model) → failure and DR.

**Likely follow-ups:**
- *Materialized view vs result cache?* → MV is persisted, indexable, shared across sessions, survives restarts, needs refresh; result cache is automatic, memory-based, invalidated on any base change, only for identical queries.
- *Materialized view vs temp table populated by ETL?* → MV knows its definition and may support rewrite/incremental refresh; ETL table gives full control but you own dependency tracking.
- *What blocks reads in PostgreSQL?* → Non-concurrent refresh takes an exclusive lock; use `CONCURRENTLY`.
- *Stakeholder wants real time but MV refreshes every 15 min?* → Define freshness SLA, show `data_as_of`, tighten refresh or move that metric to a streaming/read-model path, and agree on wording.
- *Incremental refresh worth it?* → Compare delta size, query eligibility, update/delete mix; append-only wins, heavy update-in-place often doesn't.
- *Monitoring gradual regression?* → Store each refresh's duration, rows, bytes; alert on trend and on lag vs SLA.
- *Blue-green definitions?* → Version the MV, switch via view/synonym, retire old.
- *DR?* → Rebuild vs replicate based on RTO/RPO and refresh cost.

**Signals interviewers look for:** you reason in freshness contracts and costs, know your engine's refresh semantics (blocking, incremental limits), treat the MV as an operational object with monitoring, and compare with alternatives honestly.

### 11. Check yourself
- [ ] Explain what a reader sees during a plain vs concurrent refresh in PostgreSQL.
- [ ] Give two aggregates that are self-maintainable and two that aren't, and why.
- [ ] Design a refresh job that is idempotent and restartable.
- [ ] Write the metrics and alerts you'd put on an MV to catch a slow drift in refresh time.
- [ ] Compare MV, result cache, summary table and stream read model for a "last 15 minutes of orders" dashboard.
<!-- explainer:end -->

## Practice questions

- [Basic] [What is the difference between a materialized view and a database snapshot historically in Oracle, and does the terminology matter today?](<../../../PRACTICE/01_Databases_and_SQL/Database_Objects/Materialized_Views/Q-32250_What_is_the_difference_between_a_materialized_view_and_a_dat.md>)
- [Deep] [MySQL has no native materialized views. Describe a trigger-plus-summary-table approach and its correctness pitfalls.](<../../../PRACTICE/01_Databases_and_SQL/Database_Objects/Materialized_Views/Q-32218_MySQL_has_no_native_materialized_views_Describe_a_trigger-pl.md>)
- [Deep] [How do you decide the placement of a materialized view in a data-warehouse layering (staging, core, mart)?](<../../../PRACTICE/01_Databases_and_SQL/Database_Objects/Materialized_Views/Q-32259_How_do_you_decide_the_placement_of_a_materialized_view_in_a_.md>)
- [Deep] [How would you handle materialized views defined over slowly-changing dimensions so historical aggregates remain correct as dimension attributes evolve?](<../../../PRACTICE/01_Databases_and_SQL/Database_Objects/Materialized_Views/Q-32112_How_would_you_handle_materialized_views_defined_over_slowly-.md>)
- [Deep] [Explain how you'd design a materialized-view refresh SLA dashboard that ties refresh cost and freshness directly to business KPIs so leadership can prioritize.](<../../../PRACTICE/01_Databases_and_SQL/Database_Objects/Materialized_Views/Q-32151_Explain_how_you_d_design_a_materialized-view_refresh_SLA_das.md>)
- [Expert] [How do you enable and verify that query rewrite is actually happening for a given SQL statement?](<../../../PRACTICE/01_Databases_and_SQL/Database_Objects/Materialized_Views/Q-32097_How_do_you_enable_and_verify_that_query_rewrite_is_actually_.md>)
- [Expert] [Describe how you'd handle a materialized view definition that depends on a table which is itself being partitioned or repartitioned.](<../../../PRACTICE/01_Databases_and_SQL/Database_Objects/Materialized_Views/Q-32257_Describe_how_you_d_handle_a_materialized_view_definition_tha.md>)
- [Expert] [How do you architect materialized views to coexist with row-level and column-level security so aggregation doesn't become a policy bypass?](<../../../PRACTICE/01_Databases_and_SQL/Database_Objects/Materialized_Views/Q-32134_How_do_you_architect_materialized_views_to_coexist_with_row-.md>)

## Related concepts

- [Materialized View Refresh Correctness](<../Views/Materialized_View_Refresh_Correctness.md>) *(Views)*
- [Materialized View Trade-offs](<../Views/Materialized_View_Trade-offs.md>) *(Views)*
- [Index Strategy and Governance at Scale](<../../Query_Performance/Indexing/Index_Tuning_in_Practice/Index_Strategy_and_Governance_at_Scale.md>) *(Indexing)*
- [Using Views Effectively](<../Views/Using_Views_Effectively.md>) *(Views)*

## References

- *Designing Data-Intensive Applications* — Martin Kleppmann — Processing Streams › Maintaining materialized views
- *Designing Data-Intensive Applications* — Martin Kleppmann — B › C
- *Designing Data-Intensive Applications* — Martin Kleppmann — T
- *AWS Certified Data Engineer Associate Study Guide* — Sakti Mishra et al. — SQL-Based Data Transformation Using Amazon Redshift › SQL Data Transformations
- *AWS Certified Data Engineer Associate Study Guide* — Sakti Mishra et al. — Cost Optimization for Data Pipelines › Practice Questions
- *AWS Certified Data Engineer Associate Study Guide* — Sakti Mishra et al. — Practice Exam › Chapter 9. Practice Exam
- *Data Engineering Design Patterns* — Bartosz Konieczny — Data Storage Design Patterns › Data access
- *Data Engineering Design Patterns* — Bartosz Konieczny — Data Storage Design Patterns › Examples
- *Databricks Certified Data Engineer Associate Study Guide* — Derar Alhussein — Chapter 6. Building Production Pipelines › Materialized views
- *Databricks Certified Data Engineer Associate Study Guide* — Derar Alhussein — DLT Expectations › Creating a materialized view
- *Fundamentals of Data Engineering* — Joe Reis, Matt Housley — Materialized Views, Federation, and Query Virtualization › Composable materialized views
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 16: Best Practices for T-SQL Development and Database Design › Modularization and Reuse

