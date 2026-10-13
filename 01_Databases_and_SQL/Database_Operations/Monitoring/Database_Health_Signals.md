---
id: "01_Databases_and_SQL/Database_Operations/Monitoring/Database_Health_Signals"
level: 4
type: "concept"
topic: "01_Databases_and_SQL/Database_Operations/Monitoring"
tools: ["Prometheus", "Grafana"]
---

# Database Health Signals

**L4 Advanced** · [↑ Monitoring](<_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** A database's health is read from a small set of signals — the golden four of latency, traffic, errors and saturation, plus engine-specific indicators such as locks, replication lag, log growth and disk — so that you alert on user-visible symptoms, watch resource trends for causes, and always know which signal to look at first.

### 1. The big picture
Traditional DBA monitoring asked "is the database up?" and "is utilization sustainable?". Operational visibility in modern reliability engineering asks a different question: *how is the database's behaviour affecting latency and correctness for users?* That means looking at distributions (histograms, percentiles), not just averages, and correlating database metrics with application metrics.

Health signals are layered. At the top are **symptoms** the user feels (query latency, error rate, availability). Below are **saturation and resource** signals (CPU, memory pressure, disk I/O, connections, locks, log space). At the bottom are **engine internals** (cache hit ratio, checkpoint behaviour, replication lag, vacuum/compaction progress). Good monitoring pages a human on the first layer, and lets them drill into the lower ones. Monitoring done the other way round — paging on every CPU spike — produces alert fatigue and misses the real incidents.

This lesson is the vocabulary and the "what to watch first" map; the neighbouring lesson on percentiles vs averages covers how to *summarize* latency correctly.

### 2. Core concepts and vocabulary
- **Golden signals** — latency, traffic, errors, saturation. A fifth, **correctness/freshness**, matters for data systems (replication lag, stale reads, failed constraints).
- **USE method** — for each resource: Utilization, Saturation, Errors. **RED method** — for each service: Rate, Errors, Duration.
- **Metric, log, trace** — numeric time series; discrete events; request paths across services.
- **Baseline** — what "normal" looks like by hour/day/week; alerts are meaningful only relative to it.
- **SLI / SLO** — a measured indicator (p99 query latency) and the target for it (99 % under 200 ms over 30 days).
- **Symptom-based vs cause-based alert** — page on "checkout queries slow", not on "CPU 80 %".
- **Saturation** — how much work is queued or waiting (run-queue, lock waits, connection-pool waiters, disk queue length).
- **Scrape interval / retention** — how often a metric is sampled and how long it is kept; high resolution for short windows, downsampled long-term.
- **Blind spot** — a failure mode with no signal (e.g. replication stopped but node "up").
- **Runbook** — a documented diagnostic and fix procedure attached to each alert.

### 3. How it works, step by step
**Triage order when paged for "the app is slow"**
1. **Is it the database at all?** Compare application-side latency to database-side execution time. Network, pool wait and app code often dominate.
2. **Errors first.** Connection failures, timeouts, deadlock victims, constraint violations, failovers.
3. **Latency distribution.** p50/p95/p99 over time — did everything slow, or only the tail?
4. **Saturation.** CPU run-queue, memory pressure (page life expectancy, swap), disk latency and queue, connection count versus limit, lock waits.
5. **Who/what.** Top queries by total time and by wait; blocking chains; recent deploys or schema changes.
6. **Durability and HA.** Log/WAL growth, replication lag, backup freshness, free disk.

**Signal families to instrument**

| Family | Signals | Typical alert |
|---|---|---|
| Availability | Up check that runs a real query, failover events | Query fails 3 × in a row |
| Latency | p50/p95/p99 per query class | p99 > SLO for 5 min |
| Throughput | QPS/TPS, rows read/written | Drop > 30 % vs baseline (traffic loss) or rise (capacity) |
| Errors | Failed connections, deadlocks, timeouts, constraint violations | Rate above baseline |
| Resource | CPU, memory/cache hit, IOPS and latency, network | Sustained saturation |
| Concurrency | Active vs total sessions, lock waits, blocking chain length, pool waiters | Waiters > 0 for minutes |
| Storage | Free space, growth rate, log size, tempdb | Time-to-full < 7 days |
| Replication/HA | Lag (bytes and seconds), replica state | Lag > threshold |
| Maintenance | Last successful backup, vacuum/stats update age, job failures | Backup older than RPO |

### 4. Worked example
Three quick health queries — one per engine family — that answer "what is waiting right now?".

```sql
-- SQL Server: top waits since restart and currently blocked sessions
SELECT TOP 10 wait_type, wait_time_ms/1000.0 AS wait_s, waiting_tasks_count
FROM sys.dm_os_wait_stats
WHERE wait_type NOT LIKE '%SLEEP%' AND wait_type NOT IN ('BROKER_TASK_STOP','XE_TIMER_EVENT')
ORDER BY wait_time_ms DESC;

SELECT r.session_id, r.blocking_session_id, r.wait_type, r.wait_time, t.text
FROM sys.dm_exec_requests r
CROSS APPLY sys.dm_exec_sql_text(r.sql_handle) t
WHERE r.blocking_session_id <> 0;
```

```sql
-- PostgreSQL: connection saturation, long transactions, replication lag
SELECT count(*) FILTER (WHERE state = 'active') AS active,
       count(*)                                  AS total,
       current_setting('max_connections')::int   AS max_conn
FROM pg_stat_activity;

SELECT pid, now() - xact_start AS xact_age, state, wait_event_type, left(query,80)
FROM pg_stat_activity
WHERE xact_start IS NOT NULL ORDER BY xact_age DESC LIMIT 5;

SELECT application_name, replay_lag FROM pg_stat_replication;
```

Reading the output: if `LCK_M_*` or lock wait events dominate and a blocking session sits at the head of a chain, it is concurrency, not capacity — find and fix the long transaction. If `PAGEIOLATCH_*` / `IO` waits dominate, check disk latency and cache hit ratio. If active ≈ max connections with many `idle in transaction`, it is a connection leak or pool misconfiguration — a classic cause of "latency doubled but CPU and IO look flat".

### 5. Under the hood
*(optional at this level)*
- Counters like `sys.dm_os_wait_stats` are cumulative since restart; to see *current* behaviour you take two samples and subtract. Monitoring tools do this for you; ad-hoc queries must too.
- Latency histograms are recorded in buckets so percentiles can be computed and aggregated; storing only the mean loses the tail permanently.
- Per-query statistics (`pg_stat_statements`, Query Store, performance schema) normalise query text and keep call counts and timing, which is how you rank by *total* time (frequency × cost) rather than single slowest execution.
- Connection health: a pooler that is saturated shows up as *waiters* in the application, while the database itself looks idle — so the right signal lives on the client or pooler.

### 6. Performance and design implications
- Monitoring has a cost: aggressive scraping, full query-text capture, or verbose logging can load the primary. Use sampling, short high-resolution retention (e.g. 15 s for 2 days) plus downsampled long-term data.
- Keep on-call dashboards small (5–10 panels: golden signals, saturation, replication, storage). Put the long list on drill-down pages for capacity planning and investigation.
- Alert on **rate of change and time-to-exhaustion** for resources (disk full in 6 days), on **burn rate** for SLOs, and on **symptoms** for paging.
- Correlate with change events (deploys, schema changes, config, failovers) on the same timeline.
- Trends that matter even when nothing is "alerting": deadlock frequency, WAL/redo generation rate, sequential-scan to index-scan ratio, constraint-violation rate, connection churn — each is an early signal of an application or design change.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `sys.dm_os_wait_stats`, `sys.dm_exec_requests` | Where time is spent / who is blocked (SQL Server) | First look at slowness |
| `sys.dm_io_virtual_file_stats` | File-level I/O latency | Disk suspects |
| Query Store | Plan and runtime history | Regression after change |
| `pg_stat_activity`, `pg_locks`, `pg_stat_statements` | Sessions, locks, per-query totals (PostgreSQL) | Live and cumulative diagnosis |
| `pg_stat_replication`, `sys.dm_hadr_*` | Replication state and lag | HA health |
| Slow query log / `log_min_duration_statement` | Capture slow statements | Sampled, not log everything |
| Extended Events / `performance_schema` | Low-overhead event tracing | Deadlocks, recompiles, waits |
| Exporters + Prometheus/Grafana, cloud monitors | Collect and graph | Fleet-wide visibility |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| Symptom-based paging (latency/errors) | You have no user-facing SLI yet — then start with availability + saturation |
| Static thresholds for hard limits (disk, connections) | Metrics with daily cycles — use baselines or seasonal comparison |
| Dynamic/anomaly alerts for trends | Low traffic (noisy) or when nobody tunes them |
| High scrape resolution | Storage and overhead cost; use tiered retention |
| Synthetic probe (real query) | Probe hits only a trivial table and misses lock/quorum issues |

### 9. Common mistakes
- Alerting on CPU/memory alone → noise and missed outages → page on symptoms, alert on saturation with duration.
- Averages only → tail latency invisible → record percentiles/histograms.
- No baseline → "is 70 % normal?" → keep week-over-week comparison.
- "Up" check that only opens a TCP port → replication stopped, writes blocked unnoticed → run a real read/write probe plus lag and quorum checks.
- Ignoring idle-in-transaction and idle connections → exhaustion over time → alert on age and count.
- Dashboard overload → on-call cannot act → one page per question, one runbook per alert.
- Never reviewing blind spots after incidents → same gap bites twice → each postmortem adds the missing signal.

### 10. 🎯 Interview kit
**30-second answer:** I watch the golden signals — latency percentiles, traffic, errors, saturation — plus database-specific ones: blocking/locks, connection saturation, replication lag, log/disk growth and backup freshness. I page on user-visible symptoms and time-to-exhaustion, use baselines, and keep resource and per-query detail for drill-down.

**Structure a longer answer:** clarify what "healthy" means (SLO) → signal families → triage order → alert design (symptom, duration, runbook) → cost of monitoring → close with an incident where a signal exposed the cause.

**Likely follow-ups:**
- *Latency doubled but CPU/IO flat?* → contention/locks, pool wait, network, plan change, noisy neighbour, throttling.
- *How do you forecast disk exhaustion?* → growth rate from history, alert on days-to-full.
- *Rising deadlock rate means?* → code or index change altering access order, higher concurrency, missing index.
- *How find a monitoring blind spot?* → replay recent incidents against alerts; inject failure; look for metrics with no alert.
- *Scrape interval and retention?* → 10–60 s, tiered downsampling by value of data.

**Signals interviewers look for:** symptom vs cause thinking, percentile awareness, correlation across tiers, runbooks, restraint in alerting.

### 11. Check yourself
- [ ] Name the golden signals and one database-specific example of each.
- [ ] Explain how to tell a capacity problem from a contention problem using wait stats.
- [ ] Write the query that shows the current blocking chain on your engine.
- [ ] Describe an alert you would page on and one you would only graph.
- [ ] Explain why a TCP "up" check is not a health check.
<!-- explainer:end -->

## Implementations (tools)

- [Prometheus](<../../../TOOLS/Prometheus/_index.md>)
- [Grafana](<../../../TOOLS/Grafana/_index.md>)

## Practice questions

- [Intermediate] [How do you measure and alert on deadlock frequency and what does a rising trend usually indicate?](<../../../PRACTICE/01_Databases_and_SQL/Database_Operations/Monitoring/Q-20870_How_do_you_measure_and_alert_on_deadlock_frequency_and_what_.md>)
- [Intermediate] [Describe how you monitor free disk/tablespace and forecast when it will run out.](<../../../PRACTICE/01_Databases_and_SQL/Database_Operations/Monitoring/Q-20910_Describe_how_you_monitor_free_disk_tablespace_and_forecast_w.md>)
- [Intermediate] [How do you monitor a data-integrity constraint's violation rate and what would a rising trend tell you?](<../../../PRACTICE/01_Databases_and_SQL/Database_Operations/Monitoring/Q-20951_How_do_you_monitor_a_data-integrity_constraint_s_violation_r.md>)
- [Deep] [Explain how you would monitor lock contention and identify the blocking session in a live incident.](<../../../PRACTICE/01_Databases_and_SQL/Database_Operations/Monitoring/Q-20883_Explain_how_you_would_monitor_lock_contention_and_identify_t.md>)
- [Deep] [What would make you suspect a monitoring blind spot, and how would you go about finding it?](<../../../PRACTICE/01_Databases_and_SQL/Database_Operations/Monitoring/Q-20904_What_would_make_you_suspect_a_monitoring_blind_spot_and_how_.md>)
- [Deep] [How would you monitor and interpret WAL/redo generation rate and what does a sudden increase imply?](<../../../PRACTICE/01_Databases_and_SQL/Database_Operations/Monitoring/Q-20927_How_would_you_monitor_and_interpret_WAL_redo_generation_rate.md>)
- [Deep] [How do you monitor and interpret the ratio of sequential scans to index scans on key tables?](<../../../PRACTICE/01_Databases_and_SQL/Database_Operations/Monitoring/Q-20947_How_do_you_monitor_and_interpret_the_ratio_of_sequential_sca.md>)
- [Deep] [Explain how you would monitor the health of an in-database full-text or vector index as data grows.](<../../../PRACTICE/01_Databases_and_SQL/Database_Operations/Monitoring/Q-20966_Explain_how_you_would_monitor_the_health_of_an_in-database_f.md>)

## Related concepts

- [Index Tuning in Practice](<../../Query_Performance/Indexing/Index_Tuning_in_Practice/_index.md>) *(Indexing)*
- [MySQL Performance Diagnosis](<../../../TOOLS/MySQL/MySQL_Platform/MySQL_Performance_Diagnosis.md>) *(MySQL Platform)*
- [Stored Procedure Performance and Design](<../../Database_Objects/Stored_Procedures/Stored_Procedure_Performance_and_Design.md>) *(Stored Procedures)*
- [Data Quality Monitoring](<../../../02_Data_Engineering/Data_Quality/Data_Observability/Data_Quality_Monitoring.md>) *(Data Observability)*
- [Distribution Change Alerts](<../../../02_Data_Engineering/Data_Quality/Data_Observability/Distribution_Change_Alerts.md>) *(Data Observability)*
- [Index Strategy and Governance at Scale](<../../Query_Performance/Indexing/Index_Tuning_in_Practice/Index_Strategy_and_Governance_at_Scale.md>) *(Indexing)*

## References

- *Database Reliability Engineering* — Laine Campbell, Charity Majors — Operational Visibility › Chapter 4. Operational Visibility
- *Database Reliability Engineering* — Laine Campbell, Charity Majors — Operational Visibility › Sample Database Availability Monitors
- *Database Reliability Engineering* — Laine Campbell, Charity Majors — Operational Visibility › Errors
- *Database Reliability Engineering* — Laine Campbell, Charity Majors — Operational Visibility › Internal Database Visibility
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 8: Query Optimization and Performance Tuning › Monitoring Statistics Health
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 9: Execution Plans and Query Store › Monitoring Plan Cache
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 9: Execution Plans and Query Store › Monitoring Plan Cache with DMVs
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 9: Execution Plans and Query Store › Monitoring Compilation and Recompilation Events
- *Expert Performance Indexing in SQL Server* — Jason Strate, Grant Fritchey — Page Compression › Performance Counters
- *Expert Performance Indexing in SQL Server* — Jason Strate, Grant Fritchey — Data Cleanup › Event Tracing
- *Expert Performance Indexing in SQL Server* — Jason Strate, Grant Fritchey — Performance Counters
- *Expert Performance Indexing in SQL Server* — Jason Strate, Grant Fritchey — Full Scans per Second
- *Learning Spark (2nd ed.)* — Jules S. Damji, Brooke Wenig, Tathagata Das, Denny Lee — Under the Hood of an Active Streaming Query › Monitoring an Active Query
- *SQL Server 2025 Query Performance Tuning (7th ed.)* — Grant Fritchey — FULL_BOOK_NEEDS_MANUAL_SPLIT (cont.) › Chapter 1 Query Performance Tuning
- *SQL Server 2025 Query Performance Tuning (7th ed.)* — Grant Fritchey — FULL_BOOK_NEEDS_MANUAL_SPLIT (cont.) › Chapter 3 Methods For Capturing Query Performance Metrics
- *AWS Certified Data Engineer Associate Study Guide* — Sakti Mishra et al. — Monitoring › Monitoring using system tables

