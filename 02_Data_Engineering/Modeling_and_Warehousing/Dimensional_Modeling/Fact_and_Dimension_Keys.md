---
id: "02_Data_Engineering/Modeling_and_Warehousing/Dimensional_Modeling/Fact_and_Dimension_Keys"
level: 4
type: "concept"
topic: "02_Data_Engineering/Modeling_and_Warehousing/Dimensional_Modeling"
tools: []
---

# Fact and Dimension Keys

**L4 Advanced** · [↑ Dimensional Modeling](<_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** A fact table is made of foreign keys to dimensions plus numeric measures; dimensions hold the descriptive context and are keyed with warehouse-owned surrogate keys that facts reference.

### 1. The big picture
In a Kimball model there are two kinds of tables. **Fact tables** record measurements of business events — what was sold, how many, for how much. **Dimension tables** describe the who, what, where and when of those events. The two are connected only through keys: each fact row carries a foreign key to every dimension that is meaningful at its grain, and a handful of numeric measures.

This design is why analysts can write simple queries: pick measures from the fact, attributes from dimensions, join on keys, group. Understanding what goes where (and how keys are built and looked up during loading) is the foundation for every other dimensional topic: grain, SCDs, conformance, aggregates.

### 2. Core concepts and vocabulary
- **Fact table** — narrow, tall, large; rows are events or states; contains keys and measures.
- **Measure (fact)** — numeric value observed at the grain: quantity, amount, duration. Most are additive.
- **Dimension table** — wide, short, denormalized; descriptive attributes used to filter, group and label.
- **Surrogate key** — a meaningless integer assigned by the warehouse as the dimension's primary key.
- **Natural / business key** — the identifier from the source system (customer number, SKU); kept as an attribute for lookups.
- **Foreign key in fact** — a surrogate key pointing to one dimension row.
- **Degenerate dimension** — an identifier like an order number stored in the fact with no dimension table.
- **Factless fact table** — a fact with only keys; records that an event or relationship occurred (attendance, coverage).
- **Key lookup / surrogate key pipeline** — ETL step that replaces natural keys in incoming facts with the right surrogate keys.
- **Unknown member** — a reserved dimension row (key −1) for missing or not-yet-known references.

### 3. How it works, step by step
**Designing the tables**
1. Declare the grain (one row per order line).
2. For each descriptive question the business asks, define a dimension (date, product, customer, store).
3. Put each dimension's surrogate key in the fact as a foreign key; keep the natural key as a column in the dimension.
4. Put only numeric, event-level values in the fact; textual descriptions belong in dimensions.

**Loading facts (surrogate key pipeline)**
1. Land source rows in staging with natural keys.
2. For each dimension, look up the surrogate key by natural key (for SCD2 dimensions, the version valid at event date).
3. Missing keys → assign the unknown member, or create an "inferred" dimension row and fix it when the dimension arrives.
4. Insert into the fact with all keys populated and no NULLs.

**Dimension keys at load time**
- New natural key → new row, new surrogate key (identity/sequence).
- Existing natural key → update (Type 1) or insert new version (Type 2).

### 4. Worked example
```sql
CREATE TABLE dim_customer (
    customer_key   int IDENTITY(1,1) PRIMARY KEY,   -- surrogate
    customer_id    varchar(20) NOT NULL,            -- natural key from CRM
    name           varchar(100), city varchar(60), segment varchar(20),
    valid_from date, valid_to date, is_current bit
);

CREATE TABLE fact_sales (
    date_key      int NOT NULL,
    customer_key  int NOT NULL,
    product_key   int NOT NULL,
    order_number  varchar(20) NOT NULL,   -- degenerate dimension
    quantity      int NOT NULL,           -- additive measure
    net_amount    decimal(12,2) NOT NULL  -- additive measure
);

-- load with key lookup; missing customer → unknown member (-1)
INSERT fact_sales (date_key, customer_key, product_key, order_number, quantity, net_amount)
SELECT CONVERT(int, FORMAT(s.order_date,'yyyyMMdd')),
       COALESCE(c.customer_key, -1),
       COALESCE(p.product_key, -1),
       s.order_number, s.quantity, s.net_amount
FROM stg_orders s
LEFT JOIN dim_customer c ON c.customer_id = s.customer_id AND s.order_date BETWEEN c.valid_from AND c.valid_to
LEFT JOIN dim_product  p ON p.sku = s.sku AND p.is_current = 1;
```

A factless fact for attendance: `fact_attendance(date_key, student_key, class_key)` — a row means "attended". "Who was absent?" is answered by anti-join: all enrolled students for the class on that date, minus rows in the factless table (so you also need a coverage fact of enrolments).

Columns that stay out of the fact: product name (dimension), profit margin % (non-additive; compute from sums), payment method (small descriptive attribute → dimension or junk dimension).

### 5. Under the hood
The key design separates the *warehouse's* identity from the *source's* identity. Natural keys change (re-used SKUs, merged customers, formats), come from several systems, and are often wide strings; surrogate integers are narrow (4 bytes), stable, and let one natural key have many historical versions. Narrow keys also shrink fact tables and their indexes: a billion-row fact with six 4-byte keys is far smaller than with six varchar keys, and joins compare integers.

Joins from fact to dimension are many-to-one; engines exploit this with **star-join optimisation** (build bitmap or hash filters from filtered dimensions and apply them to the fact scan). Referential integrity between fact and dimensions is usually *not* enforced by foreign key constraints in a warehouse (load cost, load order) — it is guaranteed by the ETL (every fact gets a key, unknowns map to −1) and verified with orphan checks.

Choosing INT vs BIGINT for the fact's *own* key (if it has one): INT tops out at ~2.1 billion; a fast-growing fact can exhaust it, forcing a painful migration, whereas BIGINT costs 4 more bytes per row per index. Dimension keys generally stay INT; a fact rarely needs its own surrogate at all — the grain key (dimension keys + degenerate) is the identity.

### 6. Performance and design implications
- Keep the fact narrow: integer keys, compact numeric types; push descriptive text into dimensions.
- Index/cluster the fact on the date key (or partition by it); in columnstore engines rely on compression and segment elimination.
- Dimension lookup during load should be a hash/in-memory join; make sure the dimension natural-key column is indexed.
- Denormalize a few hot attributes onto the fact only when measured benefit exists (with risk of redundancy and inconsistent updates).
- Orphan check: `SELECT COUNT(*) FROM fact f LEFT JOIN dim d ON d.key=f.key WHERE d.key IS NULL`.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `IDENTITY` / `SEQUENCE` / `GENERATED AS IDENTITY` | assigns surrogate keys | creating dimensions |
| Hash of natural key (`sha2`, `HASHBYTES`) | deterministic keys in distributed/lakehouse loads | parallel, order-independent loads |
| `LEFT JOIN … COALESCE(key, -1)` | unknown-member mapping | fact loading |
| Orphan-check query / dbt `relationships` test | verifies referential integrity | after every load |
| `MERGE` | upserts dimension rows | Type 1 maintenance |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| Sequence surrogate keys: classic RDBMS, single loader | Distributed parallel writers needing coordination |
| Hash keys: lakehouse/Spark, idempotent reloads | Hash collisions or wide hash keys (32 bytes) inflate the fact |
| Natural key directly in fact | One source, never changes, no history needed (rare) |
| Enforced FKs | Bulk loads where speed matters and ETL guarantees integrity |

### 9. Common mistakes
- Natural keys in facts → joins on strings, broken when source recycles IDs → use surrogate keys.
- NULL foreign keys → rows vanish in inner joins → map to unknown member.
- Storing the product name on every fact row → billions of repeated strings, update anomalies → keep in dimension.
- Putting ratios in the fact → un-summable → store components.
- Looking up the *current* dimension version instead of the one valid at event time → history corrupted → use range-valid lookup.

### 10. 🎯 Interview kit
**30-second answer:** A fact table holds foreign keys to dimensions and numeric measures at a declared grain; dimensions hold the descriptive attributes and have warehouse-generated surrogate keys. Loading replaces source natural keys with surrogate keys, mapping missing references to an unknown member.

**Structure a longer answer:** fact vs dimension → grain → keys (surrogate vs natural) → load pipeline → special cases (degenerate, factless, unknown) → performance.

**Likely follow-ups:**
- *Why not use the natural key as the key?* → instability, multiple sources, width, no history support.
- *INT or BIGINT?* → size for growth; dimensions INT, fact identity usually the composite grain key.
- *What's a factless fact?* → keys only; records occurrence; use anti-join with a coverage table.
- *How do you ensure referential integrity?* → ETL guarantees + orphan tests, not necessarily FK constraints.

**Signals interviewers look for:** correct vocabulary, awareness of the SCD2 lookup by date, and unknown-member handling.

### 11. Check yourself
- [ ] List what belongs in a fact table versus a dimension for a retail sale.
- [ ] Explain the surrogate key pipeline for loading facts.
- [ ] Design an attendance factless fact and write the "who was absent" query.
- [ ] Explain why NULL foreign keys are harmful and what to use instead.
- [ ] Decide INT vs BIGINT for a fact with 20 billion expected rows.
<!-- explainer:end -->

## Practice questions

- [Basic] [What is the difference between a fact table and a dimension table?](<../../../PRACTICE/02_Data_Engineering/Modeling_and_Warehousing/Dimensional_Modeling/Q-12143_What_is_the_difference_between_a_fact_table_and_a_dimension_.md>)
- [Basic] [What SQL join type is most common when querying a fact table against its dimensions?](<../../../PRACTICE/02_Data_Engineering/Modeling_and_Warehousing/Dimensional_Modeling/Q-12182_What_SQL_join_type_is_most_common_when_querying_a_fact_table.md>)
- [Basic] [Give a real-world analogy that explains a fact table and a dimension table to a beginner.](<../../../PRACTICE/02_Data_Engineering/Modeling_and_Warehousing/Dimensional_Modeling/Q-12450_Give_a_real-world_analogy_that_explains_a_fact_table_and_a_d.md>)
- [Basic] [Why are fact tables described as 'tall and narrow' and dimensions as 'short and wide'?](<../../../PRACTICE/02_Data_Engineering/Modeling_and_Warehousing/Dimensional_Modeling/Q-12501_Why_are_fact_tables_described_as_tall_and_narrow_and_dimensi.md>)
- [Intermediate] [How would you design a fact table for a payments company tracking transactions?](<../../../PRACTICE/02_Data_Engineering/Modeling_and_Warehousing/Dimensional_Modeling/Q-12201_How_would_you_design_a_fact_table_for_a_payments_company_tra.md>)
- [Intermediate] [Write a SQL query to count orders per customer city joining fact and dimension tables.](<../../../PRACTICE/02_Data_Engineering/Modeling_and_Warehousing/Dimensional_Modeling/Q-12503_Write_a_SQL_query_to_count_orders_per_customer_city_joining_.md>)
- [Deep] [What is a conformed dimension and why is it useful across multiple fact tables?](<../../../PRACTICE/02_Data_Engineering/Modeling_and_Warehousing/Dimensional_Modeling/Q-12161_What_is_a_conformed_dimension_and_why_is_it_useful_across_mu.md>)
- [Deep] [What is a multi-valued dimension and how can it be modeled with a bridge table?](<../../../PRACTICE/02_Data_Engineering/Modeling_and_Warehousing/Dimensional_Modeling/Q-12208_What_is_a_multi-valued_dimension_and_how_can_it_be_modeled_w.md>)

## Related concepts

- [Fact Dimension Integrity](<../Data_Warehousing/Fact_Dimension_Integrity.md>) *(Data Warehousing)*
- [Late Arriving Dimensions](<../../../01_Databases_and_SQL/Database_Internals/DBMS_Concepts/Late_Arriving_Dimensions.md>) *(DBMS Concepts)*
- [Denormalization Trade-offs and Patterns](<../../../01_Databases_and_SQL/Database_Design/Normalization/Denormalization_Trade-offs_and_Patterns/_index.md>) *(Normalization)*

## References

- *Fundamentals of Data Engineering* — Joe Reis, Matt Housley — Techniques for Modeling Batch Analytical Data › Fact tables
- *Fundamentals of Data Engineering* — Joe Reis, Matt Housley — Techniques for Modeling Batch Analytical Data › Dimension tables
- *Building Medallion Architectures* — Piethein Strengholt — Demystifying the Medallion Architecture › Loading the fact tables
- *Building Medallion Architectures* — Piethein Strengholt — Streamline the Gold Layer › Creation of the fact table for sales
- *Building Medallion Architectures* — Piethein Strengholt — Streamline the Gold Layer › Creation of the Semantic Model
- *Building Medallion Architectures* — Piethein Strengholt — E › F
- *Practical Lakehouse Architecture* — Gaurav Ashok Thalpati — The Big Picture Designing and Implementing a Lakehouse Platform › Dimensional modeling

