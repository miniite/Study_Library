---
id: "01_Databases_and_SQL/NoSQL_and_NewSQL/Time_Series_Databases/PromQL_Pitfalls"
level: 4
type: "concept"
topic: "01_Databases_and_SQL/NoSQL_and_NewSQL/Time_Series_Databases"
tools: ["Prometheus"]
---

# PromQL Pitfalls

**L4 Advanced** · [↑ Time Series Databases](<_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** PromQL is a vector-oriented query language over labelled time series; most real-world failures come from high-cardinality labels, regex matchers and subqueries that fan out over many series, and from vector-matching, staleness and `rate()` semantics that quietly change the answer.

### 1. The big picture
Prometheus stores each unique combination of metric name + label set as one **series**. A PromQL expression selects series with matchers, applies functions over time ranges, and aggregates across labels. Two ideas explain almost every pitfall: **(1)** cost ≈ number of series touched × number of samples in the range; **(2)** binary operators match series by *label set*, so a wrong matching rule yields empty or wrong results without an error.

This lesson sits under *Time series databases*; the companion lesson covers TSDB storage and scrape health.

### 2. Core concepts and vocabulary
- **Instant vector** — one sample per series at a single timestamp.
- **Range vector** — a window of samples per series (`http_requests_total[5m]`).
- **Selector / matcher** — `{job="api", status=~"5.."}`; operators `=`, `!=`, `=~`, `!~`.
- **Cardinality** — number of distinct series; grows multiplicatively with each label's values.
- **Counter** — monotonic value that resets to 0 on restart; use `rate()`/`increase()`.
- **Gauge** — value that goes up and down.
- **Histogram** — buckets (`_bucket{le=…}`, `_sum`, `_count`); **native histograms** use sparse exponential buckets in one series.
- **Staleness** — a series that stops being scraped is marked stale and drops out of instant queries after 5 minutes (or immediately via staleness markers).
- **Vector matching** — `on()`, `ignoring()`, `group_left()/group_right()`.
- **Recording rule** — precomputed query stored as a new series.
- **Subquery** — `expr[range:resolution]` evaluating an instant expression repeatedly.

### 3. How it works, step by step
1. The selector resolves matchers to a set of series using the inverted label index (an exact-match matcher on a rare label is cheap; a regex over a big label is not).
2. For each series, samples within the evaluation window (and lookback delta, 5 m by default) are loaded.
3. Functions run per series (`rate`, `increase`, `histogram_quantile`).
4. Aggregation (`sum by (job)`) reduces labels.
5. Binary operators align left and right vectors on the **entire label set** unless told otherwise with `on`/`ignoring`.
6. The result is returned (instant) or evaluated at each step (range query), so a dashboard with a 1 h range at 15 s steps evaluates the expression 240 times.

### 4. Worked example
```promql
# Error ratio per service (good)
sum by (service) (rate(http_requests_total{status=~"5.."}[5m]))
/
sum by (service) (rate(http_requests_total[5m]))
```
```promql
# Join to add a team label from an info metric (many-to-one)
sum by (service) (rate(http_requests_total[5m]))
* on (service) group_left (team)
  service_info
```
Typical failure modes with this pair:
- Without `sum by (service)` on both sides, the label sets (`instance`, `pod`, `status`) differ, so division returns *nothing*.
- Without `group_left`, many series on the left match one on the right and Prometheus errors ("many-to-many matching not allowed") — or, if you "fixed" it by dropping `on()`, you get silent wrong joins.
- `service_info` must have value 1 and exactly one series per `service`, otherwise duplicates cause errors or doubled values.

```promql
# Dead-man's switch pitfall
absent(up{job="api"})            # fires when NO api target exists, not when one instance is missing
count by (instance)(up{job="api"} == 0)   # per-instance down detection
```

### 5. Under the hood
- **Regex matchers** (`=~`) must be evaluated against every label value of that label; anchored to the full string. Prefer exact matchers or restructure labels. Alternation of literals (`a|b|c`) is optimised, `.*foo.*` is not.
- **High-cardinality labels** (user_id, request_id, URL with ids) create a series per value; each series costs memory in the head block (~a few KB) and index space. Per-request data belongs in logs/traces, not metrics. Rule: a label is for dimensions you aggregate or alert on and whose cardinality is bounded.
- **`rate()` vs `irate()`.** `rate` averages over the whole window and extrapolates at the edges; `irate` uses the last two samples — twitchy, good for graphing fast counters, bad for alerts and for feeding anomaly models. Window must be ≥ 4× scrape interval so at least two samples exist.
- **`increase()`** is `rate × window` with extrapolation, so it can return non-integers for integer counters.
- **Subqueries** re-evaluate the inner expression at every step of the subquery: `max_over_time(rate(x[5m])[1d:1m])` evaluates `rate` 1440 times per outer step. Replace with a **recording rule** and call `max_over_time(recorded[1d])`.
- **`label_replace`/`label_join`** run per series and can create duplicate label sets (error `vector cannot contain metrics with the same labelset`).
- **NaN and Inf.** `0/0` yields NaN; comparison with NaN is false; `sum` propagates NaN; `avg` over series containing NaN returns NaN, which silently breaks scoring models. Guard denominators: `… / (denominator > 0)` or `clamp_min`.
- **`absent()` limitations**: only works for a selector with equality matchers (it can infer labels); in a multi-tenant setup one `absent` across tenants hides single-tenant outages — build per-tenant rules.
- **Histogram quantiles** are linear interpolation inside a bucket; averaging quantiles across instances is wrong — aggregate buckets first: `histogram_quantile(0.99, sum by (le)(rate(x_bucket[5m])))`.

### 6. Performance and design implications
- **Drop at scrape time** with `metric_relabel_configs` (`action: labeldrop`/`drop`) — it removes the series before ingest and relieves TSDB. Query-time filtering only hides cost you already paid.
- Use **recording rules** for dashboard and alert expressions (`level:metric:operation` naming) and keep ranges modest.
- Cap dashboards: variable-driven queries with `.*` defaults, many panels, and short refresh multiply load; use `$__rate_interval` in Grafana and set max data points.
- Set server guard rails: `--query.max-samples` (default 50 million), `--query.timeout`, `--query.max-concurrency`.
- Count series: `topk(10, count by (__name__)({__name__=~".+"}))` (expensive: run sparingly) or use `/api/v1/status/tsdb`.
- For alerts: use `for:` to avoid flapping, include labels the router needs, keep annotations for text (annotations aren't part of identity, so they don't increase cardinality).

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `rate()`, `increase()`, `irate()` | Counter rate | Alerts (`rate`), spiky graphs (`irate`) |
| `histogram_quantile` | Percentiles from buckets | Latency SLOs |
| `on() / ignoring() / group_left()` | Vector matching | Joins with info metrics |
| `metric_relabel_configs` | Drop/rename labels at ingest | Cardinality control |
| `promtool check rules/query` | Validate and unit-test rules | CI |
| `/api/v1/status/tsdb` | Top series/labels by cardinality | Find cardinality offenders |
| `--query.max-samples` | Query cap | Protect server |
| Recording rules | Precompute | Dashboards, subquery replacement |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| Label for bounded dimensions (service, status class) | IDs, emails, full URLs |
| Recording rule | Query used once, ad hoc |
| `rate` for alerts | Needing instantaneous spikes (use `irate` for graphs only) |
| Drop labels at scrape | You might later need the label for debugging |
| Separate metrics | Differences matter semantically (don't hide with a label) |

### 9. Common mistakes
- Per-request label → series explosion → out-of-memory → move to logs/traces; drop at scrape.
- `rate(x[1m])` with a 1 m scrape interval → no/one sample → empty result → use ≥ 4× interval.
- Averaging percentiles → incorrect SLO → aggregate buckets first.
- Join without matching labels → empty graph → use `on`/`ignoring`, verify with `count`.
- Alert on `absent()` for a labelled subset → never fires → use `up == 0` or per-tenant absent rules.
- Using `increase` over very short windows with counter resets → noisy → longer window.
- Subquery in an alert → heavy CPU → recording rule.

### 10. 🎯 Interview kit
**30-second answer:** PromQL cost is series × samples × steps, so cardinality, regex matchers and subqueries are the main risks; correctness risks are vector matching, staleness, counter resets and NaN. I control both with scrape-time relabeling, recording rules and explicit `on`/`group_left`.

**Structure a longer answer:** data model → cost model → correctness traps → mitigation (rules, relabel, limits) → alert design.

**Likely follow-ups:**
- *Why does a request-ID label break Prometheus?* → one series per value; head memory and index blow up.
- *Subquery 100× slower?* → inner expression evaluated per subquery step; use recording rule.
- *rate vs irate?* → average over window vs last two samples.
- *How do you page on SLO burn?* → multi-window, multi-burn-rate rules (e.g. 5m & 1h at 14.4×, 30m & 6h at 6×).

**Signals interviewers look for:** cost-based reasoning, label design discipline, knowledge of staleness/NaN, using recording rules and validation tools.

### 11. Check yourself
- [ ] Explain what series count a metric with labels `{service, pod, status, user_id}` could create.
- [ ] Fix a query that returns nothing because of mismatched label sets.
- [ ] Rewrite a subquery into a recording rule.
- [ ] Choose between `rate` and `irate` for an alert.
- [ ] Compute a correct p99 across 10 instances.
<!-- explainer:end -->

## Implementations (tools)

- [Prometheus](<../../../TOOLS/Prometheus/_index.md>)

## Practice questions

- [Intermediate] [What is the query-time cost of a regex label matcher over millions of series, and how do you rewrite alert rules to avoid it?](<../../../PRACTICE/01_Databases_and_SQL/NoSQL_and_NewSQL/Time_Series_Databases/Q-43595_What_is_the_query-time_cost_of_a_regex_label_matcher_over_mi.md>)
- [Intermediate] [What are the risks of using label_replace() heavily in queries, and how can it hurt performance or correctness in a correlation rule?](<../../../PRACTICE/01_Databases_and_SQL/NoSQL_and_NewSQL/Time_Series_Databases/Q-43642_What_are_the_risks_of_using_label_replace_heavily_in_queries.md>)
- [Deep] [Why does high-cardinality labeling (like per-request IDs) break Prometheus, and how do you decide what belongs as a label versus a separate metric for an AIOps telemetry pipeline?](<../../../PRACTICE/01_Databases_and_SQL/NoSQL_and_NewSQL/Time_Series_Databases/Q-43564_Why_does_high-cardinality_labeling_like_per-request_IDs_brea.md>)
- [Deep] [A PromQL query using subqueries is 100x slower than expected; how do you reason about the evaluation cost and rewrite it for an alerting rule?](<../../../PRACTICE/01_Databases_and_SQL/NoSQL_and_NewSQL/Time_Series_Databases/Q-43578_A_PromQL_query_using_subqueries_is_100x_slower_than_expected.md>)
- [Deep] [How do you handle metric-relabeling to drop noisy labels at scrape time versus at query time, and which reduces TSDB pressure more for your ML feature store?](<../../../PRACTICE/01_Databases_and_SQL/NoSQL_and_NewSQL/Time_Series_Databases/Q-43579_How_do_you_handle_metric-relabeling_to_drop_noisy_labels_at_.md>)
- [Deep] [What are the pitfalls of using absent() and absent_over_time() for dead-man's-switch alerting in a multi-tenant scraping setup?](<../../../PRACTICE/01_Databases_and_SQL/NoSQL_and_NewSQL/Time_Series_Databases/Q-43587_What_are_the_pitfalls_of_using_absent_and_absent_over_time_f.md>)
- [Deep] [Explain how join semantics work with on/ignoring/group_left in PromQL and a real bug where a bad join skewed your correlation results.](<../../../PRACTICE/01_Databases_and_SQL/NoSQL_and_NewSQL/Time_Series_Databases/Q-43628_Explain_how_join_semantics_work_with_on_ignoring_group_left_.md>)

## Related concepts

- [Gaps and Islands](<../../SQL_Language/SQL_Fundamentals/Gaps_and_Islands.md>) *(SQL Fundamentals)*
- [Scaling with Contaminated Data](<../../../11_AI_and_ML/ML_Foundations/Data_Preprocessing/Scaling_with_Contaminated_Data.md>) *(Data Preprocessing)*
- [Filtered Indexes and Plan Cache](<../../../TOOLS/SQL_Server/T-SQL/Filtered_Indexes_and_Plan_Cache.md>) *(T-SQL)*

