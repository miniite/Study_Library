---
id: "02_Data_Engineering/Modeling_and_Warehousing/Data_Warehousing/Distribution_Keys"
level: 4
type: "concept"
topic: "02_Data_Engineering/Modeling_and_Warehousing/Data_Warehousing"
tools: ["AWS", "Snowflake"]
---

# Distribution Keys

**L4 Advanced** · [↑ Data Warehousing](<_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** In an MPP warehouse, the distribution key decides which node stores each row; choosing it well keeps joins local and work evenly spread, and choosing it badly moves data across the network and creates hot nodes.

### 1. The big picture
An MPP (massively parallel processing) warehouse splits each table across many compute nodes/slices so that scans and aggregations run in parallel. The question is *which rows go where*. If two big tables are joined on `customer_id` and rows with the same `customer_id` live on the same node in both tables, each node joins its own data with no network traffic — a **collocated join**. If not, the engine has to redistribute or broadcast rows at query time, which is often the dominant cost.

So distribution design is about two competing goals: **locality** (matching rows co-located for the common joins and aggregations) and **balance** (no node holds a disproportionate share). It is mostly a concern of engines with explicit distribution (Amazon Redshift, Azure Synapse dedicated SQL pool, Greenplum, Teradata). Snowflake and BigQuery manage placement automatically and expose clustering/partitioning instead.

### 2. Core concepts and vocabulary
- **Node / slice / distribution** — unit of storage and parallel processing (Redshift slices; Synapse's 60 distributions).
- **Distribution style** — how rows are assigned: by key hash, round-robin, replicated, or automatically.
- **Distribution key** — the column whose hash chooses the node.
- **Collocated join** — join where matching rows are already on the same node.
- **Data movement / redistribution** — shuffling or broadcasting rows between nodes during a query.
- **Skew** — uneven row distribution across nodes, so the slowest node dictates time.
- **Sort key** — physical ordering within a node to speed range filters and merge joins (separate from distribution).
- **Cardinality** — number of distinct values; high cardinality hashes evenly.
- **Broadcast** — copy a small table to every node for a join.

### 3. How it works, step by step
Using Amazon Redshift terms:
1. **DISTSTYLE EVEN** — rows are dealt round-robin across slices. Balanced, but no locality: joins on any column require redistribution. Good for tables not joined to big tables.
2. **DISTSTYLE KEY** (`DISTKEY(col)`) — hash of `col` picks the slice. Rows with the same value share a slice, so a join on that column between two tables with the same DISTKEY is collocated.
3. **DISTSTYLE ALL** — a full copy of the table on every node. Joins with it never need movement. Suitable for small, rarely changing dimensions; costly for big or frequently updated tables.
4. **DISTSTYLE AUTO** — the service picks and may change the style as the table grows (ALL for small, EVEN/KEY later).

Choosing a key for a star schema:
1. List the large fact tables and the joins that matter most (by frequency and size).
2. Pick the **join column of the largest joined pair** with **high cardinality** and **even distribution**.
3. Make the other large table use the same column (and data type) as its DISTKEY.
4. Distribute small dimensions with ALL.
5. Verify skew and the query plan; adjust.

In Azure Synapse dedicated pools the equivalents are `HASH(col)`, `ROUND_ROBIN`, and `REPLICATED`.

### 4. Worked example
```sql
-- Large fact and large customer-order table joined constantly on customer_id
CREATE TABLE fact_sales (
  sale_id     BIGINT,
  customer_id INT,
  product_id  INT,
  date_key    INT,
  amount      DECIMAL(12,2)
)
DISTSTYLE KEY DISTKEY (customer_id)
SORTKEY (date_key);

CREATE TABLE dim_customer (
  customer_id INT,
  name        VARCHAR(100),
  segment     VARCHAR(20)
)
DISTSTYLE KEY DISTKEY (customer_id);        -- same key/type → collocated join

CREATE TABLE dim_product (
  product_id INT,
  category   VARCHAR(50)
)
DISTSTYLE ALL;                               -- small, replicate everywhere

-- Inspect plan and skew
EXPLAIN
SELECT c.segment, SUM(f.amount)
FROM fact_sales f JOIN dim_customer c ON c.customer_id = f.customer_id
GROUP BY c.segment;
-- look for DS_DIST_NONE (no movement) on the join

SELECT "table", diststyle, skew_rows, size
FROM svv_table_info
WHERE "table" IN ('fact_sales','dim_customer');
```
How to read the output: in `EXPLAIN` the join step label tells you what happens — `DS_DIST_NONE` (collocated, ideal), `DS_DIST_ALL_NONE` (inner table is ALL, also free), `DS_BCAST_INNER` (inner broadcast to all nodes: fine if small, costly if large), `DS_DIST_INNER`/`DS_DIST_OUTER`/`DS_DIST_BOTH` (rows redistributed over the network). In `svv_table_info`, `skew_rows` is the ratio of rows on the fullest slice to the emptiest; values far above 1 (e.g., > 2–4) indicate a poor key.

### 5. Under the hood
- Rows are placed by hashing the key; the hash result maps to a slice. Hashing implies that **NULLs and heavily repeated values all land on the same slice** — a single dominant `customer_id` (or many NULLs) creates a hot slice regardless of the table's total size.
- A join is collocated only when both inputs are distributed on the *join columns* and the key types match; joining on a different column than the DISTKEY forces movement of at least one side.
- Distribution is physical placement; **sort keys** order rows within each slice and let zone maps skip blocks. A good design uses distribution for join locality and sort key for filter selectivity (typically date).
- Redistribution writes data to the network and temporary storage, so large moves show up as long steps and heavy interconnect usage.
- Changing distribution later requires a rewrite (`ALTER TABLE ... ALTER DISTSTYLE` or CTAS), which can be expensive on large tables.
- Snowflake micro-partitions and BigQuery's partition/cluster are different mechanisms: they prune data but don't expose node placement.

### 6. Performance and design implications
- Collocated joins on big tables can be orders of magnitude faster than redistributing billions of rows.
- Skew makes a cluster as slow as one hot node: balance matters as much as locality.
- Don't distribute on a filter column (like `date`): all of one day's rows land on one slice and queries on that day serialize.
- A key good for one fact table may not suit another; choose by the dominant, expensive join, and let secondary joins redistribute small tables or broadcast.
- Use AUTO when query patterns are unknown or evolving; revisit once workloads stabilize.
- Measure: `EXPLAIN` data-movement labels, `svv_table_info.skew_rows`, `SVL_QUERY_SUMMARY`/step metrics, concurrency and queue times.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `DISTSTYLE KEY DISTKEY(col)` | Hash-distribute by column | Large tables joined on same column |
| `DISTSTYLE ALL` | Replicate on every node | Small, stable dimensions |
| `DISTSTYLE EVEN` | Round-robin | No dominant join, staging tables |
| `DISTSTYLE AUTO` | Service-managed | Default for new tables |
| `SORTKEY(col)` | Physical ordering | Range filters, time series |
| `EXPLAIN` | Show join/data-movement strategy | Verifying collocation |
| `SVV_TABLE_INFO` | Style, skew, size | Auditing design |
| Synapse `HASH`, `ROUND_ROBIN`, `REPLICATED` | Equivalent options | Azure dedicated pools |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| KEY on high-cardinality join column | Few distinct values or one dominant value (skew) |
| ALL for small dimension | Table is large or updated often (storage and load cost multiply) |
| EVEN for staging/independent tables | You later join it to big tables on a common column |
| AUTO for evolving workloads | You need predictable, tuned performance at scale |
| Same key on two big tables | Keys differ in type or the join isn't dominant |

### 9. Common mistakes
- Distributing on a low-cardinality column (country, status) → few hot slices → choose a high-cardinality column.
- Mismatched key data types → no collocation → align types.
- DISTKEY = filter column (date) → single-slice scans → use as sort key instead.
- ALL on a multi-hundred-million-row table → slow loads, wasted storage → KEY or EVEN.
- Ignoring NULLs in the key → all NULLs on one slice → handle or choose another column.

### 10. 🎯 Interview kit
**30-second answer:** The distribution key determines which node each row lives on. In an MPP warehouse you want the two big tables in a frequent join distributed on that join column so it is collocated, keep the key high-cardinality to avoid skew, replicate small dimensions with ALL, and use EVEN or AUTO when there's no dominant join.

**Structure a longer answer:** clarify (workload, biggest joins) → mechanism (KEY/EVEN/ALL/AUTO, collocation, data movement) → trade-offs (locality vs balance) → scenario (join taking an hour due to DS_DIST_BOTH).

**Likely follow-ups:**
- *Distribution key vs sort key?* → placement across nodes vs ordering within nodes.
- *How do you detect skew?* → `skew_rows`, uneven slice metrics, one slow step.
- *What if a table joins on two different columns?* → pick the more expensive join; broadcast/replicate the smaller one.
- *How do Snowflake/BigQuery differ?* → no user distribution keys; micro-partitions/clustering/partitioning.

**Signals interviewers look for:** understands data movement cost, can read EXPLAIN, balances locality and skew.

### 11. Check yourself
- [ ] Explain collocated join in your own words.
- [ ] Choose distribution styles for a fact and three dimensions.
- [ ] Interpret `DS_BCAST_INNER` vs `DS_DIST_NONE`.
- [ ] Explain why NULLs and hot values cause skew.
<!-- explainer:end -->

## Implementations (tools)

- [AWS](<../../../TOOLS/AWS/_index.md>)
- [Snowflake](<../../../TOOLS/Snowflake/_index.md>)

## Practice questions

- [Basic] [What is a star schema and why is it commonly used in data warehouses?](<../../../PRACTICE/02_Data_Engineering/Modeling_and_Warehousing/Data_Warehousing/Q-18973_What_is_a_star_schema_and_why_is_it_commonly_used_in_data_wa.md>)
- [Intermediate] [What are the main advantages and disadvantages of a snowflake schema compared to a star schema?](<../../../PRACTICE/02_Data_Engineering/Modeling_and_Warehousing/Data_Warehousing/Q-18976_What_are_the_main_advantages_and_disadvantages_of_a_snowflak.md>)
- [Intermediate] [What is the role of a business key versus a surrogate key when matching records across loads?](<../../../PRACTICE/02_Data_Engineering/Modeling_and_Warehousing/Data_Warehousing/Q-19032_What_is_the_role_of_a_business_key_versus_a_surrogate_key_wh.md>)
- [Intermediate] [How would you design a simple star schema for an e-commerce orders reporting use case?](<../../../PRACTICE/02_Data_Engineering/Modeling_and_Warehousing/Data_Warehousing/Q-19065_How_would_you_design_a_simple_star_schema_for_an_e-commerce_.md>)
- [Deep] [What is a dimensional model versus a normalized (3NF) model and when is each appropriate?](<../../../PRACTICE/02_Data_Engineering/Modeling_and_Warehousing/Data_Warehousing/Q-19031_What_is_a_dimensional_model_versus_a_normalized_3NF_model_an.md>)
- [Deep] [What is Data Vault modeling and how do hubs, links, and satellites differ from the star schema?](<../../../PRACTICE/02_Data_Engineering/Modeling_and_Warehousing/Data_Warehousing/Q-19034_What_is_Data_Vault_modeling_and_how_do_hubs_links_and_satell.md>)
- [Deep] [Explain a case where you chose a snowflake schema over a star schema in production and how it affected join cost and BI usability.](<../../../PRACTICE/02_Data_Engineering/Modeling_and_Warehousing/Data_Warehousing/Q-19072_Explain_a_case_where_you_chose_a_snowflake_schema_over_a_sta.md>)
- [Deep] [What is your strategy for handling duplicate surrogate keys after two parallel dimension loads race each other?](<../../../PRACTICE/02_Data_Engineering/Modeling_and_Warehousing/Data_Warehousing/Q-19130_What_is_your_strategy_for_handling_duplicate_surrogate_keys_.md>)

## Related concepts

- [Late Arriving Dimensions](<../../../01_Databases_and_SQL/Database_Internals/DBMS_Concepts/Late_Arriving_Dimensions.md>) *(DBMS Concepts)*
- [Composite vs Surrogate Keys](<../../../01_Databases_and_SQL/Database_Internals/DBMS_Concepts/Composite_vs_Surrogate_Keys.md>) *(DBMS Concepts)*
- [Data Vault](<../Modeling_Methodologies/Data_Vault.md>) *(Modeling Methodologies)*

## References

- *AWS Certified Data Engineer Associate Study Guide* — Sakti Mishra et al. — Overview of AWS Analytics and Auxiliary Services › Amazon Redshift
- *AWS Certified Data Engineer Associate Study Guide* — Sakti Mishra et al. — SQL-Based Data Transformation Using Amazon Redshift
- *AWS Certified Data Engineer Associate Study Guide* — Sakti Mishra et al. — SQL-Based Data Transformation Using Amazon Redshift › Amazon Redshift stored procedures
- *AWS Certified Data Engineer Associate Study Guide* — Sakti Mishra et al. — Designing Data Models and Schema
- *Ultimate AWS Data Engineering* — Rathish Mohan — Exploring the Realm of AWS Data Storage Services › Amazon Redshift: The Data Warehousing Champion
- *Ultimate AWS Data Engineering* — Rathish Mohan — Exploring the Realm of AWS Data Storage Services › Introducing Amazon Redshift: Scalable Data Warehousing in the Cloud
- *Ultimate AWS Data Engineering* — Rathish Mohan — Data Warehousing Demystified › Part 2: Building and Querying Your Redshift Data Warehouse
- *Ultimate AWS Data Engineering* — Rathish Mohan — Data Warehousing Demystified › Part 3: Advanced Techniques and Best Practices Taking Your Redshift Data Warehouse to the Next Level
- *Azure Data Fundamentals* — Michael John Pena — Data Visualization with Power BI › Data Transformation and Preparation
- *Data Pipelines Pocket Reference* — James Densmore — Data Ingestion Loading Data › Loading Data into a Redshift Warehouse
- *Data Pipelines Pocket Reference* — James Densmore — Measuring and Monitoring Pipeline Performance › R
- *Data Engineering Fundamentals* — Zhaolong Liu — CHAPTER 9Real-world Data Engineering Use Cases › CHAPTER 9Real-world Data Engineering Use Cases(#contents.xhtmlch09a)
- *Database Reliability Engineering* — Laine Campbell, Charity Majors — Datastore Field Guide › The key--value model

