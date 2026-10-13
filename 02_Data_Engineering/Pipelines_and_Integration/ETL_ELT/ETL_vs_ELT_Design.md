---
id: "02_Data_Engineering/Pipelines_and_Integration/ETL_ELT/ETL_vs_ELT_Design"
level: 4
type: "concept"
topic: "02_Data_Engineering/Pipelines_and_Integration/ETL_ELT"
tools: ["Python"]
---

# ETL vs ELT Design

**L4 Advanced** · [↑ ETL ELT](<_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** ETL transforms data before loading it into the target; ELT loads raw data first and transforms inside the target platform, and modern cloud warehouses and lakehouses make ELT the usual default.

### 1. The big picture
Every pipeline has three activities: **extract** data from sources, **transform** it (clean, conform, join, aggregate, model), and **load** it where it will be used. The order in which transform and load happen defines two patterns.

- **ETL**: transform on a separate processing engine (an ETL server, Spark cluster, custom code), then load only the finished data into the warehouse. This was dominant when warehouse storage and compute were scarce and expensive.
- **ELT**: land raw data into the warehouse or lake first, then transform with the target's own engine (SQL, dbt, Spark SQL). Cheap, elastic storage and scalable compute made this practical, and it keeps raw data available for reprocessing.

Neither is universally right. The choice interacts with governance (what may be stored raw), cost, team skills, latency and how often logic changes.

### 2. Core concepts and vocabulary
- **Extract** — read from sources (queries, files, APIs, logs).
- **Transform** — cleaning, type casting, deduplication, joins, business rules, aggregation, modelling.
- **Load** — write to the target (append, overwrite, merge).
- **Staging area** — intermediate storage used between extract and load/transform.
- **Raw / bronze, cleaned / silver, curated / gold** — medallion layers: raw landed data, validated and conformed data, business-ready data.
- **Pushdown** — executing transformations inside the data store's engine rather than moving data out.
- **Reverse ETL** — pushing curated warehouse data back into operational tools.
- **Schema-on-write vs schema-on-read** — enforce structure when writing (ETL, warehouses) vs interpret structure when reading (lakes, ELT raw layers).
- **Orchestrator** — the scheduler/dependency manager (Airflow, Dagster, cloud-native) that runs steps in order.

### 3. How it works, step by step
**ETL**
1. Extract from sources to the ETL engine.
2. Transform in the engine: cleanse, join, aggregate, apply business rules.
3. Load finished tables into the warehouse.
4. Raw data is often discarded or kept only in a staging file area.

**ELT**
1. Extract (often via CDC or connectors).
2. Load raw data into raw tables/files with minimal change.
3. Transform inside the warehouse/lakehouse using SQL models in layers (raw → staging → marts), version-controlled and tested.
4. Serve curated tables to BI, ML and applications.

| Aspect | ETL | ELT |
|---|---|---|
| Where transform runs | Separate engine | Target platform |
| Raw data kept | Often not | Yes |
| Reprocessing after logic change | Re-extract from source | Re-run SQL over raw |
| Sensitive data handling | Mask before load easily | Must control raw zone access |
| Flexibility for new questions | Limited to what was modelled | High |
| Compute cost location | ETL cluster | Warehouse credits/compute |

### 4. Worked example
Daily revenue by region from an orders database.

**ELT with dbt-style SQL**

```sql
-- raw.orders loaded by ingestion tool
-- staging
CREATE OR REPLACE VIEW stg.orders AS
SELECT order_id, customer_id, CAST(amount AS DECIMAL(12,2)) AS amount,
       CAST(created_at AS TIMESTAMP) AS created_at, UPPER(currency) AS currency
FROM raw.orders
WHERE order_id IS NOT NULL;

-- mart
CREATE OR REPLACE TABLE marts.daily_revenue AS
SELECT DATE(o.created_at) AS day, c.region, SUM(o.amount) AS revenue
FROM stg.orders o JOIN stg.customers c USING (customer_id)
GROUP BY 1, 2;
```

When finance changes the revenue definition to exclude refunds, you edit one SQL model and re-run it over the retained raw data; no source re-extract is needed. In an ETL design the same change might require rewriting the transform job and re-pulling history from the source (which may no longer hold it).

A case for ETL: personal data must be tokenized *before* leaving a regulated system; the transform runs in a controlled environment and only the masked output is loaded.

### 5. Under the hood
- **Why ELT became feasible.** Columnar warehouses separate storage and compute, scale elastically, and execute set-based SQL on massive data far faster than row-by-row external engines. Moving data out to transform and back in is wasteful.
- **Layering and testing.** ELT pipelines apply software practices: models in version control, tests (not null, unique, accepted values, relationships), documentation, CI. Layers isolate concerns: staging models only rename/cast, intermediate models join, marts expose business entities.
- **Cost model.** In ELT you pay warehouse compute for every transform; poorly written models and over-frequent runs raise bills. Incremental models (process only new data) and partition pruning control cost.
- **Hybrid reality.** Many shops do light ETL at the edge (masking, format conversion, file compaction, streaming enrichment) and heavy ELT in the warehouse. Streaming ingestion may do light in-flight transformation, with raw retained for replay.
- **Data lake variants.** Raw files in object storage plus engines like Spark/Trino/Databricks give ELT on lakes; open table formats (Delta, Iceberg, Hudi) add ACID, merge and time travel.

### 6. Performance and design implications
- ELT: speed depends on warehouse sizing, clustering/partitioning, and concurrency; schedule heavy models off-peak or on separate compute.
- ETL: speed depends on the engine; moving large volumes across the network can dominate.
- Idempotent, rerunnable steps are essential in both (see the lessons on idempotent loads and safe re-runs).
- Freshness vs cost: transforms on every micro-batch cost more; decide per mart.
- Observability: track run duration, rows in/out per step, test failures, freshness and cost per model.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `dbt run`, `dbt build`, `dbt test` | Run, build and test SQL models | ELT transformation layer |
| Incremental model config | Process only new/changed data | Large fact tables |
| `COPY INTO` / bulk load | Fast load of files into warehouse | Landing raw data |
| Warehouse sizing / auto-suspend | Control compute spend | Cost management |
| Orchestrator DAG dependencies | Order extract → load → transform | All pipelines |
| Masking policies / row access policies | Protect raw zone | ELT with sensitive data |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| ELT: elastic warehouse/lakehouse, SQL-skilled team, need reprocessing and agility | Raw data cannot legally be stored in the target |
| ETL: strict pre-load masking/validation, limited target compute, heavy non-SQL processing (images, complex parsing) | You need frequent logic changes over full history |
| Hybrid: light edge transforms + warehouse modelling | Duplicated logic in both places drifting apart |
| Event-driven or streaming transforms | Batch is adequate and simpler |

### 9. Common mistakes
- Loading raw sensitive data to a wide-open schema → exposure → restrict raw, mask views, or transform before load.
- Treating raw as curated → analysts query unvalidated data → expose only modelled layers.
- Full-refreshing huge models daily → cost blow-up → incremental models and partition pruning.
- Embedding business logic in many places → inconsistent metrics → single modelled layer / semantic layer.
- Dropping raw after transforming → cannot fix mistakes → retain raw per retention policy.

### 10. 🎯 Interview kit
**30-second answer:** ETL transforms before loading; ELT loads raw first and transforms in the warehouse. ELT dominates now because cloud warehouses separate cheap storage from elastic compute, keep raw data for reprocessing, and let SQL-based, tested, version-controlled models do the work. I choose ETL when data must be masked or validated before landing or when processing isn't SQL-friendly.

**Structure a longer answer:** definitions → drivers (cost, scale, agility, governance) → example → trade-offs → hybrid → a decision for the stated scenario.

**Likely follow-ups:**
- *Where do you do PII masking in ELT?* → exclude/mask at ingestion or restrict raw zone and expose masked views.
- *Reprocessing after a business-rule change?* → re-run models over retained raw.
- *Cost control?* → incremental models, scheduling, warehouse sizing.
- *Medallion layers?* → raw, cleaned/conformed, curated.

**Signals interviewers look for:** nuanced "it depends" with criteria, and operational details (testing, cost, governance).

### 11. Check yourself
- [ ] Draw ETL and ELT flows and mark where raw data lives.
- [ ] Give two scenarios where ETL is still the right choice.
- [ ] Explain why ELT enables cheap reprocessing.
- [ ] Describe three layers of models and what each may do.
- [ ] Name three cost controls in an ELT warehouse.
<!-- explainer:end -->

## Implementations (tools)

- [Python](<../../../TOOLS/Python/_index.md>)

## Practice questions

- [Basic] [What does ETL stand for and what happens in each of the three stages?](<../../../PRACTICE/02_Data_Engineering/Pipelines_and_Integration/ETL_ELT/Q-24748_What_does_ETL_stand_for_and_what_happens_in_each_of_the_thre.md>)
- [Basic] [What SQL techniques would you use in the transform step to aggregate daily transactions by customer?](<../../../PRACTICE/02_Data_Engineering/Pipelines_and_Integration/ETL_ELT/Q-24812_What_SQL_techniques_would_you_use_in_the_transform_step_to_a.md>)
- [Intermediate] [How would you extract data from a REST API in a Python ETL script and handle pagination?](<../../../PRACTICE/02_Data_Engineering/Pipelines_and_Integration/ETL_ELT/Q-24754_How_would_you_extract_data_from_a_REST_API_in_a_Python_ETL_s.md>)
- [Intermediate] [What is a data catalog and how does it help teams find and trust datasets produced by ETL?](<../../../PRACTICE/02_Data_Engineering/Pipelines_and_Integration/ETL_ELT/Q-24802_What_is_a_data_catalog_and_how_does_it_help_teams_find_and_t.md>)
- [Intermediate] [How would you design the extract layer to be polite to a rate-limited third-party API while still ingesting millions of records a day?](<../../../PRACTICE/02_Data_Engineering/Pipelines_and_Integration/ETL_ELT/Q-24876_How_would_you_design_the_extract_layer_to_be_polite_to_a_rat.md>)
- [Deep] [What is the difference between a stateless and a stateful transformation?](<../../../PRACTICE/02_Data_Engineering/Pipelines_and_Integration/ETL_ELT/Q-24792_What_is_the_difference_between_a_stateless_and_a_stateful_tr.md>)
- [Deep] [Your team debates ETL vs ELT for a new HIPAA-style dataset with strict PII rules; how does compliance change the answer?](<../../../PRACTICE/02_Data_Engineering/Pipelines_and_Integration/ETL_ELT/Q-24877_Your_team_debates_ETL_vs_ELT_for_a_new_HIPAA-style_dataset_w.md>)
- [Deep] [How do you decide whether to fan out one source into many marts or build one wide model, given query patterns and maintenance cost?](<../../../PRACTICE/02_Data_Engineering/Pipelines_and_Integration/ETL_ELT/Q-24916_How_do_you_decide_whether_to_fan_out_one_source_into_many_ma.md>)

## Related concepts

- [Pipeline Design Patterns](<../Pipeline_Design/Pipeline_Design_Patterns.md>) *(Pipeline Design)*
- [File and Table Formats for Features](<../Pipeline_Orchestration/File_and_Table_Formats_for_Features.md>) *(Pipeline Orchestration)*
- [Transform in Pipeline vs Warehouse](<../Pipeline_Design/Transform_in_Pipeline_vs_Warehouse.md>) *(Pipeline Design)*
- [Choosing a Database for the Workload](<../../../01_Databases_and_SQL/Database_Strategy/Database_Selection/Choosing_a_Database_for_the_Workload.md>) *(Database Selection)*
- [Data Warehouse Fundamentals](<../../Modeling_and_Warehousing/Data_Warehousing/Data_Warehouse_Fundamentals.md>) *(Data Warehousing)*
- [Warehouse as System of Record](<../../../04_Data_Architecture_and_Strategy/Architecture_Patterns/Data_Architecture/Layered_Pipelines_Bronze_Silver_Gold/Warehouse_as_System_of_Record.md>) *(Data Architecture)*

## References

- *Data Quality Fundamentals* — Barr Moses, Lior Gavish, Molly Vorwerck — Collecting, Cleaning, Transforming, and Testing Data › Running Analytical Data Transformations
- *Data Quality Fundamentals* — Barr Moses, Lior Gavish, Molly Vorwerck — Collecting, Cleaning, Transforming, and Testing Data › Ensuring Data Quality During Transformation
- *Practical Lakehouse Architecture* — Gaurav Ashok Thalpati — Introduction to Lakehouse Architecture › Data processing and transformations
- *Practical Lakehouse Architecture* — Gaurav Ashok Thalpati — Introduction to Lakehouse Architecture › What Are ETL and ELT?
- *Practical Lakehouse Architecture* — Gaurav Ashok Thalpati — Introduction to Lakehouse Architecture › ETL/ELT workloads
- *Practical Lakehouse Architecture* — Gaurav Ashok Thalpati — Compute Engines for Lakehouse Architectures › Tools for data consumption
- *Building Data Integration Solutions* — Jay Borthen — Models, Architectures, Methods, and Patterns › Extract, transform, load
- *Building Data Integration Solutions* — Jay Borthen — Models, Architectures, Methods, and Patterns › Extract, load, transform
- *Building Data Integration Solutions* — Jay Borthen — Models, Architectures, Methods, and Patterns › Data Replication and Propagation Pattern
- *Cracking the Data Engineering Interview* — Kedeisha Bryan, Taamir Ransome — Must-have skillsets to showcase in your portfolio › Data processing
- *Cracking the Data Engineering Interview* — Kedeisha Bryan, Taamir Ransome — Summary › Part 3: Essentials for Data Engineers Part II
- *Cracking the Data Engineering Interview* — Kedeisha Bryan, Taamir Ransome — Technical interview questions › Data Processing and ETL
- *Cracking the Data Engineering Interview* — Kedeisha Bryan, Taamir Ransome — Fundamental concepts
- *Fundamentals of Data Engineering* — Joe Reis, Matt Housley — The Data Engineering Lifecycle › Transformation
- *Fundamentals of Data Engineering* — Joe Reis, Matt Housley — The Data Engineering Lifecycle › Reverse ETL
- *Fundamentals of Data Engineering* — Joe Reis, Matt Housley — Designing Good Data Architecture › Examples and Types of Data Architecture
- *Fundamentals of Data Engineering* — Joe Reis, Matt Housley — Ingestion › ETL Versus ELT
- *97 Things Every Data Engineer Should Know* — Tobias Macey (ed.) — Data Engineering for Autonomy and Rapid Innovation › Implement Reusable Patterns in the ETL Framework
- *97 Things Every Data Engineer Should Know* — Tobias Macey (ed.) — Five Best Practices for Stable Data Processing What to Keep in Mind When Setting Up Processes Like › Chapter 35. Five Best Practices for Stable Data Processing: What to Keep in Mind When Setting Up Processes Like ETL or ELT
- *97 Things Every Data Engineer Should Know* — Tobias Macey (ed.) — Focus on Maintainability and Break Up Those ETL Tasks › Chapter 36. Focus on Maintainability and Break Up Those ETL Tasks
- *97 Things Every Data Engineer Should Know* — Tobias Macey (ed.) — Scaling ETL How Data Pipelines Evolve as Your Business Grows › Chapter 62. Scaling ETL: How Data Pipelines Evolve as Your Business Grows

