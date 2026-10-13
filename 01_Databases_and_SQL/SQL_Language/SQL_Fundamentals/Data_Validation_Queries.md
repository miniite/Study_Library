---
id: "01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Data_Validation_Queries"
level: 4
type: "concept"
topic: "01_Databases_and_SQL/SQL_Language/SQL_Fundamentals"
tools: []
---

# Data Validation Queries

**L4 Advanced** · [↑ SQL Fundamentals](<_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** A validation query is an assertion written as `SELECT` — it returns the *violating* rows (or a count of them), so "zero rows" means pass — and you run a battery of them (nulls, ranges, uniqueness, types, referential integrity, reconciliation) before promoting data.

### 1. The big picture
Constraints (`NOT NULL`, `CHECK`, `PRIMARY KEY`, `FOREIGN KEY`) enforce rules at write time, but real pipelines load from files, APIs and staging tables where constraints are absent or deliberately relaxed for speed. Validation queries are the compensating control: they run after load and before publishing, and they either gate the pipeline or write results to a monitoring table.

The idea is simple but disciplined: **state the business rule, express its negation as a query, and expect an empty result**. Automating the battery and trending its output over time is what the data-quality-monitoring literature calls moving from ad-hoc rules to a monitored system; the SQL building blocks stay the same.

### 2. Core concepts and vocabulary
- **Assertion query** — returns violating rows; pass = 0 rows.
- **Validity** — values conform to type/format/range/domain.
- **Completeness** — required values are present.
- **Uniqueness** — a business key appears once.
- **Referential integrity** — every foreign key has a parent.
- **Consistency / reconciliation** — two sources/stages agree (row counts, sums).
- **Quarantine (reject) table** — where failing rows are diverted with a reason.
- **Set operators** — `UNION`, `INTERSECT`, `EXCEPT`; the two inputs must have the same column count and compatible types, and the result is a set (duplicates removed unless `ALL`).
- **Sargable** — a predicate form that can use an index seek.

### 3. How it works, step by step
1. List the rules per table (business owner + schema).
2. Express each as a query returning violators, with a stable rule id.
3. Run them after load against the staging copy.
4. Write `(rule_id, run_ts, violator_count, sample_keys)` to a results table.
5. Gate: fail the job on *blocking* rules, warn on *soft* rules, quarantine rows if you can continue.

Common rule patterns:

| Rule | Query shape |
|---|---|
| Not null | `WHERE col IS NULL` |
| Range | `WHERE amount < 0 OR amount > 100000` |
| Domain | `WHERE status NOT IN ('new','paid','void')` |
| Unique key | `GROUP BY k HAVING COUNT(*) > 1` |
| Orphan | `LEFT JOIN parent … WHERE parent.k IS NULL` |
| Format | `WHERE email NOT LIKE '%_@_%._%'` |
| Reconcile | compare `COUNT(*)`/`SUM` across stages; `EXCEPT` both ways |

### 4. Worked example
```sql
-- 1) Business rule: amounts must be positive; labels must be present
SELECT 'amount_positive' AS rule, COUNT(*) AS violators
FROM   dbo.orders_stage WHERE amount <= 0
UNION ALL
SELECT 'label_not_null', COUNT(*) FROM dbo.training_rows WHERE label IS NULL;

-- 2) Single-pass feature check: no unexpected NULLs and within range
SELECT COUNT(*) AS rows_total,
       SUM(CASE WHEN f_score IS NULL THEN 1 ELSE 0 END)              AS null_cnt,
       SUM(CASE WHEN f_score < 0 OR f_score > 1 THEN 1 ELSE 0 END)   AS out_of_range
FROM   dbo.features;

-- 3) Type mismatch between source and target (catch before load)
SELECT s.COLUMN_NAME, s.DATA_TYPE AS src_type, t.DATA_TYPE AS tgt_type,
       s.CHARACTER_MAXIMUM_LENGTH AS src_len, t.CHARACTER_MAXIMUM_LENGTH AS tgt_len
FROM   src.INFORMATION_SCHEMA.COLUMNS s
JOIN   tgt.INFORMATION_SCHEMA.COLUMNS t
       ON t.TABLE_NAME = s.TABLE_NAME AND t.COLUMN_NAME = s.COLUMN_NAME
WHERE  s.TABLE_NAME = 'orders'
  AND (s.DATA_TYPE <> t.DATA_TYPE
       OR ISNULL(s.CHARACTER_MAXIMUM_LENGTH,0) > ISNULL(t.CHARACTER_MAXIMUM_LENGTH,0));

-- 4) Compare two loads with EXCEPT (both directions)
SELECT 'added' AS change, * FROM (SELECT * FROM dbo.orders_new EXCEPT SELECT * FROM dbo.orders_old) a
UNION ALL
SELECT 'removed', * FROM (SELECT * FROM dbo.orders_old EXCEPT SELECT * FROM dbo.orders_new) r;

-- 5) Optional filter list in ONE parameterized query (empty list = no filter)
DECLARE @tags TABLE (tag_id int);   -- fill from a table-valued parameter
SELECT p.*
FROM   dbo.products p
WHERE  NOT EXISTS (SELECT 1 FROM @tags)
   OR  EXISTS (SELECT 1 FROM dbo.product_tags pt
               JOIN @tags t ON t.tag_id = pt.tag_id
               WHERE pt.product_id = p.product_id);
```
Interpretation: the first batch returns two rows — anything with `violators > 0` fails. In (4) `EXCEPT` treats NULLs as equal (unlike `=`), so it compares rows correctly even with NULLs; in the (3) check, a wider source `varchar(255)` than target `varchar(50)` is a truncation risk.

### 5. Under the hood (optional at L4)
`EXCEPT` and `INTERSECT` compare *whole rows* with "not distinct" semantics (NULL equals NULL), implemented as a semi-/anti-join with hashing or sorting plus duplicate removal. This differs from `NOT IN`, which becomes unknown if the subquery yields any NULL and returns no rows — the classic validation bug; prefer `NOT EXISTS` or `EXCEPT`. `VARCHAR(50)` vs `VARCHAR(255)` costs the same storage for the same text (data plus 2-byte length), but a length limit is a *validation* mechanism: it rejects/truncates longer input, and very large declared sizes can inflate memory grants for sorts and hashes because the optimizer assumes about half the declared length per row.

### 6. Performance and design implications
- Validate on the **staging** table, not production, and only the newly loaded partition/batch.
- Combine many rules in one scan with conditional aggregation.
- Keep predicates sargable: `WHERE col >= '2026-01-01'` not `WHERE YEAR(col) = 2026`.
- Index the join keys used by orphan checks (`child.fk`, `parent.pk`).
- `UNION ALL` (no dedup) for combining result sets; `UNION` adds a sort/hash.
- Store failures with sample keys so triage doesn't re-run the query.
- Prefer real constraints where the data is OLTP; `WITH CHECK` constraints are trusted and enable optimizer simplification.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `EXCEPT` / `INTERSECT` | Row-set difference / overlap | Reconcile two loads |
| `NOT EXISTS` | Null-safe anti-semi-join | Orphan checks |
| `CHECK` constraint | Enforce a rule at write | Stable domain/range rules |
| `TRY_CAST / TRY_CONVERT` | Returns NULL instead of error | Find unparsable values |
| `ISNUMERIC` (careful) | Weak numeric test | Prefer `TRY_CAST` |
| `INFORMATION_SCHEMA.COLUMNS` | Metadata | Source/target schema comparison |
| `COLLATE` | Case/accent rules | Case-insensitive ordering/compare |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| Constraints: stable rules, OLTP, need guarantees | Bulk loads where you must load first and clean later |
| Validation queries: staging/warehouse, many rules, need reports | Rules that must never be violated even for a moment |
| Quarantine rows | Loading half a batch breaks consistency — consider all-or-nothing |
| Block on failure | Soft rules — warn only, to avoid alert fatigue |

### 9. Common mistakes
- `NOT IN` with a nullable subquery → no rows returned → false pass → `NOT EXISTS`.
- `= NULL` instead of `IS NULL` → never matches → use `IS NULL`.
- `BETWEEN` assumed exclusive → boundaries included → write explicit `>= AND <`.
- Validating after publishing → bad data already served → validate before swap.
- `ORDER BY name` for "ignore case" on a case-sensitive collation → wrong order → `COLLATE` or `LOWER()`.
- Passing an "empty list" parameter as `IN ()` → syntax error → use the table-valued-parameter / `NOT EXISTS` trick.
- Validating only counts → a row swapped for another passes → also compare checksums or `EXCEPT`.

### 10. 🎯 Interview kit
**30-second answer:** "A validation query returns the rows that violate a rule, so an empty result means pass. I run a battery on the staging table — null, range, domain, uniqueness, orphan, reconciliation — log counts per rule, and gate the load on the blocking ones."

**Structure a longer answer:** clarify (rules, blocking vs soft, volume) → core mechanism (negated predicate, anti-join, `EXCEPT`, conditional aggregation) → trade-offs (constraints vs queries, quarantine vs fail) → scenario (promote staging to warehouse only if 0 violators).

**Likely follow-ups:**
- *`NOT IN` vs `NOT EXISTS`?* → NULL handling.
- *How do you compare two tables?* → `EXCEPT` both ways plus counts/sums.
- *VARCHAR(50) vs (255)?* → same storage for same data; limit is validation; memory grant implications.
- *Optional filter list?* → table-valued parameter + `NOT EXISTS (SELECT 1 FROM @list)` OR branch.

**Signals interviewers look for:** NULL semantics, set-based thinking, gating strategy, logging results.

### 11. Check yourself
- [ ] Write the orphan check for `orders.customer_id`.
- [ ] Explain why `NOT IN` can silently return nothing.
- [ ] Reconcile two tables using `EXCEPT` and counts.
- [ ] Detect source/target type mismatches from metadata.
- [ ] Describe what is blocking vs soft in your pipeline.
<!-- explainer:end -->

## Practice questions

- [Basic] [How would you write a query to list all product names in alphabetical order ignoring case?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Q-39791_How_would_you_write_a_query_to_list_all_product_names_in_alp.md>)
- [Basic] [How would you write a query to find all rows where a numeric column falls between two values, inclusive?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Q-39848_How_would_you_write_a_query_to_find_all_rows_where_a_numeric.md>)
- [Basic] [How would you write SQL to detect and count rows where a numeric amount is negative when the business says it should always be positive?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Q-40004_How_would_you_write_SQL_to_detect_and_count_rows_where_a_num.md>)
- [Basic] [What query would you write to verify that every training row has a non-null label before feeding it to the model?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Q-40539_What_query_would_you_write_to_verify_that_every_training_row.md>)
- [Intermediate] [How does specifying VARCHAR(50) versus VARCHAR(255) affect storage and validation of a column?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Q-39803_How_does_specifying_VARCHAR_50_versus_VARCHAR_255_affect_sto.md>)
- [Intermediate] [How would you write a query to cast and validate that a source string column of amounts contains only valid numbers before loading?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Q-40062_How_would_you_write_a_query_to_cast_and_validate_that_a_sour.md>)
- [Intermediate] [How do you write a query to compute the ratio of two columns as a feature while safely handling division by zero?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Q-40545_How_do_you_write_a_query_to_compute_the_ratio_of_two_columns.md>)
- [Intermediate] [Your transform casts a string amount column and silently NULLs bad rows, shrinking a revenue metric. How would you use TRY_CAST/SAFE_CAST to quarantine bad rows into a rejects table instead of dropping them?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Q-40135_Your_transform_casts_a_string_amount_column_and_silently_NUL.md>)

## Related concepts

- [CTEs in SQL Server](<../../../TOOLS/SQL_Server/SQL_Server_Platform/CTEs_in_SQL_Server.md>) *(SQL Server Platform)*
- [MySQL Query Basics](<../../../TOOLS/MySQL/MySQL_Platform/MySQL_Query_Basics.md>) *(MySQL Platform)*
- [Classic SQL Query Problems](<../../Database_Internals/DBMS_Concepts/Classic_SQL_Query_Problems.md>) *(DBMS Concepts)*

## References

- *Automating Data Quality Monitoring* — Jeremy Stanley, Paige Schwartz — Data Quality Monitoring Strategies and the Role of Automation
- *Automating Data Quality Monitoring* — Jeremy Stanley, Paige Schwartz — Operating Your Solution at Scale › Value Issues
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 1: Getting Started with T-SQL › Formatting Conventions
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 1: Getting Started with T-SQL › Validate Row Counts before DML
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 3: SELECT Statements and Filtering Logic › Column Aliases and Computed Columns
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 3: SELECT Statements and Filtering Logic › OR Operator
- *Cracking the Data Engineering Interview* — Kedeisha Bryan, Taamir Ransome — Must-know foundational SQL concepts
- *Cracking the Data Engineering Interview* — Kedeisha Bryan, Taamir Ransome — Technical interview questions
- *Fuzzy Data Matching with SQL* — Jim Lehmer — Function Junction › ISNUMERIC
- *Fuzzy Data Matching with SQL* — Jim Lehmer — Bad Characters › Chapter 8. Bad Characters
- *Fuzzy Data Matching with SQL* — Jim Lehmer — Bad Characters › Cleaning Up the Input Data
- *Fuzzy Data Matching with SQL* — Jim Lehmer — Orthogonal Data › Lather, Rinse, Repeat
- *SQL Server 2025 Query Performance Tuning (7th ed.)* — Grant Fritchey — FULL_BOOK_NEEDS_MANUAL_SPLIT (cont.) › Chapter 14 Query Design Analysis
- *T-SQL Fundamentals (4th ed.)* — Itzik Ben-Gan — Background to T-SQL querying and programming
- *T-SQL Fundamentals (4th ed.)* — Itzik Ben-Gan — Single-table queries (cont.)
- *T-SQL Fundamentals (4th ed.)* — Itzik Ben-Gan — Programmable objects

