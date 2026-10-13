---
id: "01_Databases_and_SQL/Database_Strategy/Database_Selection/Choosing_a_Database_for_the_Workload"
level: 5
type: "concept"
topic: "01_Databases_and_SQL/Database_Strategy/Database_Selection"
tools: ["AWS", "Cassandra", "MySQL", "MongoDB"]
---

# Choosing a Database for the Workload

**L5 Expert** · [↑ Database Selection](<_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** Choose a database by modelling the workload first — data shape, access patterns, consistency needs, scale and lifecycle — then let those requirements (plus team skills and total cost) pick the engine, rather than picking a favourite engine and bending the workload to it.

### 1. The big picture

"Which database?" is really several questions: *What is the shape of the data? How is it read and written? How correct must it be, and for how long? How big and how fast will it get? Who will operate it?* The answers separate transactional (OLTP) from analytic (OLAP) needs, structured from semi-structured data, strong from eventual consistency, and general-purpose engines from specialized ones (key-value, document, wide-column, graph, time-series, search, vector).

Transactional and analytic data differ in how they are written, read, stored and managed. An order system creating, updating and reading small sets of rows demands low-latency, ACID (atomic, consistent, isolated, durable) behaviour; reporting that aggregates millions of rows wants columnar storage and scan throughput. Forcing both onto one structure usually hurts one of them, and rebuilding an established system later costs far more than choosing well at the start.

This lesson belongs to *Database Strategy → Database Selection*. It leads into cost optimization (what the choice costs to run) and vendor/platform choices (which product implements the category).

### 2. Core concepts and vocabulary

- **OLTP** — many short read/write transactions on current data; priorities are latency, concurrency and integrity.
- **OLAP / analytic** — large scans and aggregations over history; priorities are throughput and compression.
- **HTAP** — one engine trying to serve both OLTP and OLAP (e.g., TiDB, SingleStore, AlloyDB).
- **Structured / semi-structured / unstructured** — rigid tables; flexible documents (JSON/XML); free-form files.
- **Access pattern** — the list of queries and write paths the application actually issues.
- **Consistency model** — strong, session, eventual; **CAP/PACELC** frame the trade-off between consistency, availability, partition tolerance and, when there is no partition, latency vs consistency.
- **Polyglot persistence** — using several datastores, each for what it does best.
- **System of record** — the authoritative store; derived stores (cache, search, vector, warehouse) can be rebuilt from it.
- **Event store / append-only log** — immutable history of facts; contrasted with mutable current-state rows.
- **CQRS** — separate models for commands (writes) and queries (reads).
- **TCO** — total cost of ownership: licensing, infrastructure, people, migration and opportunity cost.

### 3. How it works, step by step

A selection method you can apply and defend:

1. **Write the workload down.** Entities and relationships; read:write ratio; top ten queries; latency targets; data volume and growth; retention; peak vs average load.
2. **Classify the consistency need per data class.** Money, inventory and identity need strong consistency; product catalogue, feeds and counters often tolerate eventual consistency. Do not demand the strongest level for everything.
3. **Shortlist by category.**
   - Relational/ACID for relationships, joins, constraints and ad-hoc queries.
   - Document for self-contained aggregates with flexible schema.
   - Key-value for sessions, carts, caches and simple lookups at very low latency.
   - Wide-column for massive write throughput with known partition-key access.
   - Graph for deep relationship traversal.
   - Time-series for telemetry with retention/downsampling.
   - Search for relevance ranking and text queries.
   - Columnar warehouse/lakehouse for analytics.
4. **Test the unknowns.** Ask "what query will I need that this store does badly?" (e.g., ad-hoc analytics on a key-value store).
5. **Weigh the non-functional factors.** Team skills, operational surface (on-call, backup, upgrades, security), observability (plans, slow-query introspection, tracing), licensing model, ecosystem and lock-in.
6. **Prototype with realistic data and access patterns**, not vendor benchmarks.
7. **Decide, record the assumptions, and define the exit/trigger** that would make you revisit.

### 4. Worked example

A shopping product needs sessions and carts, orders, product search, and a nightly revenue report.

| Data / need | Store | Why |
|---|---|---|
| Orders, payments, inventory | Relational (ACID) | Multi-row integrity, constraints, auditing |
| Session & cart | Key-value (e.g., Redis-like) | Very low latency, simple get/put, expiry; tolerate loss or rebuild |
| Product search | Search engine | Relevance, facets; fed from the system of record |
| Revenue reporting | Columnar warehouse or replica | Heavy scans must not hurt checkout |

Sample reasoning for the cart: ephemerality is a feature — TTL expiry means abandoned carts clean themselves, and speed matters more than durability. It bites when a node restarts or evicts keys: customers lose carts. Mitigation: persistence options or a periodic snapshot to the relational store, and design the UI so a lost cart is recoverable (saved-for-later).

For the report, pick the lightest option that works:

```sql
-- Heavy aggregation: keep it OFF the OLTP primary
SELECT d.month_start, p.category, SUM(f.amount) AS revenue
FROM   fact_sales f
JOIN   dim_date d    ON d.date_key = f.date_key
JOIN   dim_product p ON p.product_key = f.product_key
GROUP  BY d.month_start, p.category;
```

Escalation path when analytics hurt production: read replica (cheapest) → columnar index or separate columnar store → HTAP engine → full warehouse/lakehouse split. Move down the list only when the previous step fails measurably.

### 5. Under the hood

- **Row vs column storage.** Row stores keep each record's fields together — ideal for fetching/updating one entity. Column stores keep each column together — compress well and read only the columns a query needs, ideal for aggregation. Numeric and date-heavy metrics suit warehouses; large text or binary payloads usually do not.
- **Partitioned stores and access paths.** Key-value, wide-column and many document stores distribute data by partition key; a query by key is a single-node lookup, a query without the key becomes a scatter or is unsupported. This is why "model the access patterns first" matters — and why discovering a new ad-hoc query pattern later hurts (e.g., needing analytics across partition keys in DynamoDB).
- **Consistency in practice.** Behind retrying, idempotent services, eventual consistency can be fine if operations are commutative or deduplicated; non-idempotent money movements need transactions or a saga with compensation.
- **PACELC.** Without a partition, you still trade latency for consistency (the "ELC" half). Geo-distributed systems usually feel this half more often than the partition half.
- **Event store vs mutable model.** An append-only log gives full audit and temporal queries by replay but needs projections for current state; a mutable relational model with history tables or temporal features is simpler for current-state queries. Choose the log when history *is* the product.
- **Embedded caching vs external cache.** In-database caching removes a network hop and a consistency problem; an external cache scales independently and protects the database but introduces invalidation and staleness risk.

### 6. Performance and design implications

- The same engine can be fast or slow depending on whether the access path matches its design; test the worst query, not the average.
- Add read replicas before adding new engines; they solve read scaling with the least complexity.
- Each extra datastore adds backup, upgrade, security and on-call surface; polyglot is a cost you pay deliberately.
- Caching is legitimate when read skew is high and staleness is tolerable; it is a smell when it masks a wrong primary choice.
- A purpose-built time-series engine usually wins on ingest, compression and retention policies for high-frequency telemetry; partitioned relational storage can be enough for moderate volumes and when joins to business data matter.
- Vector search belongs next to, not instead of, the system of record: separate the similarity requirement from the integrity requirement.
- Do not size for a company 100× your scale because an engineering blog did.

### 7. Commands and settings you should know

| Command / setting | What it does | When to use |
|---|---|---|
| Access-pattern table (query → keys → frequency) | Drives engine and key design | Before any selection |
| `EXPLAIN` / execution plans | Reveals scans vs seeks | Validating a candidate engine on real queries |
| Read replica / reader endpoint | Scales reads cheaply | First response to read pressure |
| Columnstore / columnar index | Analytic acceleration in a row store | Mixed workloads on one engine |
| TTL / expiry policies | Self-cleaning data | Sessions, caches, telemetry |
| CDC (change data capture) | Feeds derived stores from the system of record | Search, cache, warehouse sync |
| Load test with production-like data | Real performance evidence | Final shortlist |
| Decision record (ADR) | Documents assumptions and triggers | Every selection |

### 8. Trade-offs and decisions

| Choose this when | Avoid / watch out when |
|---|---|
| **Relational** — relationships, constraints, ad-hoc queries, strong consistency | Massive key-based write volume with trivially simple access |
| **Document** — aggregate fits in one document, schema evolves | Many cross-document joins and multi-document transactions |
| **Key-value** — sessions, carts, cache, simple lookups | Querying by anything other than the key |
| **Wide-column** — huge write throughput, known partition-key reads | Ad-hoc queries; small data a sharded RDBMS handles fine |
| **Graph** — deep traversals dominate | Mostly tabular access (recursive CTEs may suffice) |
| **HTAP** — consolidate OLTP and OLAP to cut data copies | Heavy analytics interfering with latency-critical transactions |
| **Polyglot** — distinct needs justify the operational cost | Small team, thin on-call capacity |
| **Single standard database** — minimal ops surface | Needs a specialized capability the standard cannot meet |
| **Serverless** — spiky, unpredictable demand | Steady load, latency-sensitive wake-ups |
| **Greenfield choice** — freedom to match the workload | **Re-platforming** — team skill, migration cost and inertia weigh heavily |

### 9. Common mistakes

- Picking the trendy engine first → workload bent to fit → write the access patterns first.
- Same consistency for everything → unnecessary cost or silent data issues → classify data by need.
- Using a key-value or wide-column store, then needing ad-hoc analytics → painful migrations → plan an analytic offload (CDC to a warehouse) at selection time.
- Treating a vector database as the system of record → lost integrity → keep a transactional source and derive the vectors.
- Ignoring operational cost of extra stores → on-call overload → count backup/upgrade/security per store.
- Using caching to hide the wrong database → fragile architecture → fix the primary model, cache deliberately.
- Trusting vendor benchmarks → surprises in production → benchmark your own queries and data distribution.
- Ignoring licensing changes (open-source vs source-available vs proprietary) → legal and cost exposure → review license terms and exit paths.

### 10. 🎯 Interview kit

**30-second answer:** "I start from the workload, not the product: data shape, access patterns, consistency per data class, scale and lifecycle. That narrows the category — relational, document, key-value, wide-column, graph, time-series, search or columnar — then I weigh team skills, operational cost and licensing, and I prototype with real queries. I default to fewer datastores and add one only when a requirement justifies its operational surface."

**Structure a longer answer:** clarify workload and constraints → candidate categories → mechanism-level reasons (partitioning, row vs column, consistency) → trade-offs and risks → a concrete scenario (cart, orders, search, reporting).

**Likely follow-ups:**
- *Greenfield vs re-platforming?* → greenfield optimizes for fit; re-platforming weighs migration cost, data gravity, team skills and risk, so incremental strangler approaches often win.
- *HTAP to collapse OLTP/OLAP?* → evaluate isolation between workloads, freshness, cost, maturity and failure modes (analytics starving transactions).
- *Polyglot vs consolidate?* → polyglot when needs differ markedly; consolidate when the operational surface outweighs marginal fit.
- *Event store vs mutable relational?* → log when history and temporal queries are core; relational with history tables when current state dominates.
- *Is caching a bandage?* → legitimate for skewed reads with tolerable staleness; a smell if it hides a wrong store.

**Signals interviewers look for:** requirement-first reasoning, explicit consistency choices, awareness of operational cost, and willingness to say "the boring relational database is enough".

### 11. Check yourself

- [ ] Fill an access-pattern table for a system you know and derive the engine category.
- [ ] Explain row vs column storage and which workload each favours.
- [ ] Argue for and against standardizing on one database.
- [ ] Describe an escalation path from "analytics hurting OLTP" to a lakehouse split.
- [ ] Explain PACELC with a concrete decision.
- [ ] Separate the vector-search requirement from the system-of-record requirement.
<!-- explainer:end -->

## Implementations (tools)

- [AWS](<../../../TOOLS/AWS/_index.md>)
- [Cassandra](<../../../TOOLS/Cassandra/_index.md>)
- [MySQL](<../../../TOOLS/MySQL/_index.md>)
- [MongoDB](<../../../TOOLS/MongoDB/_index.md>)

## Practice questions

- [Intermediate] [When choosing a datastore for user sessions or a shopping cart, why might you deliberately choose ephemerality and a key-value store over durable relational rows? Where does that bite you?](<../../../PRACTICE/01_Databases_and_SQL/Database_Strategy/Database_Selection/Q-21601_When_choosing_a_datastore_for_user_sessions_or_a_shopping_ca.md>)
- [Deep] [What's the difference in how you'd choose a database for a green-field product versus a re-platforming of a legacy monolith, and how does existing team skill and inertia weight the second case?](<../../../PRACTICE/01_Databases_and_SQL/Database_Strategy/Database_Selection/Q-21577_What_s_the_difference_in_how_you_d_choose_a_database_for_a_g.md>)
- [Deep] [How do you fold the observability and debuggability of a database into a selection decision — query plans, slow-query introspection, tracing — and rank two engines you've compared on this axis?](<../../../PRACTICE/01_Databases_and_SQL/Database_Strategy/Database_Selection/Q-21599_How_do_you_fold_the_observability_and_debuggability_of_a_dat.md>)
- [Deep] [How would you decide between an append-only immutable event store and a mutable relational model for a system that needs full audit history and temporal queries?](<../../../PRACTICE/01_Databases_and_SQL/Database_Strategy/Database_Selection/Q-21624_How_would_you_decide_between_an_append-only_immutable_event_.md>)
- [Deep] [How do you decide when to fold a caching/read layer into the primary database (e.g. a NewSQL with built-in caching) versus keeping an external cache tier? What does the coupling buy or cost?](<../../../PRACTICE/01_Databases_and_SQL/Database_Strategy/Database_Selection/Q-21645_How_do_you_decide_when_to_fold_a_caching_read_layer_into_the.md>)
- [Expert] [How would you evaluate an HTAP database (TiDB/SingleStore/AlloyDB) to collapse a separate OLTP and OLAP estate? What are the failure modes of that consolidation?](<../../../PRACTICE/01_Databases_and_SQL/Database_Strategy/Database_Selection/Q-21572_How_would_you_evaluate_an_HTAP_database_TiDB_SingleStore_All.md>)
- [Expert] [How do you decide whether to adopt a polyglot persistence strategy versus consolidating, given each additional datastore adds on-call, backup, upgrade, and security surface?](<../../../PRACTICE/01_Databases_and_SQL/Database_Strategy/Database_Selection/Q-21594_How_do_you_decide_whether_to_adopt_a_polyglot_persistence_st.md>)
- [Expert] [How do you factor a database's licensing model (open-source, SSPL, BSL, proprietary) into a selection decision, especially given recent relicensing events in the industry?](<../../../PRACTICE/01_Databases_and_SQL/Database_Strategy/Database_Selection/Q-21620_How_do_you_factor_a_database_s_licensing_model_open-source_S.md>)

## Related concepts

- [Database Vendor and Platform Choices](<../Vendor_and_Tooling/Database_Vendor_and_Platform_Choices.md>) *(Vendor and Tooling)*
- [Rollups vs On-demand Queries](<../../../04_Data_Architecture_and_Strategy/Architecture_Patterns/Data_Architecture/Tiered_Lake_Storage/Rollups_vs_On-demand_Queries.md>) *(Data Architecture)*
- [Quality Attribute Trade-off Scenarios](<../../../07_Distributed_Systems_and_Design/Software_Architecture/Trade_off_Analysis/Quality_Attribute_Trade-off_Scenarios.md>) *(Trade off Analysis)*
- [Warehouse as System of Record](<../../../04_Data_Architecture_and_Strategy/Architecture_Patterns/Data_Architecture/Layered_Pipelines_Bronze_Silver_Gold/Warehouse_as_System_of_Record.md>) *(Data Architecture)*
- [Database Cost Optimization](<../Cost_Optimization/Database_Cost_Optimization.md>) *(Cost Optimization)*

## References

- *Azure Data Fundamentals* — Michael John Pena — Introduction to Data Representation › Exam Tip
- *Azure Data Fundamentals* — Michael John Pena — Introduction to Data Representation › Making an Informed Decision About Data Representation
- *Azure Data Fundamentals* — Michael John Pena — Introduction to Data Representation › Practical Tips for Evaluating Data Structure Needs
- *Azure Data Fundamentals* — Michael John Pena — Data Storage Fundamentals › Relational Databases: The Reliable Foundation
- *Analytics Optimization with Columnstore Indexes in Microsoft SQL Server* — Edward Pollack — IntroduCtIon to analytIC data In a transaCtIonal database › Scaling Decisions and Strategy
- *Analytics Optimization with Columnstore Indexes in Microsoft SQL Server* — Edward Pollack — IntroduCtIon to analytIC data In a transaCtIonal database › CHAPTER 2
- *AWS Certified Data Engineer Associate Study Guide* — Sakti Mishra et al. — Data Modeling Strategies for Data Lakes › Conclusion
- *Database Reliability Engineering* — Laine Campbell, Charity Majors — Introducing Database Reliability Engineering › Chapter 1. Introducing Database Reliability Engineering
- *Database Reliability Engineering* — Laine Campbell, Charity Majors — Introducing Database Reliability Engineering › Guiding Principles of the DBRE
- *Database Reliability Engineering* — Laine Campbell, Charity Majors — Introducing Database Reliability Engineering › Hierarchy of Needs
- *Database Reliability Engineering* — Laine Campbell, Charity Majors — Infrastructure Engineering › Chapter 5. Infrastructure Engineering
- *Cracking the Data Engineering Interview* — Kedeisha Bryan, Taamir Ransome — Must-have skillsets to showcase in your portfolio › Data storage
- *Cracking the Data Engineering Interview* — Kedeisha Bryan, Taamir Ransome — Must-know advanced database concepts
- *Cracking the Data Engineering Interview* — Kedeisha Bryan, Taamir Ransome — Technical interview questions
- *Cracking the Data Engineering Interview* — Kedeisha Bryan, Taamir Ransome — Database Design and Optimization
- *Learning Spark (2nd ed.)* — Jules S. Damji, Brooke Wenig, Tathagata Das, Denny Lee — Building Reliable Data Lakes with Apache Spark › Databases
- *Learning Spark (2nd ed.)* — Jules S. Damji, Brooke Wenig, Tathagata Das, Denny Lee — Building Reliable Data Lakes with Apache Spark › Limitations of Databases

