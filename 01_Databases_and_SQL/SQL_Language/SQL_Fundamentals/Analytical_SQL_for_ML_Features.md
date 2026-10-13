---
id: "01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Analytical_SQL_for_ML_Features"
level: 4
type: "concept"
topic: "01_Databases_and_SQL/SQL_Language/SQL_Fundamentals"
tools: []
---

# Analytical SQL for ML Features

**L4 Advanced** · [↑ SQL Fundamentals](<_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** Building ML features in SQL is aggregation and window-function work plus one non-negotiable rule — every feature value must be computable **using only data that existed at the prediction time** — so you get time-correct training sets, deterministic splits and samples, and monitoring queries that catch broken features.

### 1. The big picture
Modeling happens in Python, but the data lives in a warehouse. Pandas is bound by memory; the database holds terabytes, enforces constraints, and can aggregate close to where data sits. SQL is therefore where **feature engineering at scale** happens: counts and sums per entity, rolling windows, lags, distinct counts, label construction, sampling, and splitting.

The new constraint compared with reporting SQL is **time-correctness**. A dashboard asks "what is true now?". A training row asks "what was knowable at the moment we made the prediction?" Violating that — *leakage* — produces models that score great offline and fail in production. The question bank repeatedly probes this: global-mean normalization over the test set, features computed without an as-of cutoff, random instead of temporal splits.

This lesson ties together aggregation (`GROUP BY`), window functions (frames, `LAG`, `ROW_NUMBER`), NULL handling and date functions, applied to ML data preparation. It is also where *data-quality monitoring* shows up: the same SQL (min/max, null rate, distinct values) powers checks that features have not drifted or broken.

### 2. Core concepts and vocabulary
- **Feature** — a numeric/categorical value derived per entity (user, order) used as model input.
- **Label (target)** — the outcome to predict; also derived by SQL.
- **Entity / grain** — one training row per what (user-day, order)?
- **Prediction (as-of) timestamp** — the moment the prediction would be made; features may use only data before it.
- **Look-ahead / label leakage** — using information from after the as-of time (or from the label itself).
- **Point-in-time (as-of) join** — joining each row to the latest feature value with `valid_from <= asof`.
- **Train/test split** — partition of rows for fitting vs evaluating; should be temporal or entity-based, not random row-based, for time-dependent data.
- **Training-serving skew** — features computed differently in training and production.
- **Class imbalance / undersampling** — rare positive class; keep all positives and sample negatives.
- **Rolling window feature** — aggregate over the previous N days/rows.
- **Feature drift** — distribution at serving time differs from training.

### 3. How it works, step by step
A time-correct training table, in order:

1. **Define the grain and as-of date.** E.g. one row per `user_id` per `asof_date`.
2. **Compute features from events strictly before `asof_date`** (`event_ts < asof_date`).
3. **Compute the label from events in the following window** (`asof_date <= event_ts < asof_date + 30 days`).
4. **Exclude internal/test accounts and duplicates.**
5. **Split deterministically** (by time or hash of entity id).
6. **Sample** classes deterministically if needed.
7. **Persist** the snapshot (feature table versioned by `asof`) so experiments are reproducible.
8. **Validate**: row counts, null rates, ranges, label balance.

### 4. Worked example
```sql
-- Entities × as-of dates (monthly snapshots)
WITH snapshots AS (
  SELECT u.user_id, d.asof_date
  FROM users u CROSS JOIN (VALUES ('2025-01-01'),('2025-02-01'),('2025-03-01')) d(asof_date)
  WHERE u.is_internal = false AND u.created_at < d.asof_date          -- filter test accounts, exist at asof
),
feat AS (
  SELECT s.user_id, s.asof_date,
         COUNT(o.order_id)                                    AS orders_90d,
         COALESCE(SUM(o.amount), 0)                           AS spend_90d,
         MAX(o.order_ts)                                      AS last_order_ts
  FROM snapshots s
  LEFT JOIN orders o
         ON o.user_id = s.user_id
        AND o.order_ts >= s.asof_date - INTERVAL '90 days'
        AND o.order_ts <  s.asof_date                          -- strictly before as-of: no look-ahead
  GROUP BY s.user_id, s.asof_date
),
label AS (
  SELECT s.user_id, s.asof_date,
         CASE WHEN COUNT(o.order_id) > 0 THEN 1 ELSE 0 END    AS bought_next_30d
  FROM snapshots s
  LEFT JOIN orders o
         ON o.user_id = s.user_id
        AND o.order_ts >= s.asof_date
        AND o.order_ts <  s.asof_date + INTERVAL '30 days'
  GROUP BY s.user_id, s.asof_date
)
SELECT f.*, l.bought_next_30d,
       -- deterministic 80/20 split by entity so one user never appears in both sets
       CASE WHEN MOD(ABS(HASH(f.user_id)), 100) < 80 THEN 'train' ELSE 'test' END AS split
FROM feat f JOIN label l USING (user_id, asof_date);
```

How to read it: features and label come from *non-overlapping* time windows on the same `asof_date`. The `LEFT JOIN` keeps users with no orders (otherwise only active users appear — selection bias), and `COALESCE(SUM, 0)` turns "no orders" into 0 for counts/sums while `last_order_ts` stays NULL (unknown). The hash-based split is repeatable across runs and engines that use the same hash; it is also entity-level, which prevents leakage between train and test for the same user.

**Other recurring recipes**

```sql
-- Change since last week; sparse history handled
SELECT user_id, wk, metric,
       metric - COALESCE(LAG(metric) OVER (PARTITION BY user_id ORDER BY wk), 0) AS delta
FROM weekly_metrics;
-- (if the weeks are not contiguous, join to a calendar first; LAG sees the previous *row*, not the previous *week*)

-- Rolling 7-row sum with explicit frame (no self-join, O(n log n) vs O(n²))
SUM(amount) OVER (PARTITION BY user_id ORDER BY d ROWS BETWEEN 6 PRECEDING AND CURRENT ROW)

-- Balanced sample: N rows per group, reproducible
SELECT * FROM (
  SELECT t.*, ROW_NUMBER() OVER (PARTITION BY user_id ORDER BY HASH(CONCAT(row_id, 'seed42'))) AS rn
  FROM training t) x
WHERE rn <= 100;

-- Detect duplicates
SELECT * FROM (SELECT t.*, ROW_NUMBER() OVER (PARTITION BY user_id, asof_date ORDER BY ingested_at DESC) rn FROM feat t) x WHERE rn > 1;

-- Unseen category at serving time (anti-join to training vocabulary)
SELECT DISTINCT s.country
FROM serving_features s
LEFT JOIN (SELECT DISTINCT country FROM training_features) t ON s.country = t.country
WHERE t.country IS NULL;

-- Out-of-range rate vs training min/max
SELECT AVG(CASE WHEN x < p.min_x OR x > p.max_x THEN 1.0 ELSE 0 END) AS out_of_range_rate
FROM serving_features s CROSS JOIN training_profile p;

-- Standard deviation for standardization
SELECT AVG(x) AS mu, STDDEV_SAMP(x) AS sigma FROM training WHERE split = 'train';   -- train only!
```

### 5. Under the hood
- **Why normalization stats must come from the training split only.** If mean/std include test rows, information about the test distribution flows into the training transformation; offline metrics become optimistic. Compute `mu`/`sigma` on the train subset, store them in a profile table, and apply the same constants in serving.
- **Why naive rolling features are O(n²).** A self-join `ON b.user_id = a.user_id AND b.d BETWEEN a.d - 30 AND a.d` pairs each row with up to 30+ others, and without an index becomes a nested loop on the user's whole history. A window aggregate sorts once by `(user_id, d)` and slides the frame: roughly O(n log n) plus O(n·frame) (and constant-time with running state for sum/count).
- **Why `ORDER BY RANDOM()` is slow.** It assigns a random key to every row and sorts the entire table — O(n log n) and non-reproducible. Alternatives: hash-based filter (`WHERE MOD(ABS(HASH(id)),100) < 10`), `TABLESAMPLE` (block-level, fast but not uniform per group), or `ROW_NUMBER() OVER (PARTITION BY g ORDER BY HASH(id))` for fixed quota per group.
- **Window-frame NULL behavior.** `AVG(x) OVER (… ROWS 6 PRECEDING)` divides by the count of *non-NULL* values in the frame, not 7. A feature can silently change meaning when sessions with NULL are skipped; audit with a companion `COUNT(x) OVER (same frame)` feature or `COALESCE` explicitly.
- **Deterministic ties.** Running balances and ranks must `ORDER BY` a unique key (`ts, id`); otherwise tied rows can appear in different orders between runs and produce different features.
- **Stale statistics → bad plans.** Ultimate T-SQL notes outdated statistics can make the optimizer expect 1,000 rows and get 1,000,000, picking nested loops and under-sized memory grants (tempdb spills). Feature queries over fast-growing tables should refresh stats; Fritchey's chapter adds that the cardinality estimator version can be controlled (`LEGACY_CARDINALITY_ESTIMATION`) when a plan regresses.
- **Data skew in Spark SQL.** One task running 10× longer than the rest is usually a skewed key (one huge entity), visible in the stage's task-duration distribution; fix with salting, broadcast join, or AQE skew handling — a slow node would show uniformly slow tasks on one executor instead.

### 6. Performance and design implications
- **Materialize snapshots** (`features_asof_YYYYMMDD`) instead of recalculating from raw events each experiment; this also fixes the data version and keeps training/serving consistent. Incremental updates (merge only changed entities) beat full nightly recomputes once volumes grow.
- **Push filters before windows** (date ranges, exclude test accounts) so the window sorts fewer rows.
- **Index/cluster/partition** event tables by `(user_id, event_ts)` or by date to enable pruning.
- **One shared definition** of each feature (a view/dbt model) used by training and serving avoids skew.
- **Document the grain and the as-of rule** next to each feature.
- **Avoid correlated subqueries per row**; use windows or pre-aggregated joins.
- **Beware live queries inside experiments**: results change as data is backfilled; snapshot or pin by ingestion time.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `AVG/SUM/COUNT … OVER (PARTITION BY e ORDER BY t ROWS BETWEEN n PRECEDING AND CURRENT ROW)` | Rolling features | 7/30/90-day aggregates |
| `LAG/LEAD` | Previous/next value | Change features, deltas, label lookahead (labels only) |
| `ROW_NUMBER() OVER (PARTITION BY …)` | Dedup, per-group sampling | Quotas, "latest per entity" |
| `HASH()/FARM_FINGERPRINT()/MD5` + `MOD` | Deterministic split/sample | Train/test, A/B bucketing |
| `STDDEV_SAMP`, `AVG`, `PERCENTILE_CONT` | Distribution stats | Standardization, outlier clipping |
| `ASOF JOIN` (Snowflake, DuckDB, ClickHouse) / `valid_from <= asof` + `ROW_NUMBER` | Point-in-time join | Slowly changing features |
| `TABLESAMPLE` | Fast approximate sampling | Exploration only |
| `LEFT JOIN … IS NULL` | Anti-join | Unseen categories, missing keys |
| `COALESCE(x, 0)` + missing indicator column | Controlled imputation | Count/sum features |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| Entity-level hash split | Time-dependent labels (use a time cut-off split) |
| Time-based split for forecasting/behaviour | Very little history |
| Window frames for rolling features | Irregular timestamps (use `RANGE` or calendar join) |
| Undersample majority with hash ordering | Need calibrated probabilities (reweight instead) |
| Precomputed snapshot tables | Features must be real-time fresh (use streaming/online store) |
| Imputation with 0 + indicator | Zero has a different business meaning than "missing" |

### 9. Common mistakes
- Computing feature windows that include the as-of day or later → leakage → use `< asof`.
- Normalizing with global mean/std → train/test leakage → train-only statistics.
- Random row-level split for user-level data → same user in both sets → split by entity/time.
- `ORDER BY RANDOM()` on a huge table → slow and irreproducible → hash-based sampling.
- Inner joining to events → drops entities with no history → left join and coalesce.
- `LAG` across missing weeks treated as "last week" → wrong deltas → calendar spine.
- Not filtering test/internal accounts → polluted labels → explicit filter.
- AVG over a window with NULL sessions → denominator drift → audit counts.
- Recomputing "latest" features from live tables during training → results change between runs → snapshots.

### 10. 🎯 Interview kit
**30-second answer:** "I do feature engineering in SQL because the data is large and already in the warehouse: aggregations and window frames per entity, with every feature computed strictly before the prediction timestamp to avoid leakage, deterministic hash-based splits and samples for reproducibility, and snapshot tables so training and serving see the same definitions."

**Structure a longer answer:** grain + as-of time → features and labels from disjoint windows → sampling/splitting → validation queries (nulls, ranges, unseen categories) → performance (windows vs self-joins, snapshots).

**Likely follow-ups:**
- *Why still SQL if I have pandas?* → scale, push-down, integrity, governance.
- *Why is a global mean risky?* → leakage.
- *Rolling 30/60/90-day features for millions of entities?* → window frames, not self-joins.
- *Deterministic train/test split?* → hash of entity id mod 100.
- *Undersample the majority class reproducibly?* → `ROW_NUMBER` ordered by hash within label.
- *Detect broken features in production?* → min/max profile, null rate, unseen category anti-join.
- *Skewed Spark task?* → read task-duration distribution; salt keys.
- *Running balance that resets per account?* → `SUM() OVER (PARTITION BY account ORDER BY ts, id ROWS UNBOUNDED PRECEDING)`.

**Signals interviewers look for:** you raise leakage and as-of correctness before being asked, choose entity/time-based splits, and know why windows beat self-joins.

### 11. Check yourself
- [ ] Write a "30-day spend" feature that cannot see the prediction day, and a matching label.
- [ ] Produce a reproducible 80/20 split by user id.
- [ ] Explain why `ORDER BY RANDOM()` doesn't scale and give two alternatives.
- [ ] Detect a new, unseen category in serving data with SQL.
- [ ] Show how to standardize a feature without leaking test statistics.
<!-- explainer:end -->

## Practice questions

- [Basic] [As an ML engineer, why is SQL still a core skill even when most modeling is done in Python and pandas?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Q-40499_As_an_ML_engineer_why_is_SQL_still_a_core_skill_even_when_mo.md>)
- [Intermediate] [How would you compute a feature capturing 'change since last week' per entity in SQL, handling entities with sparse history so the comparison doesn't silently produce NULL for everyone?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Q-40706_How_would_you_compute_a_feature_capturing_change_since_last_.md>)
- [Deep] [How would you diagnose a Spark SQL job where one task runs 10x longer than the rest? Walk through reading the stage's task-duration distribution and identifying skew versus a slow node.](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Q-40150_How_would_you_diagnose_a_Spark_SQL_job_where_one_task_runs_1.md>)
- [Deep] [How would you compute a stratified train/validation/test split in SQL that preserves the label ratio within each entity segment and is stable across re-runs via a hash of the key?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Q-40632_How_would_you_compute_a_stratified_train_validation_test_spl.md>)
- [Deep] [Explain how you'd exclude entities that were still in an observation window (right-censored) from a churn training set in SQL so you don't mislabel not-yet-churned users as retained.](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Q-40672_Explain_how_you_d_exclude_entities_that_were_still_in_an_obs.md>)
- [Deep] [Your eval dataset and production traffic have diverged. How would you write SQL comparing the distribution of query intents in your eval table versus a sampled production-log table?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Q-40857_Your_eval_dataset_and_production_traffic_have_diverged_How_w.md>)
- [Expert] [How would you design a SQL query that produces training rows only from features that were observable at each historical prediction cutoff, using a per-row asof timestamp column?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Q-40614_How_would_you_design_a_SQL_query_that_produces_training_rows.md>)
- [Expert] [Explain how you'd design SQL to compute features for a time-series forecasting model (lags, rolling means, calendar features) while strictly respecting the forecast horizon to avoid look-ahead.](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Q-40663_Explain_how_you_d_design_SQL_to_compute_features_for_a_time-.md>)

## Related concepts

- [Reproducible Training Data Versions](<../../../04_Data_Architecture_and_Strategy/Architecture_Patterns/Data_Architecture/Tiered_Lake_Storage/Reproducible_Training_Data_Versions.md>) *(Data Architecture)*
- [Clock Skew and Time Windows](<../../../07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Clock_Skew_and_Time_Windows/_index.md>) *(Distributed Systems Core)*
- [Filtered Indexes and Plan Cache](<../../../TOOLS/SQL_Server/T-SQL/Filtered_Indexes_and_Plan_Cache.md>) *(T-SQL)*
- [Time-based Windows](<../Window_Functions/Time-based_Windows.md>) *(Window Functions)*
- [Binary Search](<../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Binary_Search.md>) *(Data Structures and Algorithms)*
- [UDF Performance](<../SQL_Functions/UDF_Performance.md>) *(SQL Functions)*

## References

- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 3: SELECT Statements and Filtering Logic › Common Mistakes and Avoiding Them
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 4: Joins, Set Operations, and Subqueries › Real-World Applications
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 8: Query Optimization and Performance Tuning › Filtered Statistics
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 8: Query Optimization and Performance Tuning › Impact of Outdated Statistics
- *T-SQL Fundamentals (4th ed.)* — Itzik Ben-Gan — SQL Graph (cont.)
- *Automating Data Quality Monitoring* — Jeremy Stanley, Paige Schwartz — Automating Data Quality Monitoring with Machine Learning
- *Automating Data Quality Monitoring* — Jeremy Stanley, Paige Schwartz — Building a Model That Works on Real-World Data
- *Automating Data Quality Monitoring* — Jeremy Stanley, Paige Schwartz — Operating Your Solution at Scale › Value Issues
- *SQL Server 2025 Query Performance Tuning (7th ed.)* — Grant Fritchey — FULL_BOOK_NEEDS_MANUAL_SPLIT (cont.) › Chapter 5 Statistics, Data Distribution, And Cardinality
- *Data Engineering Design Patterns* — Bartosz Konieczny — Idempotency Design Patterns › Mutable data source
- *Data Engineering Design Patterns* — Bartosz Konieczny — Pattern: Skew Detector › Examples
- *Databricks Certified Data Engineer Associate Study Guide* — Derar Alhussein — Processing Incremental Data › Applying transformations

