---
id: "01_Databases_and_SQL/Database_Internals/DBMS_Concepts/Late_Arriving_Dimensions"
level: 4
type: "concept"
topic: "01_Databases_and_SQL/Database_Internals/DBMS_Concepts"
tools: []
---

# Late Arriving Dimensions

**L4 Advanced** · [↑ DBMS Concepts](<_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** A late-arriving dimension is a fact row whose dimension member has not been loaded yet — you must still load the fact, using a placeholder key that is later repaired, or your warehouse silently loses or mis-attributes data.

### 1. The big picture
A dimensional warehouse splits data into **fact tables** (events and measurements: sales, clicks, claims) and **dimension tables** (the descriptive context: customer, product, store, date). Facts carry foreign keys to dimensions, and the classic load order is "dimensions first, then facts", so every fact finds its parent.

In real pipelines that order is not guaranteed. A point-of-sale feed may arrive before the customer master is synchronised; an order event reaches the stream before the CRM export; a fact load is re-run for yesterday while the dimension job failed. The fact row then references a business key that does not exist in the dimension. This is the *late-arriving dimension* (also called an *early-arriving fact*) problem.

It sits between three topics: **referential integrity** (what do you do when a foreign key has no parent?), **slowly changing dimensions** (a dimension row may exist but the *right version* may not), and **idempotent, re-runnable loads** (the repair must be safe to repeat). The books on data quality add a warning: dimensional data that is updated in place without tracking when it changed makes yesterday's report impossible to reproduce, and late data multiplies that problem.

### 2. Core concepts and vocabulary
- **Fact table** — rows of measurable events at a declared *grain* (e.g. one row per order line).
- **Dimension table** — descriptive attributes, keyed by a **surrogate key** (a meaningless integer the warehouse generates) plus the source system's **business key** (natural key).
- **Surrogate key** — the warehouse-owned integer that facts store instead of the business key; it is what makes SCD Type 2 history possible.
- **Late-arriving dimension / early-arriving fact** — a fact whose business key has no matching dimension row at load time.
- **Inferred member** (placeholder, stub) — a dimension row created just to hold the business key, with attributes set to "Unknown", flagged as inferred, so the fact can be loaded.
- **Unknown member** — the single reserved row (usually surrogate key `-1` or `0`) meaning "no value known / not applicable".
- **Late-arriving fact** — different problem: the event is old, arriving after the period was loaded; with SCD2 you must pick the dimension version valid *at the event time*.
- **SCD Type 1 / Type 2** — overwrite in place vs add a new versioned row with `valid_from`/`valid_to`/`is_current`.
- **Suspense (reject) table** — holding area for rows that cannot be loaded yet.
- **Idempotency** — running the same load twice yields the same final state.

### 3. How it works, step by step
There are three accepted strategies; real warehouses combine them.

1. **Detect.** During the fact load, left-join the staged facts to the dimension on the business key. Rows where the dimension key is `NULL` are the late-arriving ones. Count them — the count is a data-quality metric in its own right.
2. **Choose a strategy.**
   - *Reject / suspense:* write unmatched facts to a suspense table and retry them after the next dimension load. Reports are *complete but delayed* — correct for financial data where mis-attribution is worse than lateness.
   - *Unknown member:* map the fact to the reserved `-1` row. Simple, but the link to the real member is lost unless you keep the business key on the fact (or in a side table) so you can repair later.
   - *Inferred member (recommended for most pipelines):* insert a stub row into the dimension (`business_key = 'C-991'`, `name = 'Unknown'`, `is_inferred = 1`), take its new surrogate key, and load the fact against it. The fact is immediately queryable and attached to the correct identity.
3. **Repair when the real row arrives.** The dimension load, on finding a business key that exists as an inferred member:
   - *Type 1 attributes:* simply update the stub in place — every fact already points to the right surrogate key, nothing else changes.
   - *Type 2 attributes:* the stub *is* the first version. Update it with the real attributes and set its validity start to the right effective date (do **not** create a second version for the first real load, or you will have an "Unknown" version that facts still point to).
4. **Late *versions* of an existing member** (an SCD2 change that arrives after facts were loaded): the facts that belong to the new version already point to the old surrogate key. Either re-key those facts (update `customer_sk` for rows with `event_date >= new valid_from`), or accept the as-loaded attribution and document it.
5. **Monitor.** Track the number of inferred members and the age of the oldest unresolved one; alert when it grows.

### 4. Worked example
```sql
-- staged facts and a customer dimension (SCD2 with inferred flag)
CREATE TABLE dim_customer (
  customer_sk   INT IDENTITY PRIMARY KEY,
  customer_bk   VARCHAR(20) NOT NULL,       -- business key
  name          VARCHAR(100) NOT NULL,
  city          VARCHAR(60),
  is_inferred   BIT NOT NULL DEFAULT 0,
  valid_from    DATE NOT NULL,
  valid_to      DATE NOT NULL DEFAULT '9999-12-31',
  is_current    BIT NOT NULL DEFAULT 1
);

-- 1) create inferred members for business keys we have never seen
INSERT INTO dim_customer (customer_bk, name, city, is_inferred, valid_from)
SELECT DISTINCT s.customer_bk, 'Unknown', NULL, 1, '1900-01-01'
FROM   stg_sales s
LEFT JOIN dim_customer d ON d.customer_bk = s.customer_bk AND d.is_current = 1
WHERE  d.customer_sk IS NULL;

-- 2) load facts, now every key resolves
INSERT INTO fact_sales (customer_sk, date_sk, amount)
SELECT d.customer_sk, s.date_sk, s.amount
FROM   stg_sales s
JOIN   dim_customer d
  ON   d.customer_bk = s.customer_bk
 AND   s.sale_date BETWEEN d.valid_from AND d.valid_to;

-- 3) later, the real customer record arrives: repair the stub in place
UPDATE d
SET    d.name = c.name, d.city = c.city, d.is_inferred = 0
FROM   dim_customer d
JOIN   stg_customer c ON c.customer_bk = d.customer_bk
WHERE  d.is_inferred = 1;
```
Step 1 is the inferred-member insert; it is idempotent because the `LEFT JOIN … IS NULL` filter skips keys already present. In step 2 the date predicate selects the dimension *version valid at sale time* — this is the SCD2 join. Step 3 keeps the same `customer_sk`, so no fact needs touching. To read the result, query `WHERE is_inferred = 1` for open items and `COUNT(*)` of facts attached to them to size the exposure.

### 5. Under the hood
- **Why the surrogate key matters:** facts hold an integer, not the business key. That is what lets you create the dimension row *after* the fact and repair it without rewriting the fact table — updating a dimension row is cheap, rewriting a billion-row fact table is not.
- **Columnar and lake engines:** in file-based systems (Parquet/Delta) an "update in place" is copy-on-write: the engine rewrites whole files containing the changed rows. Updating a handful of stub rows in a small dimension is fine; re-keying millions of facts is expensive. This is another reason to prefer the inferred-member approach to "load unknown, re-key later".
- **No enforced foreign keys:** most columnar warehouses do not enforce uniqueness or referential integrity, so the pipeline itself must enforce "every fact key resolves" — typically a post-load orphan check (`fact LEFT JOIN dim … WHERE dim.key IS NULL`).
- **Time travel is not a substitute:** warehouse time-travel features help recover recent states, but are limited to short retention and are not a record-level history model.

### 6. Performance and design implications
- The orphan-detection join is one extra pass over the staged batch; index or distribute the staging table and dimension on the business key.
- Keep inferred members in the same table (flagged) rather than a separate one — queries then need no union.
- Large volumes of late data signal an upstream ordering problem; fix the schedule (dependencies in the orchestrator) rather than relying on repair logic.
- Measure: orphan count per load, inferred-member count, mean time to repair, and facts attached to inferred members.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `LEFT JOIN … WHERE d.key IS NULL` | Finds facts with no dimension row | Pre-load detection and post-load audit |
| Unknown member row (`-1`) | Absorbs facts with no known/applicable dimension | Genuinely missing values, not just late ones |
| `is_inferred` flag | Marks placeholder rows | Inferred-member pattern |
| `MERGE` / upsert | Insert stub or update real attributes in one statement | Dimension load |
| Suspense table | Parks facts for retry | Strict correctness requirements |
| Orphan-check query | Verifies referential integrity manually | Engines without enforced FKs |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| **Inferred member** — you want facts queryable immediately and attached to the right identity | Many attributes are needed for the report; the "Unknown" bucket will mislead until repaired |
| **Suspense table** — accuracy matters more than freshness (finance) | Dashboards must be near-real-time; dropped rows will look like missing sales |
| **Unknown member** — the value is truly absent (anonymous customer) | The key is merely late; you lose the link unless you retain the business key |
| **Re-key facts on late SCD2 version** — history accuracy is a hard requirement | Fact table is huge and immutable; consider accepting as-loaded attribution |

### 9. Common mistakes
- Dropping the unmatched fact rows with an inner join → silent revenue loss → left join + inferred/suspense handling and a reconciliation count.
- Loading to `-1` and discarding the business key → can never repair → keep the business key on the fact or in a side table.
- Creating a *second* SCD2 version when the real attributes arrive for an inferred member → facts stay on "Unknown" → treat the stub as version one and update it.
- Non-idempotent repair that inserts a new stub on every re-run → duplicate members → make inserts conditional on absence of the business key.
- Assuming the warehouse enforces foreign keys → orphans accumulate → add explicit orphan checks to every load.

### 10. 🎯 Interview kit
**30-second answer:** A late-arriving dimension is a fact whose dimension row does not exist yet. I would not drop or fail the load: I create an inferred placeholder member with the business key, load the fact against its surrogate key, flag it, and update that same row in place when the real record arrives — so no facts need re-keying.

**Structure a longer answer:** clarify (is it a late dimension member or a late *version* of an existing member?) → mechanism (detect with left join, inferred member, repair in place) → trade-offs (suspense vs inferred vs unknown) → real scenario (orders stream beating the CRM export).

**Likely follow-ups:**
- *How does this differ from a late-arriving fact?* → Late fact: event is old, need the dimension version valid at event time. Late dimension: the parent is missing.
- *What if the dimension is SCD Type 2?* → Stub becomes version one; later changes create new versions; facts for late versions may need re-keying.
- *How do you make it idempotent?* → Conditional insert on business key, update keyed on the flag, no appends without existence checks.
- *How do you know it is working?* → Orphan count = 0 after load, inferred-member backlog trending to zero.

**Signals interviewers look for:** you name surrogate keys as the reason repair is cheap; you think about idempotency and monitoring; you do not default to "just inner join".

### 11. Check yourself
- [ ] Explain why an inner join between staged facts and a dimension can silently lose data.
- [ ] Write the SQL that inserts inferred members only for business keys not yet present.
- [ ] Describe what happens to the stub row when the real customer record arrives, for a Type 1 and a Type 2 attribute.
- [ ] Contrast the unknown member, inferred member and suspense table approaches.
- [ ] Design an orphan check you would run after every fact load.
<!-- explainer:end -->

## Practice questions

- [Basic] [What is the difference between a fact table and a dimension table in a data warehouse?](<../../../PRACTICE/01_Databases_and_SQL/Database_Internals/DBMS_Concepts/Q-22548_What_is_the_difference_between_a_fact_table_and_a_dimension_.md>)
- [Basic] [What does data quality mean and what dimensions (completeness accuracy consistency timeliness uniqueness) do you check?](<../../../PRACTICE/01_Databases_and_SQL/Database_Internals/DBMS_Concepts/Q-22598_What_does_data_quality_mean_and_what_dimensions_completeness.md>)
- [Intermediate] [What is a slowly changing dimension and what are the differences between Type 1 Type 2 and Type 3?](<../../../PRACTICE/01_Databases_and_SQL/Database_Internals/DBMS_Concepts/Q-22551_What_is_a_slowly_changing_dimension_and_what_are_the_differe.md>)
- [Intermediate] [What is idempotency in a data load and why must an ETL job be safe to rerun after a failure?](<../../../PRACTICE/01_Databases_and_SQL/Database_Internals/DBMS_Concepts/Q-22563_What_is_idempotency_in_a_data_load_and_why_must_an_ETL_job_b.md>)
- [Intermediate] [Why has ELT become more common than ETL now that cloud warehouses have cheap compute and storage?](<../../../PRACTICE/01_Databases_and_SQL/Database_Internals/DBMS_Concepts/Q-22588_Why_has_ELT_become_more_common_than_ETL_now_that_cloud_wareh.md>)
- [Deep] [What is a factless fact table and give an example of when you would use one?](<../../../PRACTICE/01_Databases_and_SQL/Database_Internals/DBMS_Concepts/Q-22555_What_is_a_factless_fact_table_and_give_an_example_of_when_yo.md>)
- [Deep] [What is a late-arriving dimension and how would you handle a fact row that references a dimension key that does not exist yet?](<../../../PRACTICE/01_Databases_and_SQL/Database_Internals/DBMS_Concepts/Q-22579_What_is_a_late-arriving_dimension_and_how_would_you_handle_a.md>)
- [Deep] [What is a role-playing dimension (e.g. one date dimension used as order date and ship date) and how do you implement it?](<../../../PRACTICE/01_Databases_and_SQL/Database_Internals/DBMS_Concepts/Q-22630_What_is_a_role-playing_dimension_e_g_one_date_dimension_used.md>)

## Related concepts

- [Fact Dimension Integrity](<../../../02_Data_Engineering/Modeling_and_Warehousing/Data_Warehousing/Fact_Dimension_Integrity.md>) *(Data Warehousing)*
- [Joining Type 2 Dimensions](<../../SQL_Language/SQL_Fundamentals/Joining_Type_2_Dimensions.md>) *(SQL Fundamentals)*
- [Special Dimension Types](<../../../02_Data_Engineering/Modeling_and_Warehousing/Dimensional_Modeling/Special_Dimension_Types.md>) *(Dimensional Modeling)*

## References

- *Automating Data Quality Monitoring* — Jeremy Stanley, Paige Schwartz — Operating Your Solution at Scale
- *Data Engineering Design Patterns* — Bartosz Konieczny — Data Ingestion Design Patterns › Full Load
- *Data Engineering Design Patterns* — Bartosz Konieczny — Pattern: Dynamic Joiner › Data Decoration
- *Data Engineering Design Patterns* — Bartosz Konieczny — Data Ordering
- *Data Engineering Design Patterns* — Bartosz Konieczny — Idempotency Design Patterns › Data Value Design Patterns
- *The Enterprise Big Data Lake* — Alex Gorelik — From Data PondsBig Data Warehouses to Data Lakes › Data Transformations
- *Building Data Integration Solutions* — Jay Borthen — Models, Architectures, Methods, and Patterns › Patterns
- *Data Quality Fundamentals* — Barr Moses, Lior Gavish, Molly Vorwerck — Architecting for Data Reliability › Data Transformation and Modeling
- *Data Quality Fundamentals* — Barr Moses, Lior Gavish, Molly Vorwerck — Fixing Data Quality Issues at Scale › Why Data Incident Commanders Matter
- *Mapping Data Flows in Azure Data Factory* — Mark Kromer — Slowly Changing Dimensions › ExistingProducts
- *Mapping Data Flows in Azure Data Factory* — Mark Kromer — Slowly Changing Dimensions › Set Dimension Properties
- *Mapping Data Flows in Azure Data Factory* — Mark Kromer — Monitor, Manage, and Optimize › Data Profiling

