---
id: "TOOLS/SQL_Server/04_Query_Processing/Query_Optimizer"
level: 4
type: "curriculum"
curriculum: "sql_server"
---

# The Query Optimizer

**L4 Advanced** · [↑ Query Processing and Tuning](<../_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** The query optimizer is a cost-based search engine. It turns your query into a tree, rewrites it into equivalent forms, estimates the cost of each candidate plan from statistics and metadata, and stops when it has found a plan that is good enough for the time it has spent.

### 1. The big picture
SQL is declarative: you say *what* you want, not *how* to get it. For one query there are usually many correct ways to produce the result. You can read a table by scanning or by seeking an index, join tables in different orders, and join them with nested loops, merge or hash. A five-table join already has thousands of possible plans. Picking one is the optimizer's job, and the plan it picks decides almost all of the query's performance.

The optimizer is **cost-based**. It does not run the candidates. It gives each one an estimated cost based on:
- **Statistics**: how many rows each step will produce (previous block).
- **Metadata**: data types, indexes, primary keys, unique and check constraints, trusted foreign keys, NULLability.
- **Hardware and configuration**: number of CPUs available, memory, and settings like *cost threshold for parallelism* and *MAXDOP*.

Then it picks the cheapest plan it found. Two consequences follow:
1. Wrong inputs (bad statistics, missing constraints, non-SARGable predicates) give wrong costs and so a bad plan.
2. The optimizer works under a time budget. It does not guarantee the best possible plan, only a good one found quickly.

### 2. Core concepts and vocabulary
- **Parse tree**: the internal tree created by parsing; syntax is valid.
- **Algebrizer / binding**: resolves names and types and creates the **query processor tree** that the optimizer receives. Adds implicit conversions.
- **Simplification**: rewriting the tree into a simpler, equivalent tree (for example removing joins that don't change the result, or contradictory predicates).
- **Trivial plan**: a query with only one sensible plan (for example `SELECT … FROM oneTable WHERE pk = @x` with no choices). It skips full optimization.
- **Search phases**: cost-based optimization runs in stages, *search 0* (transaction processing), *search 1* (quick plan), *search 2* (full optimization). Each stage considers more transformation rules.
- **Transformation rules**: rewrite rules (join commutativity, join associativity, pushing predicates down, aggregate pushdown…) that generate alternative plans.
- **Memo**: the internal structure that stores the alternatives explored so far, grouped by equivalent result, so the same sub-plan is not costed twice.
- **Estimated subtree cost**: the optimizer's unitless cost number for a plan or sub-plan.
- **Good Enough Plan Found / Time Out / Memory Limit Exceeded**: values of the plan property *Reason For Early Termination of Statement Optimization*.
- **Cost threshold for parallelism (CTFP)**: if the best serial plan costs more than this number, the optimizer also considers a parallel plan. Default 5.
- **MAXDOP**: the maximum number of CPUs a single parallel operator may use.
- **Query hint**: an instruction that overrides or limits the optimizer's choices (`OPTION (HASH JOIN)`, `INNER LOOP JOIN`, `WITH (INDEX(...))`, `OPTION (MAXDOP 1)`).

### 3. How it works, step by step
1. **Parse and bind.** Syntax is checked, names and types are resolved, implicit conversions are inserted. Some syntax is normalized here: `BETWEEN a AND b` becomes `>= a AND <= b`.
2. **Simplification.** The tree is cleaned up using metadata:
   - A predicate that contradicts a trusted CHECK constraint (`WHERE Age < 0` with `CHECK (Age >= 0)`) can be turned into a plan that touches no data.
   - A join to a parent table whose columns are never used can be removed when a **trusted foreign key** guarantees every row has a match.
   - Constants are folded (`WHERE x = 2 + 3` → `x = 5`), and subqueries are often turned into joins.
3. **Trivial plan check.** If only one reasonable plan exists, it is used straight away. The plan's *Optimization Level* property shows `TRIVIAL`.
4. **Load statistics.** The optimizer loads (and, if needed, auto-creates or auto-updates) the statistics it needs for cardinality estimation.
5. **Cost-based search (Optimization Level = FULL).**
   - *Search 0* tries a limited set of rules, aimed at simple OLTP queries (at least three tables). If the best plan is cheap enough, it stops.
   - *Search 1* adds more rules, including join reordering. If the best cost is below an internal threshold, it stops. Otherwise, if parallelism is possible and the serial cost is above **CTFP**, it re-optimizes search 1 for a parallel plan and compares.
   - *Search 2* (full optimization) adds every rule, such as indexed view matching. It is used only for expensive, complex queries.
   - In each phase, transformation rules generate alternatives. Each alternative's operators get a cost from estimated rows × per-row CPU and I/O costs. The cheapest are kept in the memo.
6. **Early termination.** The optimizer stops when it is satisfied (*Good Enough Plan Found*), when it has used its time budget (*Time Out*: the budget depends on the query's complexity, so it does not mean "the plan is bad", only that the search was cut short), or when it runs out of optimization memory.
7. **Output.** The plan goes to the plan cache and to execution.

| Plan property (root SELECT node) | What it tells you |
|---|---|
| Optimization Level | TRIVIAL or FULL |
| Reason For Early Termination | Good Enough Plan Found / Time Out / Memory Limit Exceeded |
| CardinalityEstimationModelVersion | 70 = legacy CE; 120+ = new CE |
| Compile CPU / Compile Time / Compile Memory | What optimization cost |
| Estimated Subtree Cost | Total plan cost, compared against CTFP |

### 4. Worked example
**a) Watch simplification use a trusted foreign key**
```sql
-- SalesOrderDetail has a trusted FK to SalesOrderHeader
SELECT sod.SalesOrderID, sod.OrderQty
FROM Sales.SalesOrderDetail AS sod
JOIN Sales.SalesOrderHeader AS soh
  ON soh.SalesOrderID = sod.SalesOrderID;    -- no soh columns used
```
The actual plan shows only a scan of `SalesOrderDetail`. The join is gone, because a trusted foreign key guarantees every detail row has exactly one header. If you disable the FK and re-enable it `WITH NOCHECK` (now `is_not_trusted = 1` in `sys.foreign_keys`), the join comes back. Constraints are not just about data quality: they are **inputs to the optimizer**.

**b) Hints: work with the optimizer, not against it**
A large table is joined to a large archive table. The optimizer picks nested loops, which is very slow at that size. A tempting fix is a join hint:
```sql
SELECT i.ImpressionUID
FROM marketing.Impressions AS i
INNER HASH JOIN dbo.ImpressionsArchive AS ia
  ON i.ImpressionUID = ia.ImpressionUID;     -- forces Hash Join (5 s)
```
Later someone adds a nonclustered index on `Impressions(ImpressionUID)`. Now a merge join over two ordered indexes would be best (about 3 s), but the hint still forces hash (about 4 s). The hint has frozen a decision that the data and schema no longer support. A safer hint *removes the bad option* and leaves the choice open:
```sql
SELECT i.ImpressionUID
FROM marketing.Impressions AS i
JOIN dbo.ImpressionsArchive AS ia ON i.ImpressionUID = ia.ImpressionUID
OPTION (MERGE JOIN, HASH JOIN);              -- anything but nested loops
```
Note that a join hint written inside the `FROM` clause (`INNER HASH JOIN`) also **fixes the join order** as written. That is another hidden side effect.

**c) Find out what costs your plans have, to set CTFP**
```sql
WITH XMLNAMESPACES (DEFAULT N'http://schemas.microsoft.com/sqlserver/2004/07/showplan')
SELECT TOP (50)
       qp.query_plan.value(N'(//RelOp/@EstimatedTotalSubtreeCost)[1]', N'float') AS est_cost,
       qs.execution_count
FROM sys.dm_exec_query_stats AS qs
CROSS APPLY sys.dm_exec_query_plan(qs.plan_handle) AS qp
ORDER BY est_cost DESC;   -- XML shredding is expensive: run off-peak
```
If most small OLTP queries cost between 5 and 30 and they go parallel needlessly, raising CTFP (for example to 35–50) keeps them serial and leaves parallelism for large reports.

