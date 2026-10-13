---
id: "02_Data_Engineering/Modeling_and_Warehousing/Slowly_Changing_Dimensions/SCD_Type_2_in_Practice"
level: 4
type: "concept"
topic: "02_Data_Engineering/Modeling_and_Warehousing/Slowly_Changing_Dimensions"
tools: []
---

# SCD Type 2 in Practice

**L4 Advanced** · [↑ Slowly Changing Dimensions](<_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** SCD Type 2 keeps a new row for every version of a dimension member, with validity dates and a current flag, so facts can be joined to the attribute values that were true *at the time*.

### 1. The big picture
Dimensions describe the things facts are about — customers, products, stores. Their attributes change slowly: a customer moves city, a product changes category, an employee changes department. If you simply overwrite the old value (Type 1), past facts silently "move" to the new value and last year's regional report changes retroactively. If you need history to stay true, you keep both versions. That is **Slowly Changing Dimension Type 2**.

The idea is time-sensitive enrichment: when joining a fact to a dimension, you want the dimension version that was valid at the fact's event time. Type 2 does it with a new row per version plus validity dates (current rows have an open end date). A related variant, **Type 4**, keeps the *current* values in one table and *all history* in a second table — same information, different physical implementation. This lesson focuses on Type 2 in practice: the table shape, the load logic, the point-in-time join, and the mistakes that corrupt history.

### 2. Core concepts and vocabulary
- **Slowly changing dimension (SCD)** — a dimension whose attributes change infrequently and unpredictably.
- **Type 1** — overwrite; no history.
- **Type 2** — add a new row per change; keeps full history.
- **Type 3** — add a column for the previous value; limited history.
- **Type 4** — separate current table and history table.
- **Surrogate key** — warehouse-generated key unique per *version* (e.g., `CustomerKey`), distinct from the **business/natural key** (`CustomerId`) that identifies the entity across versions.
- **Effective from / effective to (valid_from / valid_to)** — the period during which a version was true; the current row has `valid_to` NULL or a far-future date (`9999-12-31`).
- **Current flag (`is_current`)** — convenience indicator for the latest version.
- **Tracked attributes** — columns whose change triggers a new version (versus Type 1 attributes that just overwrite).
- **Point-in-time (as-of) join** — joining a fact to the dimension row whose validity period contains the fact's date.
- **Late-arriving dimension / fact** — a fact arriving before its dimension row, or a change that is effective in the past.

### 3. How it works, step by step
Table shape:

| CustomerKey | CustomerId | City | Segment | ValidFrom | ValidTo | IsCurrent |
|---|---|---|---|---|---|---|
| 1001 | C-77 | Paris | Retail | 2024-01-01 | 2025-06-14 | 0 |
| 1002 | C-77 | Lyon | Retail | 2025-06-15 | 2026-03-01 | 0 |
| 1003 | C-77 | Lyon | Premium | 2026-03-02 | NULL | 1 |

Load algorithm for each incoming source row (per business key):
1. Look up the current version (`IsCurrent = 1`) by business key.
2. If none exists → **insert** a new version: new surrogate key, `ValidFrom = effective date`, `ValidTo = NULL`, `IsCurrent = 1`.
3. If one exists and **no tracked attribute changed** → do nothing (or Type 1 overwrite non-tracked attributes).
4. If a tracked attribute changed → **close** the current row (`ValidTo = effective date − 1 day/second`, `IsCurrent = 0`) and **insert** a new current row with a new surrogate key.
5. If the source row was deleted → close the current row (and optionally insert a "deleted" version).
6. Facts loaded afterwards look up the surrogate key valid at the fact date.

Fact lookup ("as-of join"):
```sql
JOIN dim.Customer c
  ON c.CustomerId = f.CustomerId
 AND f.OrderDate >= c.ValidFrom
 AND f.OrderDate <  COALESCE(c.ValidTo, '9999-12-31')
```
Better: resolve the surrogate key once during fact loading and store `CustomerKey` in the fact, so reports use a simple equi-join.

### 4. Worked example
```sql
-- 1. Detect changes and expire old versions
UPDATE d
   SET d.ValidTo   = DATEADD(day, -1, s.EffectiveDate),
       d.IsCurrent = 0
FROM dim.Customer d
JOIN stg.Customer s ON s.CustomerId = d.CustomerId
WHERE d.IsCurrent = 1
  AND ( ISNULL(d.City,'')    <> ISNULL(s.City,'')
     OR ISNULL(d.Segment,'') <> ISNULL(s.Segment,'') );

-- 2. Insert new versions (changed + brand-new customers)
INSERT dim.Customer (CustomerId, City, Segment, ValidFrom, ValidTo, IsCurrent)
SELECT s.CustomerId, s.City, s.Segment, s.EffectiveDate, NULL, 1
FROM stg.Customer s
LEFT JOIN dim.Customer d
       ON d.CustomerId = s.CustomerId AND d.IsCurrent = 1
WHERE d.CustomerId IS NULL;   -- no current row: new customer or just expired in step 1
```

Run order matters: expire first, then insert rows that have no current version. Wrap both in one transaction so a failure cannot leave a customer with no current row or two.

Verify:
```sql
-- exactly one current row per business key
SELECT CustomerId FROM dim.Customer WHERE IsCurrent = 1 GROUP BY CustomerId HAVING COUNT(*) <> 1;
-- no overlapping or gap periods
SELECT a.CustomerId FROM dim.Customer a JOIN dim.Customer b
  ON a.CustomerId=b.CustomerId AND a.CustomerKey<b.CustomerKey
 AND a.ValidTo >= b.ValidFrom;
```

In ADF Mapping Data Flows or dbt snapshots the same pattern is built from three inputs: the incoming data, the current dimension rows, and a hash/compare of tracked columns that routes rows to insert, expire or ignore.

### 5. Under the hood
- **Change detection by hash.** Concatenate tracked columns (with delimiters and null handling) and hash (`HASHBYTES('SHA2_256', …)`), store as `RowHash`, and compare one column instead of many. Beware collisions of naive concatenation (`'ab'+'c'` vs `'a'+'bc'`) — use delimiters.
- **Effective date choice.** Using load time as `ValidFrom` is easy but wrong if the business change happened earlier. Use the source's change timestamp when available; backdated changes require rebuilding versions (late-arriving changes).
- **Late-arriving facts** need the version valid *at the fact's event time*, not the current one; late-arriving dimension rows need placeholder ("inferred") members that are updated when the real row arrives.
- **Join cost.** Range joins (`BETWEEN`) are slower than equi-joins and hard to optimize in distributed engines. Resolving the surrogate key at load time moves that cost to ETL once.
- **Distributed engines/lakehouse.** Use `MERGE` (Delta/Iceberg) with a staged "changes" union: rows to expire plus rows to insert — a common trick is to union the source with a copy keyed by `NULL` so a single MERGE does both.
- **Storage growth.** One row per change; volatile attributes (e.g., a balance that changes daily) don't belong in a Type 2 dimension — put them in a fact or a mini-dimension.
- **Type 2 vs Type 4.** Type 4's current table is tiny and fast for "latest" lookups; history is a separate append-only table. Same temporal semantics.

### 6. Performance and design implications
- Index/cluster on `(BusinessKey, ValidFrom)`; filtered index on `IsCurrent = 1` for load lookups.
- Keep tracked attributes few and meaningful; every extra tracked column multiplies versions.
- Reporting: provide a "current" view (`WHERE IsCurrent = 1`) and an "as-of" function for analysts.
- Data tests: one current per key, non-overlapping contiguous periods, `ValidFrom < ValidTo`, no orphan fact keys.
- Row-count growth should be monitored; sudden spikes mean a noisy source column is being tracked.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `HASHBYTES` / `MD5`/`xxhash` | Row hash of tracked columns | Fast change detection |
| `MERGE` (Delta/Iceberg/SQL Server) | Expire + insert in one statement | SCD2 loads in lakehouse/warehouse |
| dbt `snapshot` (timestamp/check strategy) | Builds SCD2 automatically | dbt projects |
| ADF Mapping Data Flow SCD2 pattern | Source + dim lookup + exists/alter-row | ADF pipelines |
| Filtered index `WHERE IsCurrent=1` | Fast current lookups | Load and "latest" queries |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| Type 2: reports must reflect attributes as of event time | Attribute changes constantly (use fact/mini-dimension) |
| Type 1: history irrelevant or corrections | Audit/regulatory history needed |
| Type 3: only "previous vs current" needed | Multiple past values matter |
| Type 4: current lookups dominate; history rare | Simpler single-table model suffices |
| Resolve key at load | Late-arriving changes frequent; need re-key process |

### 9. Common mistakes
- Joining facts to the *current* row only → history rewritten → store the versioned surrogate key in the fact.
- Overlapping or gapped periods → duplicate or missing fact rows → constrain and test.
- Tracking every column → table explosion → track only business-relevant attributes.
- Using NULL-unsafe comparisons (`NULL <> 'x'` is unknown) → missed changes → `ISNULL`/`IS DISTINCT FROM`/hash.
- Non-atomic expire + insert → two or zero current rows → transaction or single MERGE.
- Ignoring deletes and reactivation → ghost current rows → handle via closing and re-open.

### 10. 🎯 Interview kit
**30-second answer:** Type 2 preserves history by inserting a new dimension row per change with a new surrogate key, validity dates and a current flag. Facts store the surrogate key that was current at their time, so reports reflect attributes as they were. The load expires the old version and inserts the new one atomically, detecting changes by comparing tracked columns or a hash.

**Structure a longer answer:** why history matters → table shape → load algorithm → fact join/lookup → edge cases (late data, deletes, backdating) → cost and alternatives.

**Likely follow-ups:**
- *Type 2 vs Type 4?* → Same semantics; Type 2 single table with flags/dates; Type 4 splits current and history.
- *Late-arriving changes?* → Split/rewrite versions and re-key affected facts; or reload partitions.
- *Rapidly changing attributes?* → Mini-dimension or move to fact.
- *How do you test it?* → One current per key; no overlaps/gaps; fact keys all resolved.
- *Why surrogate keys?* → Natural key repeats across versions.

**Signals interviewers look for:** correctness of temporal logic, awareness of late data, and cost of tracked attributes.

### 11. Check yourself
- [ ] Draw a Type 2 table for a customer with two moves and one segment change.
- [ ] Write expire + insert as a single `MERGE`.
- [ ] Explain why facts store the surrogate key.
- [ ] List three data-quality tests for an SCD2 dimension.
- [ ] Describe handling a change that was effective three weeks ago.
<!-- explainer:end -->

## Practice questions

- [Intermediate] [What exactly is an SCD Type 6 dimension and how does it combine Types 1, 2 and 3 in one table?](<../../../PRACTICE/02_Data_Engineering/Modeling_and_Warehousing/Slowly_Changing_Dimensions/Q-39357_What_exactly_is_an_SCD_Type_6_dimension_and_how_does_it_comb.md>)
- [Intermediate] [How would you use a checksum/version column on the source to short-circuit unnecessary work in an SCD Type 2 load?](<../../../PRACTICE/02_Data_Engineering/Modeling_and_Warehousing/Slowly_Changing_Dimensions/Q-39436_How_would_you_use_a_checksum_version_column_on_the_source_to.md>)
- [Deep] [How do you orchestrate an SCD Type 2 load in Airflow so a mid-batch failure doesn't leave the dimension half-updated?](<../../../PRACTICE/02_Data_Engineering/Modeling_and_Warehousing/Slowly_Changing_Dimensions/Q-39364_How_do_you_orchestrate_an_SCD_Type_2_load_in_Airflow_so_a_mi.md>)
- [Deep] [When a fact needs the current dimension attribute AND the attribute at event time, how do you serve both without joining twice awkwardly?](<../../../PRACTICE/02_Data_Engineering/Modeling_and_Warehousing/Slowly_Changing_Dimensions/Q-39385_When_a_fact_needs_the_current_dimension_attribute_AND_the_at.md>)
- [Deep] [How would you serve both a real-time current view and a historical SCD Type 2 view of the same dimension in a lakehouse?](<../../../PRACTICE/02_Data_Engineering/Modeling_and_Warehousing/Slowly_Changing_Dimensions/Q-39405_How_would_you_serve_both_a_real-time_current_view_and_a_hist.md>)
- [Deep] [What's your strategy for handling SCD Type 2 across environment promotions so surrogate keys and history match between dev/stage/prod?](<../../../PRACTICE/02_Data_Engineering/Modeling_and_Warehousing/Slowly_Changing_Dimensions/Q-39424_What_s_your_strategy_for_handling_SCD_Type_2_across_environm.md>)
- [Expert] [With CDC-driven SCD, out-of-order events can close a version before its update arrives — how do you handle ordering and watermarking?](<../../../PRACTICE/02_Data_Engineering/Modeling_and_Warehousing/Slowly_Changing_Dimensions/Q-39350_With_CDC-driven_SCD_out-of-order_events_can_close_a_version_.md>)
- [Expert] [Walk me through detecting and repairing gaps or overlaps in the validity intervals of an existing SCD Type 2 dimension.](<../../../PRACTICE/02_Data_Engineering/Modeling_and_Warehousing/Slowly_Changing_Dimensions/Q-39393_Walk_me_through_detecting_and_repairing_gaps_or_overlaps_in_.md>)

## Related concepts

- [SCD Type 2 vs Mini-dimensions](<../Dimensional_Modeling/SCD_Type_2_vs_Mini-dimensions.md>) *(Dimensional Modeling)*
- [SCD Type 1 vs Type 2](<../Data_Warehousing/SCD_Type_1_vs_Type_2.md>) *(Data Warehousing)*
- [Joining Type 2 Dimensions](<../../../01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Joining_Type_2_Dimensions.md>) *(SQL Fundamentals)*

## References

- *Data Engineering Design Patterns* — Bartosz Konieczny — Chapter 5. Data Value Design Patterns › Slowly Changing Dimensions
- *Mapping Data Flows in Azure Data Factory* — Mark Kromer — Slowly Changing Dimensions › Data Sources

