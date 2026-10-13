---
id: "02_Data_Engineering/Modeling_and_Warehousing/Data_Warehousing/Fact_Dimension_Integrity"
level: 4
type: "concept"
topic: "02_Data_Engineering/Modeling_and_Warehousing/Data_Warehousing"
tools: []
---

# Fact Dimension Integrity

**L4 Advanced** · [↑ Data Warehousing](<_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** Fact–dimension integrity means every fact row points to exactly one valid dimension row at the right grain — so joins neither lose rows nor multiply them.

### 1. The big picture
A dimensional model works because of a simple contract: a fact table stores measurements at a declared **grain**, each fact row carries foreign keys to its dimensions, and each key resolves to exactly one dimension row. When that contract holds, `SUM(measure)` joined to any dimension is correct. When it breaks, numbers are wrong in ways that are hard to see: revenue disappears because an inner join dropped orphans, or doubles because a dimension key was duplicated.

Cloud and MPP warehouses typically *do not enforce* foreign-key constraints (they're informational, used by the optimizer), so integrity is a property you build into the load process and verify with tests. This lesson covers the grain, the key relationships, late-arriving and missing dimensions, and the checks that keep a warehouse trustworthy. It also touches on why upstream "data debt" — unannounced changes in sources — is the usual root cause.

### 2. Core concepts and vocabulary
- **Grain** — what one fact row represents ("one order line", "one account per day"). Pick the smallest unit required, but no smaller.
- **Surrogate key** — a meaningless integer assigned by the warehouse to each dimension row (or version).
- **Natural/business key** — the identifier from the source system (`customer_id`).
- **Orphan fact** — a fact row whose dimension key matches no dimension row.
- **Unknown/default member** — a reserved dimension row (e.g., key −1 "Unknown") for facts with missing or unresolved references.
- **Inferred member** — a placeholder dimension row created from a fact's business key before the real dimension data has arrived.
- **Late-arriving dimension / fact** — dimension data (or fact data) that shows up after the related rows were processed.
- **Conformed dimension** — a dimension shared consistently across fact tables, enabling cross-process analysis.
- **Fan-out (duplicate multiplication)** — a join producing more rows than the fact side because the dimension isn't unique on the join key.
- **Referential integrity** — every foreign key value exists in the referenced table.

### 3. How it works, step by step
A load process that preserves integrity:
1. **Declare the grain** in writing and a unique key for the fact (`order_id, line_number`).
2. **Load dimensions first** so new members exist before facts reference them.
3. **Assign surrogate keys** in dimension loads; keep the natural key as an attribute with a uniqueness rule (for SCD2, uniqueness is `natural key + version/current flag`).
4. **Resolve keys during fact load** by joining staging rows to dimensions on natural key (and for SCD2, on the effective-date range).
5. **Handle misses explicitly:**
   - create an inferred member and flag it for later enrichment, or
   - assign the Unknown member (−1), or
   - reject/quarantine the row if the dimension is mandatory.
6. **Test** after every load: row counts in vs out, orphan counts, uniqueness of dimension keys, uniqueness of the fact grain.
7. **Backfill/repair** late data: update inferred members when real attributes arrive; reprocess facts only if historical keys must change (usually the surrogate key stays stable).

### 4. Worked example
```sql
-- 1) Resolve keys; send misses to Unknown (-1) and record them
INSERT INTO dw.fact_orderline (order_id, line_no, date_key, customer_key, product_key, qty, net_amount)
SELECT s.order_id, s.line_no,
       COALESCE(d.date_key, -1),
       COALESCE(c.customer_key, -1),
       COALESCE(p.product_key, -1),
       s.qty, s.net_amount
FROM stg.order_lines s
LEFT JOIN dw.dim_date     d ON d.full_date   = s.order_date
LEFT JOIN dw.dim_customer c ON c.customer_nk = s.customer_id
                           AND s.order_date BETWEEN c.valid_from AND c.valid_to    -- SCD2 point-in-time
LEFT JOIN dw.dim_product  p ON p.product_nk  = s.product_id AND p.is_current = 1;

-- 2) Integrity checks
-- a) Orphans (facts pointing nowhere)
SELECT COUNT(*) AS orphan_facts
FROM dw.fact_orderline f
LEFT JOIN dw.dim_customer c ON c.customer_key = f.customer_key
WHERE c.customer_key IS NULL;

-- b) Unknown-member rate (a quality signal)
SELECT 100.0 * SUM(CASE WHEN customer_key = -1 THEN 1 ELSE 0 END) / COUNT(*) AS pct_unknown_customer
FROM dw.fact_orderline;

-- c) Dimension key uniqueness (fan-out guard)
SELECT customer_nk, valid_from, COUNT(*) FROM dw.dim_customer
GROUP BY customer_nk, valid_from HAVING COUNT(*) > 1;

-- d) Grain uniqueness
SELECT order_id, line_no, COUNT(*) FROM dw.fact_orderline GROUP BY order_id, line_no HAVING COUNT(*) > 1;

-- e) Reconciliation to the source
SELECT (SELECT SUM(net_amount) FROM stg.order_lines)      AS staged,
       (SELECT SUM(net_amount) FROM dw.fact_orderline WHERE load_batch_id = :b) AS loaded;
```
Interpretation: check (a) should be 0 because the `LEFT JOIN`+`COALESCE` maps misses to −1; (b) tells you how many facts depend on missing reference data and should be trended; (c) and (d) catch fan-out and duplicate facts; (e) proves nothing vanished. If (b) jumps from 0.1% to 12%, an upstream reference feed is broken, not the warehouse.

### 5. Under the hood
- **Why inner joins are dangerous at load time:** an `INNER JOIN` to a dimension silently drops facts with no match; a `LEFT JOIN` with a default key keeps the measure (so totals still reconcile) and exposes the data issue as "Unknown".
- **Why duplicates multiply:** if a dimension has two current rows for one natural key (a failed SCD2 close-out), every fact for that customer joins twice and `SUM` doubles — grain is inflated.
- **Late-arriving dimension and SCD2:** a fact may arrive before the customer row. With an inferred member, the fact gets a real surrogate key immediately; when attributes arrive, the inferred row is updated in place (type 1) so history is consistent, and no fact rewrite is needed.
- **Late-arriving facts** (dated in the past) should be assigned the dimension version valid at the *event date*, not the current version.
- **Constraint enforcement:** many MPP engines accept `PRIMARY KEY`/`FOREIGN KEY` declarations but do not validate them; the optimizer may *trust* them for join elimination, so a violated constraint can produce wrong results — keep declarations truthful through tests.
- **Data debt/contracts:** because analytics is tightly coupled to upstream definitions, an unannounced upstream change propagates into facts; data contracts, tests at ingestion, and lineage reduce blast radius.

### 6. Performance and design implications
- Integer surrogate keys make joins small and fast; wide natural keys inflate fact tables.
- Orphan-check queries on large facts are expensive; run them on the loaded partition only and keep daily counters.
- Declaring informational constraints can help optimizers but only if they are guaranteed true.
- Model at the minimum grain needed; extra granularity multiplies row counts and storage.
- Metrics: orphan count, % unknown per dimension, duplicate-key count, reconciliation difference, late-arriving rate.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `LEFT JOIN … COALESCE(key, -1)` | Preserve facts with unknown dims | Fact loads |
| Unique/PK/FK constraints (informational) | Document and hint the optimizer | MPP warehouses |
| dbt `unique`, `not_null`, `relationships` tests | Automated integrity tests | Every model |
| Great Expectations / custom SQL | Row/aggregate validation | Gate before publish |
| `COUNT(*)` vs `COUNT(DISTINCT)` on grain | Duplicate detection | Post-load checks |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| Unknown member (−1) — reports must reconcile | Business needs to *stop* on missing reference data |
| Inferred members — facts arrive before dimensions | Dimension attributes drive security/regulatory use |
| Reject/quarantine facts — mandatory dimension | You can't reprocess later |
| Enforced FKs (OLTP-like engines) | MPP load speed matters; use tests instead |
| Coarser grain — stable reporting needs | Future questions need line-level detail |

### 9. Common mistakes
- Inner-joining to dimensions in the load → vanishing revenue → left join + unknown member + counts.
- Non-unique dimension natural keys in SCD1 → doubled sums → uniqueness tests.
- Mixed grains in one fact table (header and line amounts) → double counting → separate fact tables or allocate.
- Resolving SCD2 keys with `is_current` for historical facts → wrong attributes → point-in-time join.
- Not monitoring % Unknown → silent upstream breakage → trend and alert.

### 10. 🎯 Interview kit
**30-second answer:** Integrity in a star schema means each fact row, at a declared grain, has valid keys that resolve to exactly one dimension row. I load dimensions first, resolve surrogate keys with left joins and an Unknown member, handle late-arriving dimensions with inferred members, and verify with orphan, duplicate, and reconciliation tests.

**Structure a longer answer:** clarify (grain, sources, tolerance) → mechanism (load order, key lookup, defaults) → trade-offs (reject vs unknown vs inferred) → scenario (revenue drops after a join change).

**Likely follow-ups:**
- *Where do orphans come from?* → late dimensions, deleted/merged source records, bad joins, upstream changes.
- *Why not enforce FKs?* → load performance and engine support; use tests.
- *How do you avoid double counting?* → unique dimension keys, one grain per fact, bridge weighting.
- *SCD2 lookups?* → by event date range.

**Signals interviewers look for:** grain discipline, explicit missing-data policy, automated verification.

### 11. Check yourself
- [ ] Write the grain statement for a sales fact.
- [ ] Explain how an inner join silently loses data.
- [ ] Implement an Unknown-member lookup.
- [ ] List four integrity tests and what each catches.
<!-- explainer:end -->

## Practice questions

- [Basic] [Explain the difference between a fact table and a dimension table with an example.](<../../../PRACTICE/02_Data_Engineering/Modeling_and_Warehousing/Data_Warehousing/Q-18972_Explain_the_difference_between_a_fact_table_and_a_dimension_.md>)
- [Basic] [How does a GROUP BY with aggregate functions support building summary reports on a fact table?](<../../../PRACTICE/02_Data_Engineering/Modeling_and_Warehousing/Data_Warehousing/Q-19044_How_does_a_GROUP_BY_with_aggregate_functions_support_buildin.md>)
- [Intermediate] [What is a window function and how could you use it to compute a running total in warehouse reporting?](<../../../PRACTICE/02_Data_Engineering/Modeling_and_Warehousing/Data_Warehousing/Q-19045_What_is_a_window_function_and_how_could_you_use_it_to_comput.md>)
- [Deep] [What is a junk dimension and when would you create one?](<../../../PRACTICE/02_Data_Engineering/Modeling_and_Warehousing/Data_Warehousing/Q-18992_What_is_a_junk_dimension_and_when_would_you_create_one.md>)
- [Deep] [What are additive, semi-additive, and non-additive measures? Give an example of each.](<../../../PRACTICE/02_Data_Engineering/Modeling_and_Warehousing/Data_Warehousing/Q-18996_What_are_additive_semi-additive_and_non-additive_measures_Gi.md>)
- [Deep] [What is a late-arriving dimension and how would you resolve the fact row that references it?](<../../../PRACTICE/02_Data_Engineering/Modeling_and_Warehousing/Data_Warehousing/Q-19017_What_is_a_late-arriving_dimension_and_how_would_you_resolve_.md>)
- [Deep] [How would you handle currency or unit conversion when facts come from multiple countries?](<../../../PRACTICE/02_Data_Engineering/Modeling_and_Warehousing/Data_Warehousing/Q-19067_How_would_you_handle_currency_or_unit_conversion_when_facts_.md>)
- [Deep] [How do you handle currency conversion in a multi-country revenue fact so both reporting-currency and local-currency totals stay auditable?](<../../../PRACTICE/02_Data_Engineering/Modeling_and_Warehousing/Data_Warehousing/Q-19091_How_do_you_handle_currency_conversion_in_a_multi-country_rev.md>)

## Related concepts

- [Late Arriving Dimensions](<../../../01_Databases_and_SQL/Database_Internals/DBMS_Concepts/Late_Arriving_Dimensions.md>) *(DBMS Concepts)*
- [Fact and Dimension Keys](<../Dimensional_Modeling/Fact_and_Dimension_Keys.md>) *(Dimensional Modeling)*
- [Special Dimension Types](<../Dimensional_Modeling/Special_Dimension_Types.md>) *(Dimensional Modeling)*

## References

- *Data Pipelines Pocket Reference* — James Densmore — Transforming Data › Modeling Fully Refreshed Data
- *Data Contracts: Developing Production-Grade Pipelines at Scale* — Chad Sanderson, Mark Freeman — Why the Industry Now Needs Data Contracts › What Is Data Debt?
- *The Enterprise Big Data Lake* — Alex Gorelik — Cataloging the Data Lake › Data profiling
- *The Enterprise Big Data Lake* — Alex Gorelik — Cataloging the Data Lake › Tagging

