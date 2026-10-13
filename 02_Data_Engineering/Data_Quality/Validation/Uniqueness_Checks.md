---
id: "02_Data_Engineering/Data_Quality/Validation/Uniqueness_Checks"
level: 4
type: "concept"
topic: "02_Data_Engineering/Data_Quality/Validation"
tools: ["dbt"]
---

# Uniqueness Checks

**L4 Advanced** · [↑ Validation](<_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** A uniqueness check proves that the column (or column combination) that *defines* a row appears only once, and it must be run after every load because constraints are often absent, disabled, or bypassed in pipelines.

### 1. The big picture
Duplicates are the most common and most expensive data defect: they inflate counts, double revenue, and break joins (a duplicate key on the "one" side of a join multiplies the "many" side). In an OLTP database a `PRIMARY KEY`/`UNIQUE` constraint prevents them. In warehouses and lakes, constraints are frequently *informational only* (not enforced), files are appended blindly, and jobs retry — so duplicates arrive and must be **detected** by tests.

There are two very different problems: **exact duplicates** (same key or same full row twice — usually a replay, a double-run or a missing constraint) and **near duplicates** (the same real-world entity written differently — "John Smith"/"Jon Smyth", `' Ravi Kumar'`/`'ravi kumar'`). The first is solved with `GROUP BY … HAVING`; the second needs normalisation and fuzzy matching.

### 2. Core concepts and vocabulary
- **Grain** — what one row represents (one order line, one customer per day). Uniqueness is always defined *at the grain*.
- **Natural vs. surrogate key** — business identifier vs. generated one; test uniqueness of the natural key even when a surrogate exists.
- **Composite key** — several columns together are unique (`order_id, line_no`).
- **Exact match** — strict equality on key fields; fast, misses typos.
- **Fuzzy match** — similarity scoring (Levenshtein, Jaccard, cosine, SOUNDEX) for near duplicates.
- **Blocking** — restricting fuzzy comparisons to candidates sharing a cheap key (zip, first letters) to avoid O(n²).
- **Idempotent load** — re-running produces the same final state; the defence against retry-duplicates.
- **Trailing/duplicate load** — a day's batch loaded twice.
- **Survivorship** — rule choosing which duplicate wins (latest `modified_on`, most complete).

### 3. How it works, step by step
Where duplicates come from (and therefore where to look):
1. Multiple entry points capturing the same entity without a shared ID.
2. Manual entry variation (spelling, casing, whitespace).
3. Integration lag or partial reprocessing of the same batch.
4. Missing/unenforced primary-key constraints.
5. Schema drift (`customer_id` vs `cust_id`) preventing correct matching.

Detection ladder:
1. **Normalise** first: `TRIM`, `LOWER`, strip punctuation, standard date/phone formats — so "same" values compare equal.
2. **Exact test on the declared key**: group by key, `HAVING COUNT(*) > 1`.
3. **Test the composite or business key** when the surrogate is always unique.
4. **Compare counts**: `COUNT(*)` vs `COUNT(DISTINCT key)`.
5. **Detect replayed loads**: group by batch/load id and compare row counts, or hash the batch.
6. **Near duplicates**: blocking → fuzzy score → threshold → human review or auto-merge.
7. **Automate**: auto-discover columns that have always been unique historically and alert if duplicates appear.

### 4. Worked example
```sql
-- 1. exact duplicates on the business key -> the test passes when this returns 0
SELECT order_id, COUNT(*) AS n
FROM   orders
GROUP  BY order_id
HAVING COUNT(*) > 1;

-- 2. which duplicate to keep (latest wins) and which to quarantine
WITH ranked AS (
  SELECT *, ROW_NUMBER() OVER (PARTITION BY order_id ORDER BY modified_on DESC) AS rn
  FROM   orders
)
SELECT * FROM ranked WHERE rn > 1;      -- the losers

-- 3. same day loaded twice?
SELECT load_date, COUNT(*) AS rows_loaded, COUNT(DISTINCT order_id) AS distinct_orders
FROM   orders GROUP BY load_date
HAVING COUNT(*) <> COUNT(DISTINCT order_id);

-- 4. foreign-key existence (orphans)
SELECT f.customer_id
FROM   fact_orders f LEFT JOIN dim_customer d ON d.customer_id = f.customer_id
WHERE  d.customer_id IS NULL;
```
How to read: query 1 returning rows = failing test (the count of rows is the metric). Query 2 deterministically picks winners. Query 3 exposes a replayed load because rows exceed distinct keys for that date only. In a framework the same test is `unique` in dbt or "expect column values to be unique" in Great Expectations; the pipeline book pattern is "run query A, compare with expected value 0, warn or fail".

### 5. Under the hood   *(optional at L4)*
- `GROUP BY` duplicate detection is a hash aggregation or sort — one pass; cost grows with table size and key width. Hashing the concatenated key columns into a single fingerprint speeds up wide composite keys.
- **Fuzzy at scale**: naive pairwise comparison is O(n²). Use blocking (compare only within same zip/prefix/soundex), MinHash/LSH, or n-gram inverted indexes.
- **Idempotent upserts** (`MERGE` on the key) prevent most replay duplicates, but a retry after a partial write can still leave phantom rows if the key isn't truly unique or the dedup window is wrong — validate with a post-load uniqueness test and a batch/attempt-id audit column.
- Self-serve tables: duplicates often appear only when an analyst joins on a *non-unique* key; document the grain and test it.

### 6. Performance and design implications
- Run uniqueness tests on the **new partition/batch** plus a cheaper global `COUNT` comparison, not a full-table scan every run.
- Declare keys even if not enforced — optimizers and BI tools use the metadata, and tests can read it.
- Window-based dedup (`ROW_NUMBER`) needs a deterministic tie-breaker or reruns can pick different survivors.
- Distinguish true duplicates from legitimate repeats (two identical purchases): dedup on event/transaction id, not on content alone.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `GROUP BY … HAVING COUNT(*) > 1` | Lists duplicate keys | Any uniqueness test |
| `ROW_NUMBER() OVER (PARTITION BY key ORDER BY …)` | Ranks duplicates | Dedup and survivorship |
| `COUNT(*) − COUNT(DISTINCT key)` | One-number duplicate metric | Dashboards/alerts |
| `UNIQUE`/`PRIMARY KEY` constraint | Prevents duplicates at write | OLTP; enforced systems |
| dbt `unique`, `unique_combination_of_columns` | Declarative test | ELT pipelines |
| `SOUNDEX`, `EDIT_DISTANCE`/Levenshtein | Fuzzy similarity | Near-duplicate customers |
| `MERGE … WHEN MATCHED/NOT MATCHED` | Idempotent upsert | Incremental loads |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| Exact-key test | Data has typos/format variants → also normalise or fuzzy-match |
| Fuzzy matching | Threshold set too low → false merges of different people |
| Auto-dedup (keep latest) | Business needs history of every version |
| Quarantine duplicates | You can't tell which version is authoritative |

### 9. Common mistakes
- Testing only the surrogate key → natural-key dupes slip through → test the business key.
- Ignoring casing/whitespace → near duplicates → normalise first.
- Setting strict fuzzy thresholds and calling it done → misses legitimate near-duplicates; tune with labelled samples.
- Dedup without a tie-breaker → non-deterministic results → add a second ordering column.
- Passing in staging but failing in prod → data volume/sources differ; test on production-like data and on the composite key.

### 10. 🎯 Interview kit
**30-second answer:** "I define the grain, then run `GROUP BY key HAVING COUNT(*)>1` after each load and compare count vs distinct count. For retries I make loads idempotent with MERGE and validate afterwards; for near duplicates I normalise and use blocked fuzzy matching."

**Structure a longer answer:** clarify what "duplicate" means for this table → exact check → replay/idempotency → near duplicates → what to do with losers (keep latest/quarantine) → monitoring.

**Likely follow-ups:**
- *Pipeline ran twice for one day?* → compare rows vs. distinct keys per load date/batch id; add load-id and make loads overwrite-by-partition.
- *Fuzzy matching without O(n²)?* → blocking keys, LSH/MinHash, then score only candidate pairs.
- *dbt incremental with update + delete for same key?* → order by event time/sequence and apply the last op per key before merge.
- *Uniqueness passes on composite key in staging but dupes in prod?* → different source/volume, late-arriving rows, nulls in key parts (NULLs aren't equal in uniqueness).

**Signals interviewers look for:** grain awareness, idempotency, and a plan for near-duplicates beyond `DISTINCT`.

### 11. Check yourself
- [ ] Write SQL that finds duplicate `(order_id, line_no)` pairs and keeps the latest.
- [ ] Explain why `UNIQUE` plus nullable columns can still allow "duplicates".
- [ ] Describe how blocking makes fuzzy matching scale.
- [ ] Explain how to detect a loaded-twice batch.
<!-- explainer:end -->

## Implementations (tools)

- [dbt](<../../../TOOLS/dbt/_index.md>)

## Practice questions

- [Basic] [How would you check for duplicate rows in a table using SQL?](<../../../PRACTICE/02_Data_Engineering/Data_Quality/Validation/Q-14354_How_would_you_check_for_duplicate_rows_in_a_table_using_SQL.md>)
- [Basic] [What problems can inconsistent letter casing cause and how would you normalize it?](<../../../PRACTICE/02_Data_Engineering/Data_Quality/Validation/Q-14392_What_problems_can_inconsistent_letter_casing_cause_and_how_w.md>)
- [Basic] [How would you use a GROUP BY and HAVING clause to find duplicate values in a column?](<../../../PRACTICE/02_Data_Engineering/Data_Quality/Validation/Q-14415_How_would_you_use_a_GROUP_BY_and_HAVING_clause_to_find_dupli.md>)
- [Intermediate] [How would you detect that a column expected to be unique like a user id has started getting duplicates?](<../../../PRACTICE/02_Data_Engineering/Data_Quality/Validation/Q-14363_How_would_you_detect_that_a_column_expected_to_be_unique_lik.md>)
- [Intermediate] [How would you validate that a foreign key value actually exists in the referenced dimension table?](<../../../PRACTICE/02_Data_Engineering/Data_Quality/Validation/Q-14395_How_would_you_validate_that_a_foreign_key_value_actually_exi.md>)
- [Intermediate] [How would you detect trailing duplicate loads if a pipeline accidentally ran twice for the same day?](<../../../PRACTICE/02_Data_Engineering/Data_Quality/Validation/Q-14405_How_would_you_detect_trailing_duplicate_loads_if_a_pipeline_.md>)
- [Deep] [What is fuzzy matching and when would you use it to detect near duplicate customer records?](<../../../PRACTICE/02_Data_Engineering/Data_Quality/Validation/Q-14434_What_is_fuzzy_matching_and_when_would_you_use_it_to_detect_n.md>)
- [Deep] [How do you validate that an idempotent upsert didn't create phantom duplicates when a job retried after a partial write?](<../../../PRACTICE/02_Data_Engineering/Data_Quality/Validation/Q-14477_How_do_you_validate_that_an_idempotent_upsert_didn_t_create_.md>)

## Related concepts

- [Reject vs Quarantine Bad Rows](<../../../11_AI_and_ML/ML_Foundations/Data_Preprocessing/Reject_vs_Quarantine_Bad_Rows.md>) *(Data Preprocessing)*
- [Unique Constraints for Dedup](<../../../01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Unique_Constraints_for_Dedup.md>) *(SQL Fundamentals)*
- [Unique and Deferred Constraints](<../../../01_Databases_and_SQL/Database_Design/Constraints_and_Keys/Unique_and_Deferred_Constraints.md>) *(Constraints and Keys)*

## References

- *Automating Data Quality Monitoring* — Jeremy Stanley, Paige Schwartz — Operating Your Solution at Scale
- *Ultimate Data Engineering Design Patterns* — Bragadeesh Sundararajan — Data Quality Patterns › 10. Data Quality Patterns
- *Fuzzy Data Matching with SQL* — Jim Lehmer — The Big Score › Eliminating Duplicates
- *Fuzzy Data Matching with SQL* — Jim Lehmer — The Big Score › Beware Automated Processes!
- *Fuzzy Data Matching with SQL* — Jim Lehmer — The Big Score › Duplicated Data
- *Data Pipelines Pocket Reference* — James Densmore — Data Validation in Pipelines › Duplicate Records After Ingestion
- *AWS Certified Data Engineer Associate Study Guide* — Sakti Mishra et al. — Data Preparation for Nontechnical Personas › Identify Duplicate Records
- *Data Engineering Design Patterns* — Bartosz Konieczny — Error Management Design Patterns › Duplicated Records

