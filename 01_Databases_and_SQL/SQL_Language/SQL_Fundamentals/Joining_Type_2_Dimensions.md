---
id: "01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Joining_Type_2_Dimensions"
level: 4
type: "concept"
topic: "01_Databases_and_SQL/SQL_Language/SQL_Fundamentals"
tools: []
---

# Joining Type 2 Dimensions

**L4 Advanced** · [↑ SQL Fundamentals](<_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** A Type 2 slowly changing dimension keeps one row per *version* of an entity (with surrogate key and valid-from/valid-to dates), so facts must join to the version that was effective at event time — either through a surrogate key stamped at load, or a range-containment join.

### 1. The big picture
Dimension attributes change: a customer moves city, a product is repriced, a rep changes region. If you simply overwrite (Type 1), old facts are re-described with new attributes and history is falsified. A **Type 2** dimension preserves truth: every change creates a new row, so a purchase in year 2 still reports the customer's year‑2 state. Kimball's original motivation: "each transaction corresponds to the person's state at the time of the transaction."

The price is complexity in joins: the *natural key* (customer_id) is no longer unique in the dimension. Join on it naively and you fan out (see Join Fan-out). This lesson is how to model it and how to join correctly.

### 2. Core concepts and vocabulary
- **Natural (business) key** — the identifier from the source system (`customer_id`). Repeats across versions.
- **Surrogate key** — warehouse-generated unique id per dimension row (`customer_sk`). Facts reference this.
- **Effective dates** — `valid_from`, `valid_to` bounding when a version was true.
- **Current flag** — `is_current` (or `valid_to = '9999-12-31'`) marking the latest version.
- **Open-ended row** — current version, whose `valid_to` is a far-future sentinel (preferred) or `NULL`.
- **Type 1 / 2 / 3** — overwrite / add row / add column for previous value.
- **Late-arriving fact** — a fact whose event date is older than the dimension version now current; must map to an older version.
- **Point-in-time (as-of) join** — join each fact to the dimension row valid at the fact's timestamp.

### 3. How it works, step by step
**Loading changes (ETL):**
1. Stage today's source snapshot or change feed.
2. Compare to current dimension rows by natural key, hashing the tracked attributes to detect change.
3. For changed keys: **close** the existing row (`valid_to = change_time`, `is_current = 0`).
4. **Insert** a new row with a new surrogate key, `valid_from = change_time`, `valid_to = sentinel`, `is_current = 1`.
5. Insert brand-new keys as first versions.

**Loading facts — two approaches:**
- *Surrogate-key lookup at load* (preferred): during fact load, look up the version effective at the event date and store `customer_sk` in the fact. Queries then use a plain equi-join on the surrogate key — fast and unambiguous.
- *Range join at query time:* store only the natural key in the fact and join with `f.event_date >= d.valid_from AND f.event_date < d.valid_to`.

### 4. Worked example
```sql
CREATE TABLE dim_customer (
  customer_sk int IDENTITY PRIMARY KEY,
  customer_id int NOT NULL,           -- natural key
  city        varchar(50),
  segment     varchar(20),
  valid_from  date NOT NULL,
  valid_to    date NOT NULL DEFAULT '9999-12-31',
  is_current  bit  NOT NULL DEFAULT 1,
  UNIQUE (customer_id, valid_from)
);

-- Close changed rows, then insert new versions (staging = stg_customer)
UPDATE d SET valid_to = @load_date, is_current = 0
FROM dim_customer d
JOIN stg_customer s ON s.customer_id = d.customer_id
WHERE d.is_current = 1
  AND (d.city <> s.city OR d.segment <> s.segment);

INSERT dim_customer (customer_id, city, segment, valid_from)
SELECT s.customer_id, s.city, s.segment, @load_date
FROM stg_customer s
LEFT JOIN dim_customer d
  ON d.customer_id = s.customer_id AND d.is_current = 1
WHERE d.customer_id IS NULL;      -- no current row => new key or just closed

-- As-of join for facts keyed by natural key
SELECT d.segment, SUM(f.amount) AS revenue
FROM   fact_sales f
JOIN   dim_customer d
  ON   d.customer_id = f.customer_id
 AND   f.sale_date  >= d.valid_from
 AND   f.sale_date  <  d.valid_to       -- half-open interval: no overlaps
GROUP  BY d.segment;
```
Reading it: the half-open interval `[valid_from, valid_to)` guarantees each sale matches exactly one version, because one version's `valid_to` equals the next one's `valid_from`. If you used `BETWEEN` (inclusive both ends), a sale on the boundary date matches **two** versions and revenue double-counts.

Also note the `UPDATE` in the first statement compares with `<>`; if attributes can be NULL, use `NOT EXISTS (… INTERSECT …)` or a row hash, because `NULL <> 'x'` is unknown and the change is missed.

### 5. Under the hood
- **Range joins are hard for optimizers.** Equality on `customer_id` can seek, but the date inequality is a residual predicate; cardinality estimates for inequality ranges are poor, so you may get nested loops over wide version lists. Index `(customer_id, valid_from) INCLUDE (valid_to, …)` so the seek lands on a customer's few versions.
- Stamping the surrogate key at fact load turns a range join into a PK equi-join — which is why warehouses do it. The cost moves to the load, where a *temporal lookup* is performed once.
- **Open-ended `valid_to = NULL`** breaks `f.date < d.valid_to` (comparison with NULL is unknown → row dropped). Use a sentinel (`9999-12-31`) or `COALESCE(d.valid_to, '9999-12-31')`.
- **Overlap/gap constraints** (a row's range overlapping another for the same key) aren't enforced by plain constraints; use exclusion constraints (PostgreSQL `EXCLUDE USING gist` on a range), temporal tables, or ETL tests.
- Lakehouse/medallion note: the Gold layer typically builds SCD2 with `MERGE`, reading the "current" rows via a flag filter before upserting.
- SQL Server **system-versioned temporal tables** keep history automatically and offer `FOR SYSTEM_TIME AS OF`, but track *system* time, not business-effective time — not a replacement for a modelled SCD2 in a warehouse.

### 6. Performance and design implications
- Fact tables keep a narrow surrogate key (int), so joins are cheap; dimension growth is bounded by *changes*, not by facts.
- Use `is_current` filters for "today" reports; use the as-of join only when history matters.
- Detect fan-out: `SELECT COUNT(*) FROM fact` must equal the row count after the dimension join; add this as an automated test.
- Late-arriving facts require the lookup to use the event date, not the load date; late-arriving *dimension* changes may require re-pointing earlier facts.
- Too-frequently-changing attributes (e.g., a balance) don't belong in SCD2 — move them to a mini-dimension or the fact.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `MERGE` / close-and-insert pattern | Maintains versions | Dimension load |
| `HASHBYTES`/row hash column | Cheap change detection | Many tracked columns |
| `valid_from <= d AND d < valid_to` | As-of join | Query-time temporal join |
| `UNIQUE (natural_key, valid_from)` | Prevents duplicate versions | Always |
| `FOR SYSTEM_TIME AS OF` | System-versioned lookup | Audit-style history |
| `dbt snapshot` | Declarative SCD2 | dbt-based warehouses |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| SCD2 for attributes that affect historical reporting | Attribute is corrected typo (use Type 1) |
| Surrogate-key stamp at load | Fact arrives before dimension row (needs inferred member) |
| Query-time range join | Large facts with many versions per key |
| Sentinel `valid_to` | Downstream tools can't handle fake far-future dates |
| Type 3 (previous column) | You need more than one prior value |

### 9. Common mistakes
- Joining on natural key alone → every fact matches all versions → inflated totals → add the date range or use the surrogate key.
- Inclusive `BETWEEN` on both ends → boundary double-count → half-open intervals.
- `NULL` `valid_to` in range predicates → current rows silently dropped → sentinel date.
- Using load date instead of event date in the lookup → facts attributed to the wrong version.
- Forgetting to close the prior row → two "current" rows → unique filtered index on `(customer_id) WHERE is_current = 1`.
- NULL-unsafe change detection → missed updates → hash or null-safe comparison.

### 10. 🎯 Interview kit
**30-second answer:** Type 2 stores a new dimension row per change with a surrogate key and effective dates. Facts carry the surrogate key of the version valid at event time, so history stays accurate; otherwise you join with a date-range containment predicate on the natural key.
**Structure a longer answer:** why history matters → table design (surrogate, natural key, from/to, current flag) → load logic (close + insert) → join patterns → pitfalls (overlaps, NULL end dates, fan-out) → validation tests.
**Likely follow-ups:**
- *Why does my SUM double count after joining a Type 2 dim?* → multiple versions match; constrain to effective row.
- *Why not `BETWEEN`?* → inclusive upper bound matches two rows on boundaries.
- *How would you handle late-arriving facts?* → lookup by event date; re-point if dimension arrives later.
- *Price history with as-of order date: join or denormalize?* → denormalize price onto the order at transaction time for immutability; use history table for analysis.
**Signals interviewers look for:** surrogate vs natural keys, half-open intervals, fan-out awareness, tests and idempotent loads.

### 11. Check yourself
- [ ] Design a Type 2 customer dimension and list every column's purpose.
- [ ] Write the close-and-insert load in two statements.
- [ ] Write the as-of join and explain half-open intervals.
- [ ] Explain what breaks with `valid_to IS NULL` in the join.
- [ ] Describe how surrogate-key stamping avoids query-time range joins.
<!-- explainer:end -->

## Practice questions

- [Basic] [In a data warehouse, what is the difference between a star schema and a snowflake schema, and how does each affect your JOIN queries?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Q-39979_In_a_data_warehouse_what_is_the_difference_between_a_star_sc.md>)
- [Intermediate] [How would you write a query to assign the correct surrogate dimension key to each fact row during load by joining on the natural key?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Q-40046_How_would_you_write_a_query_to_assign_the_correct_surrogate_.md>)
- [Deep] [How would you model Type-2 slowly-changing dimensions so a report reflects the price a customer had at transaction time? What keys and valid-from/valid-to columns do you add?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Q-39916_How_would_you_model_Type-2_slowly-changing_dimensions_so_a_r.md>)
- [Deep] [How would you write SQL to implement a Type 2 slowly changing dimension by closing the old row and inserting a new one with effective dates?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Q-39987_How_would_you_write_SQL_to_implement_a_Type_2_slowly_changin.md>)
- [Deep] [For a Type 2 dimension, how would you write a JOIN that picks the dimension row that was effective at the fact's event date?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Q-40047_For_a_Type_2_dimension_how_would_you_write_a_JOIN_that_picks.md>)
- [Deep] [Your fact-to-dimension SUM double-counts because the dimension is Type-2 and matches multiple versions per fact. How would you constrain the join to the version effective at the fact's event date?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Q-40139_Your_fact-to-dimension_SUM_double-counts_because_the_dimensi.md>)
- [Deep] [How would you write a query to find the effective Type-2 dimension row for each fact using a range containment join on valid_from/valid_to, and why can an open-ended NULL valid_to break the BETWEEN?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Q-40196_How_would_you_write_a_query_to_find_the_effective_Type-2_dim.md>)
- [Deep] [You must model product prices that change over time and support 'price as of order date'. Compare a price-history table with valid ranges joined by date-containment vs. denormalizing the price onto each order. Trade-offs on correctness and reporting.](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Q-40467_You_must_model_product_prices_that_change_over_time_and_supp.md>)

## Related concepts

- [Late Arriving Dimensions](<../../Database_Internals/DBMS_Concepts/Late_Arriving_Dimensions.md>) *(DBMS Concepts)*
- [SCD Type 2 in Practice](<../../../02_Data_Engineering/Modeling_and_Warehousing/Slowly_Changing_Dimensions/SCD_Type_2_in_Practice.md>) *(Slowly Changing Dimensions)*
- [SCD Type 1 vs Type 2](<../../../02_Data_Engineering/Modeling_and_Warehousing/Data_Warehousing/SCD_Type_1_vs_Type_2.md>) *(Data Warehousing)*

## References

- *The Enterprise Big Data Lake* — Alex Gorelik — Historical Perspective › Slowly changing dimensions
- *Data Engineering Best Practices* — Luiz Fernando F. Dos Santos — Data Models › Temporal misalignment
- *Building Medallion Architectures* — Piethein Strengholt — Streamline the Gold Layer › Load the current data from the dimension tables
- *T-SQL Fundamentals (4th ed.)* — Itzik Ben-Gan — Temporal tables

