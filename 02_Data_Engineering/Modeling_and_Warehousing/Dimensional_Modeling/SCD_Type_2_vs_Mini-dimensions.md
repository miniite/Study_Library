---
id: "02_Data_Engineering/Modeling_and_Warehousing/Dimensional_Modeling/SCD_Type_2_vs_Mini-dimensions"
level: 4
type: "concept"
topic: "02_Data_Engineering/Modeling_and_Warehousing/Dimensional_Modeling"
tools: []
---

# SCD Type 2 vs Mini-dimensions

**L4 Advanced** · [↑ Dimensional Modeling](<_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** SCD Type 2 keeps history by adding a new dimension row per change; when attributes change too often or the dimension is huge, split the volatile attributes into a mini-dimension (Type 4) so the main dimension does not explode.

### 1. The big picture
Dimension attributes change: a customer moves city, a product changes category, an employee changes department. A **slowly changing dimension (SCD)** strategy decides what the warehouse does when that happens. The business question is always: *should a report for last year show the attribute as it was then, or as it is now?*

Type 1 (overwrite) answers "as it is now" and loses history. Type 2 (new row) answers "as it was then" and preserves it. Type 2 is the workhorse, but it has a failure mode: attributes that change rapidly (credit band, loyalty tier, fraud score) multiply rows in a large dimension until it is slow and unmanageable. Mini-dimensions, outriggers, and fact-level attributes are the standard remedies.

### 2. Core concepts and vocabulary
- **SCD Type 0** — retain original; never changes (e.g. date of birth).
- **Type 1** — overwrite the attribute; no history.
- **Type 2** — insert a new row with a new surrogate key; old row is closed out with an end date. Facts keep pointing to the version current when they occurred.
- **Type 3** — add a column for the previous value; one level of history.
- **Type 4 / mini-dimension** — move rapidly changing attributes into a separate small dimension; the fact (or a bridge) carries both keys.
- **Type 6 (hybrid 1+2+3)** — Type 2 rows plus a "current value" attribute overwritten on all rows.
- **Effective date / end date / current flag** — columns that delimit a Type 2 row's validity.
- **Row hash** — hash of tracked attributes to detect change cheaply.
- **Band** — grouping a continuous value (income 50,000–60,000) so the number of distinct combinations stays small.
- **Late-arriving dimension / fact** — records that arrive after the time they describe, requiring insertion into history.

### 3. How it works, step by step
**Type 1 vs Type 2 for "customer changes city"**
- Type 1: `UPDATE dim_customer SET city = 'Pune' WHERE customer_id = 'C42'`. All past facts now report Pune.
- Type 2:
  1. Detect change by comparing incoming attributes (or hash) with the current row.
  2. Close the current row: `valid_to = change_date − 1`, `is_current = 0`.
  3. Insert a new row: new `customer_key`, same `customer_id`, `valid_from = change_date`, `valid_to = '9999-12-31'`, `is_current = 1`.
  4. New facts look up the key valid at their event date; old facts keep the old key.

**Mini-dimension (Type 4)**
1. Identify volatile attributes (credit band, age band, income band, loyalty tier).
2. Build `dim_customer_profile` with one row per *distinct combination* of banded values (say 5 × 6 × 4 = 120 rows).
3. Fact rows carry `customer_key` (stable attributes) **and** `profile_key` (the combination in effect at event time).
4. When a customer's band changes, no new customer row is needed; only subsequent facts point to a different profile row.

| Approach | History kept | Dimension growth | Query simplicity | Best for |
|---|---|---|---|---|
| Type 1 | none | none | simplest | corrections, unimportant attributes |
| Type 2 | full | one row per change | simple, needs date-aware join | slow, meaningful changes |
| Type 3 | one previous value | none | simple | "before/after" comparisons |
| Mini-dimension | full for volatile attrs, via facts | bounded by combos | extra join | rapidly changing attributes |
| Fact attribute | per event | none | simplest | very high-velocity values |

### 4. Worked example
```sql
CREATE TABLE dim_customer (
    customer_key  int IDENTITY(1,1) PRIMARY KEY,
    customer_id   varchar(20) NOT NULL,
    name varchar(100), city varchar(60), segment varchar(20),
    row_hash      binary(32) NOT NULL,
    valid_from    date NOT NULL, valid_to date NOT NULL, is_current bit NOT NULL
);
CREATE INDEX ix_dim_customer_nk ON dim_customer (customer_id, valid_from) INCLUDE (valid_to, customer_key);

-- Type 2 in two statements using MERGE output
DECLARE @today date = CAST(GETDATE() AS date);

INSERT dim_customer (customer_id, name, city, segment, row_hash, valid_from, valid_to, is_current)
SELECT s.customer_id, s.name, s.city, s.segment, s.row_hash, @today, '9999-12-31', 1
FROM stg_customer s
LEFT JOIN dim_customer d ON d.customer_id = s.customer_id AND d.is_current = 1
WHERE d.customer_id IS NULL OR d.row_hash <> s.row_hash;     -- new or changed

UPDATE d SET valid_to = DATEADD(day,-1,@today), is_current = 0
FROM dim_customer d
JOIN dim_customer n ON n.customer_id = d.customer_id AND n.is_current = 1 AND n.valid_from = @today
WHERE d.is_current = 1 AND d.valid_from < @today;
```

Point-in-time fact load: `JOIN dim_customer c ON c.customer_id = s.customer_id AND s.order_date BETWEEN c.valid_from AND c.valid_to`. The composite index on `(customer_id, valid_from)` makes that lookup a seek; without it, loading is a scan per row.

Mini-dimension for volatile attributes:

```sql
CREATE TABLE dim_customer_profile (
    profile_key int PRIMARY KEY,
    credit_band varchar(10), income_band varchar(15), loyalty_tier varchar(10)
);
-- the fact references both
CREATE TABLE fact_sales (date_key int, customer_key int, profile_key int, product_key int, amount decimal(12,2));
```

Very high-velocity values (a fraud score updated every second): do not make it a dimension attribute at all — store the score (banded or numeric) on the fact row at event time, or in a separate time-series fact.

### 5. Under the hood
Type 2 changes the *meaning* of the natural key: it is no longer unique in the dimension; uniqueness is `(natural key, valid_from)`. Fact foreign keys resolve to exactly one version, which is why history is preserved with no change to old fact rows. The cost is that "count customers" must use `COUNT(DISTINCT customer_id)` or the `is_current` flag, a common source of wrong answers.

The growth problem is arithmetic: a 50-million-customer dimension where 10% change a tracked attribute monthly adds 5 million rows per month, ~60 million per year, and every ETL run compares incoming rows against a large current partition. Mini-dimensions break the combinatorial growth: the number of rows is capped by the cross-product of banded values, independent of customer count. The price is that the relationship customer ↔ profile is recorded only in facts (no fact in a period → no recorded link); a "customer profile history" bridge or periodic snapshot fills that gap.

Type 6 adds a `current_city` column updated on all of a customer's rows so users can group by "historical city" or "current city" without extra joins; it requires an update pass across versions.

### 6. Performance and design implications
- Index `(natural_key, valid_from)` for lookups; keep `is_current` filtered index for the current-state join.
- Use a row hash (SHA-256 of tracked columns) instead of comparing many columns; separate hashes for Type 1 vs Type 2 attributes.
- Large Type 2 dimensions slow ETL and BI filters; split into mini-dimension, outrigger, or banding before reaching tens of versions per entity.
- Late-arriving facts need the range lookup; late-arriving dimension changes require inserting a version in the middle and re-pointing affected facts (or accepting approximate keys).
- Decide per **attribute**, not per table: one dimension typically mixes Type 1, Type 2 and Type 0 columns.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `MERGE … OUTPUT` | upsert + capture changed rows | Type 2 detection in one pass |
| `HASHBYTES('SHA2_256', …)` / `sha2()` | row hash for change detection | wide dimensions |
| Filtered index `WHERE is_current = 1` | fast current-state lookup | joins to current dimension |
| `BETWEEN valid_from AND valid_to` | point-in-time join | fact loading and as-of reports |
| dbt `snapshot` | managed Type 2 | dbt projects |
| Delta `MERGE` with `whenMatchedUpdate` + insert | lakehouse SCD2 | Databricks/Spark |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| Type 1: corrections, attributes nobody reports historically | Business needs "as was" reporting |
| Type 2: slowly changing, meaningful attributes (region, segment) | Attributes change monthly for most rows in a huge dimension |
| Mini-dimension: volatile banded attributes, large dimension | Values are continuous and cannot be banded |
| Fact-level attribute | Value is inherently per event and not reused |
| Type 3 | Only the previous value matters (old vs new sales territory) |
| Bitemporal / history table (Type 4 in compliance) | Simple reporting is enough; extra complexity |

### 9. Common mistakes
- Using Type 2 on every column → dimension explodes → track only attributes the business reports on.
- Looking up the current row for historical facts → history lost → range-valid lookup.
- `COUNT(*)` on a Type 2 dimension → duplicate customers → `COUNT(DISTINCT natural_key)` or `is_current`.
- Banding at too fine a level → mini-dimension becomes huge → coarser bands.
- Not indexing the validity lookup → slow loads → composite index.

### 10. 🎯 Interview kit
**30-second answer:** Type 1 overwrites and loses history; Type 2 closes the old row and inserts a new one with a new surrogate key, so facts keep the version valid at the time. If an attribute changes too frequently, I move it into a mini-dimension (banded combinations) referenced from the fact, avoiding dimension explosion.

**Structure a longer answer:** business question → Type 1 vs 2 mechanics (columns, keys) → pitfalls (point-in-time join, counts) → scaling problem → mini-dimension/fact attribute → trade-offs.

**Likely follow-ups:**
- *Why a new surrogate key per version?* → so each fact row can point to one version without date logic at query time.
- *When is Type 1 enough?* → correcting a typo, attributes not analysed historically.
- *How do you handle a high-velocity score?* → band it into a mini-dimension or store on the fact/time-series; never SCD2 per change.
- *Type 2 vs Type 4 vs bitemporal for HR?* → Type 2 for reporting; history table/bitemporal when both business time and recorded time must be queryable for audit.

**Signals interviewers look for:** you tie the choice to a reporting need, know the lookup and index, and have a scaling answer.

### 11. Check yourself
- [ ] Show the rows before/after a Type 2 change for one customer.
- [ ] Write the point-in-time join and the index that supports it.
- [ ] Design a mini-dimension for credit band and income band.
- [ ] Explain why `COUNT(*)` is wrong on a Type 2 dimension.
- [ ] Say when a fact-level attribute beats any SCD type.
<!-- explainer:end -->

## Practice questions

- [Basic] [What does 'slowly changing dimension' mean in plain language?](<../../../PRACTICE/02_Data_Engineering/Modeling_and_Warehousing/Dimensional_Modeling/Q-12475_What_does_slowly_changing_dimension_mean_in_plain_language.md>)
- [Intermediate] [In SCD Type 2, how do you know which row is the current version of a customer?](<../../../PRACTICE/02_Data_Engineering/Modeling_and_Warehousing/Dimensional_Modeling/Q-12478_In_SCD_Type_2_how_do_you_know_which_row_is_the_current_versi.md>)
- [Deep] [Explain the difference between SCD Type 1, Type 2, and Type 3.](<../../../PRACTICE/02_Data_Engineering/Modeling_and_Warehousing/Dimensional_Modeling/Q-12159_Explain_the_difference_between_SCD_Type_1_Type_2_and_Type_3.md>)
- [Deep] [What is a slowly changing fact and how does it differ from a slowly changing dimension?](<../../../PRACTICE/02_Data_Engineering/Modeling_and_Warehousing/Dimensional_Modeling/Q-12237_What_is_a_slowly_changing_fact_and_how_does_it_differ_from_a.md>)
- [Deep] [Which columns would you pick as a hash key for change detection in an SCD Type 2 load, and how do you handle NULLs and column additions in the hash?](<../../../PRACTICE/02_Data_Engineering/Modeling_and_Warehousing/Dimensional_Modeling/Q-12296_Which_columns_would_you_pick_as_a_hash_key_for_change_detect.md>)
- [Deep] [How would you implement SCD Type 2 with dbt snapshots, and which edge cases (hard deletes, out-of-order arrivals, timezone-shifted timestamps) have bitten you?](<../../../PRACTICE/02_Data_Engineering/Modeling_and_Warehousing/Dimensional_Modeling/Q-12559_How_would_you_implement_SCD_Type_2_with_dbt_snapshots_and_wh.md>)
- [Deep] [How would you support both a Type 2 history column and a fast current-value column on the same customer dimension without maintaining two tables?](<../../../PRACTICE/02_Data_Engineering/Modeling_and_Warehousing/Dimensional_Modeling/Q-12605_How_would_you_support_both_a_Type_2_history_column_and_a_fas.md>)
- [Expert] [How do you decide between implementing SCD Type 2 versus a separate mini-dimension when an attribute changes very frequently?](<../../../PRACTICE/02_Data_Engineering/Modeling_and_Warehousing/Dimensional_Modeling/Q-12244_How_do_you_decide_between_implementing_SCD_Type_2_versus_a_s.md>)

## Related concepts

- [SCD Type 2 in Practice](<../Slowly_Changing_Dimensions/SCD_Type_2_in_Practice.md>) *(Slowly Changing Dimensions)*

