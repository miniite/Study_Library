---
id: "01_Databases_and_SQL/Database_Design/Normalization/Denormalization_in_Lakes"
level: 4
type: "concept"
topic: "01_Databases_and_SQL/Database_Design/Normalization"
tools: []
---

# Denormalization in Lakes

**L4 Advanced** · [↑ Normalization](<_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** In data lakes and lakehouses, denormalization is the default shape of analytical data — you flatten and pre-join in later layers because storage is cheap and scans are columnar — but the raw copy stays untouched and every redundant copy needs a lineage and a rebuild path.

### 1. The big picture
OLTP databases normalize to protect writes. Lakes and warehouses mostly *read*: huge scans, aggregations, ML feature extraction. Joins across billions of rows are expensive (shuffles in a distributed engine, spills to disk), while redundant columns are cheap (columnar compression, object storage). So analytical layers deliberately denormalize.

A lake also has a second kind of redundancy: *whole-copy* redundancy — the landing zone is an exact copy of source data that is then copied again into curated zones. Good design distinguishes **intentional, derived** redundancy (flattened gold tables, rebuilt from silver) from **accidental** redundancy (many unmanaged copies nobody owns).

Related: Denormalization Trade-offs and Patterns (the general toolkit), Normalization Fundamentals, and the warehouse modeling topics (star schema, SCD).

### 2. Core concepts and vocabulary
- **Medallion layers** — bronze (raw, as received), silver (cleaned, conformed, often normalized or vault), gold (business-ready, denormalized/aggregated).
- **Star schema** — central fact table (events/measures) + denormalized dimension tables.
- **OBT / wide table** — a single pre-joined table with facts and dimension attributes.
- **Nested / repeated fields** — arrays and structs inside a row (Parquet, BigQuery, Spark) — a native way to denormalize one-to-many.
- **Slowly changing dimension (SCD)** — technique for keeping history of dimension attributes (type 1 overwrite, type 2 new version row).
- **Columnar storage** — data stored by column; scans read only needed columns; redundant values compress by dictionary/run-length encoding.
- **Data vault** — normalized integration modeling (hubs, links, satellites) in the middle layer.
- **Lineage** — record of which sources and jobs produced a table, needed to trust and rebuild copies.
- **Logical / virtual lake** — federation/views over sources to avoid physical copies.

### 3. How it works, step by step
**Typical flow**
1. *Land* data unchanged (bronze). Immutable, partitioned by ingest date; this is your replay source.
2. *Cleanse and conform* (silver): dedupe, type, apply keys, optionally normalize into 3NF/data vault so one entity = one table.
3. *Model for consumption* (gold): join dimensions onto facts, flatten nested data, aggregate, add pre-computed measures.
4. *Publish* with a schema contract and refresh schedule; consumers query gold.
5. *Rebuild* gold from silver whenever logic or reference data changes — never patch gold in place.

**Where to denormalize which way**

| Need | Shape | Why |
|---|---|---|
| BI dashboards | star schema | simple joins, conformed dims |
| Ad-hoc exploration / ML | wide table or nested records | no joins for analysts |
| Event analytics | flattened events + dim attributes at event time | history preserved |
| Serving API | key-value / document projection | single-key reads |

### 4. Worked example
Silver (normalized):
```sql
-- silver.orders(order_id, customer_id, order_ts, status)
-- silver.order_lines(order_id, product_id, qty, unit_price)
-- silver.customers(customer_id, name, segment, country)
-- silver.products(product_id, name, category)
```
Gold (denormalized fact for BI), e.g. Spark SQL / Delta:
```sql
CREATE OR REPLACE TABLE gold.sales_fact
USING DELTA PARTITIONED BY (order_date) AS
SELECT o.order_id, CAST(o.order_ts AS DATE) AS order_date,
       c.customer_id, c.segment, c.country,
       p.product_id, p.category,
       l.qty, l.qty * l.unit_price AS revenue
FROM silver.orders o
JOIN silver.order_lines l USING (order_id)
JOIN silver.customers  c USING (customer_id)
JOIN silver.products   p USING (product_id);
```
Each row repeats `segment`, `country`, `category`: redundant, but a dashboard now scans one table, prunes by `order_date` partition, and reads just the three columns it needs.

What to watch: if a customer's segment changes, gold either (a) is rebuilt (type-1 behaviour: history rewritten) or (b) keeps the value *as of the order* (type-2 join on validity range), which is usually what finance wants.

Reconciliation test after the build:
```sql
SELECT (SELECT SUM(qty*unit_price) FROM silver.order_lines) AS silver_rev,
       (SELECT SUM(revenue)        FROM gold.sales_fact)     AS gold_rev;  -- must match
```

### 5. Under the hood
- **Cost model.** Distributed joins shuffle data across the network; wide pre-joined tables convert that shuffle to one-time ETL cost. Broadcast joins avoid shuffles when one side is small, so tiny dimensions are cheap even normalized.
- **Columnar compression.** A repeated `country` string over 100M rows compresses to a few bits per row; denormalization cost in bytes is far lower than row-store intuition suggests. Parquet min/max statistics and partition pruning make filters on denormalized columns cheap.
- **Immutability and files.** Object-store files are immutable; updating a copied attribute means rewriting files (copy-on-write) or writing delete vectors/merge-on-read (Delta, Iceberg, Hudi). That's why gold tables are rebuilt or incrementally merged, not row-updated casually.
- **Nested types.** One order with an array of lines in a single row is a denormalization of a one-to-many; queries on lines require `explode`/`UNNEST`, and column pruning works on nested leaf fields.
- **Redundancy of copies.** Frictionless ingestion means the landing zone duplicates the source; every extra copy costs storage and refresh. A virtual/federated approach or table-format features (shallow clones, views) reduce physical duplication but trade away performance isolation from sources.
- **Consistency.** Lakes have no cross-table transactions by default; denormalized tables are consistent only *as of the build*. Table formats give atomic commits per table, not across tables.

### 6. Performance and design implications
- Partition by the common filter (date), cluster/z-order on the next (customer/product), keep files at ~128 MB–1 GB.
- Prefer incremental merges (`MERGE` by key) over full rebuilds for large facts; full rebuild for small dims.
- Keep a normalized silver layer: it is the single place where each fact is corrected once, then propagated.
- Track lineage and freshness (job run timestamp column, SLAs); a stale denormalized table is worse than a slower accurate one.
- Control sprawl: one owner per gold table, naming conventions, deprecation of unused copies.
- Compute cost vs storage cost: pre-joining pays when a table is queried many times per refresh.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `CREATE TABLE AS SELECT` / `INSERT OVERWRITE` | build a denormalized table | rebuild gold |
| `MERGE INTO` (Delta/Iceberg/Snowflake/BigQuery) | incremental upsert | large facts |
| `explode` / `UNNEST` | flatten nested arrays | converting nested to rows |
| Partition by / cluster by / Z-ORDER | prune and co-locate | big gold tables |
| Broadcast hint | avoid shuffle for small dims | joins with small tables |
| dbt tests (`unique`, `relationships`) | validate keys/integrity | warehouses with unenforced constraints |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| Denormalize in gold for BI/ML scans | the same attribute is updated often and consumers need current values at read time |
| Keep silver normalized (3NF/vault) | no one needs integration and the source is already clean — fewer layers |
| Nested fields for one-to-many | consumers/tools can't handle arrays |
| Rebuild from source of truth | upstream is not retained — keep bronze |
| Virtual layer / views | heavy concurrent analytical load on source systems |

### 9. Common mistakes
- Patching gold tables by hand → next rebuild loses the fix → fix upstream and rebuild.
- Denormalizing straight from raw → every consumer redoes cleansing differently → silver first.
- Treating redundant copies as independent datasets → conflicting numbers → define one metric layer and lineage.
- Losing history through type-1 overwrites when the business needs as-of reporting → SCD2.
- Assuming constraints are enforced → duplicates/orphans slip in → tests and reconciliation.
- Unbounded copies ("just copy the table for my project") → cost and governance debt → shared gold products.

### 10. 🎯 Interview kit
**30-second answer:** In a lake, analytical layers are deliberately denormalized — star schemas or wide tables — because scans are columnar and cheap but joins at scale are expensive. I keep raw immutable, normalize or conform in silver, and denormalize in gold with a rebuildable pipeline, lineage and reconciliation checks.

**Structure a longer answer:** access pattern (scans/aggregates) → layering (bronze/silver/gold) → denormalization choices (star, OBT, nested) → history (SCD) → maintenance (merge/rebuild, tests, lineage) → cost trade-offs.

**Likely follow-ups:**
- *Why not just normalize in a warehouse?* → Join and shuffle cost, analyst usability; but keep a normalized core for integrity.
- *How do you handle dimension changes?* → SCD type 2 with effective dates; fact keys to the version valid at the event.
- *What does redundancy cost in a lake?* → Storage plus refresh/lineage burden; compression makes the bytes cheap, governance isn't.
- *Star schema vs OBT?* → Star for reuse and governance; OBT for simplicity and speed on a single use case.

**Signals interviewers look for:** you separate raw/integrated/serving concerns and know that denormalization is owned and rebuildable, not ad hoc.

### 11. Check yourself
- [ ] Explain why denormalized columns are cheaper in Parquet than in a row store.
- [ ] Sketch bronze → silver → gold for an orders dataset and say which layer is normalized.
- [ ] Describe how to keep gold consistent when a dimension attribute changes.
- [ ] Write a reconciliation query between silver and gold.
- [ ] Name three costs of uncontrolled copies in a lake.
<!-- explainer:end -->

## Practice questions

- [Intermediate] [In a Parquet-based data lake why might you keep data denormalized in one wide file?](<../../../PRACTICE/01_Databases_and_SQL/Database_Design/Normalization/Q-34159_In_a_Parquet-based_data_lake_why_might_you_keep_data_denorma.md>)
- [Intermediate] [How do file formats like CSV versus Parquet change how you think about storing denormalized wide data?](<../../../PRACTICE/01_Databases_and_SQL/Database_Design/Normalization/Q-34179_How_do_file_formats_like_CSV_versus_Parquet_change_how_you_t.md>)
- [Intermediate] [In a simple pipeline what steps would you follow to turn one denormalized export into 1NF then 2NF then 3NF?](<../../../PRACTICE/01_Databases_and_SQL/Database_Design/Normalization/Q-34184_In_a_simple_pipeline_what_steps_would_you_follow_to_turn_one.md>)
- [Deep] [When ingesting CDC from a normalized OLTP source into a lakehouse how do you decide whether to rebuild denormalized dimensions with merge or with full reload?](<../../../PRACTICE/01_Databases_and_SQL/Database_Design/Normalization/Q-34245_When_ingesting_CDC_from_a_normalized_OLTP_source_into_a_lake.md>)
- [Deep] [How do you reason about the correctness boundary between a Lambda architecture's batch normalized layer and its speed denormalized layer?](<../../../PRACTICE/01_Databases_and_SQL/Database_Design/Normalization/Q-34079_How_do_you_reason_about_the_correctness_boundary_between_a_L.md>)
- [Deep] [Why might you move from Lambda to Kappa architecture and how does that change your denormalization maintenance model?](<../../../PRACTICE/01_Databases_and_SQL/Database_Design/Normalization/Q-34080_Why_might_you_move_from_Lambda_to_Kappa_architecture_and_how.md>)
- [Expert] [For a GDPR delete, how do you cascade an erasure through denormalized copies that have spread into search indexes caches and analytics extracts?](<../../../PRACTICE/01_Databases_and_SQL/Database_Design/Normalization/Q-34298_For_a_GDPR_delete_how_do_you_cascade_an_erasure_through_deno.md>)
- [Expert] [Why might moving from Lambda to Kappa change how you maintain denormalized aggregates and what do you give up?](<../../../PRACTICE/01_Databases_and_SQL/Database_Design/Normalization/Q-34306_Why_might_moving_from_Lambda_to_Kappa_change_how_you_maintai.md>)

## Related concepts

- [End to End No Data Loss](<../../../04_Data_Architecture_and_Strategy/Architecture_Patterns/Data_Architecture/Tiered_Lake_Storage/End_to_End_No_Data_Loss.md>) *(Data Architecture)*
- [Stream Join Data Structures](<../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Stream_Join_Data_Structures.md>) *(Data Structures and Algorithms)*
- [Unified Batch and Streaming Semantics](<../../../03_Big_Data_and_Streaming/Distributed_Processing/Batch_vs_Stream/Batch_and_Stream_Reconciliation/Unified_Batch_and_Streaming_Semantics.md>) *(Batch vs Stream)*

## References

- *The Enterprise Big Data Lake* — Alex Gorelik — Architecting the Data Lake › Eliminating Redundancy
- *Data Engineering Fundamentals* — Zhaolong Liu — CHAPTER 4Data Transformation and Processing › CHAPTER 4Data Transformation and Processing(#contents.xhtmlch04a)
- *Data Algorithms with Spark* — Mahmoud Parsian — **Serverless Spark** › CHAPTER 2
- *Data Algorithms with Spark* — Mahmoud Parsian — **CHAPTER 3** › CHAPTER 3
- *Data Engineering Design Patterns* — Bartosz Konieczny — Error Management Design Patterns › Pattern: Dynamic Late Data Integrator

