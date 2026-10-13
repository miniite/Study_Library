---
id: "01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Deduplication_in_SQL"
level: 4
type: "concept"
topic: "01_Databases_and_SQL/SQL_Language/SQL_Fundamentals"
tools: []
---

# Deduplication in SQL

**L4 Advanced** · [↑ SQL Fundamentals](<_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** Deduplication is "define what makes two rows the same (the business key), decide which one survives (a deterministic rule), then either filter the extras on read or delete/merge them on write" — and prevent the problem upstream with a unique key and idempotent loads.

### 1. The big picture
Duplicates arrive from retries, at-least-once delivery, overlapping extracts, re-run jobs, and merged sources. They distort counts and sums, break joins (fan-out), and violate uniqueness assumptions. Dedup work has four separate sub-problems that are often confused:

1. **Detect** — find keys that occur more than once.
2. **Decide** — pick the survivor (latest by `updated_at`? highest version? first seen?).
3. **Remove or hide** — delete extras, or select only survivors in a view.
4. **Prevent** — unique constraints, `MERGE`/upsert, idempotency keys.

The same machinery (anti-joins, `EXCEPT`, `ROW_NUMBER`) also powers incremental loads and diffing two snapshots.

### 2. Core concepts and vocabulary
- **Natural / business key** — the columns that define identity (e.g. `email`, or `customer_id + minute + amount`).
- **Exact duplicate** — all columns identical.
- **Near duplicate / key duplicate** — same key, different payload or surrogate id.
- **Survivor** — the row kept per key.
- **`GROUP BY … HAVING COUNT(*) > 1`** — detection pattern.
- **`ROW_NUMBER()`** — numbers rows per key so `rn > 1` are extras.
- **`EXCEPT`** — set difference with row-wise NULL-safe comparison.
- **Upsert / `MERGE`** — insert if new, update if exists.
- **Watermark** — the high-water mark of the last extracted change timestamp.
- **Hard delete detection** — noticing a row vanished from a full snapshot.

### 3. How it works, step by step
**Detect.** Group by the business key and keep groups with more than one row. To see the offending rows, join the keys back, or use a window `COUNT(*) OVER (PARTITION BY key)`.

**Choose a survivor.** Number rows within each key group using a deterministic order, e.g. latest `updated_at`, tie-broken by a unique `id`. `rn = 1` survives.

**Remove.** Delete through a CTE so the engine deletes the underlying table rows (T-SQL allows deleting from a CTE that references one table). Batch large deletes to avoid long locks and log growth.

**Prevent.** Add a `UNIQUE` constraint/index on the business key (after cleaning), and write loads as `MERGE` or `UPDATE … ; INSERT … WHERE NOT EXISTS`.

**Incremental loading.** Select source rows whose `updated_at` is greater than the stored watermark (minus a small overlap for late arrivals), then upsert; because upsert is idempotent, overlap just re-applies the same rows.

### 4. Worked example
```sql
-- A) Detect duplicate emails
SELECT email, COUNT(*) AS cnt
FROM   dbo.users
GROUP  BY email
HAVING COUNT(*) > 1;

-- B) Same customer, amount, and minute but different ids = suspected duplicate transactions
SELECT customer_id, amount,
       DATEADD(MINUTE, DATEDIFF(MINUTE, 0, created_at), 0) AS minute_bucket,
       COUNT(*) AS cnt, MIN(txn_id) AS keep_id
FROM   dbo.transactions
GROUP  BY customer_id, amount, DATEADD(MINUTE, DATEDIFF(MINUTE, 0, created_at), 0)
HAVING COUNT(*) > 1;

-- C) Delete extras, keep latest per email (deterministic)
WITH ranked AS (
    SELECT *, ROW_NUMBER() OVER (PARTITION BY email
                                 ORDER BY updated_at DESC, user_id DESC) AS rn
    FROM   dbo.users
)
DELETE FROM ranked WHERE rn > 1;

-- D) Non-destructive: a view that exposes only survivors
CREATE OR ALTER VIEW dbo.users_current AS
SELECT user_id, email, name, updated_at
FROM  (SELECT *, ROW_NUMBER() OVER (PARTITION BY email ORDER BY updated_at DESC, user_id DESC) AS rn
       FROM dbo.users) x
WHERE rn = 1;

-- E) Incremental upsert from staging
MERGE dbo.dim_customer AS t
USING (SELECT * FROM (SELECT *, ROW_NUMBER() OVER (PARTITION BY customer_id ORDER BY updated_at DESC) rn
                      FROM stg.customer WHERE updated_at > @watermark) s WHERE rn = 1) AS s
   ON t.customer_id = s.customer_id
WHEN MATCHED AND s.updated_at > t.updated_at THEN
     UPDATE SET t.name = s.name, t.updated_at = s.updated_at
WHEN NOT MATCHED THEN
     INSERT (customer_id, name, updated_at) VALUES (s.customer_id, s.name, s.updated_at);

-- F) Which rows vanished from today's full snapshot (hard deletes)?
SELECT t.customer_id
FROM   dbo.dim_customer t
WHERE  t.is_active = 1
  AND  NOT EXISTS (SELECT 1 FROM stg.customer_snapshot s WHERE s.customer_id = t.customer_id);
```
Notes: (E) dedups the *source* batch first — `MERGE` raises an error if several source rows match one target row. (B) uses minute-bucket arithmetic so near-simultaneous retries collapse. (F) is how you detect deletes when the source only gives full snapshots.

### 5. Under the hood (optional at L4)
`GROUP BY … HAVING` is a hash or stream aggregate; the cost is a full scan unless an index on the key makes a stream aggregate possible. `ROW_NUMBER` needs the data ordered by `(partition, order)` — an index on `(email, updated_at DESC)` avoids a Sort. A `UNIQUE` index also stores *one row per key* in its B-tree and checks on insert via a seek, which is why uniqueness enforcement costs an extra index maintenance but makes duplicates impossible. `EXCEPT` removes duplicates in its output (set semantics) — `EXCEPT ALL` is not available in T-SQL, so row *counts* must be compared separately.

### 6. Performance and design implications
- Index the business key (`CREATE UNIQUE INDEX … WHERE …` for filtered keys) before the cleanup so the final constraint is cheap.
- Delete in batches (`DELETE TOP (10000) …` in a loop) to keep the log small and avoid lock escalation.
- On columnar warehouses where DML is expensive, rebuild the table (`CTAS`/`INSERT OVERWRITE`) from the deduped query instead of deleting.
- Always dedup the *incoming batch* before `MERGE`.
- Measure duplicate rate as a metric; a jump signals an upstream retry storm.
- Wide `GROUP BY` keys (many columns) are slow; hash the key (`HASHBYTES`) into a persisted computed column if needed.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `GROUP BY … HAVING COUNT(*) > 1` | Find duplicate keys | Detection |
| `ROW_NUMBER() OVER (…)` + `DELETE` via CTE | Keep one per key | Cleanup |
| `EXCEPT` / `INTERSECT` | Compare two sets | Diff old vs new |
| `MERGE` / `INSERT … SELECT WHERE NOT EXISTS` | Idempotent load | Upserts |
| `CREATE UNIQUE INDEX` | Enforce uniqueness | Prevention |
| `IGNORE_DUP_KEY = ON` | Silently skip duplicate inserts | Bulk loads (use carefully) |
| `SELECT DISTINCT` | Remove exact duplicates | Only if rows are fully identical |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| Dedup view: keep raw history, expose clean data | Query cost on every read — materialize it |
| Physical delete: the duplicates are errors | You need an audit trail — archive first |
| `SELECT DISTINCT` | Rows differ in any column (e.g. a load timestamp) — it won't dedupe |
| `MERGE` | Concurrency-heavy tables (documented race/bug history) — use explicit `UPDATE`+`INSERT` in a transaction with `UPDLOCK, HOLDLOCK` |

### 9. Common mistakes
- Using `DISTINCT` to "fix" a fan-out join → hides a wrong join → find the cardinality problem.
- Non-deterministic `ORDER BY` in `ROW_NUMBER` → survivors differ per run → add a unique tie-breaker.
- Deleting duplicates before adding a unique constraint, in the wrong order → new dupes slip in → constraint first on the clean data.
- Treating `NULL` keys as unique → many "NULL-key" rows survive → decide policy (`COALESCE` or exclude).
- `MERGE` with a duplicated source → error 8672 → dedup staging first.
- Watermark = `MAX(updated_at)` with no overlap → late rows missed → subtract a safety interval.
- Case/whitespace differences (`'A@x.com '`) → "different" keys → normalize (`LOWER(LTRIM(RTRIM()))`) before comparing.

### 10. 🎯 Interview kit
**30-second answer:** "Define the business key, find extras with `GROUP BY HAVING COUNT(*)>1`, pick survivors with `ROW_NUMBER` over the key ordered by a deterministic rule, delete or filter `rn>1`, and add a unique constraint plus idempotent upserts so it doesn't recur."

**Structure a longer answer:** clarify (key definition, survivor rule, volume, history needs) → mechanism → trade-offs (view vs delete, `MERGE` vs insert-where-not-exists) → scenario (retry storm created duplicate payments; fix and prevent).

**Likely follow-ups:**
- *Why not `DISTINCT`?* → needs identical rows; hides logic errors.
- *Incremental load design?* → watermark with overlap + idempotent upsert.
- *Detect hard deletes from full snapshots?* → anti-join snapshot vs current.
- *Diff two loads?* → `EXCEPT` both directions.

**Signals interviewers look for:** deterministic survivor rule, batching deletes, prevention (constraint/idempotency), awareness of NULLs.

### 11. Check yourself
- [ ] Write detect → survivor → delete for a table with `email` duplicates.
- [ ] Explain why a `MERGE` can fail with duplicate source rows.
- [ ] Choose a watermark strategy tolerant of late arrivals.
- [ ] Write the hard-delete detection query.
- [ ] State when `DISTINCT` is and is not valid.
<!-- explainer:end -->

## Practice questions

- [Basic] [How would you write a query to verify a column is unique (no duplicate keys) before promoting a staging table to the warehouse?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Q-40010_How_would_you_write_a_query_to_verify_a_column_is_unique_no_.md>)
- [Intermediate] [Write a query to find all duplicate emails in a Users table.](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Q-40231_Write_a_query_to_find_all_duplicate_emails_in_a_Users_table.md>)
- [Intermediate] [How do you write a query to reconcile a source sum (e.g. total revenue) against the loaded warehouse sum to prove the load was complete?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Q-40012_How_do_you_write_a_query_to_reconcile_a_source_sum_e_g_total.md>)
- [Intermediate] [How would you write a query to detect a broken pipeline where a table has zero rows for today but had data yesterday?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Q-40073_How_would_you_write_a_query_to_detect_a_broken_pipeline_wher.md>)
- [Intermediate] [How do you deduplicate repeated identical alerts (same alert_name + host) keeping only the latest, using ROW_NUMBER?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Q-40988_How_do_you_deduplicate_repeated_identical_alerts_same_alert_.md>)
- [Intermediate] [How would you capture hard deletes from a source that only sends a daily full snapshot, using an EXCEPT/anti-join between yesterday's and today's keys, and mark them soft-deleted downstream?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Q-40171_How_would_you_capture_hard_deletes_from_a_source_that_only_s.md>)
- [Deep] [How would you handle a source that resends the last hour of data on every extract to avoid missing rows, and dedupe it in SQL?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Q-40006_How_would_you_handle_a_source_that_resends_the_last_hour_of_.md>)
- [Deep] [How would you write a query that appends only new partitions to a target table, skipping partitions that already loaded?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Q-40071_How_would_you_write_a_query_that_appends_only_new_partitions.md>)

## Related concepts

- [Classic SQL Query Problems](<../../Database_Internals/DBMS_Concepts/Classic_SQL_Query_Problems.md>) *(DBMS Concepts)*
- [Deterministic Deduplication](<../Window_Functions/Deterministic_Deduplication.md>) *(Window Functions)*
- [Reject vs Quarantine Bad Rows](<../../../11_AI_and_ML/ML_Foundations/Data_Preprocessing/Reject_vs_Quarantine_Bad_Rows.md>) *(Data Preprocessing)*

## References

- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 3: SELECT Statements and Filtering Logic › Eliminating Duplicates with DISTINCT
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 3: SELECT Statements and Filtering Logic › Best Practices
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 4: Joins, Set Operations, and Subqueries › UNIONALL Variant
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 6: CTEs, MERGE, Pivoting, JSON, and XML › Adding DELETE Logic
- *T-SQL Fundamentals (4th ed.)* — Itzik Ben-Gan — Data ��difi�ati�n
- *Azure Data Fundamentals* — Michael John Pena — Relational Data Concepts › Exam Tip
- *Delta Lake: The Definitive Guide* — Denny Lee et al. — Essential Delta Lake Operations › Python
- *Analytics Optimization with Columnstore Indexes in Microsoft SQL Server* — Edward Pollack — Delete anD UpDate OperatiOns UP ATE OPERATI
- *Delta Lake: Up and Running* — Bennie Haelen, Dan Davis — Table Deletes, Updates, and Merges › Modifying unmatched rows using MERGE
- *Delta Lake: Up and Running* — Bennie Haelen, Dan Davis — Using Time Travel › count records where VendorId = 1 using version number\

