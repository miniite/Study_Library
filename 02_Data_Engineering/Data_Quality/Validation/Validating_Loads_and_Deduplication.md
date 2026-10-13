---
id: "02_Data_Engineering/Data_Quality/Validation/Validating_Loads_and_Deduplication"
level: 4
type: "concept"
topic: "02_Data_Engineering/Data_Quality/Validation"
tools: ["Python"]
---

# Validating Loads and Deduplication

**L4 Advanced** · [↑ Validation](<_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** Validating a load means proving, right after it lands, that what arrived is complete (counts and checksums reconcile with the source), structurally sound (schema, types, formats), and sensible (business ranges and cross-field rules) — at a cost you can afford on big tables.

### 1. The big picture
A load can "succeed" (no exception, job green) and still be wrong: a pagination bug drops the last page, a partition filter skips a region, a retry double-writes, a CSV column shifts. Load validation is the set of checks that sit between "data landed" and "data is promoted for consumers". It combines three families:

- **Reconciliation** — source vs. target (row counts, sums, checksums, key sets).
- **Technical validation** — schema, datatype, length, format, nullability.
- **Business validation** — value ranges, allowed sets, cross-field rules, referential integrity.

In a lakehouse the first processing step on cleansed data is schema validation: records that violate the agreed schema are rejected (or the schema is evolved deliberately), then quality rules decide which rows load to target tables and which go to an error table.

### 2. Core concepts and vocabulary
- **Reconciliation** — proving two datasets agree (counts, totals, hashes).
- **Checksum / hash** — fixed-size fingerprint (MD5, SHA-256) of a file or row set; equal fingerprints ⇒ same content, used to prove a transfer wasn't corrupted.
- **Technical rule** — based on metadata: type, length, pattern, nullability.
- **Business rule** — based on meaning: `balance >= minimum`, `shipped_at >= ordered_at`.
- **Cross-field validation** — a rule over two or more columns.
- **Smoke test** — the minimal post-load sanity check (rows > 0, key columns non-null, latest date present).
- **Pre-build / build / post-build validation** — in schema/ETL deployments: rule analysis before, actual execution during, verification after.
- **Placeholder value** — default such as `0`, `'N/A'`, `1900-01-01` that masks missing data.
- **Stratified sampling** — sampling in proportion to business segments so rare segments are represented.

### 3. How it works, step by step
1. **Capture expected control totals** at the source: row count, sum of key measures, max watermark, file checksum, number of pages/files.
2. **Land the data** in a staging area (not the final table).
3. **Structural checks**: schema matches, types castable, mandatory columns present, no shifted columns.
4. **Reconcile** to the control totals; compare key sets for missing/extra IDs; hash comparisons for exactness.
5. **Column-level checks**: nulls vs. empty strings, ranges (`0–100`, age > 0), allowed sets, formats (regex, 10-digit phones, pincodes), whitespace and casing.
6. **Cross-field and referential checks** (dates in order, FK exists, timezone-safe comparison).
7. **Decide**: promote, quarantine bad rows, or fail the batch (see *Validation Coverage and Quarantine*).
8. **Record the result** (metrics table) for trend monitoring.

### 4. Worked example
```sql
-- 1. completeness vs source (control total supplied by extractor)
SELECT (SELECT COUNT(*) FROM stg.orders)            AS loaded_rows,
       c.expected_rows,
       (SELECT COUNT(*) FROM stg.orders) - c.expected_rows AS diff
FROM   etl.control_totals c WHERE c.batch_id = :batch;

-- 2. null vs empty string vs placeholder
SELECT SUM(CASE WHEN email IS NULL THEN 1 END)               AS null_email,
       SUM(CASE WHEN LTRIM(RTRIM(email)) = '' THEN 1 END)    AS blank_email,
       SUM(CASE WHEN email = 'N/A' THEN 1 END)               AS placeholder_email
FROM stg.customers;

-- 3. range, set and format rules (each returns offending rows)
SELECT * FROM stg.customers WHERE age <= 0 OR age > 120;
SELECT * FROM stg.accounts  WHERE status NOT IN ('active','inactive');
SELECT * FROM stg.customers WHERE phone NOT LIKE REPLICATE('[0-9]',10);

-- 4. cross-field, timezone-safe: compare in UTC, allow a small clock-skew buffer
SELECT * FROM stg.shipments
WHERE  shipped_at_utc < DATEADD(minute, -15, ordered_at_utc);

-- 5. missing keys between source and target
SELECT s.order_id FROM src.orders s
LEFT JOIN stg.orders t ON t.order_id = s.order_id
WHERE t.order_id IS NULL;
```
Reading it: query 1's `diff` must be 0; any nonzero value fails the batch. Query 2 shows why `COALESCE(x, 0)` / blank-vs-NULL distinctions matter — three different "missing" shapes. Query 5 finds *which* rows are missing, not just how many (the page that was dropped will appear as a contiguous key range).

### 5. Under the hood   *(optional at L4)*
- **Why per-page validity hides a dropped page:** each page passes checks individually. Only an aggregate against an independent count (`total` header, next-page token, max sequence id, source `COUNT`) detects it. Check *contiguity* of sequence numbers or last-cursor reached.
- **Partition pruning pitfalls:** a wrong predicate or a missing partition directory makes a query silently return fewer rows. Validate by comparing partition lists against the expected set (all regions × dates) before trusting a report.
- **Replacement of NULL with 0:** makes a sum look complete, but averages become wrong and the information "unknown" is destroyed.
- **File formats:** Parquet carries a schema and per-block statistics (min/max/null count), so some checks are answered from metadata; CSV has no schema, so column shifts and type problems must be detected by parsing and explicit schema enforcement.
- **Hashes** compare bit-exact content; a row-level hash (over normalised columns) lets you compare two large tables efficiently by comparing hash aggregates per partition first and drilling in only where they differ.

### 6. Performance and design implications
- Validate incrementally (new batch only); reserve full-table checks for periodic audits.
- On very large tables, **sample** (stratified) for expensive rule checks but run cheap exact checks (counts, nulls, keys) on everything.
- Prioritise columns by business impact, dependency (join keys) and incident history.
- Combine all column metrics in one scan rather than one query per rule.
- Heavy validation during retries slows recovery — keep a fast path for known-good re-runs.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `COUNT(*)` source vs target | Row-count reconciliation | Every load, migration |
| `HASHBYTES`/`MD5`/`sha256sum` | Content fingerprint | File transfer, table compare |
| `EXCEPT` / `LEFT JOIN … IS NULL` | Missing/extra keys | Pinpoint lost rows |
| Regex (`REGEXP_LIKE`, `LIKE '[0-9]…'`) | Format validation | Email, phone, pincode |
| `TRY_CAST` / `TRY_CONVERT` | Detect uncastable values without failing | Type validation |
| Great Expectations, Glue DQ, dbt tests | Declarative rule frameworks | Production pipelines |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| Full check of every row | Table is huge and the check is expensive |
| Sampling | Rare segments matter (not stratified) |
| Validate before load (stop bad data early) | Source can't be re-extracted cheaply |
| Validate after load (visible in target) | Bad rows already visible to consumers |
| Do both | Never — they protect different failures |

### 9. Common mistakes
- Checking only parsing/format, not business meaning → "valid" but wrong values pass.
- Treating placeholders as real data → analysis skews → count and alert on placeholder rates.
- Assuming CSV is self-describing → shifted columns go unnoticed → enforce an explicit schema.
- Using local timestamps in cross-field rules → midnight false failures → compare in UTC with a buffer.
- Missing partition validation → silently missing region → compare to expected partition list.

### 10. 🎯 Interview kit
**30-second answer:** "After a load I reconcile counts and totals to the source, check schema and formats, then apply business and cross-field rules, and only then promote — failing or quarantining on breach. For large tables I validate the new batch fully and sample the expensive rules."

**Structure a longer answer:** what can go wrong in this load → control totals → structural → business rules → decision (fail/quarantine) → recording and alerting.

**Likely follow-ups:**
- *Upstream pagination bug dropped the last page?* → compare to source count/`total`, check contiguous ids, final cursor, last-page marker.
- *Validate a very large table cheaply?* → incremental partitions, aggregates, stratified sampling, metadata stats.
- *Checksum use?* → compute before/after transfer; mismatch = corruption.
- *Which columns first?* → business impact, join keys, historical failures.

**Signals interviewers look for:** reconciliation against an *independent* control total, not just internal consistency; awareness of silent failures.

### 11. Check yourself
- [ ] Write the reconciliation query for counts and sums between staging and source.
- [ ] Explain NULL vs empty string vs placeholder with a query that distinguishes all three.
- [ ] Describe how you'd prove no partition was skipped.
- [ ] Explain how to make a cross-field date rule timezone-safe.
<!-- explainer:end -->

## Implementations (tools)

- [Python](<../../../TOOLS/Python/_index.md>)

## Practice questions

- [Basic] [How can you use a regular expression to validate that an email column has a proper format?](<../../../PRACTICE/02_Data_Engineering/Data_Quality/Validation/Q-14345_How_can_you_use_a_regular_expression_to_validate_that_an_ema.md>)
- [Basic] [How would you handle leading and trailing whitespace in text data that causes join mismatches?](<../../../PRACTICE/02_Data_Engineering/Data_Quality/Validation/Q-14391_How_would_you_handle_leading_and_trailing_whitespace_in_text.md>)
- [Intermediate] [What is a checksum or hash and how can it be used to verify a file was not corrupted during transfer?](<../../../PRACTICE/02_Data_Engineering/Data_Quality/Validation/Q-14386_What_is_a_checksum_or_hash_and_how_can_it_be_used_to_verify_.md>)
- [Intermediate] [How would you decide which columns in a table are most important to validate first?](<../../../PRACTICE/02_Data_Engineering/Data_Quality/Validation/Q-14417_How_would_you_decide_which_columns_in_a_table_are_most_impor.md>)
- [Deep] [How would you approach validating data quality on a very large table where scanning every row is expensive?](<../../../PRACTICE/02_Data_Engineering/Data_Quality/Validation/Q-14439_How_would_you_approach_validating_data_quality_on_a_very_lar.md>)
- [Deep] [A cross-field rule says shipped_date >= order_date, but timezone skew makes valid rows fail near midnight. How do you make cross-field validation timezone-safe?](<../../../PRACTICE/02_Data_Engineering/Data_Quality/Validation/Q-14472_A_cross-field_rule_says_shipped_date_order_date_but_timezone.md>)
- [Deep] [How do you validate that partition pruning didn't skip a required partition, causing a report to look complete while silently missing a region?](<../../../PRACTICE/02_Data_Engineering/Data_Quality/Validation/Q-14507_How_do_you_validate_that_partition_pruning_didn_t_skip_a_req.md>)
- [Deep] [How do you validate that an upstream API pagination bug didn't silently drop the last page of records during ingestion, given each page individually looks valid?](<../../../PRACTICE/02_Data_Engineering/Data_Quality/Validation/Q-14542_How_do_you_validate_that_an_upstream_API_pagination_bug_didn.md>)

## Related concepts

- [Cleaning Text and Numeric Fields](<../../../11_AI_and_ML/ML_Foundations/Data_Preprocessing/Cleaning_Text_and_Numeric_Fields.md>) *(Data Preprocessing)*
- [Reject vs Quarantine Bad Rows](<../../../11_AI_and_ML/ML_Foundations/Data_Preprocessing/Reject_vs_Quarantine_Bad_Rows.md>) *(Data Preprocessing)*
- [Data Validation Queries](<../../../01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Data_Validation_Queries.md>) *(SQL Fundamentals)*
- [Quality SLAs and Rule Testing](<../Quality_Frameworks_and_SLAs/Quality_SLAs_and_Rule_Testing.md>) *(Quality Frameworks and SLAs)*
- [End to End No Data Loss](<../../../04_Data_Architecture_and_Strategy/Architecture_Patterns/Data_Architecture/Tiered_Lake_Storage/End_to_End_No_Data_Loss.md>) *(Data Architecture)*
- [Denormalization Trade-offs and Patterns](<../../../01_Databases_and_SQL/Database_Design/Normalization/Denormalization_Trade-offs_and_Patterns/_index.md>) *(Normalization)*

## References

- *Data Quality Fundamentals* — Barr Moses, Lior Gavish, Molly Vorwerck — Assembling the Building Blocks of a Reliable Data System › Step 4: Health check
- *Data Quality Fundamentals* — Barr Moses, Lior Gavish, Molly Vorwerck — Collecting, Cleaning, Transforming, and Testing Data › Schema Checking and Type Coercion
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 6: CTEs, MERGE, Pivoting, JSON, and XML › Validating XML with Schema Collections
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 14: Unit Testing T-SQL with tSQLt › Assertions: Validating Results
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 14: Unit Testing T-SQL with tSQLt › Combining Fake Tables and Assertions
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 15: Case Studies and the Future of T-SQL › Data Migration and Validation
- *Practical Lakehouse Architecture* — Gaurav Ashok Thalpati — The Big Picture Designing and Implementing a Lakehouse Platform › Schema and data quality validations
- *Database Reliability Engineering* — Laine Campbell, Charity Majors — Operational Visibility › Is the Data Safe?
- *Database Reliability Engineering* — Laine Campbell, Charity Majors — Release Management › Test data
- *Database Reliability Engineering* — Laine Campbell, Charity Majors — Release Management › Pre-build
- *Database Reliability Engineering* — Laine Campbell, Charity Majors — Security › Input validation
- *Data Pipelines Pocket Reference* — James Densmore — Data Validation in Pipelines › Running a Validation Test
- *Data Pipelines Pocket Reference* — James Densmore — Data Validation in Pipelines › Metric Value Fluctuations
- *Cassandra: The Definitive Guide* — Jeff Carpenter, Eben Hewitt — handling. Instead of failing immediately on a bad input line (or worse, failing silently and (cont.)

