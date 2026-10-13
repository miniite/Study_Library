---
id: "TOOLS/SQL_Server/07_Monitoring_and_Troubleshooting/Extended_Events"
level: 4
type: "curriculum"
curriculum: "sql_server"
---

# Extended Events

**L4 Advanced** · [↑ Monitoring and Troubleshooting](<../_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** Extended Events (XE) is SQL Server's built-in, lightweight event-tracing engine: you pick *events*, enrich them with *actions*, narrow them with *predicates*, and send them to *targets*; it replaces SQL Trace/Profiler for query capture, wait analysis, plan capture and diagnostics.

### 1. The big picture
Counters, DMVs and wait statistics tell you *that* something is wrong and roughly *where*. They are aggregates. They don't tell you which exact statement, with which parameters, from which application, took 4 seconds at 14:02. To answer that you need **event tracing**: recording individual occurrences of things the engine does.

SQL Server has had two tracing technologies:

- **SQL Trace / Profiler**: the legacy tool. Deprecated, heavier, fixed column set.
- **Extended Events**: introduced in SQL Server 2008, GUI in SSMS from 2012. More functional, far lower overhead, and the only one that exposes the newer engine internals (wait info, plan capture, deadlock graphs, and so on).

Both books used here make the same recommendation: use Extended Events and avoid SQL Trace for anything new. For compliance logging (who did what, tamper-evident), the right feature is **SQL Server Audit**, not XE. XE is for *diagnostics*.

XE sits at the "most detailed" end of the monitoring stack: you turn to it after counters and waits narrowed the problem and you need event-level proof. Query Store covers a lot of "which query regressed" ground; XE is what you use when you need things Query Store doesn't keep, such as per-execution parameter values, waits per statement, or the sequence of compiles and recompiles.

### 2. Core concepts and vocabulary
- **Event** — a point in the engine's code that can fire and be recorded, e.g. `sql_batch_completed`, `rpc_completed`, `wait_info`, `auto_stats`, `query_post_execution_showplan`.
- **Package** — the module that defines events, actions and targets (`sqlserver`, `sqlos`, `package0`). You'll see names like `sqlserver.rpc_completed`.
- **Channel** — event category: Admin, Operational, Analytic, Debug. The Debug channel can be dangerous; leave it alone unless directed.
- **Event fields (data columns)** — the payload that comes with an event, e.g. `duration`, `cpu_time`, `logical_reads`, `statement`. Some are always present; some are *optional* and can be switched off to cut overhead.
- **Action (global field)** — extra data you attach to any event at the moment it fires: `sql_text`, `database_name`, `client_app_name`, `client_hostname`, `username`, `session_id`, `query_hash`, `query_plan_hash`, `tsql_stack`, `plan_handle`.
- **Predicate (filter)** — a condition evaluated *before* the event is recorded, e.g. `database_name = 'AdventureWorks' AND duration > 1000000`.
- **Target** — where captured events go: `event_file` (files on disk), `ring_buffer` (memory), `histogram` (counters grouped by a field), `event_counter`, `pair_matching`.
- **Session** — the named definition that bundles events + actions + predicates + targets. It's a *server-level* object (a database-scoped variant exists in Azure SQL Database).
- **Causality tracking** — a session option that stamps related events with GUIDs and sequence numbers so you can reconstruct the order in which related events happened.
- **Live Data window** — the SSMS viewer that streams a running session or reads `.xel` files.

### 3. How it works, step by step
1. **Choose events.** Common starting pair: `rpc_completed` (stored-procedure/parameterized calls) and `sql_batch_completed` (ad-hoc batches). These are the XE equivalents of Trace's `RPC:Completed` and `SQL:BatchCompleted`. Their payload includes `duration` (in **microseconds**), `cpu_time`, `logical_reads`, `physical_reads`, `writes`, `row_count`, and the statement/batch text.
2. **Add actions** to enrich: who and where (`client_app_name`, `client_hostname`, `username`, `database_name`), and the query "fingerprints" `query_hash` and `query_plan_hash`, which let you group different literal values of the same query.
3. **Add predicates.** This is the single most important overhead control. Filter on database, minimum duration, or application. Predicates are evaluated early, and the order matters: put the cheapest, most selective ones first.
4. **Choose targets.** Prefer `event_file`, and put it on a drive that does *not* hold data or log files. The `ring_buffer` lives in memory that otherwise belongs to the OS and SQL Server, and is not recommended in production beyond short tests. Use `histogram` when you only want counts (e.g. count `rpc_completed` grouped by `object_name` to find the most-called procedures; default 256 buckets, more buckets means more load).
5. **Set session options**: memory, event-loss policy, whether to start with the instance (`STARTUP_STATE = ON`), and causality tracking if you need to correlate related events (it adds overhead).
6. **Create and start** the session in T-SQL or through SSMS (Management › Extended Events › Sessions › New Session; the New Session window is more capable than the wizard, and is the one you use to edit sessions).
7. **Consume the data.** Output is XML (`event_data`). Read it with Live Data (right-click session › Watch Live Data), open the `.xel` file in SSMS, query with `sys.fn_xe_file_target_read_file` plus XQuery, or use the dbatools PowerShell module.
8. **Stop and drop** the session when you're done. Sessions are cheap but not free.

SSMS also ships **templates** worth knowing: *Count Query Locks* (histogram of locks per `query_hash`), *TSQL* (Profiler-equivalent), *Query Batch Sampling* and *Query Wait Statistics* (sample about 20% of sessions to cap overhead), and *Query Detail Tracking* (statements, batches and RPCs, with system queries filtered out).

### 4. Worked example
Capture slow statements in one database, with enough context to act on them:

```sql
CREATE EVENT SESSION [SlowQueries] ON SERVER
ADD EVENT sqlserver.rpc_completed
(
    ACTION (sqlserver.client_app_name, sqlserver.client_hostname,
            sqlserver.database_name,  sqlserver.username,
            sqlserver.query_hash,     sqlserver.query_plan_hash,
            sqlserver.sql_text)
    WHERE sqlserver.database_name = N'AdventureWorks'
      AND duration > 1000000                 -- microseconds => > 1 second
),
ADD EVENT sqlserver.sql_batch_completed
(
    ACTION (sqlserver.client_app_name, sqlserver.database_name,
            sqlserver.query_hash,      sqlserver.sql_text)
    WHERE sqlserver.database_name = N'AdventureWorks'
      AND duration > 1000000
)
ADD TARGET package0.event_file
(   SET filename = N'D:\XE\SlowQueries.xel',
        max_file_size = 100, max_rollover_files = 5 )
WITH (MAX_MEMORY = 4096 KB,
      EVENT_RETENTION_MODE = ALLOW_SINGLE_EVENT_LOSS,
      STARTUP_STATE = OFF);
GO
ALTER EVENT SESSION [SlowQueries] ON SERVER STATE = START;
```

Read it back:

```sql
SELECT
    x.event_data.value('(event/@name)[1]','sysname')                         AS event_name,
    x.event_data.value('(event/data[@name="duration"]/value)[1]','bigint')/1000 AS duration_ms,
    x.event_data.value('(event/data[@name="logical_reads"]/value)[1]','bigint') AS logical_reads,
    x.event_data.value('(event/action[@name="query_hash"]/value)[1]','varchar(30)') AS query_hash
FROM (SELECT CAST(event_data AS xml) AS event_data
      FROM sys.fn_xe_file_target_read_file(N'D:\XE\SlowQueries*.xel',NULL,NULL,NULL)) x;
```

What to look at: group by `query_hash` and sum `duration`/`logical_reads` to find the *families* of statements that cost most; a statement taking 1.1 s once matters less than one taking 0.3 s a million times (lower the threshold or use a histogram for that). Then take the `query_plan_hash` and look the plan up in the plan cache or Query Store.

A second pattern is the **histogram**, for "which statements cause this wait":

```sql
CREATE EVENT SESSION [ex_cxpacket] ON SERVER
ADD EVENT sqlos.wait_info
(
    ACTION (sqlserver.plan_handle, sqlserver.tsql_stack)
    WHERE wait_type = 191 AND sqlserver.is_system = 0   -- numeric wait-type id varies by version
)
ADD TARGET package0.histogram
(   SET filtering_event_name = N'sqlos.wait_info',
        slots = 2048,
        source = N'sqlserver.tsql_stack', source_type = 1 )
WITH (STARTUP_STATE = ON);
```

It groups parallelism waits by the T-SQL call stack, so the output is a small table of "stack → count" rather than millions of events.

### 5. Under the hood
- **Synchronous firing, asynchronous dispatch.** When code reaches an event point the engine evaluates the predicate in the firing thread; only matching events are packaged and buffered, then dispatched to targets *asynchronously* (the session's memory buffers are flushed on a timer or when full). That is why XE is cheap compared with Trace: non-matching events cost very little, and no rows are written synchronously.
- **Event loss.** If targets can't keep up, buffers fill and events are dropped according to the retention mode: `NO_EVENT_LOSS` (can *block* the workload; avoid on production), `ALLOW_SINGLE_EVENT_LOSS` (default), `ALLOW_MULTIPLE_EVENT_LOSS`. Chosen deliberately, this is your trade-off between completeness and impact.
- **Event data is XML.** That is flexible but awkward; `event_data` for the file target is an XML fragment per event. The target-specific XML for ring buffer and histogram targets is read from `sys.dm_xe_session_targets`.
- **Metadata DMVs.** `sys.dm_xe_objects` / `sys.dm_xe_object_columns` describe available events and fields; `sys.dm_xe_packages` the packages; `sys.server_event_sessions` the *defined* sessions; `sys.dm_xe_sessions` the *running* ones.
- **Always-on sessions.** `system_health` ships enabled and records deadlocks, severe errors, long waits and similar; check it first when something odd happened and you have no custom session running.
- **Plan capture events** differ in cost and what they return: `query_post_compilation_showplan` (after compile), `query_pre_execution_showplan`, `query_post_execution_showplan` (plan + runtime metrics, expensive), and `query_post_execution_plan_profile` (SQL Server 2017+, uses lightweight profiling; prefer it when you want runtime metrics).

### 6. Performance and design implications
- **Overhead is dominated by what you choose.** Rough ranking from cheap to expensive: completed-event sessions with predicates < wait_info on all waits < statement-level events < plan-capture events with no filter. Plan capture without tight predicates can seriously hurt a busy server.
- **Always filter.** Database, duration, application, `session_id`, `query_hash`. Prefer sampling templates when you just need a representative picture.
- **Disk placement.** Write `.xel` files away from data and log volumes, and cap size with `max_file_size`/`max_rollover_files` so a forgotten session can't fill a drive.
- **Optional fields.** Turning off `statement` on `rpc_completed` reduces overhead but loses parameter values; decide consciously.
- **Causality tracking** is great for compile/recompile investigations but adds cost; enable it only for the duration of the investigation.
- **Don't parse XML with XQuery against large files on production.** Copy the `.xel` away, or open it in SSMS / dbatools.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `CREATE EVENT SESSION … ON SERVER` | Defines a session (events, actions, predicates, targets, options) | Start of any capture |
| `ALTER EVENT SESSION … STATE = START / STOP` | Starts or stops a session | Begin/end an investigation |
| `DROP EVENT SESSION … ON SERVER` | Removes the definition | Clean up |
| `STARTUP_STATE = ON` | Session starts with the instance | Long-lived diagnostics such as a wait-by-stack histogram |
| `sys.fn_xe_file_target_read_file` | Reads `.xel` files as rows of XML | Offline analysis |
| `sys.dm_xe_session_targets` | Returns live target data (ring buffer, histogram) | Read in-memory targets |
| `sys.server_event_sessions` / `sys.dm_xe_sessions` | Defined vs. running sessions | Inventory and audit |
| `sys.dm_xe_objects`, `sys.dm_xe_object_columns` | Catalog of events, actions, fields | Discovering what exists |
| Watch Live Data (SSMS) | Streams events as they arrive | Interactive debugging |
| `system_health` session | Built-in always-on capture (deadlocks, errors) | First stop for "what happened?" |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| `event_file` target: durable, bigger volumes, survives restart | Slow or shared disks; unbounded file growth |
| `ring_buffer`: quick tests, no file path needed | Production: it consumes memory and wraps, losing older events |
| `histogram`: you want *counts by X* with minimal volume | You need individual events or parameter values |
| Extended Events for diagnostics and tuning | You need tamper-evident compliance logging, which is SQL Server Audit |
| Query Store for "which plan regressed over days" | You need per-execution detail or wait/compile sequences, which is XE |
| Sampling templates (about 20% of sessions) | You must see *every* occurrence (rare errors, deadlocks) |
| Causality tracking for compile/recompile chains | Always-on, high-volume production captures |

### 9. Common mistakes
- **Capturing without a predicate** → huge volume, measurable CPU and I/O hit → filter by database and duration, or sample.
- **Leaving sessions running** → files fill the drive, or ring buffers keep consuming memory → set rollover limits and drop sessions when done.
- **Misreading `duration`** → in XE it is **microseconds**, while `cpu_time` and DMV columns can be in other units; a "1000" filter is 1 ms, not 1 s → check the field description.
- **Using the Debug channel** → unexpected impact and noise → leave it alone.
- **Using SQL Trace/Profiler against production** → higher overhead and deprecated → use XE.
- **Using XE as an audit log** → no tamper evidence, events can be lost under pressure → use SQL Server Audit for compliance.
- **Relying on `NO_EVENT_LOSS` on a busy server** → sessions can stall the workload → accept single-event loss unless you truly can't.
- **Capturing plans for everything** → very high overhead → capture plans only for filtered queries, and prefer the lightweight plan-profile event.

### 10. 🎯 Interview kit
**30-second answer:** Extended Events is SQL Server's lightweight event tracing framework. You define a session with events, optional actions for extra context, predicates to filter early, and targets like files or histograms. It replaces SQL Trace/Profiler, is much cheaper, and is what you use to capture slow queries, waits per statement, plans, deadlocks and compile activity.

**Structure a longer answer:** clarify what you need to observe → pick the narrowest event → add actions and predicates → choose a target (file for detail, histogram for counts) → analyze by grouping on `query_hash` → stop and clean up. Close with a scenario: "Parallelism waits were high; I ran a `wait_info` histogram grouped by T-SQL stack for an hour, found two statements missing a covering index, fixed them and confirmed waits dropped."

**Likely follow-ups:**
- *Why is XE lighter than Trace?* → Predicates evaluated before event creation, asynchronous buffered dispatch, no per-event row writes, configurable loss.
- *Which target would you use for finding the most frequently executed procedures?* → Histogram on `object_name` of `rpc_completed`.
- *How do you see the query plan with XE?* → plan-capture events (prefer `query_post_execution_plan_profile`), view in the Query Plan tab, with filters.
- *XE vs. SQL Server Audit?* → XE diagnostics and tuning; Audit compliance and tamper-evident logs.
- *What is causality tracking?* → GUID plus sequence stamps tying related events together; useful for recompile analysis; adds overhead.

**Signals interviewers look for:** you filter by default and know the overhead levers; you can name `system_health`; you know `duration` units; you differentiate XE, Query Store, Audit, and Trace; you can describe a concrete investigation.

### 11. Check yourself
- [ ] Name the four building blocks of a session and what each does.
- [ ] Write a session that captures `rpc_completed` over 2 seconds in one database to an `event_file`.
- [ ] Explain why a ring buffer is a poor production target.
- [ ] Describe how a histogram target finds the statements behind a wait type.
- [ ] Explain the retention modes and which one is the default.
- [ ] State when to use Audit, Query Store and XE respectively.
<!-- explainer:end -->

## References

- *Expert Performance Indexing in SQL Server* — Jason Strate, Grant Fritchey — Extended Events
- *SQL Server 2025 Query Performance Tuning (7th ed.)* — Grant Fritchey — FULL_BOOK_NEEDS_MANUAL_SPLIT (cont.) › Chapter 3 Methods For Capturing Query Performance Metrics
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 8: Query Optimization and Performance Tuning › Using Extended Events for Statistics Monitoring
- *SQL Server 2025 Query Performance Tuning (7th ed.)* — Grant Fritchey — FULL_BOOK_NEEDS_MANUAL_SPLIT (cont.)
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 7: Dynamic SQL, Triggers, and Table-Valued Functions (TVFs) › Auditing with Extended Events Integration
- *SQL Server 2025 Query Performance Tuning (7th ed.)* — Grant Fritchey — FULL_BOOK_NEEDS_MANUAL_SPLIT (cont.) › Chapter 4 Analyzing Query Behavior Using Execution Plans
- *Expert Performance Indexing in SQL Server* — Jason Strate, Grant Fritchey — Data Cleanup › Event Tracing
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 13: Security, Permissions, and Auditing › SQL Server Audit vs. SQL Trace vs. Extended Events
- *SQL Server 2025 Query Performance Tuning (7th ed.)* — Grant Fritchey — FULL_BOOK_NEEDS_MANUAL_SPLIT (cont.) › Chapter 21 Intelligent Query Processing
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 11: Automation with SSIS, SQL Agent, and PowerShell › Event Handling

