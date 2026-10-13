---
id: "02_Data_Engineering/Modeling_and_Warehousing/Dimensional_Modeling/OLTP_vs_Star_Schema_Performance"
level: 4
type: "concept"
topic: "02_Data_Engineering/Modeling_and_Warehousing/Dimensional_Modeling"
tools: []
---

# OLTP vs Star Schema Performance

**L4 Advanced** · [↑ Dimensional Modeling](<_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** OLTP schemas are normalized for fast, correct small writes; star schemas are denormalized for fast large reads — so running analytics on the OLTP model is slow, risky, and hard to understand.

### 1. The big picture
The same business can be stored two ways. An **OLTP** (online transaction processing) database supports the application: place an order, update stock, record a payment. It is normalized (usually third normal form) so each fact lives in one place, writes are short, and constraints keep data consistent under high concurrency. An **OLAP/warehouse** model supports analysis: scan millions of rows, group by several attributes, compare periods. It is denormalized into stars so queries need few joins and read data in large sequential chunks.

A skeptical stakeholder asks, "Why not just report from the OLTP replicas?" The honest answer is a set of trade-offs: complexity, performance, history, isolation, and consistency of definitions. A replica gives you the current state in the application's shape; a star schema gives you history, conformed meaning and predictable performance in the analyst's shape.

### 2. Core concepts and vocabulary
- **OLTP** — transactional workload: many concurrent short reads/writes touching few rows, point lookups by key.
- **OLAP** — analytical workload: few concurrent users, long scans and aggregations over many rows.
- **Normalization (3NF)** — organising tables to remove redundancy and update anomalies; many narrow tables joined by keys.
- **Star schema** — central fact surrounded by denormalized dimensions; few, simple joins.
- **Snowflake schema** — a star whose dimensions are partly normalized into sub-tables.
- **One Big Table (OBT)** — a fully flattened table of facts plus all dimension attributes.
- **Row store vs column store** — rows stored together (good for OLTP), or each column stored together (good for scans/aggregates).
- **Workload isolation** — keeping heavy reads away from the production database.
- **Slowly changing history** — warehouse tracks how attributes changed; OLTP typically only holds current values.

### 3. How it works, step by step
**What an analytic question looks like on OLTP**
1. "Revenue by product category and customer region last quarter" requires joining orders → order lines → products → categories → customers → addresses → regions: 6–8 tables.
2. The optimizer must pick join order and algorithms with imperfect cardinality estimates across many joins.
3. Indexes built for point lookups don't help scans over a quarter of data; the engine reads huge portions of several tables.
4. The query competes for CPU, I/O, buffer cache and locks with the application.

**The same question on a star**
1. Filter `dim_date` (quarter), group by `dim_product.category` and `dim_customer.region`.
2. One fact scan, three small dimension joins.
3. Columnar storage reads only the needed columns; partition pruning reads only the quarter.

| Dimension | OLTP (3NF) | Star schema |
|---|---|---|
| Goal | correct, concurrent updates | fast, understandable reads |
| Typical query | by key, few rows | scans millions, aggregates |
| Joins per query | many (depth of normalization) | few (one per dimension) |
| History | current state, overwritten | versioned (SCD) |
| Redundancy | minimised | deliberate in dimensions |
| Storage | row store, B-tree indexes | columnar, partitioned |
| Optimizer behaviour | complicated join-order search | recognises star join, simple plans |

### 4. Worked example
The same report in both models:

```sql
-- OLTP: eight joins, current-state attributes only
SELECT r.region_name, c.category_name, SUM(ol.qty * ol.unit_price) AS revenue
FROM orders o
JOIN order_lines ol ON ol.order_id = o.order_id
JOIN products p     ON p.product_id = ol.product_id
JOIN categories c   ON c.category_id = p.category_id
JOIN customers cu   ON cu.customer_id = o.customer_id
JOIN addresses a    ON a.address_id = cu.default_address_id
JOIN regions r      ON r.region_id = a.region_id
WHERE o.order_date >= '2026-04-01' AND o.order_date < '2026-07-01'
GROUP BY r.region_name, c.category_name;

-- Star: three joins, history-correct attributes
SELECT cu.region, p.category, SUM(f.net_amount) AS revenue
FROM fact_sales f
JOIN dim_date d     ON d.date_key = f.date_key
JOIN dim_product p  ON p.product_key = f.product_key
JOIN dim_customer cu ON cu.customer_key = f.customer_key
WHERE d.fiscal_quarter = 'FY26-Q1'
GROUP BY cu.region, p.category;
```

Check the difference with `SET STATISTICS IO ON` and the execution plan: the OLTP version shows scans of `orders` and `order_lines` and a long chain of nested loops or hash joins with large estimates errors; the star version should show a columnstore scan with partition elimination and a few hash joins to small dimensions.

A deeper difference the SQL hides: if a customer moves region, the OLTP query reports **all** their past orders under the new region (current address only). The star, with SCD Type 2 on `dim_customer`, keeps past orders under the old region.

### 5. Under the hood
- **Join count and estimation:** each extra join multiplies estimation error; with 7 joins optimizers can pick poor plans, causing spills and unpredictable runtimes. A star has a recognisable shape (one big table, many small dimension joins), enabling dedicated star-join strategies — build hash tables or bitmaps from filtered dimensions, push them as filters into the fact scan.
- **Storage layout:** OLTP row stores read whole rows including columns the report doesn't need; column stores read only needed columns and compress them heavily (similar values, run-length and dictionary encoding), often 5–10× fewer bytes read.
- **Concurrency:** analytic scans hold shared locks or generate row versions (snapshot isolation), increasing tempdb/version store pressure and delaying writers; replicas avoid this but lag, and still run on a schema not designed for scans.
- **Semantic stability:** the OLTP schema changes with application releases; the warehouse isolates consumers from that via an ETL contract.
- **Data quality:** OLTP stores whatever the application accepted; the warehouse's ETL deduplicates, standardises and conforms values.

### 6. Performance and design implications
- A star should reduce join depth and read volume by an order of magnitude; verify by comparing logical reads and elapsed time on representative queries.
- Wide dimensions are cheap because they're small; denormalizing them avoids joins with little storage cost. Denormalizing into the fact (OBT) increases scanned bytes and update cost, though columnar compression softens that.
- Use a materialized view or aggregate table if the same rollup runs repeatedly.
- If a materialized view refresh is slow and blocks readers, diagnose by checking whether it's a full recompute, what locks it takes, and switch to incremental or partition-swap refresh.
- For machine-generated OBT views per consumer: virtualise over the star when freshness and consistency matter and compute is cheap; materialise when the same expensive flattening is hit constantly and staleness is acceptable.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `SET STATISTICS IO, TIME ON` | logical reads and time | comparing OLTP vs star query cost |
| Actual execution plan | shows join order, scans, estimates | diagnosing OLTP analytic queries |
| `READ COMMITTED SNAPSHOT` / replicas | reduces reader-writer blocking | when forced to report from OLTP |
| `CREATE CLUSTERED COLUMNSTORE INDEX` | columnar storage on fact | star facts |
| Partitioning by date | partition elimination | large facts |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| Report straight from an OLTP replica: small data, few reports, need near-real-time, no history needed | Data is large, queries are heavy, or definitions must be consistent across teams |
| Star schema: many reports, history, multiple sources, consistent KPIs | Data is tiny or one-off analysis; building ETL isn't worth it |
| Snowflake schema | Dimensions rarely need normalization; joins add little benefit |
| OBT for one consumer | Multiple consumers need the same dimension meaning |

### 9. Common mistakes
- Running heavy reports on the primary OLTP database → lock/IO contention slows the app → use replica or warehouse.
- Believing a replica solves the model problem → still deep joins and current-state-only → build a dimensional layer.
- Over-normalizing the warehouse → analysts face the same join maze → denormalize dimensions.
- Ignoring history → reports change retroactively → use SCD Type 2.
- Flattening everything into the fact → massive scans and update cost → keep dimensions separate.

### 10. 🎯 Interview kit
**30-second answer:** OLTP is normalized for fast concurrent writes, so analytics on it needs many joins and large scans competing with the application. A star schema denormalizes dimensions around a central fact, so queries need few joins, can use columnar storage and partition pruning, and keep history.

**Structure a longer answer:** workload contrast → why analytics hurt OLTP (joins, scans, contention) → how star helps (join shape, storage, history, definitions) → trade-offs and when direct replica reporting is fine → measure with plans and reads.

**Likely follow-ups:**
- *How does a star affect the optimizer?* → recognisable many-to-one star-join shape, fewer joins, better estimates, bitmap/hash filters.
- *Why not just add indexes to OLTP?* → they slow writes and don't fix deep joins or history.
- *Materialized view vs star?* → MVs speed specific rollups; the star provides the general model.
- *OBT obsoletes dimensional modeling?* → wins for single-consumer simplicity; costs in duplication, update anomalies, and inconsistent definitions at scale.

**Signals interviewers look for:** quantitative thinking (join count, reads), mention of history, and a balanced "it depends".

### 11. Check yourself
- [ ] List four reasons not to report straight from OLTP.
- [ ] Explain why a star's join shape helps the optimizer.
- [ ] Convert a six-table OLTP report query to a star query.
- [ ] Explain how a customer address change distorts OLTP-based historical reports.
- [ ] Decide when an OBT is acceptable.
<!-- explainer:end -->

## Practice questions

- [Basic] [What are OLAP and OLTP, and which schema style suits each?](<../../../PRACTICE/02_Data_Engineering/Modeling_and_Warehousing/Dimensional_Modeling/Q-12171_What_are_OLAP_and_OLTP_and_which_schema_style_suits_each.md>)
- [Basic] [What is denormalization and why is it acceptable in a warehouse but avoided in OLTP?](<../../../PRACTICE/02_Data_Engineering/Modeling_and_Warehousing/Dimensional_Modeling/Q-12511_What_is_denormalization_and_why_is_it_acceptable_in_a_wareho.md>)
- [Intermediate] [Why are OLTP databases usually normalized but OLAP data warehouses often use star schemas?](<../../../PRACTICE/02_Data_Engineering/Modeling_and_Warehousing/Dimensional_Modeling/Q-12151_Why_are_OLTP_databases_usually_normalized_but_OLAP_data_ware.md>)
- [Intermediate] [How is a surrogate key different from an auto-increment primary key in an OLTP table?](<../../../PRACTICE/02_Data_Engineering/Modeling_and_Warehousing/Dimensional_Modeling/Q-12466_How_is_a_surrogate_key_different_from_an_auto-increment_prim.md>)
- [Intermediate] [Why does a normalized OLTP schema make analytical queries slower than a star schema?](<../../../PRACTICE/02_Data_Engineering/Modeling_and_Warehousing/Dimensional_Modeling/Q-12512_Why_does_a_normalized_OLTP_schema_make_analytical_queries_sl.md>)
- [Intermediate] [A stakeholder asks why the warehouse cannot just query the OLTP tables directly; how do you explain the dimensional-modeling payoff with concrete trade-offs?](<../../../PRACTICE/02_Data_Engineering/Modeling_and_Warehousing/Dimensional_Modeling/Q-12278_A_stakeholder_asks_why_the_warehouse_cannot_just_query_the_O.md>)
- [Intermediate] [How do you convert an existing normalized OLTP schema into a star schema; what is your step-by-step approach for identifying facts and dimensions?](<../../../PRACTICE/02_Data_Engineering/Modeling_and_Warehousing/Dimensional_Modeling/Q-12319_How_do_you_convert_an_existing_normalized_OLTP_schema_into_a.md>)
- [Intermediate] [How do you measure whether a star schema redesign actually improved query performance rather than relying on intuition?](<../../../PRACTICE/02_Data_Engineering/Modeling_and_Warehousing/Dimensional_Modeling/Q-12328_How_do_you_measure_whether_a_star_schema_redesign_actually_i.md>)

## Related concepts

- [Separating OLTP and Analytics](<../OLTP_vs_OLAP/Separating_OLTP_and_Analytics/_index.md>) *(OLTP vs OLAP)*
- [Physical Design for OLTP vs OLAP](<../OLTP_vs_OLAP/Physical_Design_for_OLTP_vs_OLAP.md>) *(OLTP vs OLAP)*

## References

- *AWS Certified Data Engineer Associate Study Guide* — Sakti Mishra et al. — Chapter 5. Data Store Management › AWS Cloud Databases
- *Designing Data-Intensive Applications* — Martin Kleppmann — Storage and Retrieval › The divergence between OLTP databases and data warehouses
- *Building Medallion Architectures* — Piethein Strengholt — Demystifying the Medallion Architecture › Optimizing loads
- *Analytics Optimization with Columnstore Indexes in Microsoft SQL Server* — Edward Pollack — transaCtional vs. analytiC Workloads RANSA TIONAL VS. NALYTI
- *The Enterprise Big Data Lake* — Alex Gorelik — Historical Perspective › Massively parallel processing (MPP) systems

