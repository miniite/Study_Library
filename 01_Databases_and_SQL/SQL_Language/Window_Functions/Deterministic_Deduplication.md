---
id: "01_Databases_and_SQL/SQL_Language/Window_Functions/Deterministic_Deduplication"
level: 4
type: "concept"
topic: "01_Databases_and_SQL/SQL_Language/Window_Functions"
tools: []
---

# Deterministic Deduplication

**L4 Advanced** · [↑ Window Functions](<_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** To remove duplicates reliably, number the rows in each duplicate group with ROW_NUMBER using an ORDER BY that is unique, keep number 1, and the same survivor is chosen on every run.

### 1. The big picture
Duplicates arrive from retries, CDC replays, overlapping extracts and late events. "Deduplicate" always hides a decision: *which* copy survives — the latest, the first, the most complete? `SELECT DISTINCT` cannot express that; it only collapses rows that are identical in every column. The window-function pattern can: partition by the business key, order by "which copy wins", keep row 1.

Deterministic means: rerun the job tomorrow, in another environment, on a different physical row order, and you get the same survivor. That property is what makes pipelines idempotent and MERGEs safe.

### 2. Core concepts and vocabulary
- **Dedup key** — the column(s) that define "the same entity" (e.g. `order_id`).
- **Survivor** — the one row kept per key.
- **Ordering rule** — what makes a row win (e.g. latest `updated_at`).
- **Tiebreaker** — extra ORDER BY column(s) that make the ordering total (e.g. `ingest_seq`, `file_offset`).
- **Deterministic** — same input set → same output regardless of physical order or run.
- **Idempotent** — running twice gives the same final state.
- **Dedup scope** — the data compared: current batch, batch plus target table, or a time window in streaming.
- **Exact duplicate** — all columns equal; **key duplicate** — same key, different payload (versions).

### 3. How it works, step by step
1. Decide the dedup key and the winning rule ("latest by `event_time`, then highest `sequence`").
2. Number rows: `ROW_NUMBER() OVER (PARTITION BY key ORDER BY winning_rule DESC, tiebreaker DESC)`.
3. Keep `rn = 1` (select) or delete `rn > 1` (in place).
4. Add a guard: count of survivors must equal count of distinct keys.

### 4. Worked example
```sql
-- keep latest version per customer; ties on updated_at broken by load_id
WITH ranked AS (
  SELECT *,
         ROW_NUMBER() OVER (
           PARTITION BY customer_id
           ORDER BY updated_at DESC, load_id DESC) AS rn
  FROM stg.customer)
SELECT * FROM ranked WHERE rn = 1;
```
Delete duplicates in place through the CTE (T-SQL lets you delete from a CTE over one table):
```sql
WITH d AS (
  SELECT ROW_NUMBER() OVER (PARTITION BY Col1, Col2, Col3 ORDER BY (SELECT NULL)) AS rn
  FROM #Dupes)
DELETE FROM d WHERE rn > 1;
```
For *exact* duplicates partition by every column; any survivor is equivalent, so an arbitrary order is acceptable. For *key* duplicates the order must be explicit and total.

Data-quality guard (should return zero rows):
```sql
SELECT customer_id FROM final GROUP BY customer_id HAVING COUNT(*) > 1;
```

Pre-MERGE pattern, avoiding "multiple source rows matched" (Delta/Iceberg/SQL Server MERGE):
```sql
MERGE INTO dim_customer t
USING (SELECT * FROM (SELECT s.*, ROW_NUMBER() OVER (
                PARTITION BY customer_id ORDER BY op_ts DESC, seq DESC) rn
              FROM cdc_batch s) x WHERE rn = 1) s
ON t.customer_id = s.customer_id
WHEN MATCHED THEN UPDATE SET ...
WHEN NOT MATCHED THEN INSERT ...;
```

### 5. Under the hood
- The plan is a sort on `(partition cols, order cols)`, a Segment/Sequence Project pair that numbers rows, and a Filter on `rn = 1`. Everything is computed before the filter, so the whole input is sorted even though only one row per key survives.
- If the ORDER BY has ties, the engine picks among tied rows by whatever order the sort emits. That can depend on degree of parallelism, spill behaviour, statistics-driven plan changes, storage layout, or in a distributed engine, shuffle order. A test environment with a different physical order silently picks a different survivor.
- Because the numbering is a *derived* value, the result must be materialised or used immediately; there's no persistent "rn".
- In Spark, the same pattern shuffles by the partition key; `dropDuplicates(keys)` is cheaper but picks an arbitrary row, so it is only correct for exact duplicates. A "max-struct" aggregate (`max(struct(order_cols, *))`) avoids a full sort and is deterministic if the struct ordering is total.
- In streaming (Flink SQL `ROW_NUMBER() ... WHERE rn = 1` over a time attribute) the engine keeps per-key **state**; state size grows with key cardinality unless bounded by a TTL or window.

### 6. Performance and design implications
- Support the sort with an index on `(key, order_col DESC, tiebreaker DESC)` INCLUDE (payload); the plan can stream without sorting.
- Dedup as early as possible — fewer rows to join, smaller shuffles — but after you have assembled everything that defines the winner.
- For huge tables, make it incremental: dedup only the new partition/batch and MERGE into the target; or process by key ranges.
- Compare with alternatives: `DISTINCT`/`GROUP BY` (exact duplicates only); `GROUP BY key` with `MAX(updated_at)` then join back (breaks on ties, duplicates the join); `CROSS APPLY TOP 1` (good with index, few keys).
- A unique constraint or MERGE key on the target is the last line of defence, not a substitute for dedup.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `ROW_NUMBER() OVER (PARTITION BY k ORDER BY t DESC, id DESC)` | Numbers copies, winner = 1 | Standard dedup |
| `DELETE FROM cte WHERE rn > 1` | In-place cleanup (T-SQL) | Remove duplicates from a table |
| `DISTINCT` | Collapses fully identical rows | Exact duplicates |
| `QUALIFY ROW_NUMBER() ... = 1` | Filter on window (Snowflake, BigQuery, Teradata) | Avoid the wrapping CTE |
| `dropDuplicates(cols)` | Spark exact dedup | Only when any copy is fine |
| Unique index / constraint | Rejects future duplicates | After cleanup |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| Key duplicates with a clear "latest" rule | Rows are full duplicates → DISTINCT is simpler |
| You can add a total ordering | No tiebreaker exists → fix upstream (add sequence/offset) |
| Batch scope | Scope includes history → compute cost grows; use MERGE with target |
| Streaming | State unbounded → use window/TTL and accept late-duplicate leakage |

### 9. Common mistakes
- ORDER BY only on a timestamp that ties → analyst sees two "latest" rows or the survivor flips between runs → add a deterministic tiebreaker (sequence, offset, load id, then primary key).
- Dropping with `rn > 1` when ordering is `ASC` → keeps the oldest → check direction; test with a fixture.
- Deduping on a subset of the key → deletes legitimately distinct rows → agree on the key.
- ORDER BY `(SELECT NULL)` for key duplicates → arbitrary winner → only for exact duplicates.
- Ordering by `updated_at` only, ignoring NULLs → NULL sorts first or last depending on engine → `COALESCE` or `NULLS LAST`.
- Deduping before applying late updates (incremental model appends only) → history drift → reload the affected window.

### 10. 🎯 Interview kit
**30-second answer:** Partition by the business key, order by the rule that picks the winner plus a unique tiebreaker, assign ROW_NUMBER, and keep row 1. The tiebreaker is what makes the survivor deterministic across reruns and environments.

**Structure a longer answer:** clarify key and "latest" rule → ROW_NUMBER pattern → why DISTINCT/GROUP BY don't suffice → determinism and tiebreaker → scale (index, incremental, MERGE) → validation query.

**Likely follow-ups:**
- *Why ROW_NUMBER not RANK?* → RANK keeps ties; you need exactly one row.
- *What breaks when ordering only by op_ts in CDC?* → two events in the same millisecond; add sequence/LSN.
- *How do you test it?* → fixture with deliberate ties, assert same survivor over shuffled input.
- *500M-row timeouts?* → incremental by partition, index, pre-filter, or Spark max-struct.

**Signals interviewers look for:** you raise ties unprompted, you know the survivor is a business decision, and you add a post-condition check.

### 11. Check yourself
- [ ] Write the dedup query for "latest per key" with a three-level tiebreaker.
- [ ] Build a 4-row fixture with ties and show how two plans can pick different survivors without the tiebreaker.
- [ ] Write the guard query that proves one row per key.
- [ ] Explain how this changes inside a MERGE and in a streaming job.
<!-- explainer:end -->

## Practice questions

- [Intermediate] [In a data pipeline, why is ROW_NUMBER() commonly used to deduplicate rows from a source table?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/Window_Functions/Q-44187_In_a_data_pipeline_why_is_ROW_NUMBER_commonly_used_to_dedupl.md>)
- [Intermediate] [Explain why you must add a tie-breaker column to ORDER BY when using ROW_NUMBER for stable dedup across reruns.](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/Window_Functions/Q-44216_Explain_why_you_must_add_a_tie-breaker_column_to_ORDER_BY_wh.md>)
- [Intermediate] [How would you enforce a data-quality check that flags any partition where ROW_NUMBER()=1 count doesn't equal the distinct-key count, and why does that catch dedup bugs?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/Window_Functions/Q-44281_How_would_you_enforce_a_data-quality_check_that_flags_any_pa.md>)
- [Intermediate] [An analyst reports duplicate 'latest' rows after your ROW_NUMBER dedup; walk through how you'd reproduce it and what non-unique ORDER BY key is the usual root cause.](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/Window_Functions/Q-44312_An_analyst_reports_duplicate_latest_rows_after_your_ROW_NUMB.md>)
- [Deep] [How do you compute a de-duplicated 'latest version per key' from a CDC stream where ties on the ordering timestamp are possible?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/Window_Functions/Q-44060_How_do_you_compute_a_de-duplicated_latest_version_per_key_fr.md>)
- [Deep] [Describe a production incident where a window function silently produced wrong results because the ORDER BY was not unique, and how you'd add a deterministic tiebreaker.](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/Window_Functions/Q-44068_Describe_a_production_incident_where_a_window_function_silen.md>)
- [Deep] [How do you decide the ORDER BY tiebreaker set for a ROW_NUMBER-based dedup so the same survivor is chosen deterministically across environments and reruns?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/Window_Functions/Q-44146_How_do_you_decide_the_ORDER_BY_tiebreaker_set_for_a_ROW_NUMB.md>)
- [Deep] [In a lakehouse merge (Delta/Iceberg), how do you use ROW_NUMBER() to pick the winning source row before a MERGE INTO to avoid the 'multiple source rows matched' error?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/Window_Functions/Q-44275_In_a_lakehouse_merge_Delta_Iceberg_how_do_you_use_ROW_NUMBER.md>)

## Related concepts

- [Deduplication in SQL](<../SQL_Fundamentals/Deduplication_in_SQL.md>) *(SQL Fundamentals)*
- [Classic SQL Query Problems](<../../Database_Internals/DBMS_Concepts/Classic_SQL_Query_Problems.md>) *(DBMS Concepts)*
- [Large Batch Updates](<../SQL_Fundamentals/Large_Batch_Updates.md>) *(SQL Fundamentals)*

## References

- *Data Engineering Design Patterns* — Bartosz Konieczny — Error Management Design Patterns › Solution
- *Beginning T-SQL* — Kathi Kellenberger, Lee Everest — Discovering Windowing Functions › Removing Duplicates
- *Mapping Data Flows in Azure Data Factory* — Mark Kromer — Data Deduplication › Type 1: Distinct Rows
- *Mapping Data Flows in Azure Data Factory* — Mark Kromer — Data Deduplication › Scoring Your Data for Duplication Evaluation
- *Databricks Certified Data Engineer Associate Study Guide* — Derar Alhussein — Processing Change Data Capture › Extending DLT Pipelines with New Notebooks
- *Designing Data-Intensive Applications* — Martin Kleppmann — Sequence Number Ordering › Lamport timestamps
- *Designing Data-Intensive Applications* — Martin Kleppmann — Chapter 12. The Future of Data Systems › Ordering events to capture causality
- *Optimizing Databricks Workloads* — Anirudh Kala, Anshul Bhatnagar, Sarthak Sarbahi — Batch and Real-Time Processing in Databricks › Dropping duplicate rows

