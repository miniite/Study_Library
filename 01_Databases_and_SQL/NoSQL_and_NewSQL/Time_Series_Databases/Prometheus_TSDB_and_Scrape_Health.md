---
id: "01_Databases_and_SQL/NoSQL_and_NewSQL/Time_Series_Databases/Prometheus_TSDB_and_Scrape_Health"
level: 4
type: "concept"
topic: "01_Databases_and_SQL/NoSQL_and_NewSQL/Time_Series_Databases"
tools: ["Prometheus"]
---

# Prometheus TSDB and Scrape Health

**L4 Advanced** · [↑ Time Series Databases](<_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** Prometheus pulls metrics from targets on a schedule into a local TSDB (in-memory head + write-ahead log + immutable two-hour blocks), and its own self-metrics (`up`, `scrape_*`, `prometheus_tsdb_*`) tell you whether collection, storage and alerting are healthy.

### 1. The big picture
Prometheus is a pull-based monitoring system with a single-node local time-series database. It is deliberately simple: one server, local disk, no clustering. Scale and durability come from sharding by function (one Prometheus per team/region), from `remote_write` to long-term stores (Thanos, Mimir, Cortex, VictoriaMetrics, managed services), and from federation for rolled-up views. Operating it means watching three loops: **scrape** (are targets reachable and cheap?), **storage** (is the head, WAL and compaction keeping up?), and **rules/alerts** (are evaluations on time and delivered?).

### 2. Core concepts and vocabulary
- **Target / job / instance** — an endpoint scraped, a group of targets, and one endpoint's address.
- **Service discovery + relabeling** — finds targets (Kubernetes, EC2, Consul, files) and rewrites labels before scrape (`relabel_configs`) or after (`metric_relabel_configs`).
- **Scrape interval / timeout** — how often and how long to wait (global default 1 m; timeout 10 s).
- **`up`** — synthetic series: 1 if the last scrape succeeded, 0 otherwise.
- **Head block** — in-memory (and WAL-backed) recent data, about the last 2–3 hours.
- **WAL** — write-ahead log used to rebuild the head after a crash.
- **Block** — immutable on-disk directory covering a time range, with chunks, an index and tombstones.
- **Compaction** — merges blocks into larger ones (2 h → 6 h → 18 h …).
- **Retention** — `--storage.tsdb.retention.time` (default 15 d) and `.size`.
- **Remote write/read** — streaming samples to/from a remote store.
- **Federation** — `/federate` endpoint to pull selected series from another Prometheus.
- **Alertmanager** — receives alerts, groups, silences, inhibits and routes them.

### 3. How it works, step by step
1. The scrape manager fetches `/metrics` from each target every interval (jittered to spread load).
2. Samples are parsed, relabeled with `metric_relabel_configs`, and appended to the **head**: the series is found/created by label hash, the sample goes into an open chunk (120 samples per XOR-compressed chunk), and the append is logged to the WAL.
3. Every 2 hours the head's oldest data is cut into a persistent **block**; WAL segments covered by it are truncated.
4. Background **compaction** merges blocks; **retention** deletes blocks older than the limit.
5. Rule groups evaluate every interval (default 1 m): recording rules write new series; alerting rules move through `pending` → `firing` (after `for:`) and are sent to Alertmanager.
6. Queries read head + blocks through the index; the query engine evaluates PromQL.
7. If `remote_write` is configured, a queue reads from the WAL and ships batches; slow endpoints build lag and can drop data when the queue is full.

### 4. Worked example
Health checks you should have as alert rules (names are standard Prometheus self-metrics):
```promql
# 1. Targets down
up == 0

# 2. Scrape slow (approaching timeout)
scrape_duration_seconds > 0.8 * 10

# 3. Targets hitting sample limits or exploding
scrape_samples_post_metric_relabeling > 50000

# 4. Series growth in head
rate(prometheus_tsdb_head_series_created_total[1h]) > 1000

# 5. Compactions failing / WAL corruption
increase(prometheus_tsdb_compactions_failed_total[1h]) > 0
increase(prometheus_tsdb_wal_corruptions_total[1h]) > 0

# 6. Rule evaluations missing their deadlines
increase(prometheus_rule_group_iterations_missed_total[10m]) > 0

# 7. Notifications failing
increase(prometheus_notifications_dropped_total[10m]) > 0

# 8. remote_write falling behind
prometheus_remote_storage_highest_timestamp_in_seconds
  - ignoring(remote_name,url) group_right
    prometheus_remote_storage_queue_highest_sent_timestamp_seconds > 120
```
Multi-window, multi-burn-rate SLO alert (99.9% availability, 30 days):
```promql
(
  sum(rate(http_requests_total{code=~"5.."}[1h])) / sum(rate(http_requests_total[1h])) > (14.4 * 0.001)
and
  sum(rate(http_requests_total{code=~"5.."}[5m]))  / sum(rate(http_requests_total[5m]))  > (14.4 * 0.001)
)
```
The long window proves the problem is sustained; the short window makes the alert reset quickly and avoids paging on a transient blip. Pair it with a slower rule (6× over 6 h and 30 m) that opens a ticket instead of paging.

### 5. Under the hood
- **Memory model.** Memory is dominated by the head: each active series carries labels, chunk and index structures (roughly 3–8 KB/series including overhead). Doubling cardinality roughly doubles RSS; ingestion rate matters less than series count and churn. Churn (pods restarting with new labels) creates many short-lived series that stay in the head index until the next block cut.
- **Compression.** Gorilla-style delta-of-delta timestamps and XOR values give ~1–2 bytes per sample on typical data.
- **Staleness.** When a target disappears or a series is no longer exposed, Prometheus writes a staleness marker so instant queries drop it immediately (otherwise a 5 m lookback would show it). Federation and `honor_timestamps` can bypass this behaviour.
- **NaN values** are valid samples; special staleness NaN is a distinct bit pattern. Aggregations over regular NaN return NaN.
- **Native histograms** (enabled by a feature flag in recent Prometheus releases; check your version's status) store a sparse, exponentially-bucketed histogram in one sample: far fewer series than classic `_bucket` series, with schema-based resolution and queries via `histogram_quantile` directly. Downstream: classic dashboards using `le` labels change, and remote stores/exporters must support them; mixed environments run both for a transition period.
- **Federation vs remote_write.** Federation pulls an aggregated subset on a schedule — simple, loses detail, adds scrape load on the leaf. `remote_write` ships *all* (or relabel-filtered) samples via WAL — durable-ish, near real time, enables long-term retention, global query and multi-tenant isolation, but costs bandwidth and the remote store's price.
- **HA.** Run two identical Prometheus replicas scraping the same targets; Alertmanager de-duplicates; long-term stores deduplicate by an external replica label.

### 6. Performance and design implications
- **Budget cardinality** per team/job: `sample_limit`, `label_limit` and `target_limit` in scrape config stop one bad exporter.
- **Pick scrape intervals deliberately**: 15–30 s for service metrics, 60 s for infrastructure; shorter intervals multiply storage and make `rate()` windows shrink.
- **Dashboard cost**: expensive panels refreshed by dozens of users hit the same TSDB; use recording rules, `$__rate_interval`, query-frontend caching (Thanos/Mimir) and sensible time ranges.
- **Disk**: estimate `retention_seconds × samples/s × ~1.5 bytes`; leave 20–30% headroom for compaction and WAL.
- **Alert labels vs annotations**: labels define alert identity and routing; unbounded values in labels create a new alert per value. Put free text in annotations; keep labels like `service`, `severity`, `team`, `runbook`.
- **Hybrid stacks** (Prometheus + InfluxDB): query each in Grafana, or normalise to one store through remote_write/remote_read; do correlation on the common labels (service, instance) and a common time base.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `up`, `scrape_duration_seconds`, `scrape_samples_scraped` | Target health & cost | Scrape-health alerts |
| `sample_limit`, `label_limit` | Reject bad scrapes | Guard cardinality |
| `--storage.tsdb.retention.time/size` | Retention | Disk planning |
| `promtool tsdb analyze <dir>` | Cardinality and churn report | Capacity & clean-up |
| `/api/v1/status/tsdb`, `/targets` | Live health/cardinality | Troubleshooting |
| `remote_write` + `queue_config` | Ship samples | Long-term store |
| `/-/reload`, `/-/ready`, `/-/healthy` | Operations endpoints | Deploys |
| `promtool check config/rules` | Validate | CI/CD |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| Single Prometheus per domain | Global query or HA/long retention requirements |
| `remote_write` to long-term store | Bandwidth/cost or ordering concerns on small teams |
| Federation | You only need a few rolled-up series centrally |
| Native histograms | Consumers and stores can't read them yet |
| Short scrape interval | Storage and CPU limits; most alerts don't need seconds |

### 9. Common mistakes
- No retention/disk sizing → disk full → TSDB corruption → set size limits and alert on free space.
- Pod/instance ID in labels with high churn → memory spikes → relabel away, use stable labels.
- Alerting only on app metrics → silent scrape failure → `up == 0` and absent-dead-man-switch rules.
- `remote_write` queue ignored → data loss on slow endpoint → monitor lag and drop counters.
- Huge `rate` windows on dashboards → slow queries → recording rules.
- Putting unique IDs in alert labels → notification storm → move to annotations.

### 10. 🎯 Interview kit
**30-second answer:** Prometheus scrapes targets on an interval and writes samples into an in-memory head backed by a WAL; every two hours it persists blocks and compacts them. Health is read from `up`, `scrape_*` and `prometheus_tsdb_*`; scale comes from cardinality control, recording rules, sharding and `remote_write`.

**Structure a longer answer:** scrape → head/WAL → blocks/compaction → query and rules → what to monitor → scaling and HA.

**Likely follow-ups:**
- *Federation vs remote_write?* → pull a subset vs stream everything; consistency and cost trade-offs.
- *How to build SLO alerts?* → burn-rate across multiple windows to catch fast and slow burns.
- *Why does `irate` distort a model's inputs?* → it uses the last two samples; use `rate` for stable signals.
- *Native histograms change?* → one series per histogram, sparse buckets, different query and exporter support.

**Signals interviewers look for:** you reason in series count and churn; you monitor the monitor; you know failure signals (WAL corruption, missed rule iterations, remote-write lag).

### 11. Check yourself
- [ ] Describe what happens to a sample from scrape to disk block.
- [ ] Write rules that detect a target down vs a whole job missing.
- [ ] Estimate memory for 2 M active series.
- [ ] Explain staleness markers and the 5-minute lookback.
- [ ] Design a two-window burn-rate alert for a 99.95% SLO.
<!-- explainer:end -->

## Implementations (tools)

- [Prometheus](<../../../TOOLS/Prometheus/_index.md>)

## Practice questions

- [Intermediate] [What's the difference between rate() and irate() in PromQL and when would picking the wrong one distort an anomaly detector's input signal?](<../../../PRACTICE/01_Databases_and_SQL/NoSQL_and_NewSQL/Time_Series_Databases/Q-43566_What_s_the_difference_between_rate_and_irate_in_PromQL_and_w.md>)
- [Intermediate] [How would you measure and reduce query cost of a heavy Grafana dashboard that a whole ops team keeps open, straining the TSDB?](<../../../PRACTICE/01_Databases_and_SQL/NoSQL_and_NewSQL/Time_Series_Databases/Q-43609_How_would_you_measure_and_reduce_query_cost_of_a_heavy_Grafa.md>)
- [Intermediate] [How do you keep alert-rule labels and annotations rich enough for the automation engine to act, without exploding notification cardinality?](<../../../PRACTICE/01_Databases_and_SQL/NoSQL_and_NewSQL/Time_Series_Databases/Q-43637_How_do_you_keep_alert-rule_labels_and_annotations_rich_enoug.md>)
- [Deep] [Walk me through choosing between remote_write to a long-term store versus federation for feeding a central AIOps analytics cluster.](<../../../PRACTICE/01_Databases_and_SQL/NoSQL_and_NewSQL/Time_Series_Databases/Q-43573_Walk_me_through_choosing_between_remote_write_to_a_long-term.md>)
- [Deep] [How would you build an SLO burn-rate alert with multi-window multi-burn-rate PromQL and validate it won't page on transient noise?](<../../../PRACTICE/01_Databases_and_SQL/NoSQL_and_NewSQL/Time_Series_Databases/Q-43596_How_would_you_build_an_SLO_burn-rate_alert_with_multi-window.md>)
- [Deep] [What's your strategy for querying across Prometheus and InfluxDB in a hybrid observability stack feeding one correlation model?](<../../../PRACTICE/01_Databases_and_SQL/NoSQL_and_NewSQL/Time_Series_Databases/Q-43608_What_s_your_strategy_for_querying_across_Prometheus_and_Infl.md>)
- [Deep] [How does Prometheus handle NaN and Inf values in float samples, and how could they silently break aggregations in your anomaly scoring?](<../../../PRACTICE/01_Databases_and_SQL/NoSQL_and_NewSQL/Time_Series_Databases/Q-43625_How_does_Prometheus_handle_NaN_and_Inf_values_in_float_sampl.md>)
- [Expert] [Your team wants native histograms in Prometheus for lower-cardinality latency tracking; what changes for storage, queries, and your downstream anomaly models?](<../../../PRACTICE/01_Databases_and_SQL/NoSQL_and_NewSQL/Time_Series_Databases/Q-43575_Your_team_wants_native_histograms_in_Prometheus_for_lower-ca.md>)

## Related concepts

- [Scaling with Contaminated Data](<../../../11_AI_and_ML/ML_Foundations/Data_Preprocessing/Scaling_with_Contaminated_Data.md>) *(Data Preprocessing)*
- [Binary Search](<../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Binary_Search.md>) *(Data Structures and Algorithms)*
- [Provisional and Final Results](<../../../03_Big_Data_and_Streaming/Streaming_and_Messaging/Streaming_Architecture/Provisional_and_Final_Results/_index.md>) *(Streaming Architecture)*
- [Reproducible Telemetry State](<../../../04_Data_Architecture_and_Strategy/Architecture_Patterns/Data_Architecture/Layered_Pipelines_Bronze_Silver_Gold/Reproducible_Telemetry_State.md>) *(Data Architecture)*
- [OpenTelemetry Instrumentation](<../../../TOOLS/Python/Python_Language/OpenTelemetry_Instrumentation.md>) *(Python Language)*

## References

- *Ultimate Data Engineering Design Patterns* — Bragadeesh Sundararajan — Operationalizing Data Pipelines › 14. Operationalizing Data Pipelines
- *Data Quality Fundamentals* — Barr Moses, Lior Gavish, Molly Vorwerck — L › M
- *Data Pipelines with Apache Airflow* — Bas P. Harenslak, Julian de Ruiter — Operating Airflow in production
- *Data Engineering Fundamentals* — Zhaolong Liu — CHAPTER 2Data Ingestion and Acquisition › CHAPTER 2Data Ingestion and Acquisition(#contents.xhtmlch02a)
- *AWS Certified Data Engineer Associate Study Guide* — Sakti Mishra et al. — Data Security and Governance › Amazon Managed Service for Prometheus and Grafana
- *AWS Certified Data Engineer Associate Study Guide* — Sakti Mishra et al. — E › G
- *Database Reliability Engineering* — Laine Campbell, Charity Majors — G
- *Database Reliability Engineering* — Laine Campbell, Charity Majors — J › L
- *Database Reliability Engineering* — Laine Campbell, Charity Majors — M
- *Database Reliability Engineering* — Laine Campbell, Charity Majors — S

