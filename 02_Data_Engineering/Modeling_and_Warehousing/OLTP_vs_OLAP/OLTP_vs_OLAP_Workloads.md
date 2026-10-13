---
id: "02_Data_Engineering/Modeling_and_Warehousing/OLTP_vs_OLAP/OLTP_vs_OLAP_Workloads"
level: 4
type: "concept"
topic: "02_Data_Engineering/Modeling_and_Warehousing/OLTP_vs_OLAP"
tools: []
---

# OLTP vs OLAP Workloads

**L4 Advanced** · [↑ OLTP vs OLAP](<_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** OLTP systems run the business one small, fast, correct transaction at a time; OLAP systems study the business by scanning huge amounts of history — and the two want opposite things from a database.

### 1. The big picture
Every company's data lives in two worlds. The first is the **operational world**: the application that takes an order, moves money between accounts, or updates a shopping cart. Each action touches a handful of rows, must finish in milliseconds, and must never lose or corrupt data. The second is the **analytical world**: questions such as "what was month-over-month sales growth for the last five years?" or "which products are bought together?". These questions touch millions of rows, read only a few columns, and are happy to run for seconds or minutes.

The first world is called **OLTP** (online transaction processing); the second **OLAP** (online analytical processing). Trying to serve both from one database is the classic early-stage mistake: a single big analytical query scans the production tables, evicts the cache, takes locks or burns I/O, and the checkout page slows to a crawl. Understanding precisely *how* the two workloads differ is the foundation for every later design decision in data engineering — star schemas, columnar storage, ETL/ELT, CDC, lakehouses all exist because of this split.

This lesson is the vocabulary and mental model. The sibling lessons cover physical design (indexes, storage layout), how to separate the systems, and how to feed the analytical side from the operational side.

### 2. Core concepts and vocabulary
- **OLTP** — a system optimized for many concurrent, short read/write transactions that maintain the *current state* of an application (orders, balances, inventory).
- **OLAP** — a system optimized for complex queries that read large volumes of (mostly historical) data to produce aggregates, trends and reports. "Online" means it constantly listens for queries and serves interactive analysis; it does not have to mean OLAP *cubes*.
- **Transaction** — a unit of work that either fully happens or not at all (an ATM withdrawal: check balance, debit, record the event).
- **ACID** — Atomicity (all or nothing), Consistency (rules always hold), Isolation (concurrent transactions don't see each other's half-done work), Durability (once committed, survives a crash). OLTP is defined by its need for ACID.
- **Point lookup** — fetching one row (or a few) by key: `WHERE order_id = 42`.
- **Scan / aggregation** — reading a large fraction of a table and summarizing it: `SUM(amount) GROUP BY region`.
- **Concurrency** — how many users/queries run at once. OLTP: thousands of small ones. OLAP: few users, each running heavy queries.
- **Data latency (freshness)** — how stale the data is. OLTP is live; OLAP typically tolerates minutes to a day.
- **Data warehouse / data lake** — the typical storage systems that host OLAP workloads.
- **Mixed workload (HTAP)** — running both kinds of work against the same copy of data; tempting, hard to do well.

### 3. How it works, step by step
Compare what the two engines do for their typical request.

**An OLTP request ("add item to cart", "withdraw €50")**
1. The application opens a transaction and sends a short statement keyed on a primary key.
2. The engine uses an index to find the single row (a few page reads, usually already in memory).
3. It takes row-level locks (or uses row versioning) so concurrent customers don't collide.
4. It modifies the row and writes the change to the transaction log *before* acknowledging — durability.
5. It commits; locks release; the user sees a response in milliseconds.
6. Repeat thousands of times per second from thousands of connections.

**An OLAP request ("revenue by region by month for 3 years")**
1. An analyst or BI tool submits one large query.
2. The optimizer chooses a plan that reads the needed *columns* of a very large table, ideally skipping partitions/segments by date.
3. The engine scans, filters, joins to small dimension tables, and aggregates — using many cores or many nodes in parallel.
4. Intermediate results spill to temporary storage if memory is short.
5. Result returns after seconds to minutes; the analyst refines the query and runs it again (analysis is iterative).

| Dimension | OLTP | OLAP |
|---|---|---|
| Purpose | Run the business | Understand the business |
| Typical op | INSERT/UPDATE/DELETE, point SELECT | Large SELECT with joins + aggregates |
| Rows per query | 1 – a few hundred | Millions – billions |
| Columns per query | Whole row | A handful of columns |
| Latency target | ms | seconds – minutes |
| Concurrency | Very high | Low–moderate |
| Data | Current, detailed | Historical, often summarized/integrated |
| Schema | Normalized (many small tables) | Denormalized/star (few wide tables) |
| Consistency need | Strict ACID, immediate | Accurate, but may be hours stale |
| Availability | Down = lost revenue | Down = delayed report |
| Writes | Constant, tiny | Batch/bulk loads, rarely updated |
| Storage layout | Row-oriented | Column-oriented |

### 4. Worked example
The same fact — a customer's orders — queried in both worlds.

```sql
-- OLTP: "how many orders did user 452 place today?"  (touches a few rows, via an index)
SELECT COUNT(*)
FROM   dbo.Orders
WHERE  CustomerId = 452
  AND  OrderDate >= CAST(SYSUTCDATETIME() AS date);

-- OLAP: "revenue by region and month, last 3 years" (touches hundreds of millions of rows)
SELECT  d.CalendarMonth,
        c.Region,
        SUM(f.SalesAmount) AS Revenue,
        COUNT(DISTINCT f.CustomerKey) AS ActiveCustomers
FROM    fact.Sales f
JOIN    dim.Date     d ON d.DateKey     = f.DateKey
JOIN    dim.Customer c ON c.CustomerKey = f.CustomerKey
WHERE   d.CalendarYear >= YEAR(GETDATE()) - 3
GROUP BY d.CalendarMonth, c.Region;
```

What to notice: the first query is satisfied by an index seek on `(CustomerId, OrderDate)` and reads a few pages. The second cannot be helped by a row index — it must read the `SalesAmount`, `DateKey` and `CustomerKey` columns for most of the table. Its cost is dominated by *bytes scanned*, which is why analytic systems store columns separately, compress them, and partition by date. If the second query is run against the production OLTP tables during business hours, it competes for buffer pool and I/O with the first — the root cause of "the dashboard took down checkout".

A useful sanity test when classifying a request: ask "how many rows, how many columns, how fresh, how many at once?"

### 5. Under the hood
*(optional at this level, but worth knowing)*

- **Why ACID drives OLTP design.** Write-ahead logging, row locks/MVCC and small pages exist so that many writers can safely update the same table. Each of these has overhead that an analytic system, which writes in big batches, can avoid or reduce (e.g., minimally logged bulk loads).
- **Why indexes help OLTP but not always OLAP.** A B-tree index finds a few rows fast. When a query needs 30% of the table, following an index row-by-row is slower than a sequential scan, and every index also taxes every write. Modern analytic engines therefore lean on scans, compression, partition pruning and min/max metadata instead of many indexes.
- **Why normalization differs.** OLTP normalizes so each fact is stored once and updates touch one row (avoiding anomalies). OLAP deliberately denormalizes (star schema) so queries need few joins; since data is loaded once and rarely updated, duplication is cheap and anomalies are controlled by the load process.
- **Different definitions of consistency.** OLTP needs *transactional* consistency now. OLAP needs *analytical* consistency: numbers must reconcile and be reproducible, but may reflect data as of the last load.
- **Mixed workloads** are possible (HTAP engines, columnstore on a replica) but each such design is a compromise; the safest default is two systems connected by a pipeline.

### 6. Performance and design implications
- Cost in OLTP is measured in **latency and throughput** (transactions/sec, p99 ms, lock waits, log flush time).
- Cost in OLAP is measured in **bytes scanned, query duration and cluster cost**; cloud warehouses often bill by data scanned or compute time.
- Keep OLTP rows narrow and hot; keep OLAP fact tables partitioned by date and scanned by column.
- An unpartitioned, always-on warehouse is the usual reason analytic bills double: scan volume and idle compute, not query count.
- Alert on **freshness** for OLAP (time since last successful load) — a "healthy" dashboard that silently shows yesterday's data is the classic failure.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `SET STATISTICS IO ON` | Shows pages read per table | Prove a query is scan-heavy vs lookup-heavy |
| Execution plan (Seek vs Scan) | Shows access pattern | Classify a query as OLTP-like or OLAP-like |
| Read-only replica / readable secondary | Offloads analytic reads | First step off the primary |
| Resource Governor / workload groups | Caps CPU/memory per workload | Protect OLTP when mixed |
| Partitioning by date | Lets scans skip old data | Large OLAP fact tables |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| One OLTP database is enough: small data, a few simple reports | Reports scan big tables during business hours |
| A read replica for reporting: moderate data, near-real-time needed | Reports need history the source purges, or heavy joins across systems |
| A separate warehouse/lake fed by pipelines: large history, many sources | You cannot afford the pipeline's operational cost yet |
| HTAP / hybrid engine: need real-time analytics on live data | You assume it removes the need to model data for analytics |

### 9. Common mistakes
- Believing OLAP is "write-heavy because it stores lots of data" → it is **read-heavy**; writes arrive as scheduled batches.
- Running reports directly on production → lock waits, cache churn, outages → move reads to a replica or warehouse.
- Judging an OLAP system by OLTP latency expectations → disappointment with 20-second queries that are normal for scanning 5 years of data.
- Putting thousands of single-row lookups per second on a columnar warehouse → "brings it to its knees" → add a serving layer/cache or keep that access in an OLTP store.
- Assuming "OLAP means cubes" → in practice most modern analytics is SQL on column stores.
- Saying duplication is "bad" everywhere → it's bad where updates must stay consistent (OLTP), a deliberate trade in read-optimized OLAP.

### 10. 🎯 Interview kit
**30-second answer:** OLTP handles live business transactions — many short, concurrent reads and writes on current data with strict ACID and millisecond latency, in normalized, row-oriented stores. OLAP analyzes large volumes of historical data — few users running heavy scans and aggregations over a handful of columns, in denormalized, columnar stores that can be hours stale. You separate them so analytics can't hurt the application.

**Structure a longer answer:** clarify (what's being asked: workload shape, freshness, scale) → core mechanism (access pattern: rows/columns per query, index vs scan) → trade-offs (latency vs throughput, normalization vs denormalization, freshness vs cost) → close with a real scenario (ATM withdrawal vs executive quarterly sales analysis).

**Likely follow-ups:**
- *Is "how many orders did user 452 place today" OLTP or OLAP?* → OLTP: one key, few rows, needs fresh data, served by an index.
- *Why is availability more critical for OLTP?* → Downtime stops revenue-generating transactions immediately; a delayed report rarely does.
- *Why is duplication acceptable in OLAP?* → Data is loaded in controlled batches and not updated in place, so update anomalies are not a runtime risk, and duplication saves joins.
- *Can one system do both?* → HTAP exists; needs resource isolation and still usually a separate model; explain the trade-off.
- *How would you set freshness alerts?* → Track last-successful-load time per table against its expected cadence; page when lag exceeds cadence + tolerance, not on every batch tick.

**Signals interviewers look for:** you reason from access patterns (rows × columns × freshness × concurrency), not from definitions; you mention ACID and its cost; you connect to normalization vs star schema; you know the failure mode of mixing.

### 11. Check yourself
- [ ] Classify five requests (cart add, monthly trend, fraud check at swipe, executive dashboard, ML training pull) as OLTP/OLAP and justify with rows/columns/freshness.
- [ ] Explain why the same table needs a different index strategy in each world.
- [ ] Describe what happens to checkout latency when a 3-year report scans production, and name two ways to fix it.
- [ ] State what "consistency" means in each system.
- [ ] Draw the dataflow OLTP → pipeline → OLAP → BI tool and label where latency is introduced.
<!-- explainer:end -->

## Practice questions

- [Basic] [What does the 'On-Line' part of both OLTP and OLAP actually refer to?](<../../../PRACTICE/02_Data_Engineering/Modeling_and_Warehousing/OLTP_vs_OLAP/Q-34897_What_does_the_On-Line_part_of_both_OLTP_and_OLAP_actually_re.md>)
- [Basic] [In your own words, what is the 'shape' of a typical OLTP query versus an OLAP query?](<../../../PRACTICE/02_Data_Engineering/Modeling_and_Warehousing/OLTP_vs_OLAP/Q-34930_In_your_own_words_what_is_the_shape_of_a_typical_OLTP_query_.md>)
- [Basic] [If a query filters WHERE id = 5 versus WHERE created_at BETWEEN two years, which is OLTP and which is OLAP?](<../../../PRACTICE/02_Data_Engineering/Modeling_and_Warehousing/OLTP_vs_OLAP/Q-34972_If_a_query_filters_WHERE_id_5_versus_WHERE_created_at_BETWEE.md>)
- [Basic] [Which system typically stores large volumes of historical data — OLTP or OLAP?](<../../../PRACTICE/02_Data_Engineering/Modeling_and_Warehousing/OLTP_vs_OLAP/Q-34608_Which_system_typically_stores_large_volumes_of_historical_da.md>)
- [Basic] [How does data volume typically grow over time in OLAP versus OLTP systems?](<../../../PRACTICE/02_Data_Engineering/Modeling_and_Warehousing/OLTP_vs_OLAP/Q-34660_How_does_data_volume_typically_grow_over_time_in_OLAP_versus.md>)
- [Intermediate] [Why is 'a few very large joins' typical of OLAP while 'a lookup by ID' is typical of OLTP?](<../../../PRACTICE/02_Data_Engineering/Modeling_and_Warehousing/OLTP_vs_OLAP/Q-34960_Why_is_a_few_very_large_joins_typical_of_OLAP_while_a_lookup.md>)
- [Intermediate] [How would you explain OLTP versus OLAP to a non-technical business stakeholder?](<../../../PRACTICE/02_Data_Engineering/Modeling_and_Warehousing/OLTP_vs_OLAP/Q-34667_How_would_you_explain_OLTP_versus_OLAP_to_a_non-technical_bu.md>)
- [Intermediate] [When migrating from an OLTP-only stack to adding a warehouse, how do you decide what stays as operational reporting on the OLTP side vs what moves to OLAP?](<../../../PRACTICE/02_Data_Engineering/Modeling_and_Warehousing/OLTP_vs_OLAP/Q-34774_When_migrating_from_an_OLTP-only_stack_to_adding_a_warehouse.md>)

## Related concepts

- [Data Warehouse Fundamentals](<../Data_Warehousing/Data_Warehouse_Fundamentals.md>) *(Data Warehousing)*
- [OLTP vs Star Schema Performance](<../Dimensional_Modeling/OLTP_vs_Star_Schema_Performance.md>) *(Dimensional Modeling)*
- [Databases and Schemas](<../../../01_Databases_and_SQL/Database_Internals/DBMS_Concepts/Databases_and_Schemas.md>) *(DBMS Concepts)*
- [Warehouse as System of Record](<../../../04_Data_Architecture_and_Strategy/Architecture_Patterns/Data_Architecture/Layered_Pipelines_Bronze_Silver_Gold/Warehouse_as_System_of_Record.md>) *(Data Architecture)*
- [Real-time OLAP Serving Layer](<../../../04_Data_Architecture_and_Strategy/Architecture_Patterns/Data_Architecture/Tiered_Lake_Storage/Real-time_OLAP_Serving_Layer.md>) *(Data Architecture)*
- [Choosing a Database for the Workload](<../../../01_Databases_and_SQL/Database_Strategy/Database_Selection/Choosing_a_Database_for_the_Workload.md>) *(Database Selection)*

## References

- *Azure Data Fundamentals* — Michael John Pena — Understanding Data Workloads › Data Workloads in Everyday Terms
- *Azure Data Fundamentals* — Michael John Pena — Understanding Data Workloads › Transactional Workloads (OLTP)
- *Azure Data Fundamentals* — Michael John Pena — Understanding Data Workloads › Warning
- *Azure Data Fundamentals* — Michael John Pena — Understanding Data Workloads › Current, operational data focus
- *AWS Certified Data Engineer Associate Study Guide* — Sakti Mishra et al. — Prerequisite Knowledge for Aspiring Data Engineers › OLTP Versus OLAP
- *Fundamentals of Data Engineering* — Joe Reis, Matt Housley — Data Generation in Source Systems › APIs
- *Fundamentals of Data Engineering* — Joe Reis, Matt Housley — Data Generation in Source Systems › Online Analytical Processing System
- *Fundamentals of Data Engineering* — Joe Reis, Matt Housley — Serving Data for Analytics, Machine Learning, and Reverse ETL › Streaming Systems
- *Data Contracts: Developing Production-Grade Pipelines at Scale* — Chad Sanderson, Mark Freeman — Data Quality Isn't About Pristine Data › A Brief Summary of OLTP and OLAP
- *Data Contracts: Developing Production-Grade Pipelines at Scale* — Chad Sanderson, Mark Freeman — M › O
- *Cracking the Data Engineering Interview* — Kedeisha Bryan, Taamir Ransome — Must-know foundational database concepts › OLTP versus OLAP databases
- *Analytics Optimization with Columnstore Indexes in Microsoft SQL Server* — Edward Pollack — IntroduCtIon to analytIC data In a transaCtIonal database › OLAP Data in an OLTP Table in a Transactional Database
- *Analytics Optimization with Columnstore Indexes in Microsoft SQL Server* — Edward Pollack — transaCtional vs. analytiC Workloads RANSA TIONAL VS. NALYTI › The Need for Two Systems

