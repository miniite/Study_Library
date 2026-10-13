---
id: "TOOLS/SQL_Server/07_Monitoring_and_Troubleshooting/Wait_Statistics"
level: 4
type: "curriculum"
curriculum: "sql_server"
---

# Wait Statistics

**L4 Advanced** · [↑ Monitoring and Troubleshooting](<../_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** Wait statistics record *what each worker thread waited for* and for how long; read as a proportion of elapsed time (after removing idle "benign" waits) they point at the resource that is the bottleneck: CPU, disk, memory, locks, or parallelism.

### 1. The big picture
SQL Server runs requests on worker threads. A thread is either **running** on a CPU, **runnable** (ready, queued for a CPU), or **suspended** (waiting for something it needs: a data page from disk, a lock, a memory grant, log flush, another parallel thread). Every time a thread is suspended the engine records the reason (the *wait type*) and the duration. Summed up, this is `sys.dm_os_wait_stats`.

The result is a very good first-pass diagnostic: instead of guessing, "the server spent 55% of its wait time on `PAGEIOLATCH_SH`" tells you queries are mostly waiting for data pages to be read from storage. This is the "waits and queues" approach and sits right after the baseline/counter check in the troubleshooting method.

Waits are *general health indicators for the instance*. They don't point at a specific query or index by themselves. They tell you *which resource family to investigate*; then you use plans, Extended Events or Query Store to find the statements.

### 2. Core concepts and vocabulary
- **Wait type** — the named reason for a suspension (`PAGEIOLATCH_SH`, `LCK_M_X`, `CXPACKET`, `WRITELOG`, `SOS_SCHEDULER_YIELD`…).
- **Resource wait** — time waiting for a resource (disk, lock, memory…).
- **Signal wait** — time between the resource becoming available and the thread actually getting back on a CPU. `signal_wait_time_ms` is the runnable-queue part; `wait_time_ms − signal_wait_time_ms` is the pure resource wait. A high signal fraction suggests CPU pressure.
- **`waiting_tasks_count`** — how many times a wait of this type happened; `wait_time_ms / waiting_tasks_count` is the average wait.
- **`max_wait_time_ms`** — the longest single wait seen.
- **Benign / ignorable waits** — waits that tick up regardless of load (background threads sleeping or idling: `LAZYWRITER_SLEEP`, `SLEEP_TASK`, `XE_TIMER_EVENT`, `CHECKPOINT_QUEUE`, `REQUEST_FOR_DEADLOCK_SEARCH`, `BROKER_*`, `WAITFOR`, etc.). Filter them out.
- **Cumulative since restart** — `sys.dm_os_wait_stats` never resets until restart or an explicit clear, so you must analyze *deltas* between two snapshots.
- **Wait percentage / pressure** — a wait's share of total (non-benign) wait time, or of elapsed wall-clock time.

### 3. How it works, step by step
1. **Snapshot** `sys.dm_os_wait_stats` into a table with a timestamp (e.g. every 5–15 minutes).
2. **Compute deltas** between consecutive snapshots into a history table (current minus previous). If the later value is lower, the server restarted: discard that pair.
3. **Drop benign waits** so they don't dominate.
4. **Rank** wait types by delta `wait_time_ms`; compute each as a percentage of total wait time and as a share of elapsed time (`wait_time_ms / elapsed_ms`).
5. **Check the signal ratio** (`signal_wait_time_ms / wait_time_ms`) for CPU pressure.
6. **Map the top waits** to the resource (table below), confirm with counters (e.g. `Avg. Disk sec/Read`, `Page life expectancy`, `Lock Wait Time (ms)`), then drill down to queries.
7. **Fix, re-measure** against the baseline.

A key point from the book: you keep *all* wait types in the history, not only index-related ones, because a wait is only meaningful *relative to the others*. If `CXPACKET` is the smallest slice of wait time, tuning parallelism queries wouldn't pay off.

**Common waits and where to look**

| Wait type | Meaning | Typical direction |
|---|---|---|
| `PAGEIOLATCH_SH/EX/UP/…` | Waiting for a data page to be read from disk into the buffer pool | Missing/poor indexes (scans), too little memory, slow storage |
| `LCK_M_*` | Waiting to acquire a lock (`S`, `X`, `U`, `IX`, `SCH_M`, range locks…) | Blocking, long transactions, scan-heavy queries, isolation level |
| `CXPACKET` / `CXCONSUMER` | Parallel threads synchronizing; some threads wait on slower ones | Skewed parallelism, scans, poor estimates; MAXDOP/cost threshold |
| `IO_COMPLETION` | Synchronous non-data-page I/O (sorts, spills) | Memory grants, spills to `tempdb` |
| `WRITELOG` | Waiting for transaction log flush | Slow log disk, many tiny commits |
| `SOS_SCHEDULER_YIELD` | Thread used its quantum and yielded the CPU | CPU pressure |
| `RESOURCE_SEMAPHORE` | Waiting for a query memory grant | Memory pressure, huge grants |
| `ASYNC_NETWORK_IO` | Client is slow consuming results | App-side, not the engine |

### 4. Worked example
A minimal snapshot-and-delta setup (the book's pattern), then a ranking query:

```sql
CREATE TABLE dbo.wait_stats_snapshot (
    snapshot_id INT IDENTITY PRIMARY KEY,
    create_date DATETIME NOT NULL DEFAULT GETDATE(),
    wait_type   NVARCHAR(60) NOT NULL,
    waiting_tasks_count BIGINT, wait_time_ms BIGINT,
    max_wait_time_ms BIGINT,    signal_wait_time_ms BIGINT);

-- run on a schedule
INSERT dbo.wait_stats_snapshot (wait_type, waiting_tasks_count, wait_time_ms,
                                max_wait_time_ms, signal_wait_time_ms)
SELECT wait_type, waiting_tasks_count, wait_time_ms,
       max_wait_time_ms, signal_wait_time_ms
FROM sys.dm_os_wait_stats
WHERE waiting_tasks_count > 0;
```

Quick ad-hoc version without tables (sample twice, 60 s apart):

```sql
SELECT wait_type, wait_time_ms, signal_wait_time_ms, waiting_tasks_count
INTO #w1 FROM sys.dm_os_wait_stats;
WAITFOR DELAY '00:01:00';
SELECT TOP (10)
       w2.wait_type,
       w2.wait_time_ms - w1.wait_time_ms               AS wait_ms,
       w2.signal_wait_time_ms - w1.signal_wait_time_ms AS signal_ms,
       w2.waiting_tasks_count - w1.waiting_tasks_count AS waits,
       100.0 * (w2.wait_time_ms - w1.wait_time_ms) / 60000 AS pct_of_elapsed
FROM sys.dm_os_wait_stats w2
JOIN #w1 w1 ON w1.wait_type = w2.wait_type
WHERE w2.wait_type NOT IN (N'LAZYWRITER_SLEEP', N'SLEEP_TASK', N'XE_TIMER_EVENT',
      N'CHECKPOINT_QUEUE', N'REQUEST_FOR_DEADLOCK_SEARCH', N'WAITFOR',
      N'LOGMGR_QUEUE', N'DIRTY_PAGE_POLL', N'BROKER_TO_FLUSH', N'SQLTRACE_BUFFER_FLUSH')
ORDER BY wait_ms DESC;
```

How to read the result: `pct_of_elapsed` can exceed 100% because many threads wait concurrently, so it's a *pressure* indicator, not a share of a single timeline. Suppose the top rows are `PAGEIOLATCH_SH` 380%, `LCK_M_X` 60%, `SOS_SCHEDULER_YIELD` 25% with signal time about 20% of its total. Conclusion: the dominant problem is reading pages from disk; locks are secondary; CPU is fine. Next steps: check `Avg. Disk sec/Read` and PLE; list top queries by physical reads; look for scans on large tables.

### 5. Under the hood
- **Scheduler model.** Each logical CPU has a scheduler with a runnable queue, a waiter list and one running worker. A worker blocks → moves to the waiter list with a wait type and timestamp; when the resource is signaled it moves to the *runnable queue*; the time in that queue is the **signal wait**. So total wait = resource wait + signal wait.
- **Where counters live.** `sys.dm_os_wait_stats` is instance-wide and cumulative; `sys.dm_os_waiting_tasks` shows who is waiting *right now* (and `blocking_session_id`); `sys.dm_exec_session_wait_stats` (2016+) has per-session waits; Query Store keeps wait categories per plan (2017+).
- **Latches versus locks.** `PAGEIOLATCH_*` are *buffer latches* held while a page is being read from disk: latches are lightweight, internal physical-consistency protection; locks (`LCK_M_*`) enforce transactional isolation. Both appear as waits but call for different fixes. The `_SH/_EX/_UP/_KP/_DT` suffix is the mode (shared, exclusive, update, keep, destroy).
- **`LCK_M_*` suffix** encodes lock mode: `S`, `X`, `U`, intent modes (`IS`, `IX`, `IU`, `SIX`, `SIU`, `UIX`), schema locks (`SCH_S`, `SCH_M`), bulk update (`BU`) and key-range modes (`RS_S`, `RIn_X`, `RX_X`…) used under serializable.
- **`CXPACKET` has two causes** per the book: (1) a parallel thread is blocked by something else (more common), (2) one thread has more rows to process than its siblings, so the others wait. Cause (2) is addressable by better indexing and statistics. Newer versions split the benign "consumer" part into `CXCONSUMER`.
- **Wait IDs vs. names.** Extended Events refers to waits by numeric map values, which differ between versions (e.g. 187 on 2008, 191 on 2012); look them up in `sys.dm_xe_map_values` instead of hard-coding.

### 6. Performance and design implications
- **Measure over time, not once.** A single since-restart total is dominated by history; use deltas and compare against a baseline for the same hour/weekday.
- **Rank by pressure.** Top 3–5 non-benign waits normally explain most of the time lost.
- **Confirm with a second signal.** `PAGEIOLATCH` + high `Avg. Disk sec/Read` → storage; + low PLE → memory; + many full scans → indexing/query.
- **Index-related waits** (the focus of the index book): `PAGEIOLATCH_*` (reading too many pages → fewer scans, narrower covering indexes), `LCK_M_*` (queries touching too many rows hold locks longer), `CXPACKET` (parallel plans caused by scanning too much), `IO_COMPLETION`.
- **Tools.** Third-party monitors usually collect waits already; check them before building your own history.
- **Cost.** Reading `dm_os_wait_stats` is cheap; snapshotting every few minutes is negligible.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `sys.dm_os_wait_stats` | Cumulative waits since start/clear | Instance-level baseline and top-waits |
| `sys.dm_os_waiting_tasks` | Current waiting tasks and their blockers | Live blocking investigations |
| `sys.dm_exec_session_wait_stats` | Waits per session | Isolating one workload |
| `DBCC SQLPERF('sys.dm_os_wait_stats', CLEAR)` | Resets the wait counters | Start a clean measurement window (use sparingly; it destroys history) |
| `sys.dm_os_performance_counters` | Counters to confirm waits | Cross-check `Lock Wait Time`, PLE |
| Extended Events `sqlos.wait_info` | Per-wait events with statement context | Find which statements generate a specific wait |
| `max degree of parallelism`, `cost threshold for parallelism` | Parallelism controls | Dealing with `CXPACKET` pressure after tuning |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| Snapshot + delta history | Looking at raw cumulative totals after months of uptime |
| Instance-level waits as the *first* triage step | You already know the specific query; go straight to its plan |
| Per-session or Query Store waits for workload-level attribution | Assuming instance waits equal your app's waits (a background job may dominate) |
| Raising memory when `PAGEIOLATCH` + low PLE | Memory won't fix scans caused by missing indexes or non-SARGable predicates |
| Tuning the queries behind `CXPACKET` | Blindly lowering MAXDOP to hide the symptom |

### 9. Common mistakes
- **Ignoring benign waits** → they top the list and mislead → filter them out first.
- **Using cumulative totals** → old history swamps current behavior → delta between snapshots.
- **Treating a wait as a root cause** → e.g. "CXPACKET is bad, set MAXDOP 1" → you may just trade it for CPU-bound slow queries → find the queries and indexes causing the skew.
- **Blaming disks for all `PAGEIOLATCH`** → may be scans and too little memory → check PLE, full scans/sec and the plans.
- **Forgetting the signal ratio** → missing CPU pressure hidden among resource waits → compare signal to resource time.
- **Clearing the stats routinely** → you lose the baseline → snapshot to history instead.

### 10. 🎯 Interview kit
**30-second answer:** Wait statistics show where worker threads spend time suspended. I snapshot `sys.dm_os_wait_stats`, take deltas, filter out benign waits, rank by share of time and map the top ones to resources (PAGEIOLATCH → disk/memory, LCK_M → blocking, CXPACKET → parallelism, WRITELOG → log, SOS_SCHEDULER_YIELD → CPU). Then I confirm with counters and find the queries.

**Structure a longer answer:** definition of wait and signal wait → why deltas and benign filtering → how you rank → top-waits-to-resource mapping → confirm with a second signal → find responsible queries (XE, Query Store, plan cache) → fix and re-measure.

**Likely follow-ups:**
- *Signal vs. resource wait?* → signal is time in runnable queue after resource is free; high signal ⇒ CPU pressure.
- *What does PAGEIOLATCH_SH mean and what do you do?* → reading a page from disk; check disk latency, PLE, scans; add indexes or memory.
- *Is CXPACKET always bad?* → no; parallel threads wait for each other normally; it's a problem when it dominates, usually from skew or blocked threads.
- *Why not look at the biggest wait absolutely?* → relative size and elapsed time matter; benign waits dominate otherwise.
- *How would you find which query causes LCK_M_X?* → `dm_os_waiting_tasks` / blocking chain live; XE lock count by `query_hash`; Query Store waits.

**Signals interviewers look for:** you analyze deltas, filter benign waits, correlate with counters, avoid knee-jerk configuration changes, and can drill from instance to statement.

### 11. Check yourself
- [ ] Define signal wait and explain what a high signal ratio suggests.
- [ ] Explain why `sys.dm_os_wait_stats` must be sampled twice.
- [ ] Map `PAGEIOLATCH_SH`, `LCK_M_X`, `CXPACKET`, `WRITELOG` and `SOS_SCHEDULER_YIELD` to likely causes.
- [ ] Write a query that returns the top 10 non-benign waits over a 60-second window.
- [ ] Describe the two scenarios that cause `CXPACKET` and which one indexing can fix.
- [ ] Explain how to find the statements behind one wait type with Extended Events.
<!-- explainer:end -->

## References

- *Expert Performance Indexing in SQL Server* — Jason Strate, Grant Fritchey — PAGEIOLATCH
- *Expert Performance Indexing in SQL Server* — Jason Strate, Grant Fritchey — Wait Statistics
- *Expert Performance Indexing in SQL Server* — Jason Strate, Grant Fritchey — LCKM
- *Expert Performance Indexing in SQL Server* — Jason Strate, Grant Fritchey — CXPACKET
- *Expert Performance Indexing in SQL Server* — Jason Strate, Grant Fritchey — Buffer Allocation

