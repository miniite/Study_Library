---
id: "01_Databases_and_SQL/Database_Operations/Monitoring/Percentiles_vs_Averages"
level: 4
type: "concept"
topic: "01_Databases_and_SQL/Database_Operations/Monitoring"
tools: []
---

# Percentiles vs Averages

**L4 Advanced** · [↑ Monitoring](<_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** Query latency is a skewed distribution with a long slow tail, so the average describes almost nobody's experience — report percentiles (p50, p95, p99) from histograms, never average the percentiles, and size and alert on the tail that users actually feel.

### 1. The big picture
Every database request takes a different amount of time. Most finish quickly; a minority are delayed by locks, cache misses, plan changes, garbage collection or a noisy neighbour. A few seconds of delay is invisible in an average over thousands of fast queries, yet those few are exactly what users complain about — and for a page that fires 20 queries, the chance that *at least one* hits the slow tail is large.

So the question "how fast is the database?" has no single-number answer. The mean answers "how much total time are we spending per request?" — useful for capacity and cost. Percentiles answer "what experience do most users get (p50), and how bad is it for the unlucky ones (p95, p99, p99.9)?" — useful for SLOs, alerting and diagnosis. Book guidance on operational visibility says the same: look at histograms of latency to understand distribution and outliers rather than only asking whether the database is up and utilised.

### 2. Core concepts and vocabulary
- **Mean (average)** — sum of values ÷ count; sensitive to outliers and hides shape.
- **Median / p50** — half the requests are faster, half slower; the "typical" case.
- **Percentile pN** — the value below which N % of observations fall. p99 = the slowest 1 % starts here.
- **Tail latency** — the high percentiles (p95 and above).
- **Histogram** — counts of observations per latency bucket; the data structure from which percentiles can be computed and merged.
- **Summary / quantile sketch** — pre-computed quantiles (client side) or compact sketches (t-digest, HDR histogram, DDSketch) that approximate percentiles with bounded error.
- **Tail amplification** — a request that needs N sub-requests is as slow as the slowest; P(at least one slow) = 1 − (1 − p)^N.
- **Coordinated omission** — load generators or monitors that wait for a slow response before sending the next measurement undercount slow cases.
- **SLI/SLO** — e.g. "99 % of read queries complete in under 100 ms over 30 days".
- **Apdex-like thresholds** — classify each request as satisfied/tolerating/frustrated.

### 3. How it works, step by step
**Computing a percentile**
1. Collect each request's duration (or increment a histogram bucket).
2. Sort the sample (or walk cumulative bucket counts).
3. pN is the value at rank ⌈N/100 × count⌉.

**Why the mean misleads — a concrete case**
- 990 queries at 10 ms and 10 queries at 3 000 ms.
- Mean = (990 × 10 + 10 × 3 000) / 1 000 = **39.9 ms**. No query actually took 40 ms.
- p50 = 10 ms, p99 = 10 ms (the 990th value), p99.9 = 3 000 ms.
- If the slow ones double to 6 000 ms, the mean moves 39.9 → 69.9 ms, which looks "okay"; p99.9 doubles to 6 s, which users notice.

**Rules for aggregating**
1. Percentiles cannot be averaged. The average of two servers' p99 is not the p99 of the combined traffic.
2. Aggregate *histograms* (add bucket counts) and then compute the percentile of the merged histogram.
3. Choose buckets that are fine near your SLO threshold (e.g. 5, 10, 25, 50, 100, 250, 500, 1000 ms) or use a log-linear scheme (HDR).
4. Pick the window to match decisions: 1 min for alerting, 1 h–1 day for trends, 30 days for SLO.

### 4. Worked example
```sql
-- PostgreSQL: percentiles from raw timings you logged to a table
SELECT date_trunc('minute', ts)                         AS minute,
       count(*)                                         AS n,
       avg(duration_ms)                                 AS mean_ms,
       percentile_cont(0.50) WITHIN GROUP (ORDER BY duration_ms) AS p50,
       percentile_cont(0.95) WITHIN GROUP (ORDER BY duration_ms) AS p95,
       percentile_cont(0.99) WITHIN GROUP (ORDER BY duration_ms) AS p99
FROM   query_log
WHERE  ts > now() - interval '1 hour'
GROUP  BY 1 ORDER BY 1;

-- SQL Server: same idea (percentile_cont is a window function there)
SELECT DISTINCT
       DATEADD(minute, DATEDIFF(minute,0,ts),0)           AS minute,
       PERCENTILE_CONT(0.99) WITHIN GROUP (ORDER BY duration_ms)
           OVER (PARTITION BY DATEDIFF(minute,0,ts))      AS p99
FROM   dbo.query_log;
```

Reading it: graph mean and p99 on the same panel. Healthy shape — p99 a few times the p50, both flat. Warning shapes — mean flat but p99 rising (a growing slow minority: lock waits, a plan for an unusual parameter value, a failing disk); p50 up with p99 (global slowdown: CPU, I/O, cache); bimodal histogram (two populations, e.g. cache hit vs miss, or two plans).

Built-in statistics also expose only means: SQL Server's Query Store reports `avg_duration` plus `stdev_duration` and `min/max_duration` per plan interval, and `pg_stat_statements` gives `mean_exec_time` and `stddev_exec_time`. A large standard deviation relative to the mean is your hint that the distribution has a tail worth investigating; use extended events, tracing, or application-side histograms for real percentiles.

### 5. Under the hood
*(optional at this level)*
- **Storage cost of exactness.** Exact percentiles need all raw values. Histograms trade a bounded relative error (HDR/DDSketch: ~1 % relative) for constant memory and mergeability — which is why metrics systems use them.
- **Statistical stability.** p99 on 100 samples is one data point; p99.9 on a low-traffic endpoint is noise. Require enough samples per window or widen the window.
- **Network vs execution time.** Client-observed latency = queue/pool wait + network + server execution + result transfer. Server-side metrics (Query Store, `pg_stat_statements`) exclude the first and last. Instrument both to attribute the tail.
- **Tail-at-scale math.** With 1 % of calls slow and a page making 100 calls, 63 % of page loads see at least one slow call (1 − 0.99^100). Fan-out systems must therefore engineer p99 of components, not p50.
- **Queueing.** As utilization approaches 100 %, tails grow much faster than means, so the first sign of saturation is usually the p99, long before the average moves.

### 6. Performance and design implications
- **Alert on percentile SLOs** over a window with burn-rate logic, not on mean thresholds.
- **Use the mean for capacity and cost** (total time, throughput); **use percentiles for experience.** Sum of durations × count ranks the *impact* of queries; p95/p99 per query ranks *user pain*.
- **Segment before you summarise**: by query class, endpoint, tenant, replica, plan. One combined percentile mixes populations with different SLOs.
- **Keep an eye on maximum and counts above a threshold** ("number of queries > 1 s") — often more actionable than p99.9.
- **Scrape interval and retention**: keep full histograms at high resolution short term, downsampled histograms for months; do not store only pre-aggregated percentiles because they cannot be re-aggregated.
- **Averages are not useless**: a slowly rising mean row-fetch time for a critical table is a valid early warning for growth or fragmentation; pair it with the tail.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `PERCENTILE_CONT / PERCENTILE_DISC` | Compute percentiles in SQL | Offline analysis of logged timings |
| Query Store `avg_duration`, `stdev_duration` | Per-plan mean and spread | Spot unstable plans |
| `pg_stat_statements` (`mean_exec_time`, `stddev_exec_time`, `calls`) | Per-query totals and spread | Find high-variance queries |
| `log_min_duration_statement`, slow query log | Capture the tail | Investigating p99 |
| `SET STATISTICS TIME ON` | Single-query CPU/elapsed | Dev comparisons (run several times; cache state matters) |
| Prometheus histograms + `histogram_quantile` | Aggregatable percentiles | Fleet monitoring |
| HDR histogram / t-digest libraries | Accurate mergeable sketches | App-side latency capture |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| **Percentiles (p95/p99)** for user-facing SLOs | Very low volume (unstable); never average them across hosts |
| **Mean** for throughput, cost, capacity totals | Judging experience or detecting tail regressions |
| **Histograms** for aggregation across nodes/time | Buckets mis-chosen around the SLO; too coarse hides shifts |
| **Raw logs** for exact analysis and forensic work | High volume and cost; sample or keep short retention |
| **Max / count > threshold** for hard timeouts | Single outliers can be noise; pair with p99 |

### 9. Common mistakes
- Reporting the mean latency in an SLO → tail regressions invisible → define SLO on a percentile.
- Averaging per-host p99s → wrong number, usually too low → merge histograms.
- Looking only at p50 → missing the minority who time out → show p95/p99 alongside.
- Using too few samples for p99.9 → noisy alerts → widen window or lower percentile.
- Measuring only server-side time → network/pool wait invisible → instrument the client path as well.
- Combining all queries into one percentile → fast reads hide slow reports → segment by class.
- Load testing with a closed loop that pauses on slow responses → tail underestimated (coordinated omission) → use open-loop arrival-rate testing.

### 10. 🎯 Interview kit
**30-second answer:** Latency distributions are skewed with a long tail, so the mean hides the slow minority: 990 queries at 10 ms and 10 at 3 s average 40 ms although nobody experiences 40 ms. I report p50/p95/p99 from mergeable histograms, set SLOs and alerts on percentiles, and use the mean only for capacity and cost.

**Structure a longer answer:** define the skew with an example → what each statistic answers → how to compute and aggregate correctly (histograms, not averaged percentiles) → pitfalls (sample size, segmentation, coordinated omission) → close with a diagnosis: "p99 doubled, CPU flat" → look for lock waits, a plan for a rare parameter, pool wait, network.

**Likely follow-ups:**
- *Why can't you average percentiles?* → percentile is a rank statistic; the combined distribution's p99 depends on counts and shapes, merge histograms instead.
- *p99 doubled but CPU/IO flat — investigate how?* → check blocking/lock waits, plan regression for specific parameters, connection-pool queueing, network, throttling, GC/stalls.
- *Why does fan-out make tails worse?* → 1 − (1 − p)^N.
- *How separate network from DB time?* → client timing vs server-reported execution time, trace spans.
- *How choose scrape interval and retention?* → by decision value: high resolution short-term, downsampled long-term; keep histograms.

**Signals interviewers look for:** can compute a concrete example; knows aggregation pitfalls; ties percentile choice to SLOs and capacity; mentions segmentation and sample size.

### 11. Check yourself
- [ ] Compute mean, p50, p99 for 95 queries at 20 ms and 5 at 2 000 ms.
- [ ] Explain why averaging p99s across servers is wrong and what to do instead.
- [ ] Show how fan-out of 50 sub-queries turns a 1 % tail into a common event.
- [ ] Name two situations where the mean is the right metric.
- [ ] Describe how you would set an SLO and alert for read-query latency.
<!-- explainer:end -->

## Practice questions

- [Intermediate] [How do you use pg_stat_statements (or equivalent) to find the queries actually hurting your database health?](<../../../PRACTICE/01_Databases_and_SQL/Database_Operations/Monitoring/Q-20869_How_do_you_use_pg_stat_statements_or_equivalent_to_find_the_.md>)
- [Intermediate] [Describe how you would build a top-N slow queries report that engineers actually use and act on.](<../../../PRACTICE/01_Databases_and_SQL/Database_Operations/Monitoring/Q-20924_Describe_how_you_would_build_a_top-N_slow_queries_report_tha.md>)
- [Intermediate] [Explain how you would monitor and act on a slowly increasing average row-fetch time for a critical table.](<../../../PRACTICE/01_Databases_and_SQL/Database_Operations/Monitoring/Q-20942_Explain_how_you_would_monitor_and_act_on_a_slowly_increasing.md>)
- [Deep] [Your p99 query latency doubled but CPU and IO look flat. How would you investigate?](<../../../PRACTICE/01_Databases_and_SQL/Database_Operations/Monitoring/Q-20885_Your_p99_query_latency_doubled_but_CPU_and_IO_look_flat_How_.md>)
- [Deep] [What is the role of percentiles versus averages when reporting query latency, and why do averages mislead?](<../../../PRACTICE/01_Databases_and_SQL/Database_Operations/Monitoring/Q-20908_What_is_the_role_of_percentiles_versus_averages_when_reporti.md>)
- [Deep] [Explain how you'd instrument end-to-end query latency to separate network time from actual DB execution time.](<../../../PRACTICE/01_Databases_and_SQL/Database_Operations/Monitoring/Q-20914_Explain_how_you_d_instrument_end-to-end_query_latency_to_sep.md>)
- [Deep] [How do you decide the right scrape interval and retention for database metrics without exploding storage?](<../../../PRACTICE/01_Databases_and_SQL/Database_Operations/Monitoring/Q-20935_How_do_you_decide_the_right_scrape_interval_and_retention_fo.md>)
- [Deep] [How would you measure the blast radius of a single slow query across dependent services?](<../../../PRACTICE/01_Databases_and_SQL/Database_Operations/Monitoring/Q-20953_How_would_you_measure_the_blast_radius_of_a_single_slow_quer.md>)

## Related concepts

- [Reading EXPLAIN](<../../SQL_Language/SQL_Fundamentals/Reading_EXPLAIN.md>) *(SQL Fundamentals)*
- [Statistics and Sargability](<../../Query_Performance/Execution_Plans/Statistics_and_Sargability.md>) *(Execution Plans)*
- [Index Tuning in Practice](<../../Query_Performance/Indexing/Index_Tuning_in_Practice/_index.md>) *(Indexing)*

## References

- *Expert T-SQL Window Functions in SQL Server* — Kathi Kellenberger, Clayton Groom, Ed Pollack — Performing Time Comparisons
- *97 Things Every Data Engineer Should Know* — Tobias Macey (ed.) — Know Your Latencies › Chapter 44. Know Your Latencies
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 9: Execution Plans and Query Store › Query Store Metrics and Analysis
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 9: Execution Plans and Query Store › Analyzing Query Store Metrics
- *Database Reliability Engineering* — Laine Campbell, Charity Majors — Service-Level Management › Latency versus Response Time
- *Database Reliability Engineering* — Laine Campbell, Charity Majors — Service-Level Management › Why Is Latency Critical?
- *Database Reliability Engineering* — Laine Campbell, Charity Majors — Datastore Field Guide › Consistency Latency Trade-offs
- *Database Reliability Engineering* — Laine Campbell, Charity Majors — A Data Architecture Sampler › Latency
- *Designing Data-Intensive Applications* — Martin Kleppmann — Reliable, Scalable, and Maintainable Applications › Latency and response time
- *SQL Server 2025 Query Performance Tuning (7th ed.)* — Grant Fritchey — FULL_BOOK_NEEDS_MANUAL_SPLIT (cont.) › Chapter 3 Methods For Capturing Query Performance Metrics