### 5. Under the hood
- **Memo and groups.** The memo holds *groups*. Each group is a set of logically equivalent expressions (for example "A join B" in every order and algorithm). Rules add new expressions to groups. Costing picks the best physical expression per group, so the search reuses work instead of enumerating whole plans.
- **Logical vs physical operators.** Rules first explore *logical* alternatives (join order, where aggregates go), then *implementation* rules turn them into physical operators (Inner Join → Nested Loops / Merge / Hash). The plan shows both names on each operator.
- **Costing model.** Operator costs are built from estimated rows, row size, and fixed CPU and I/O constants tuned for a reference machine from the late 1990s. The absolute number does not mean seconds on your server. Only the *relative* values matter, which is why cost % is a guide and never a measurement.
- **Parallel plans.** These need Exchange operators (Distribute/Repartition/Gather Streams). They cost more CPU and memory in total but finish sooner. Things that block parallelism include scalar UDFs that are not inlined, writes to table variables, and some system functions. The plan property *NonParallelPlanReason* tells you which one applied.
- **Version changes.** Each compatibility level can change the cardinality estimator and add Intelligent Query Processing features. A database upgrade can therefore change plans even when no code changed. Query Store exists to catch exactly that.

### 6. Performance and design implications
- Give the optimizer good inputs: up-to-date statistics; **trusted** constraints and foreign keys; correct, matching data types (an implicit conversion on a column blocks seeks); SARGable predicates.
- Simpler queries optimize better. A 15-table query with views nested in views can hit *Time Out* and get a mediocre plan. Splitting it into steps with a temp table often helps.
- Compile time is real CPU. Check *CompileCPU* in the plan for complex queries that run often.
- CTFP default 5 is very low for modern hardware. Most practitioners raise it (often 25–50) after looking at their workload's costs. Set MAXDOP according to Microsoft's per-NUMA guidance (often 8 or less) instead of 0 (unlimited).

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `sp_configure 'cost threshold for parallelism', 35` | Raises the cost at which parallel plans are considered | After checking plan costs on your workload |
| `sp_configure 'max degree of parallelism', 8` / `ALTER DATABASE SCOPED CONFIGURATION SET MAXDOP = 8` | Limits CPUs per parallel operator | Every server; per database if needed |
| `OPTION (MAXDOP n)` | Per-query parallelism limit | Rare, targeted fixes |
| `OPTION (HASH JOIN, MERGE JOIN)` | Limits join algorithms for the whole query | To rule out a known-bad choice |
| `INNER LOOP/HASH/MERGE JOIN` | Forces one algorithm **and** the join order | Almost never |
| `OPTION (FORCE ORDER)` | Joins tables in the written order | Diagnosing join-order problems |
| `WITH (INDEX(ix))` table hint | Forces an index | Last resort, breaks if the index is renamed or dropped |
| `ALTER TABLE … WITH CHECK CHECK CONSTRAINT ALL` | Re-validates constraints so they are trusted | After bulk loads with NOCHECK |
| `sys.dm_exec_query_optimizer_info` | Server-wide counters (optimizations, trivial plans, timeouts) | Seeing how often each phase or timeout occurs |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| Fix the inputs (stats, indexes, constraints, rewrite) | Always the first option |
| Hint that *excludes* one bad option (`OPTION (MERGE JOIN, HASH JOIN)`) | Data volumes may change a lot; review it periodically |
| Hint that *forces* one choice | Schema or data will change; the hint freezes today's decision |
| Query Store plan forcing or Query Store hints | Need to fix without changing code; still review over time |
| Higher CTFP | Many small queries going parallel and waiting on CXPACKET/CXCONSUMER |
| `MAXDOP 1` for a query | Parallelism really hurts it, not just to silence waits |

