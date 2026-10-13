---
id: "02_Data_Engineering/Data_Quality/Data_Observability/Distribution_Change_Alerts"
level: 4
type: "concept"
topic: "02_Data_Engineering/Data_Quality/Data_Observability"
tools: []
---

# Distribution Change Alerts

**L4 Advanced** · [↑ Data Observability](<_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** Distribution-change alerts compare today's statistical shape of the data (volume, null rate, value ranges, category shares) with an expected baseline so you catch silent breakage — and the craft is choosing baselines and thresholds that catch real incidents without drowning on-call in false alarms.

### 1. The big picture
Pipelines often "succeed" while producing wrong data: a wrong date filter processes zero rows, a timezone bug shifts every timestamp by hours, a new country code floods a categorical column, a unit change multiplies amounts by 100. None of these throw errors. Distribution monitoring turns the table's *shape* into time series — row count, null %, mean, percentiles, distinct counts, category frequencies, freshness gaps — and alerts when a new observation departs from what history predicts.

The books frame detection as a prediction problem: even a simple freshness rule ("any outage longer than X days is an anomaly") is a model with a parameter, and you can evaluate it with precision, recall and F-score against labelled historical incidents. This is the key mindset: alerts are classifiers, and their false positives and false negatives can be measured and tuned. The other half is human: legitimate business events (a marketing campaign, a sale) look like anomalies, so alert design needs communication with business owners, not only math. Notification design must also avoid alert fatigue and send each alert to one audience.

### 2. Core concepts and vocabulary
- **Baseline** — expected value or range, learned from history or declared.
- **Static threshold** — fixed bound (null rate < 2%).
- **Dynamic threshold** — bound computed from a rolling window or seasonal model.
- **Seasonality** — predictable weekly/daily/annual patterns (weekend dips, month-end spikes).
- **Precision / recall** — share of alerts that are real incidents / share of real incidents that were alerted.
- **False positive / false negative** — noise alert / missed incident.
- **Drift** — gradual shift in distribution (feature drift for ML).
- **Population Stability Index (PSI), KS test, chi-square, JS divergence** — measures of distribution difference.
- **Skew detector** — compares size or key concentration of a dataset to previous runs.
- **Fatality loop** — a bad day becomes the baseline, so the next good day looks anomalous.
- **Alert fatigue** — responders ignore alerts after too much noise.

### 3. How it works, step by step
1. **Choose metrics per column/table**: row count, freshness lag, null %, distinct count, min/max, mean/stddev/percentiles, top-k category shares.
2. **Compute them per partition/batch** (daily, hourly) and store in a metrics table.
3. **Pick a baseline method**:
   - same weekday median/mean of last N weeks (handles weekly seasonality),
   - rolling window with robust statistics (median/MAD, IQR),
   - forecasting model (Prophet/ARIMA) with prediction interval,
   - distribution distance (PSI/KS) versus a reference window.
4. **Set thresholds** from historical residuals (e.g., 3 MAD) and calibrate on labelled incidents using precision/recall.
5. **Add calendar context**: weekends/holidays/known campaigns are expected-quiet or expected-spike days.
6. **Suppress duplicates** and group alerts by root cause using lineage.
7. **Handle the alert**: send to one owner with the metric, expected range, deviation, sample rows and recent deploys.
8. **Feed back**: label each alert real/false and re-tune.

### 4. Worked example
```sql
-- Metrics captured per daily batch
CREATE TABLE ops.col_metrics AS
SELECT CURRENT_DATE AS dt, 'fct_orders' AS tbl, 'amount' AS col,
       count(*)                                   AS n_rows,
       avg(CASE WHEN amount IS NULL THEN 1.0 ELSE 0 END) AS null_rate,
       avg(amount) AS mean_v, approx_percentile(amount, 0.5) AS p50,
       approx_percentile(amount, 0.99) AS p99,
       count(DISTINCT customer_id) AS distinct_cust
FROM fct_orders WHERE order_date = CURRENT_DATE;

-- Robust same-weekday z-score on row count
WITH h AS (
  SELECT n_rows FROM ops.col_metrics
  WHERE tbl='fct_orders' AND dt < CURRENT_DATE AND dt >= CURRENT_DATE - 56
    AND dayofweek(dt) = dayofweek(CURRENT_DATE)
),
s AS (SELECT median(n_rows) m, median(abs(n_rows - median(n_rows) OVER ())) mad FROM h)
SELECT today.n_rows, s.m,
       (today.n_rows - s.m) / NULLIF(1.4826 * s.mad, 0) AS robust_z
FROM ops.col_metrics today, s
WHERE today.tbl='fct_orders' AND today.dt = CURRENT_DATE;

-- Categorical shift: share of each country vs last 28 days
SELECT country,
       today_share, base_share, abs(today_share - base_share) AS delta
FROM country_share_today JOIN country_share_baseline USING (country)
ORDER BY delta DESC LIMIT 10;
```
Reading it: |robust_z| above ~4 on a tier-1 table warrants an alert; a new `country` with no baseline share is itself a signal. A zero-row success is caught by `n_rows = 0` or ratio near 0, regardless of baseline sophistication — keep a hard "never zero for tier-1" rule.

**Fatality loop example:** one day's load is a third of normal and fails the skew check. If not repaired by next day, the next *valid* day is compared to the bad day and looks 3x too large — so it fails too. Fix: compare against a baseline built from validated/accepted days only, or against the same weekday median, and exclude failed runs from the baseline.

### 5. Under the hood
- **Why same-weekday baselines:** many business datasets have strong weekly cycles; comparing Monday to Sunday generates constant false alarms.
- **Rolling windows vs fixed thresholds:** fixed thresholds are transparent but go stale with growth; rolling windows adapt but can *learn* a slow degradation or outage as normal. Combine: dynamic band plus hard floors/ceilings.
- **Distance metrics:** PSI compares binned distributions (rule of thumb: <0.1 stable, 0.1–0.25 moderate shift, >0.25 major); KS suits continuous values; chi-square or JS divergence for categories. With huge samples every tiny difference is "significant", so use effect-size thresholds, not p-values alone.
- **Detecting code-change artefacts:** annotate deployments (commit SHA, job version) in the metrics timeline. A step change aligned with a deploy and affecting only data produced after it points to pipeline logic; a change aligned with a source system release points upstream; a change affecting source and target equally is real business change.
- **Freshness vs newest record age:** table update time can be recent while the newest event timestamp is old (a backfill, or late source). Monitor both the load time and `max(event_ts)`; partition freshness and record freshness differ.
- **Kafka lag → freshness:** consumer lag in messages divided by consumption rate gives expected delay; translate to "data in the lake is N minutes behind" for consumers.
- **Skew in Spark:** detect with per-key counts (`top key share`) and per-task duration spread (max/median); alert when a single key exceeds a configured share of rows or stage max task time exceeds a multiple of the median.
- **Timezone bugs:** shift in hourly volume profile (peak moves by whole hours), events dated "in the future", or a date boundary mismatch with source counts.

### 6. Performance and design implications
- Compute metrics incrementally per partition, store compactly, and avoid scanning full history.
- Tier checks: cheap volume/freshness everywhere; distribution on key columns of critical tables.
- Alert policy: page only for tier-1 breaches with business impact; ticket or digest the rest; send each alert to one audience.
- Sales events: pre-announce campaigns to widen bands (or use event calendars), keep a hard ceiling for runaway duplicates.
- Evaluate detectors: track precision/recall per monitor; retire monitors with <20% precision.
- Metrics: alerts per week, % actionable, time to acknowledge, missed incidents found by users.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `approx_percentile`, `stddev`, `count(distinct)` | Cheap distribution summaries | Metric collection |
| Great Expectations `expect_column_mean_to_be_between` etc. | Declarative distribution rules | Static ranges |
| dbt `dbt_utils.recency`, anomaly packages (e.g., elementary) | Freshness/anomaly tests | dbt projects |
| Soda checks `anomaly score` | ML-based anomaly | Volume/seasonal metrics |
| PSI/KS in Python (scipy) | Drift scoring | ML feature tables |
| Maintenance windows / calendars | Suppress expected quiet | Weekends, holidays |
| Alert grouping by lineage | Single incident per root cause | Reduce noise |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| Static thresholds | Strong growth/seasonality |
| Same-weekday baseline | Few weeks of history |
| ML forecasting | You cannot explain alerts to owners |
| Real-time alerting | Source has legitimate spiky behaviour — batch up/aggregate |
| Strict gate (block) | Late partial data is better than none — warn and mark |

### 9. Common mistakes
- Baseline includes broken days → fatality loop / normalised outages → build baseline from accepted data.
- Ignoring weekday/holiday patterns → constant false alarms → seasonal baselines and calendars.
- Alerting on every column → noise → select high-value columns, tier by criticality.
- Relying on p-values with huge samples → everything "significant" → effect-size thresholds.
- No zero-row guard → "successful" empty loads go unnoticed → hard minimums.
- Never labelling alerts → can't tune → record outcome and revisit.

### 10. 🎯 Interview kit
**30-second answer:** I turn table shape into time series — row counts, freshness, null rates, quantiles, category shares — compare each batch to a seasonality-aware baseline, and alert on deviations beyond calibrated thresholds, tuned for precision and recall. I keep hard guards for zero rows, exclude bad days from baselines, and route each alert to one owner with lineage and deploy context.

**Structure a longer answer:** clarify → core mechanism → trade-offs → close with a real scenario.

**Likely follow-ups:**
- *Job ran but processed zero rows?* → volume floor vs source count, filter-date audit, reconcile with source.
- *Weekends/holidays?* → expected cadence calendar or same-weekday baseline.
- *Legit sales spikes?* → campaign calendar, wider bands, ceiling guard.
- *Rolling vs fixed thresholds?* → rolling adapts but learns outages; fixed is stable but stale.
- *Real anomaly vs deploy artefact?* → align with deploy annotations and compare source vs target.
- *ML feature drift?* → PSI/KS per feature versus training reference.

**Signals interviewers look for:** measures detector quality, understands seasonality, handles alert fatigue and ownership.

### 11. Check yourself
- [ ] Explain precision vs recall for a freshness alert with a labelled incident history.
- [ ] Describe a fatality loop and two ways to prevent it.
- [ ] Choose a baseline method for a table with weekly seasonality and a growth trend.
- [ ] List three signals that distinguish a timezone bug from a real traffic change.
<!-- explainer:end -->

## Practice questions

- [Intermediate] [How would you detect that a scheduled job ran but processed zero rows due to a wrong date filter?](<../../../PRACTICE/02_Data_Engineering/Data_Quality/Data_Observability/Q-12774_How_would_you_detect_that_a_scheduled_job_ran_but_processed_.md>)
- [Intermediate] [Explain how you would set up freshness monitoring that accounts for weekends and holidays when no data is expected.](<../../../PRACTICE/02_Data_Engineering/Data_Quality/Data_Observability/Q-12797_Explain_how_you_would_set_up_freshness_monitoring_that_accou.md>)
- [Deep] [How do you detect and alert on data drift in a feature table feeding a production ML model?](<../../../PRACTICE/02_Data_Engineering/Data_Quality/Data_Observability/Q-12760_How_do_you_detect_and_alert_on_data_drift_in_a_feature_table.md>)
- [Deep] [How do you balance real-time alerting against alert noise when a data source has legitimate spikes during sales events?](<../../../PRACTICE/02_Data_Engineering/Data_Quality/Data_Observability/Q-12772_How_do_you_balance_real-time_alerting_against_alert_noise_wh.md>)
- [Deep] [How do you observe and alert on a sudden change in the distribution of a categorical column (e.g. country codes)?](<../../../PRACTICE/02_Data_Engineering/Data_Quality/Data_Observability/Q-12780_How_do_you_observe_and_alert_on_a_sudden_change_in_the_distr.md>)
- [Deep] [How would you detect and quantify data skew that causes a Spark job to slow down and eventually produce late data?](<../../../PRACTICE/02_Data_Engineering/Data_Quality/Data_Observability/Q-12790_How_would_you_detect_and_quantify_data_skew_that_causes_a_Sp.md>)
- [Deep] [What is your approach to setting dynamic thresholds using rolling windows versus fixed thresholds, and what breaks with each?](<../../../PRACTICE/02_Data_Engineering/Data_Quality/Data_Observability/Q-12807_What_is_your_approach_to_setting_dynamic_thresholds_using_ro.md>)
- [Deep] [How would you distinguish a real anomaly from an artifact of a deployment/code change in your pipeline?](<../../../PRACTICE/02_Data_Engineering/Data_Quality/Data_Observability/Q-12818_How_would_you_distinguish_a_real_anomaly_from_an_artifact_of.md>)

## Related concepts

- [Scaling with Contaminated Data](<../../../11_AI_and_ML/ML_Foundations/Data_Preprocessing/Scaling_with_Contaminated_Data.md>) *(Data Preprocessing)*
- [Clock Skew and Time Windows](<../../../07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Clock_Skew_and_Time_Windows/_index.md>) *(Distributed Systems Core)*
- [Adaptive Thresholds](<../../../11_AI_and_ML/Applied_ML/Anomaly_Detection/Adaptive_Thresholds.md>) *(Anomaly Detection)*

## References

- *Data Quality Fundamentals* — Barr Moses, Lior Gavish, Molly Vorwerck — Monitoring and Anomaly Detection for Your Data Pipelines › Scaling Anomaly Detection with Python and Machine Learning
- *Data Quality Fundamentals* — Barr Moses, Lior Gavish, Molly Vorwerck — Monitoring and Anomaly Detection for Your Data Pipelines › Improving Data Monitoring Alerting with Machine Learning
- *Data Quality Fundamentals* — Barr Moses, Lior Gavish, Molly Vorwerck — Monitoring and Anomaly Detection for Your Data Pipelines › Accounting for False Positives and False Negatives
- *Data Quality Fundamentals* — Barr Moses, Lior Gavish, Molly Vorwerck — Monitoring and Anomaly Detection for Your Data Pipelines › Improving Precision and Recall
- *Automating Data Quality Monitoring* — Jeremy Stanley, Paige Schwartz — Implementing Notifications While Avoiding Alert Fatigue
- *Data Engineering Design Patterns* — Bartosz Konieczny — Chapter 10. Data Observability Design Patterns › Consequences
- *Data Engineering Design Patterns* — Bartosz Konieczny — Pattern: Skew Detector › Consequences
- *Data Engineering Design Patterns* — Bartosz Konieczny — Pattern: Skew Detector › Communication
- *Data Engineering Design Patterns* — Bartosz Konieczny — Time Detectors
- *Fundamentals of Data Engineering* — Joe Reis, Matt Housley — Designing Good Data Architecture › Serving
- *Fundamentals of Data Engineering* — Joe Reis, Matt Housley — Security and Privacy › Logging, Monitoring, and Alerting
- *Chaos Engineering: System Resiliency in Practice* — Casey Rosenthal, Nora Jones — Encountering Complex Systems › Example 3: Holiday Code Freeze
- *Chaos Engineering: System Resiliency in Practice* — Casey Rosenthal, Nora Jones — Google DiRT Disaster Recovery Testing › Treat disaster tests as you would actual outages
- *Chaos Engineering: System Resiliency in Practice* — Casey Rosenthal, Nora Jones — Google DiRT Disaster Recovery Testing › Monitoring and alerting
- *Database Reliability Engineering* — Laine Campbell, Charity Majors — Service-Level Management › Monitoring Latency

