---
id: "02_Data_Engineering/Data_Quality/Validation/Detecting_Unit_Changes"
level: 4
type: "concept"
topic: "02_Data_Engineering/Data_Quality/Validation"
tools: []
---

# Detecting Unit Changes

**L4 Advanced** · [↑ Validation](<_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** A unit change (rupees → paise), a sudden volume drop or a stale feed produces data that is *valid* row by row but wrong in aggregate, so you catch it by comparing today's distribution, volume and freshness against a learned baseline.

### 1. The big picture
Rule-based validation (types, nulls, ranges) catches data that is *malformed*. The nastiest incidents are data that is perfectly well-formed but *behaves differently*: an upstream team starts sending amounts in paise instead of rupees, a feed silently stops for three days, a field starts defaulting to zero, a categorical column grows a new value. Every row passes every constraint; the finance report is simply 100× off.

This is why production pipelines add **behaviour-based monitoring** on top of rule-based checks: sudden drops in row counts, revenue outside a threshold band, unexpected new categories, shifts in a distribution. Techniques range from fixed thresholds, through statistical bands (z-score, IQR, seasonal baselines), to ML-based drift detection.

### 2. Core concepts and vocabulary
- **Baseline** — what "normal" looks like, computed from history (same weekday, trailing N days).
- **Volume anomaly** — row count much lower or higher than the baseline.
- **Freshness anomaly** — the newest record (or the gap between loads) exceeds the allowed lag.
- **Distribution drift** — the statistical shape of a column changes (mean, quantiles, category mix, null/zero rate).
- **Unit change** — a multiplicative shift in a numeric column (×100, ×1000, currency/time-unit change). Often recognisable as the median jumping by a clean factor.
- **Outlier** — an individual value far from the rest (z-score, IQR fence).
- **Monotonic column** — a column that should only increase (sequence number, cumulative total, watermark); any decrease is a defect.
- **Seasonality** — predictable weekly/holiday cycles that must be in the baseline to avoid false alerts.
- **Fabricated / imputed value** — a default such as `0` or `-1` replacing missing data, which makes completeness checks pass while the data is meaningless.

### 3. How it works, step by step
1. **Choose the metric** per table/column: `COUNT(*)`, null rate, zero rate, mean/median, p95, distinct count, max(timestamp).
2. **Compute it per period** (per day/hour) and store it in a metrics table — this is your history.
3. **Build the expected value**: simple (average of last 14 days), seasonal (same weekday over last 8 weeks) or model-based.
4. **Define a band**: e.g. ±20 %, or mean ± 3σ, or IQR fences. Prefer robust statistics (median, MAD) when outliers pollute the history.
5. **Compare** today's value to the band; use `LAG()` to compare to the previous period when no model is needed.
6. **Alert** with context: metric, expected range, actual, first date the shift appeared (so you can find the deploy or upstream change that caused it).
7. **Investigate**: find the first date a metric changed, then segment (by source, country, app version) to isolate the cause.

Unit-change specific tests:
- Ratio of today's median to trailing median ≈ 100 (or 0.01, 1000 …) → suspect unit change.
- Min/max of the column shifts by the same factor.
- Cross-check with a second source or a business invariant (`amount_paise / 100 ≈ amount_rupees` in another table).

### 4. Worked example
Detect the first day a column's zero-rate went from 0 to non-zero, and stale days in a feed:

```sql
-- 1. zero-rate regression: first date the rate became non-zero
WITH zero_rates AS (
  SELECT load_date,
         1.0 * SUM(CASE WHEN habitability = 0 THEN 1 ELSE 0 END) / COUNT(*) AS zero_rate
  FROM   planets
  GROUP  BY load_date
), with_prev AS (
  SELECT load_date, zero_rate,
         LAG(zero_rate) OVER (ORDER BY load_date) AS prev_zero_rate
  FROM zero_rates
)
SELECT * FROM with_prev
WHERE  prev_zero_rate = 0 AND zero_rate <> 0;

-- 2. freshness: gaps between loads larger than one day
WITH loads AS (SELECT DISTINCT load_date FROM planets)
SELECT load_date,
       DATEDIFF(day, LAG(load_date) OVER (ORDER BY load_date), load_date) AS days_since_prev
FROM   loads
QUALIFY days_since_prev > 1;      -- or wrap in a subquery where QUALIFY is unavailable

-- 3. unit change: median jumps by a clean factor
SELECT load_date,
       PERCENTILE_CONT(0.5) WITHIN GROUP (ORDER BY amount) AS median_amount,
       median_amount / LAG(median_amount) OVER (ORDER BY load_date) AS ratio_vs_prev
FROM   payments GROUP BY load_date;
```
Reading it: query 1 returns the exact onset date — correlate with deploys/upstream releases. Query 2 lists the days the feed "went offline" (e.g. gaps of 8, 4, 8 days). Query 3's `ratio_vs_prev ≈ 100` on one day, then staying there, is the signature of rupees→paise (a single spike would be an outlier instead).

### 5. Under the hood   *(optional at L4)*
- Aggregate monitoring answers "did the shape change?" but not "which rows?". Pair it with row-level segmentation to find the responsible partition.
- **Gradual changes** (a feature flag rolled out slowly) never cross a fixed threshold; compare against a *long* baseline or use change-point detection / trend tests.
- **Categorical drift** is invisible to means and null rates. Use distribution distance (population stability index, chi-square, KL/JS divergence) on category frequencies.
- Tools: Deequ anomaly-detection strategies (relative-rate-of-change, online normal), custom statistical thresholds, and ML-based monitors that learn per-column baselines automatically.

### 6. Performance and design implications
- Compute metrics incrementally per partition/day and store only the summaries; never re-scan history for every check.
- Seasonal baselines cut false positives drastically (a weekend dip is not an incident).
- Avoid *overfitting thresholds*: too tight → alert fatigue; too loose → missed drifts. Review thresholds after each incident.
- Monitor a small number of business-critical metrics tightly (active listings, revenue) in addition to broad automatic coverage.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `LAG()/LEAD() OVER (ORDER BY date)` | Compare to previous period | Day-over-day deltas, onset detection |
| `PERCENTILE_CONT`, `STDDEV` | Robust/standard spread measures | Bands and z-scores |
| `df.isnull().mean()` (pandas) | % missing per column | Quick profiling |
| `df.describe()` / IQR fence | Spot outliers | Ad-hoc analysis |
| Deequ `AnomalyDetection` | Spark-scale metric anomaly detection | Big batch tables |
| dbt `dbt_utils.recency`, `expect_*` | Freshness / volume tests | ELT pipelines |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| Fixed thresholds on stable, critical metrics | Metrics are seasonal or trending |
| Seasonal/statistical baselines | Not enough history (new table) |
| ML-based monitors for hundreds of tables | You need explainable, auditable rules (finance/regulatory) |
| Median/MAD | Distribution is multi-modal and needs per-segment baselines |

### 9. Common mistakes
- No historical baseline → can't tell abnormal from normal → persist daily metrics.
- Alerting only on the aggregate → a small segment breaks unnoticed → segment key metrics (region, source).
- Ignoring small, consistent deviations → slow drift accumulates → add trend/long-window checks.
- Treating `0` defaults as real values → completeness passes, data is fabricated → monitor zero-rate and distinct-count too.
- Not considering business context → "spike" was a promotion → include a calendar/annotations.

### 10. 🎯 Interview kit
**30-second answer:** "I'd compare each load's volume, freshness and column distributions with a baseline built from history — same-weekday medians and a band — and alert on deviation. A unit change shows up as the median jumping by a clean factor like 100, which I catch with a ratio check on the column, ideally cross-checked against another source."

**Structure a longer answer:** clarify what "valid" already guarantees → define metrics → baseline & band → seasonality → alert with onset date and segment → remediation (quarantine/backfill) → prevention (data contract with units in the schema).

**Likely follow-ups:**
- *Row counts with weekly/holiday seasonality?* → seasonal baseline (same weekday), holiday calendar, relative bands, alert on persistent not single-point deviation.
- *Column that should only increase drops?* → compare to `LAG()` / running max; flag `value < prev_value`.
- *Drift in categorical distribution but mean/null-rate fine?* → PSI / chi-square on category proportions with thresholds tuned from history.
- *Completeness passes because nulls became zeros?* → track zero/default rate and distinct count; compare against source nullability.

**Signals interviewers look for:** you distinguish rule-based vs behaviour-based checks, know false-positive control, and can pinpoint the onset date and segment.

### 11. Check yourself
- [ ] Write a `LAG()` query that finds days where row count fell more than 30 % below the previous 7-day average.
- [ ] Explain how you'd distinguish an outlier from a unit change.
- [ ] Design a freshness check and say what threshold you'd pick and why.
- [ ] Name two ways slow drift evades threshold alerts.
<!-- explainer:end -->

## Practice questions

- [Intermediate] [How would you detect outliers in a numeric column such as an unusually large transaction amount?](<../../../PRACTICE/02_Data_Engineering/Data_Quality/Validation/Q-14347_How_would_you_detect_outliers_in_a_numeric_column_such_as_an.md>)
- [Intermediate] [What is a data freshness or timeliness check and why does a stale dataset cause problems?](<../../../PRACTICE/02_Data_Engineering/Data_Quality/Validation/Q-14367_What_is_a_data_freshness_or_timeliness_check_and_why_does_a_.md>)
- [Intermediate] [How would you use pandas to find columns with missing values and the percentage missing?](<../../../PRACTICE/02_Data_Engineering/Data_Quality/Validation/Q-14370_How_would_you_use_pandas_to_find_columns_with_missing_values.md>)
- [Intermediate] [How would you monitor row counts over time to catch a day where far fewer rows arrived than usual?](<../../../PRACTICE/02_Data_Engineering/Data_Quality/Validation/Q-14419_How_would_you_monitor_row_counts_over_time_to_catch_a_day_wh.md>)
- [Deep] [How would you detect that a numeric column has suddenly changed units for example rupees becoming paise?](<../../../PRACTICE/02_Data_Engineering/Data_Quality/Validation/Q-14385_How_would_you_detect_that_a_numeric_column_has_suddenly_chan.md>)
- [Deep] [How would you catch a situation where a column that should always increase suddenly drops in value?](<../../../PRACTICE/02_Data_Engineering/Data_Quality/Validation/Q-14401_How_would_you_catch_a_situation_where_a_column_that_should_a.md>)
- [Deep] [What is data drift and give an example of how it could break a downstream report.](<../../../PRACTICE/02_Data_Engineering/Data_Quality/Validation/Q-14402_What_is_data_drift_and_give_an_example_of_how_it_could_break.md>)
- [Deep] [What is anomaly detection in the context of data volume and why might row count spikes matter?](<../../../PRACTICE/02_Data_Engineering/Data_Quality/Validation/Q-14420_What_is_anomaly_detection_in_the_context_of_data_volume_and_.md>)

## Related concepts

- [Distribution Change Alerts](<../Data_Observability/Distribution_Change_Alerts.md>) *(Data Observability)*
- [Scaling with Contaminated Data](<../../../11_AI_and_ML/ML_Foundations/Data_Preprocessing/Scaling_with_Contaminated_Data.md>) *(Data Preprocessing)*
- [Drift and Threshold Detection](<../../../11_AI_and_ML/Applied_ML/Anomaly_Detection/Drift_and_Threshold_Detection.md>) *(Anomaly Detection)*

## References

- *Ultimate Data Engineering Design Patterns* — Bragadeesh Sundararajan — Data Ingestion Patterns in Data Engineering › 4. Data Ingestion Patterns in Data Engineering
- *Ultimate Data Engineering Design Patterns* — Bragadeesh Sundararajan — Data Quality Patterns › 10. Data Quality Patterns
- *Automating Data Quality Monitoring* — Jeremy Stanley, Paige Schwartz — Data Quality Monitoring Strategies and the Role of Automation
- *Automating Data Quality Monitoring* — Jeremy Stanley, Paige Schwartz — Assessing the Business Impact of Automated Data Quality Monitoring
- *Automating Data Quality Monitoring* — Jeremy Stanley, Paige Schwartz — Automating Data Quality Monitoring with Machine Learning
- *Automating Data Quality Monitoring* — Jeremy Stanley, Paige Schwartz — Building a Model That Works on Real-World Data
- *Data Quality Fundamentals* — Barr Moses, Lior Gavish, Molly Vorwerck — Monitoring and Anomaly Detection for Your Data Pipelines › Building an Anomaly Detection Algorithm
- *Data Quality Fundamentals* — Barr Moses, Lior Gavish, Molly Vorwerck — Monitoring and Anomaly Detection for Your Data Pipelines › Understanding Distribution
- *Data Quality Fundamentals* — Barr Moses, Lior Gavish, Molly Vorwerck — Monitoring and Anomaly Detection for Your Data Pipelines › Investigating a Data Anomaly
- *Data Quality Fundamentals* — Barr Moses, Lior Gavish, Molly Vorwerck — Monitoring and Anomaly Detection for Your Data Pipelines › Scaling Anomaly Detection with Python and Machine Learning
- *Data Contracts: Developing Production-Grade Pipelines at Scale* — Chad Sanderson, Mark Freeman — Data Quality Isn't About Pristine Data › Quarterly incident count
- *Data Contracts: Developing Production-Grade Pipelines at Scale* — Chad Sanderson, Mark Freeman — The Data Contract Components Detection and Prevention › Detection
- *Data Contracts: Developing Production-Grade Pipelines at Scale* — Chad Sanderson, Mark Freeman — D
- *Data Pipelines Pocket Reference* — James Densmore — Data Validation in Pipelines › Unexpected Change in Row Count After Ingestion
- *Ultimate Data Engineering with Databricks* — Mayank Malhotra — Chapter 5: Data Quality and Validation › Handling Data Anomalies and Outliers
- *Ultimate Data Engineering with Databricks* — Mayank Malhotra — Chapter 5: Data Quality and Validation › Strategies for Managing and Correcting Data Anomalies and Outliers