### 9. Common mistakes
- Forcing a join type with a join hint → it stays forced after an index makes a better plan possible → fix the cause, or use a query-level hint that gives the optimizer choices.
- Loading data with `NOCHECK` and never re-checking → constraints untrusted, simplification lost → `WITH CHECK CHECK CONSTRAINT`.
- Reading "Reason For Early Termination: Time Out" as "plan is bad" → it only means the search stopped → compare estimated vs actual rows before blaming it; simplify the query if the plan really is poor.
- Leaving CTFP at 5 and MAXDOP at 0 → tiny queries go parallel across all cores → set both deliberately.
- Comparing two queries by "cost relative to the batch" → costs are estimates → compare reads, CPU and duration.

### 10. 🎯 Interview kit
**30-second answer:** "SQL Server's optimizer is cost-based. After parsing and binding, it simplifies the query tree using constraints and metadata, uses a trivial plan if there's only one option, and otherwise explores alternatives in phases using transformation rules stored in a memo. It costs each alternative from cardinality estimates and stops when it finds a good-enough plan or hits a time-out. It also decides on parallelism based on cost threshold for parallelism and MAXDOP. Its plans are only as good as the statistics and metadata it gets."
**Structure a longer answer:** clarify (why care?) → phases → inputs (stats, constraints, types) → parallelism → hints as a last resort → scenario (a trusted FK removing a join, or a hint that became wrong after a new index).
**Likely follow-ups:** *Why might a query get a bad plan?* → bad estimates, non-SARGable predicates, parameter sniffing, optimizer timeout on complex queries. *What does cost threshold for parallelism do?* → serial plan cost above it makes the optimizer consider a parallel plan. *Should you use hints?* → rarely; prefer fixing inputs; prefer hints that limit options over hints that force one.
**Signals interviewers look for:** knows the optimizer does not find "the best" plan; ties constraints and statistics to plan quality; cautious with hints; can name plan properties.

