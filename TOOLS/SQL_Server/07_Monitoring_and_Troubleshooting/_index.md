---
id: "TOOLS/SQL_Server/07_Monitoring_and_Troubleshooting"
level: 3
type: "curriculum"
curriculum: "sql_server"
---

# Monitoring and Troubleshooting

**L3 Practitioner** · [↑ SQL Server](<../_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** Monitoring SQL Server means collecting the right signals continuously (performance counters, wait statistics, DMVs, Extended Events, Query Store, error log) against a known baseline. Troubleshooting means following a method: start broad (where is the server waiting?), narrow to the cause (which queries, which resource), fix, and verify against the baseline.

### 1. The big picture
Earlier blocks explained *how* SQL Server works: memory, storage, indexes, plans, locking. This block is about *observing* it. Every engine component exposes information about itself. The skill is knowing which tool answers which question, and following a repeatable method instead of guessing.

The tools, from broadest to most detailed:

| Tool | Answers | Child lesson |
|---|---|---|
| **Performance counters** (PerfMon, `sys.dm_os_performance_counters`) | Is the server under resource pressure? Trends over time | This lesson |
| **Wait statistics** | What are threads waiting on? | Wait Statistics |
| **DMVs** (dynamic management views) | What's running now, what's cached, what's missing? | Dynamic Management Views |
| **Extended Events** | Exactly what happened, event by event | Extended Events |
| **Query Store** | Which queries regressed, and with which plan? | Query Processing block |
| **Error log, Agent job history** | Failures, restarts, I/O errors, failed jobs | This lesson |

Cloud variants add their own layer: Azure Monitor metrics and alerts, Database Watcher, and elastic pool metrics.

### 2. Core concepts and vocabulary
- **Monitoring**: continuous, low-overhead collection of signals, with alerts.
- **Troubleshooting**: focused investigation of a specific problem.
- **Baseline**: recorded normal values of key metrics for your workload (by hour and weekday). Without it you can't tell whether 2,000 batch requests/sec or a 70% CPU is a problem.
- **Performance counter**: a numeric metric exposed by Windows and SQL Server (objects like `SQLServer:Buffer Manager`). Some are point-in-time values; others are cumulative and must be sampled twice and divided by the interval.
- **Wait type**: the reason a worker thread is waiting (e.g. `PAGEIOLATCH_SH` = waiting for a data page read from disk).
- **DMV / DMF**: dynamic management views and functions (`sys.dm_*`) exposing internal state. Most reset on restart.
- **Extended Events (XE)**: lightweight event tracing framework that replaces SQL Trace/Profiler.
- **Query Store**: per-database store of query texts, plans and runtime statistics over time.
- **Alert**: a notification when a condition is met: SQL Agent alerts on errors/severity/counters, or monitoring-tool alerts.
- **Methodology (USE / waits-and-queues)**: check Utilization, Saturation, Errors for each resource; or start from waits and map them to queues (resources).

### 3. How it works, step by step
**A troubleshooting method that works:**
1. **Define the symptom precisely.** What is slow (one query, one app, everything)? Since when? What changed (deployment, data volume, statistics, patch, failover)?
2. **Check health basics.** Error log (I/O errors 823/824/825, memory messages), Agent job failures, disk free space, blocking chains, AG/replication state.
3. **Look at waits** for the period (delta of `sys.dm_os_wait_stats`, or Query Store wait categories). They tell you which resource is the bottleneck.
4. **Confirm with counters.** CPU waits → `% Processor Time`, `Batch Requests/sec`, `SQL Compilations/sec`. I/O waits → `Avg. Disk sec/Read`, `Page life expectancy`. Memory → `Memory Grants Pending`.
5. **Find the queries responsible.** `sys.dm_exec_query_stats` / Query Store top consumers by CPU, reads, duration; `sys.dm_exec_requests` for what's running now.
6. **Analyze the plans** of those queries (scans, spills, bad estimates, parameter sniffing).
7. **Fix one thing at a time** and **verify** against the baseline.

**Key performance counters and what they mean:**

| Counter | Healthy / watch for | Indicates |
|---|---|---|
| `Processor: % Processor Time` | Sustained > 80% | CPU pressure (pair with `SOS_SCHEDULER_YIELD`, signal waits) |
| `SQLServer:SQL Statistics: Batch Requests/sec` | Compare to baseline | Workload volume |
| `SQL Compilations/sec`, `Re-Compilations/sec` | Compilations > ~10% of batches | Ad hoc / non-parameterized code |
| `SQLServer:Buffer Manager: Page life expectancy` | Sudden drops; compare to baseline (old "300 s" rule is outdated for large memory) | Buffer pool churn |
| `Memory Manager: Memory Grants Pending` | > 0 regularly | Queries waiting for workspace memory |
| `LogicalDisk: Avg. Disk sec/Read`, `/Write` | Data reads < ~10–20 ms, log writes < ~2–5 ms on modern storage | I/O latency |
| `General Statistics: User Connections` | Unusual growth | Connection leaks |
| `Locks: Lock Waits/sec`, `Number of Deadlocks/sec` | Rising | Concurrency problems |
| `Access Methods: Full Scans/sec`, `Page Splits/sec` | Compare to baseline | Missing indexes, fill factor issues |
| `Databases: Log Flush Wait Time` | Rising | Log I/O bottleneck |

### 4. Worked example
**Reading counters from T-SQL** (cumulative counters need two samples):
```sql
-- Point-in-time counters
SELECT RTRIM(object_name) AS object, RTRIM(counter_name) AS counter, cntr_value
FROM sys.dm_os_performance_counters
WHERE counter_name IN (N'Page life expectancy', N'Memory Grants Pending', N'User Connections')
  AND (instance_name = N'' OR object_name LIKE N'%Buffer Manager%');

-- Per-second counters: sample twice, 10 seconds apart
DECLARE @t1 TABLE (counter_name nvarchar(128), v bigint);
INSERT @t1
SELECT RTRIM(counter_name), cntr_value FROM sys.dm_os_performance_counters
WHERE counter_name IN (N'Batch Requests/sec', N'SQL Compilations/sec', N'SQL Re-Compilations/sec');

WAITFOR DELAY '00:00:10';

SELECT RTRIM(p.counter_name) AS counter, (p.cntr_value - t.v) / 10 AS per_second
FROM sys.dm_os_performance_counters AS p
JOIN @t1 AS t ON t.counter_name = RTRIM(p.counter_name);
```
**Reading it:** 1,500 batches/sec with 600 compilations/sec means 40% of batches compile: look for non-parameterized ad hoc SQL (consider `optimize for ad hoc workloads`, forced parameterization, or fixing the app). PLE that drops from 20,000 to 500 every night at 02:00 points to a big scan job (index maintenance, a report), not necessarily too little memory.

**A quick "what's happening right now" check:**
```sql
SELECT r.session_id, r.status, r.wait_type, r.wait_time, r.blocking_session_id,
       r.cpu_time, r.logical_reads, r.total_elapsed_time,
       DB_NAME(r.database_id) AS db, t.text
FROM sys.dm_exec_requests AS r
CROSS APPLY sys.dm_exec_sql_text(r.sql_handle) AS t
WHERE r.session_id <> @@SPID AND r.session_id > 50
ORDER BY r.total_elapsed_time DESC;
```

### 5. Under the hood
- **Counters inside SQL Server.** `sys.dm_os_performance_counters` exposes the SQL Server counters (not Windows CPU/disk counters). `cntr_type` tells you how to read it: 65792 = point-in-time value; 272696576 = cumulative per-second counter (needs delta); 537003264 + base = ratio (divide by the matching `base` counter).
- **Reset behavior.** DMVs and counters reset at service restart (and wait stats can be cleared with `DBCC SQLPERF('sys.dm_os_wait_stats', CLEAR)`). Persist snapshots to tables if you need history.
- **Overhead.** Querying DMVs is cheap; Extended Events sessions with targeted filters are cheap; Profiler/SQL Trace and capturing actual plans for every query (`query_post_execution_showplan`) are expensive.
- **Default trace and system_health.** SQL Server runs a `system_health` XE session by default: deadlocks, severe errors, long latch/lock waits, memory issues. Check it first after an incident.
- **Monitoring in pipelines and jobs.** SQL Agent job history (`msdb.dbo.sysjobhistory`), job step logs, and alerts on failure. ETL pipelines should log row counts, durations and errors to a table so trends are visible.

### 6. Performance and design implications
- Collect a baseline *before* problems: counters every 1–5 minutes, wait-stat deltas, top queries from Query Store, file latency.
- Alert on symptoms that matter: failed backups/jobs, severity ≥ 17 errors, errors 823/824/825, long blocking, AG not synchronizing, low disk space, log full (9002).
- Don't chase single numbers (PLE < 300, CPU 90% for a few seconds). Trends and correlations matter.
- Prefer Query Store for "what got slower since last week" questions; DMV plan-cache data is lost when plans are evicted.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `sys.dm_os_performance_counters` | SQL Server counters from T-SQL | Scripts, baselines |
| PerfMon / Data Collector Sets | Windows + SQL counters over time | Baselines, OS-level issues |
| `sys.dm_os_wait_stats` | Cumulative waits | Bottleneck identification |
| `sys.dm_exec_requests` / `sessions` | Current activity | Live troubleshooting |
| `sp_who2`, `sp_WhoIsActive` (community) | Activity overview | Quick look |
| `system_health` XE session | Built-in diagnostics | After incidents |
| Query Store reports | Regressed / top queries | Plan regressions |
| `xp_readerrorlog` / `sys.sp_readerrorlog` | Read error log | Health check |
| SQL Agent alerts (severity 17–25, 823–825) | Notify on errors | Always configure |
| Azure Monitor metrics/alerts | Cloud metrics | Azure SQL / MI |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| Counters for trends and capacity | They don't name the query responsible |
| Waits to find the bottleneck | Benign waits (e.g. `SLEEP_TASK`, `BROKER_*`) inflate totals → filter them |
| DMVs for current state | History is lost on restart/eviction |
| XE for precise event capture | Unfiltered sessions on busy servers create overhead |
| Query Store for regressions | Needs to be enabled and sized; capture mode AUTO |
| Third-party monitoring tools | Cost; still need to understand the raw signals |

### 9. Common mistakes
- No baseline → can't tell what's abnormal → collect one now.
- Reading a cumulative counter once → meaningless number → take deltas.
- Using outdated thresholds (PLE 300) → wrong conclusions → compare to your baseline and memory size.
- Running Profiler on production → added load → Extended Events.
- Changing several things at once → can't tell what helped → one change, then measure.
- Clearing wait stats without saving them → lost evidence → snapshot first.

### 10. 🎯 Interview kit
**30-second answer:** I monitor SQL Server with a baseline of performance counters, wait-statistics deltas, Query Store and the system_health session, plus alerts on errors, failed jobs and blocking. When something's slow I start broad (waits show which resource is the bottleneck), confirm with counters, find the top queries in DMVs or Query Store, analyze their plans, change one thing and verify against the baseline.

**Structure a longer answer:** tools and what each answers → the method (symptom → health → waits → counters → queries → plans → fix → verify) → a real incident.

**Likely follow-ups:**
- *Which counters do you watch?* → CPU, batch requests, compilations, PLE trend, grants pending, disk latency, log flush waits.
- *Why baseline?* → normal values depend on the workload.
- *DMVs vs. Query Store?* → current/volatile vs. persisted history per database.
- *Profiler vs. XE?* → XE is lighter and more flexible; Profiler is deprecated.

**Signals interviewers look for:** a method rather than random checks; correct counter interpretation; awareness of baselines and tool overhead.

### 11. Check yourself
- [ ] Capture Batch Requests/sec and Compilations/sec as deltas and compute the compile ratio.
- [ ] Write down your method for "the app is slow since this morning".
- [ ] Read the `system_health` session for recent deadlocks.
- [ ] Configure Agent alerts for severities 17–25 and errors 823–825.
- [ ] Explain why a PLE drop at night might not mean memory pressure.
<!-- explainer:end -->

## In this section (study in order)

1. [Dynamic Management Views](<Dynamic_Management_Views/_index.md>) (L4 Advanced)
2. [Extended Events](<Extended_Events/_index.md>) (L4 Advanced)
3. [Wait Statistics](<Wait_Statistics/_index.md>) (L4 Advanced)

## References

- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 15: Case Studies and the Future of T-SQL › Monitoring and Validation
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 12: Azure SQL and Synapse Integration › Monitoring and Performance Management
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 11: Automation with SSIS, SQL Agent, and PowerShell › Monitoring and Recovery
- *Expert Performance Indexing in SQL Server* — Jason Strate, Grant Fritchey — Page Compression › Performance Counters
- *Expert Performance Indexing in SQL Server* — Jason Strate, Grant Fritchey — Performance Counters
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 11: Automation with SSIS, SQL Agent, and PowerShell › Monitoring and Maintenance
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 12: Azure SQL and Synapse Integration › Monitoring with Azure Metrics and Alerts
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 11: Automation with SSIS, SQL Agent, and PowerShell › Monitoring and Logging in Pipelines
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 11: Automation with SSIS, SQL Agent, and PowerShell › Job Monitoring and Logging
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 12: Azure SQL and Synapse Integration › Monitoring and Alerts for Security and Cost

