---
id: "02_Data_Engineering/01_Lifecycle_and_Ingestion/Queries_Modeling_and_Transformation"
level: 4
type: "curriculum"
curriculum: "data_engineering"
---

# Queries Modeling and Transformation

**L4 Advanced** · [↑ Data Engineering Lifecycle and Ingestion](<../_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** Queries read data, transformations persist the results of (often complex, orchestrated) queries for reuse, and data modelling — Inmon, Kimball, data vault, wide tables, or flexible schemas for streams — decides how business logic is shaped into tables.

### 1. The big picture

Ingestion and storage only move and hold data. The *transformation* stage is where raw ingredients become something an analyst, data scientist or application can use. Three ideas work together here:

- **Queries** retrieve and act on data (the R, and with updates the C, U, D of CRUD).
- **Data models** impose structure and business logic on raw data so that many people can interpret it the same way.
- **Transformations** take the logic of the model and the results of queries and *persist* them, so expensive work happens once and downstream users read a simplified result.

Without transformation there is no unified view of the organisation: every user would re-parse, re-clean and re-join from scratch, each in a slightly different way. This lesson is the bridge from the ingestion lessons to the warehousing, dbt and orchestration topics later in the curriculum.

### 2. Core concepts and vocabulary

- **Query** — a request to read (or change) data matching a pattern, e.g. `SELECT … WHERE …`; an OLAP query typically scans large volumes.
- **Transformation** — a query-like operation whose output is *stored* (temporarily or permanently) for further queries or transformations.
- **Data model** — a deliberate representation of data that reflects how the business works.
- **Grain** — what one row of a table represents (one order line, one daily balance).
- **Inmon (top-down)** — build a normalised, enterprise-wide data warehouse first; serve department data marts from it.
- **Kimball (bottom-up)** — model business processes directly as **fact tables** (measurements) surrounded by **dimension tables** (descriptive context) in a **star schema**.
- **Data vault** — insert-only model separating keys, relationships and attributes into **hubs**, **links** and **satellites**; no notion of "conformed" data.
- **Wide denormalised table** — very wide table, often with nested fields, typically in a columnar store.
- **Slowly changing dimension (SCD)** — a dimension whose attributes change over time (Type 2 keeps history rows).
- **Data virtualization / federation** — a query engine (Trino, Presto) that queries external sources without storing the data.
- **Query pushdown** — pushing filters and computation into the source system.
- **Streaming DAG** — a graph of stream enrichments, merges and splits processed continuously.

### 3. How it works, step by step

A typical batch analytical flow:

1. **Land raw data** (from ingestion) with minimal changes.
2. **Clean and conform**: fix types, deduplicate, standardise keys and units.
3. **Model**: reshape into the chosen pattern — facts/dimensions, vault hubs/links/satellites, or a wide table at a defined grain.
4. **Transform**: persist the heavy joins, aggregates and derived metrics as tables or materialised results.
5. **Orchestrate**: run the steps in dependency order on a schedule or on events.
6. **Serve**: expose the modelled tables to BI, notebooks, ML, reverse ETL.

How the major batch modelling styles compare, using an ecommerce order system:

| | Inmon | Kimball | Data vault | Wide table |
|---|---|---|---|---|
| Direction | Top-down, enterprise first | Bottom-up, by business process | Source-aligned, agile | Pragmatic, load and query |
| Structure | Normalised (3NF) warehouse + marts | Star schema: `fact_order_line` + `dim_customer`, `dim_product`, `dim_date` | `hub_customer`, `hub_order`, `link_order_customer`, `sat_order_details` | One `orders_wide` with nested line items |
| Strength | Integration, single version of truth | Fast to deliver, easy for analysts | Auditable history, parallel loading, handles change | Fewest joins, fast scans, easy schema growth |
| Cost | Slow, heavy upfront design | Redundancy, looser integration | Many tables, needs a presentation layer | Less rigour, potential sprawl and unclear semantics |

### 4. Worked example

A Kimball-style star for order analytics, plus the transformation that persists a daily metric.

```sql
-- Dimension and fact tables at the grain "one row per order line"
CREATE TABLE dim_customer (customer_key INT PRIMARY KEY, customer_id VARCHAR(20),
                           segment VARCHAR(20), region VARCHAR(30),
                           valid_from DATE, valid_to DATE, is_current BOOLEAN); -- SCD type 2
CREATE TABLE dim_product  (product_key INT PRIMARY KEY, sku VARCHAR(30), category VARCHAR(40));
CREATE TABLE fact_order_line (
    order_id     BIGINT, line_no INT,
    date_key     INT, customer_key INT, product_key INT,
    quantity     INT, net_amount DECIMAL(12,2)
);

-- Transformation: persist the expensive aggregate once, not per dashboard refresh
CREATE TABLE mart_daily_sales AS
SELECT d.calendar_date, p.category, c.region,
       SUM(f.net_amount) AS revenue, COUNT(DISTINCT f.order_id) AS orders
FROM   fact_order_line f
JOIN   dim_date     d ON d.date_key     = f.date_key
JOIN   dim_product  p ON p.product_key  = f.product_key
JOIN   dim_customer c ON c.customer_key = f.customer_key
GROUP  BY d.calendar_date, p.category, c.region;
```

What to look at: `mart_daily_sales` is a **transformation** — the query result is saved, so dashboards read a small table rather than re-running multi-join aggregation. The SCD Type 2 `dim_customer` keeps the region the customer had at order time. One trap: joining facts at different grains can "fan out" and double-count revenue; always aggregate to a shared grain first.

### 5. Under the hood

**Why wide tables suddenly make sense.** Strict Kimball/Inmon were designed when warehouses were expensive, on-premises and tightly coupled compute/storage. In columnar cloud systems, storage is cheap and a query reads only the columns it selects. Nulls in sparse columns take almost no space, and adding a column is initially only a metadata change (new files are added as data arrives). In a row-store relational database the same wide schema is painful: each row allocates space for every column, and reads load whole rows. Removing joins can sharply improve scan performance; the price is less modelling discipline — be explicit about the grain and keys.

**Why streaming resists classic modelling.** Continuously maintaining a Type 2 slowly changing dimension over an unbounded stream would overwhelm a warehouse, and payload schemas may change at any time (a firmware update adds a field; a CDC feed recasts a datetime as string). Practitioners therefore favour flexible schemas: assume the source is correct as of today, store recent streams together with history so they can be queried jointly, and automate responses to anomalies. Data vault is the most-cited batch model that adapts, because it is insert-only and source-aligned.

**Query vs transformation.** A query retrieves based on filter and join logic; a transformation persists the output. Complex pipelines can be written as one giant query with CTEs, but that quickly becomes unwieldy and inconsistent — which is why transformations are tied to orchestration: discrete steps store intermediate results for downstream steps, increasingly across multiple tables and systems.

**Virtualization and pushdown.** A virtualization engine stores no data; it pulls from sources at query time. Pushdown moves filters and computation to the source to cut data transferred over the network, the main bottleneck. Beware: virtualising a production database does not stop analytics from hitting it — each query still reads production. A safer use is a scheduled extract (e.g., nightly) saved to object storage for downstream consumption.

### 6. Performance and design implications

- **Persist what is expensive and reused**; leave cheap, ad hoc logic as queries.
- **Grain discipline**: define it in writing; mixing grains causes fan-out and double counting.
- **Partition and cluster** large facts by date and common filters; prune scans.
- **Reduce joins** via denormalised marts for dashboards; keep normalised or vault layers for integration and audit.
- **Incremental builds** (process only new partitions) beat full rebuilds on large tables.
- **Measure**: query runtime and bytes scanned, mart refresh time, freshness lag, cost per dashboard.
- **Stakeholder fit**: analysts, data scientists and ML engineers need confidence in quality and completeness; the business needs trust that numbers are actionable.

### 7. Commands and settings you should know

| Command / setting | What it does | When to use |
|---|---|---|
| `CREATE TABLE AS SELECT` / `INSERT … SELECT` | Persists a query result | Building marts and intermediate layers |
| `MERGE` / upsert | Applies inserts and updates by key | Incremental dimension and fact maintenance |
| Materialized view | Stores and refreshes a query result | Repeated aggregates on slowly changing data |
| Partitioning / clustering clauses | Organises storage for pruning | Large fact tables |
| `EXPLAIN` / query profile | Shows scan, join and pushdown behaviour | Tuning slow transformations |
| Federated / external table | Queries data in another system | Virtualisation and one-off integration |
| Orchestrator DAG (Airflow, dbt) | Runs transformations in order | Any multi-step pipeline |

### 8. Trade-offs and decisions

| Choose this when | Avoid / watch out when |
|---|---|
| Inmon — enterprise-wide integration and one version of truth are required | Teams need results in weeks; heavy upfront modelling will stall |
| Kimball — fast delivery of analytics for known business processes | Many sources with constantly changing structure |
| Data vault — many sources, change, audit and parallel loading | Small team needing quick BI without a presentation layer |
| Wide table — columnar cloud store, fast iteration, nested data | Semantics unclear; dozens of teams writing conflicting columns |
| Persist transformation — result reused or expensive | Data changes constantly and freshness beats cost |
| Virtualization — quick access to dispersed sources | Heavy repeated queries against fragile production systems |

### 9. Common mistakes

- **Re-running the same 30-minute query for every report** → cost and delays → persist it as a transformation.
- **Unclear grain** → fan-out, double counting → declare the grain; aggregate before joining.
- **One mega-query with 20 CTEs** → unmaintainable → break into orchestrated, tested steps.
- **Copying batch Kimball patterns onto streams** → heavy SCD processing, brittle → flexible schema, append-only history.
- **Virtualising a production DB for analytics** → same load, new tool → extract off-peak to the lake.
- **Choosing a model by fashion** → friction with the organisation → match the model to team size, change rate and audit needs.
- **Ignoring downstream consumers** → unusable data → agree on definitions and SLAs with analysts and ML users.

### 10. 🎯 Interview kit

**30-second answer:** "A query reads data; a transformation persists the result so expensive logic runs once. Modelling decides how business logic is shaped: Inmon is top-down normalised, Kimball is bottom-up stars of facts and dimensions, data vault is insert-only hubs/links/satellites for agility and audit, and wide tables trade rigour for scan speed in columnar stores. For streams, I keep schemas flexible."

**Structure a longer answer:** clarify (consumers, volume, change rate, audit) → mechanism (land, clean, model, persist, orchestrate) → trade-offs (model choice, normalised vs wide) → scenario.

**Likely follow-ups:**
- *Star schema vs snowflake?* → Star keeps dimensions denormalised for fewer joins; snowflake normalises dimensions to reduce redundancy.
- *Explain SCD Type 2.* → New row per change with validity dates and current flag; facts join on surrogate key.
- *Why are wide tables acceptable in BigQuery-like systems?* → Columnar storage, cheap nulls, column pruning, metadata-only schema changes.
- *Transformation vs view?* → A view re-computes at query time; a transformation materialises.

**Signals interviewers look for:** clarity on grain, knowing when not to model rigidly, awareness of orchestration and incremental processing.

### 11. Check yourself

- [ ] Explain the difference between a query and a transformation.
- [ ] Draw a star schema for orders and state its grain.
- [ ] Describe hubs, links and satellites using customers and orders.
- [ ] Explain why a wide table is cheap in a columnar store but costly in a row store.
- [ ] Explain why Type 2 SCDs are hard to maintain on streams.
- [ ] Describe a safe use of data virtualization with a production database.
<!-- explainer:end -->

## References

- *Fundamentals of Data Engineering* — Joe Reis, Matt Housley — Modeling Streaming Data
- *Fundamentals of Data Engineering* — Joe Reis, Matt Housley — Materialized Views, Federation, and Query Virtualization › Transformations and queries are a continuum
- *Fundamentals of Data Engineering* — Joe Reis, Matt Housley — Techniques for Modeling Batch Analytical Data
- *Fundamentals of Data Engineering* — Joe Reis, Matt Housley — Chapter 8. Queries, Modeling, and Transformation
- *Fundamentals of Data Engineering* — Joe Reis, Matt Housley — Modeling Streaming Data › Transformations
- *Fundamentals of Data Engineering* — Joe Reis, Matt Housley — Techniques for Modeling Batch Analytical Data › Data vault
- *Fundamentals of Data Engineering* — Joe Reis, Matt Housley — Additional Resources
- *Fundamentals of Data Engineering* — Joe Reis, Matt Housley — Chapter 8. Queries, Modeling, and Transformation › Queries
- *Fundamentals of Data Engineering* — Joe Reis, Matt Housley — Upstream Stakeholders › Downstream Stakeholders
- *Fundamentals of Data Engineering* — Joe Reis, Matt Housley — Materialized Views, Federation, and Query Virtualization › Data virtualization