### 11. Check yourself
- [ ] List the optimizer's inputs and give one example of each changing a plan.
- [ ] Show a join being removed by a trusted FK, then make the FK untrusted and watch it come back.
- [ ] Explain the difference between `INNER HASH JOIN` and `OPTION (HASH JOIN)`.
- [ ] Find *Optimization Level* and *Reason For Early Termination* in a plan's properties.
- [ ] Describe how you would choose a value for cost threshold for parallelism.
<!-- explainer:end -->

## References

- *100 SQL Server Mistakes and How to Avoid Them* — Peter A. Carter — Optimization › 10.5 #58 Working against the optimizer
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 8: Query Optimization and Performance Tuning › Query Hints
- *SQL Server 2025 Query Performance Tuning (7th ed.)* — Grant Fritchey — FULL_BOOK_NEEDS_MANUAL_SPLIT (cont.) › Chapter 2 Execution Plan Generation And The Query Optimizer
- *Practical Graph Structures in SQL Server and Azure SQL* — Louis Davidson — SQL Graph Table Basics › Object Creation
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 12: Azure SQL and Synapse Integration › Cost Management in Azure SQL and Synapse
- *Expert Performance Indexing in SQL Server* — Jason Strate, Grant Fritchey — Temporary Objects
- *Practical Graph Structures in SQL Server and Azure SQL* — Louis Davidson — A Graph For Testing › The Queries
- *Beginning T-SQL* — Kathi Kellenberger, Lee Everest — Grouping and Summarizing Data › Aggregate Queries with More Than One Table
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 12: Azure SQL and Synapse Integration › Cost Model
- *Beginning T-SQL* — Kathi Kellenberger, Lee Everest — Expanding on Data Type Concepts › Creating a Hierarchy

