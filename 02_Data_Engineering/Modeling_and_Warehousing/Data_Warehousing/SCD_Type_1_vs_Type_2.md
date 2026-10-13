---
id: "02_Data_Engineering/Modeling_and_Warehousing/Data_Warehousing/SCD_Type_1_vs_Type_2"
level: 4
type: "concept"
topic: "02_Data_Engineering/Modeling_and_Warehousing/Data_Warehousing"
tools: []
---

# SCD Type 1 vs Type 2

**L4 Advanced** · [↑ Data Warehousing](<_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** A slowly changing dimension decides what happens to a descriptive attribute when it changes: Type 1 overwrites it (history lost), Type 2 adds a new row (history kept).

### 1. The big picture
Dimension attributes change over time: a customer moves from the USA to the UK, a product is re-categorized, an employee changes department. Source systems usually keep only the current value. If the warehouse also keeps only the current value, an old order will suddenly be reported in the customer's *new* country — rewriting the past. If the business needs "orders by the country the customer lived in at the time," history must be preserved.

Slowly changing dimension (SCD) patterns are the standard answers. **Type 1** overwrites; **Type 2** versions the row. Both are common; the choice is per attribute and driven by the question "does anyone need the old value?"

### 2. Core concepts and vocabulary
- **Natural/business key** — the source identifier (`customer_id`); repeated across versions.
- **Surrogate key** — warehouse-generated key, unique per dimension *row* (per version in Type 2).
- **Type 0** — retain original; never changes.
- **Type 1** — overwrite in place; no history.
- **Type 2** — new row per change with validity range and current flag.
- **Type 3** — add a column holding the previous value (limited history).
- **Type 6 (hybrid)** — Type 2 rows that also carry the current value of selected attributes.
- **Effective dates** — `valid_from` / `valid_to` (or `start_date` / `end_date`) bounding when the row was true.
- **`is_current` flag** — marks the latest version per business key.
- **Row hash** — a hash over tracked attributes used to detect change cheaply.
- **Point-in-time lookup** — joining a fact to the dimension version valid at the fact's event date.

### 3. How it works, step by step
**Type 1:**
1. Match incoming row to dimension by business key.
2. If attributes differ, `UPDATE` them in place.
3. If not found, `INSERT`.
Result: one row per business key; facts show the *current* attribute for all history.

**Type 2:**
1. Compute a hash of the tracked attributes for incoming rows and for current dimension rows.
2. For new business keys: insert with `valid_from` = effective date, `valid_to` = a far-future date (e.g., 9999-12-31), `is_current` = 1.
3. For changed keys (hash differs): **close** the current row (`valid_to` = change date − 1 or change timestamp, `is_current` = 0) and **insert** a new row with a new surrogate key.
4. For unchanged keys: do nothing.
5. During fact loading, look up the surrogate key whose validity range contains the fact's date (or `is_current = 1` if loading only the latest).
Result: each fact is permanently tied to the attributes in force at the time.

Source behavior matters: if the source gives full refreshes, change history is lost between loads (you detect only differences between snapshots); if it gives incremental/CDC changes, you can capture every intermediate version.

### 4. Worked example
Customer 100 moves from USA to UK on 2020-06-20.

| customer_key | customer_id | country | valid_from | valid_to | is_current |
|---|---|---|---|---|---|
| 1 | 100 | USA | 2019-01-01 | 2020-06-19 | 0 |
| 7 | 100 | UK  | 2020-06-20 | 9999-12-31 | 1 |

```sql
-- Step A: close changed current rows
MERGE INTO dw.dim_customer AS d
USING (
  SELECT s.customer_id, s.country, s.name,
         MD5(CONCAT_WS('|', s.country, s.name)) AS row_hash,
         s.effective_date
  FROM stg.customer s
) AS s
ON d.customer_id = s.customer_id AND d.is_current = 1
WHEN MATCHED AND d.row_hash <> s.row_hash THEN
  UPDATE SET valid_to = DATEADD(day, -1, s.effective_date), is_current = 0;

-- Step B: insert new versions (new keys + changed keys that were just closed)
INSERT INTO dw.dim_customer (customer_id, country, name, row_hash, valid_from, valid_to, is_current)
SELECT s.customer_id, s.country, s.name,
       MD5(CONCAT_WS('|', s.country, s.name)), s.effective_date, DATE '9999-12-31', 1
FROM stg.customer s
LEFT JOIN dw.dim_customer d ON d.customer_id = s.customer_id AND d.is_current = 1
WHERE d.customer_id IS NULL;

-- Fact analysis by country at time of order
SELECT c.country, SUM(f.net_amount)
FROM dw.fact_orders f
JOIN dw.dim_customer c ON c.customer_key = f.customer_key   -- key resolved at load time
GROUP BY c.country;
```
With Type 2, an order placed on 2020-07-12 carries surrogate key 7 (UK), and an order from 2019 stays on key 1 (USA). With Type 1, both would report UK. Note the two-step shape (close, then insert) — some engines use a single `MERGE` with a union trick; the order matters so you don't insert before the old row is closed.

### 5. Under the hood
- **Why surrogate keys:** the business key repeats across versions, so facts need a per-version identifier. Facts store the surrogate key; the historical attribute lookup is a plain integer join.
- **Hashing:** comparing a single hash is faster and simpler than comparing every attribute, and null-safe if you coalesce values with a sentinel before hashing. Use a stable delimiter and normalization (trim, case) to avoid false changes.
- **Mixing types:** apply Type 2 only to attributes the business wants history for; track Type 1 attributes (typos, formatting fixes) separately — a Type 1 correction should update *all* versions of that business key rather than creating a new row.
- **Deletes:** sources that remove a record need a policy (soft-delete flag, closing the current row, or ignore).
- **Out-of-order/late changes:** inserting a change dated before the current version requires splitting an interval — rebuild the affected key's history rather than appending.
- **Bronze/silver placement:** SCD2 is typically built in silver/dimension layers from raw history; defining what counts as a "change" can differ between source and target, which is why business keys and hashes are central.

### 6. Performance and design implications
- Type 2 dimensions grow over time; a customer table with frequent attribute churn can balloon — track only attributes that need history.
- Point-in-time joins use a range predicate (`BETWEEN valid_from AND valid_to`), which is slower than equality; resolve to the surrogate key during fact load so queries use a simple equality join.
- Current-state queries should filter `is_current = 1` (or use a view) to avoid double counting versions.
- Index/sort/cluster by business key and `valid_from` for efficient close-out merges.
- Metrics: version count per key, change rate by attribute, merge duration.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `MERGE` / `UPDATE` + `INSERT` | Close and open versions | SCD2 loads |
| Row hash (`MD5`, `SHA2`, `HASH`) | Detect attribute change | Large dimensions |
| `valid_from`, `valid_to`, `is_current` | Version window and flag | All Type 2 tables |
| Current-view (`WHERE is_current=1`) | Simple consumer access | BI tools |
| Delta/Snowflake streams, CDC | Provide change events | Capturing every version |
| ADF Mapping Data Flows / dbt snapshots | Prebuilt SCD2 patterns | Tool-driven pipelines |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| Type 1 — corrections, attributes nobody analyzes historically | Reports must reproduce past values |
| Type 2 — history matters (region, segment, price tier) | Attribute changes constantly; table explodes |
| Type 3 — only "previous vs current" needed | More than one prior value is needed |
| Type 6 — need both historical and current-attribute reporting | Complexity outweighs benefit |
| Full-refresh source with snapshot diffing | You need every intermediate change (use CDC instead) |

### 9. Common mistakes
- Using `is_current = 1` when joining historical facts → past reported with current attributes → resolve by date at load.
- Forgetting to close the old row → two "current" rows → duplicates and double sums → close-then-insert in one transaction + uniqueness test.
- Hashing without null handling → false changes → coalesce and normalize.
- Applying Type 2 to every column → bloated dimension → track only needed attributes.
- Treating a correction as a change → fake history → separate corrections (Type 1 across versions).

### 10. 🎯 Interview kit
**30-second answer:** SCD Type 1 overwrites an attribute and loses history; Type 2 closes the current row and inserts a new version with validity dates and a new surrogate key, so facts keep the attributes true at the time. I choose Type 2 only for attributes whose history the business analyzes.

**Structure a longer answer:** clarify (which attributes, reports, source type) → mechanism (hash compare, close/insert, key lookup) → trade-offs (growth, complexity) → scenario (customer relocates; revenue by country).

**Likely follow-ups:**
- *How do you detect changes?* → row hash/column compare; CDC if available.
- *How do facts find the right version?* → surrogate key at load via date-range lookup.
- *What about late-arriving changes?* → split intervals/rebuild the key's history.
- *Type 1 vs 2 on one table?* → yes, per attribute; corrections update all versions.

**Signals interviewers look for:** surrogate-key reasoning, point-in-time correctness, awareness of growth and edge cases.

### 11. Check yourself
- [ ] Show the table after a Type 1 vs a Type 2 change for the same customer.
- [ ] Write the close-and-insert logic for Type 2.
- [ ] Explain why facts store the surrogate key.
- [ ] Describe how to handle a change that arrives late.
<!-- explainer:end -->

## Practice questions

- [Intermediate] [Explain the difference between SCD Type 1 and SCD Type 2 with an example.](<../../../PRACTICE/02_Data_Engineering/Modeling_and_Warehousing/Data_Warehousing/Q-18981_Explain_the_difference_between_SCD_Type_1_and_SCD_Type_2_wit.md>)
- [Intermediate] [How does SCD Type 2 preserve history and what extra columns does it typically add?](<../../../PRACTICE/02_Data_Engineering/Modeling_and_Warehousing/Data_Warehousing/Q-18982_How_does_SCD_Type_2_preserve_history_and_what_extra_columns_.md>)
- [Intermediate] [What is an upsert (MERGE) statement and how is it used when loading dimension tables?](<../../../PRACTICE/02_Data_Engineering/Modeling_and_Warehousing/Data_Warehousing/Q-19025_What_is_an_upsert_MERGE_statement_and_how_is_it_used_when_lo.md>)
- [Deep] [When would you use SCD Type 3 instead of Type 2?](<../../../PRACTICE/02_Data_Engineering/Modeling_and_Warehousing/Data_Warehousing/Q-18983_When_would_you_use_SCD_Type_3_instead_of_Type_2.md>)
- [Deep] [What is a conformed dimension and why is it useful across multiple data marts?](<../../../PRACTICE/02_Data_Engineering/Modeling_and_Warehousing/Data_Warehousing/Q-18991_What_is_a_conformed_dimension_and_why_is_it_useful_across_mu.md>)
- [Deep] [What is a slowly changing dimension Type 4 (history table) and how does it differ from Type 2?](<../../../PRACTICE/02_Data_Engineering/Modeling_and_Warehousing/Data_Warehousing/Q-19046_What_is_a_slowly_changing_dimension_Type_4_history_table_and.md>)
- [Deep] [Your nightly SCD Type 2 merge started producing two open (current) rows per business key. How would you debug and fix the root cause?](<../../../PRACTICE/02_Data_Engineering/Modeling_and_Warehousing/Data_Warehousing/Q-19071_Your_nightly_SCD_Type_2_merge_started_producing_two_open_cur.md>)
- [Deep] [Explain how you would model a many-to-many relationship (e.g., a claim with multiple diagnoses) with a bridge table and its weighting factor.](<../../../PRACTICE/02_Data_Engineering/Modeling_and_Warehousing/Data_Warehousing/Q-19082_Explain_how_you_would_model_a_many-to-many_relationship_e_g_.md>)

## Related concepts

- [SCD Type 2 in Practice](<../Slowly_Changing_Dimensions/SCD_Type_2_in_Practice.md>) *(Slowly Changing Dimensions)*
- [SCD Type 2 vs Mini-dimensions](<../Dimensional_Modeling/SCD_Type_2_vs_Mini-dimensions.md>) *(Dimensional Modeling)*
- [Joining Type 2 Dimensions](<../../../01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Joining_Type_2_Dimensions.md>) *(SQL Fundamentals)*

## References

- *Building Medallion Architectures* — Piethein Strengholt — Demystifying the Medallion Architecture › Slowly changing dimensions
- *Mapping Data Flows in Azure Data Factory* — Mark Kromer — Slowly Changing Dimensions › Building a Slowly Changing Dimension Pattern in Mapping Data Flows
- *Data Pipelines Pocket Reference* — James Densmore — Transforming Data › Slowly Changing Dimensions for Fully Refreshed Data
- *The Enterprise Big Data Lake* — Alex Gorelik — From Data PondsBig Data Warehouses to Data Lakes › Preserving History Using Slowly Changing Dimensions
- *The Enterprise Big Data Lake* — Alex Gorelik — From Data PondsBig Data Warehouses to Data Lakes › Implementing Slowly Changing Dimensions in a Data Pond
- *Data Engineering Design Patterns* — Bartosz Konieczny — Chapter 5. Data Value Design Patterns › Examples

