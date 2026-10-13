---
id: "TOOLS/SQL_Server/01_Architecture_and_Storage/Engine_Architecture"
level: 4
type: "curriculum"
curriculum: "sql_server"
---

# Engine Architecture

**L4 Advanced** · [↑ Architecture and Storage](<../_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** A query travels through the protocol layer, the relational engine (parser, algebrizer, optimizer, executor) and the storage engine (access methods, buffer, lock and log managers), all running on SQLOS — and each hop has its own failure modes and diagnostics.

### 1. The big picture
The Database Engine is the part of SQL Server that stores, processes and secures data. Knowing its components is what lets you read an execution plan, interpret a wait type or explain why a query that "should be fast" is not. When someone says "the optimizer chose a bad plan", "the query is waiting on I/O" or "it's blocked", they are naming a specific engine component.

The guiding example from the books: a web shop shows a product page. The site runs a query that reads `dbo.Products` for one `ProductID`, returns its name, description and image, and uses a `CASE` expression to print "In Stock", "n left in stock" or "Out of stock". We will follow that query through the engine.

### 2. Core concepts and vocabulary
- **TDS (Tabular Data Stream)** — the application-layer protocol clients use to talk to SQL Server.
- **SNI (SQL Server Network Interface)** — part of the protocol layer that handles the network transport (TCP/IP, named pipes, shared memory).
- **Relational engine (query processor)** — parses, binds, optimizes and executes queries.
- **Parser** — checks syntax and builds a parse tree; syntax errors come from here.
- **Algebrizer (binding)** — resolves names to object IDs, checks columns and types, and produces a normalized *query tree*.
- **Query optimizer** — cost-based component that searches for an efficient physical plan using statistics, indexes, constraints and data types.
- **Plan cache** — memory where compiled plans are stored for reuse.
- **Query executor** — runs the plan's operators (seek, join, sort…) and asks the storage engine for rows.
- **Storage engine** — access methods (heaps, B-trees, columnstore), buffer manager, lock manager, transaction/log manager.
- **Buffer pool** — memory cache of data pages.
- **SQLOS** — SQL Server's internal operating-system layer: schedulers, workers, memory clerks, waits, I/O completion.

### 3. How it works, step by step
![The flow of a query through the Database Engine, from the client over TDS to the protocol layer, relational engine and storage engine](<../../../../_assets/100_SQL_Server_Mistakes_and_How_to_Avoid_-_Peter_A_Carter/CH01_F03_Carter.png>)

1. **Client → protocol layer.** The application sends the batch inside TDS packets. SNI receives them and hands the request to the relational engine. *(Diagnostics: `sys.dm_exec_connections`, `sys.dm_exec_sessions`; `ASYNC_NETWORK_IO` waits when the client reads results slowly.)*
2. **Parse.** The parser checks syntax. A missing comma fails here, before anything else.
3. **Algebrize (bind).** Names (`dbo.Products`, `StockQty`) become object and column IDs; types are checked; the result is a normalized query tree. A misspelled column fails here.
4. **Plan cache lookup.** If a matching plan already exists (same text, same SET options, same database), it is reused and optimization is skipped.
5. **Optimize.** SQL is *declarative* — you describe the result, not the steps. The optimizer turns the tree into a physical plan by exploring alternatives (which index, which join type, join order, parallel or not) and choosing the lowest estimated cost. Estimates come from statistics, so stale statistics → wrong row counts → poor choices. Trivial queries get a *trivial plan* without full search.
6. **Execute.** The executor runs the plan iterator by iterator. For our product query it asks the storage engine for one row via a clustered index seek on `ProductID`.
7. **Storage engine: access methods** navigate the B-tree to the right page.
8. **Buffer manager** returns the page from the buffer pool or reads it from disk first (`PAGEIOLATCH_SH` waits).
9. **Lock manager** takes a shared lock (or skips it under `NOLOCK`, which the books flag as a mistake: you can read uncommitted, duplicated or missing rows).
10. **Results** flow back through the executor to the protocol layer and to the client as TDS packets.

For a write, the storage engine also takes exclusive locks and generates log records; commit waits for the log flush (see *Transaction Log and Recovery*).

| Component | Typical problem | Where you see it |
|---|---|---|
| Protocol layer | Slow client consumption | `ASYNC_NETWORK_IO` |
| Parser/algebrizer | Syntax/binding errors | Error messages |
| Optimizer | Bad estimates → bad plan | Estimated vs actual rows in plans |
| Plan cache | Bloat, parameter sniffing | `sys.dm_exec_cached_plans`, Query Store |
| Access methods | Scans instead of seeks | Plan operators, `STATISTICS IO` |
| Buffer manager | Physical I/O | `PAGEIOLATCH_*`, Page Life Expectancy |
| Lock manager | Blocking, deadlocks | `LCK_M_*`, `sys.dm_tran_locks` |
| Log manager | Slow commits | `WRITELOG` |
| SQLOS | CPU pressure | `SOS_SCHEDULER_YIELD`, signal wait time |

### 4. Worked example
```sql
DECLARE @ProductID INT = 42;

SELECT ProductName,
       ProductDescription AS Description,
       CASE WHEN StockQty >= 10 THEN N'In Stock'
            WHEN StockQty > 0   THEN CAST(StockQty AS NVARCHAR(10)) + N' left in stock'
            ELSE N'Out of stock' END AS Availability,
       ProductImage
FROM dbo.Products            -- no NOLOCK
WHERE ProductID = @ProductID;

-- Which phase is the request in right now, and what is it waiting on?
SELECT r.session_id, r.status, r.command, r.wait_type, r.wait_time, r.cpu_time, r.logical_reads
FROM sys.dm_exec_requests r
WHERE r.session_id > 50;

-- Was the plan reused?
SELECT cp.usecounts, cp.objtype, st.text
FROM sys.dm_exec_cached_plans cp
CROSS APPLY sys.dm_exec_sql_text(cp.plan_handle) st
WHERE st.text LIKE '%Out of stock%' AND st.text NOT LIKE '%dm_exec%';
```
Reading the results:
- In the actual execution plan you should see a **Clustered Index Seek** (storage engine access) feeding a **Compute Scalar** (the `CASE`, evaluated by the executor).
- `sys.dm_exec_requests` shows the status: `running` (on CPU), `runnable` (waiting for a scheduler — CPU pressure), or `suspended` with a `wait_type` that names the component being waited on.
- `usecounts` > 1 in the plan cache means the optimizer was skipped on later executions.

### 5. Under the hood
- **SQLOS scheduling.** One scheduler per logical CPU. A *task* (part of a request) is bound to a *worker* thread which runs on a scheduler. Scheduling is *cooperative*: a worker voluntarily yields every few milliseconds (quantum ≈ 4 ms) or when it must wait. Time spent waiting for a resource is *resource wait*; time waiting in the runnable queue for CPU after the resource is ready is *signal wait*. High signal wait ⇒ CPU pressure.
- **Memory.** SQLOS manages memory through *memory clerks* (buffer pool, plan cache, lock manager, memory grants for sorts/hashes). Query memory grants are requested before execution; `RESOURCE_SEMAPHORE` waits mean queries are queuing for grant memory.
- **Optimizer phases.** Simplification (remove contradictions, push predicates), trivial plan check, then cost-based search in stages (search 0, 1, 2) that stop when a "good enough" plan is found or a timeout is reached (`StatementOptmEarlyAbortReason` in plan XML).
- **Plan reuse.** Plans are keyed by the exact text and session settings. Ad hoc literals create a new plan per value unless parameterized; parameters are *sniffed* at compile time, which can produce a plan good for one value and bad for another.
- **Read-ahead.** The storage engine prefetches pages it predicts a scan will need, so large scans issue big I/Os ahead of the executor.
- **Cloud variants.** Azure SQL Database exposes the same engine but sizes compute as DTUs (bundled CPU/IO/memory) or vCores; Synapse dedicated pools add workload management with *resource classes* that allocate memory per query so big ETL batches don't starve small queries.

### 6. Performance and design implications
- Start triage with **waits** (`sys.dm_os_wait_stats`, `sys.dm_exec_requests`): they tell you which component is the bottleneck.
- Most "slow query" issues are optimizer issues caused by **estimates** — compare estimated and actual rows before adding hardware.
- **Parameterize** application queries to avoid plan-cache bloat and compile CPU.
- Don't use `NOLOCK` to "speed up" reads; consider read committed snapshot isolation instead.
- Avoid `sp_` prefixes for user procedures — SQL Server first looks for them in `master` (a tiny but avoidable cost and a naming collision risk).

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `sys.dm_exec_requests` | Live requests, status, waits | What is running now |
| `sys.dm_os_wait_stats` | Cumulative waits since restart | Instance-level bottleneck |
| `sys.dm_os_schedulers` | Scheduler queues (`runnable_tasks_count`) | CPU pressure |
| `sys.dm_exec_cached_plans` + `dm_exec_query_plan` | Plans in cache | Plan reuse, bloat |
| `SET STATISTICS XML ON` / actual plan | Runtime plan with actual rows | Optimizer issues |
| Query Store | Plan and runtime history per query | Regressions, plan forcing |
| `OPTION (RECOMPILE)` | Compile for current parameter values | Sniffing problems on rare queries |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| Plan reuse (parameterized queries) — high-frequency OLTP | Parameter values with very different selectivity → sniffing issues |
| `OPTION (RECOMPILE)` — infrequent, highly variable queries | Hot queries (compile CPU every call) |
| Row versioning (RCSI) for read/write concurrency | Heavy tempdb version store load |
| DTU model — simple, small workloads | Need to tune CPU/memory independently → vCore |

### 9. Common mistakes
- **Blaming the disk first** → buying storage when the problem is a bad plan → check waits and estimated vs actual rows.
- **Using `WITH (NOLOCK)` everywhere** → dirty, missing or duplicated rows → use proper isolation (RCSI/snapshot).
- **Concatenating literals into SQL** → a new plan per value, cache bloat, injection risk → use parameters.
- **Reading cumulative wait stats without a baseline** → misleading totals since last restart → capture deltas over an interval.

### 10. 🎯 Interview kit
**30-second answer:** A request arrives over TDS to the protocol layer, then the relational engine parses it, binds names to objects, looks for a cached plan, and otherwise lets the cost-based optimizer build one from statistics. The executor runs the plan and calls the storage engine, which navigates heaps/B-trees, reads pages via the buffer pool, takes locks and writes log records. Everything runs on SQLOS, which schedules workers cooperatively and records waits.

**Structure a longer answer:** follow one query through the layers → name each component's job → map wait types to components → close with a triage scenario: slow query → `dm_exec_requests` shows `PAGEIOLATCH_SH` → plan shows scan due to bad estimate → update stats/add index.

**Likely follow-ups:**
- *Parser vs algebrizer?* → syntax vs name/type resolution.
- *What does the optimizer need?* → statistics, indexes, constraints, data types, cardinality model.
- *Signal wait vs resource wait?* → time waiting for CPU vs time waiting for a resource.
- *Why is SQL "declarative"?* → you state the result; the optimizer decides the steps.

**Signals interviewers look for:** clear separation of relational vs storage engine, use of waits as the first diagnostic, and awareness that estimates drive plans.

### 11. Check yourself
- [ ] Draw a query's path from client to storage engine and back, naming each component.
- [ ] Classify these errors/waits by component: syntax error, invalid column, `WRITELOG`, `LCK_M_X`, `SOS_SCHEDULER_YIELD`.
- [ ] Show whether a query's plan was reused using the plan cache DMVs.
- [ ] Explain cooperative scheduling and what signal wait means.
- [ ] Explain why `NOLOCK` is a correctness risk.
<!-- explainer:end -->

## References

- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 12: Azure SQL and Synapse Integration › Workload Management and Resource Classes
- *100 SQL Server Mistakes and How to Avoid Them* — Peter A. Carter — Introducing SQL Server › 1.2.1 Overview of the Database Engine
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 12: Azure SQL and Synapse Integration › Resource Allocation (DTUs vs. vCores)
- *SQL Server Integration Services Design Patterns* — Tim Mitchell et al. — SQL Server Integration Services Design Patterns › Contents

