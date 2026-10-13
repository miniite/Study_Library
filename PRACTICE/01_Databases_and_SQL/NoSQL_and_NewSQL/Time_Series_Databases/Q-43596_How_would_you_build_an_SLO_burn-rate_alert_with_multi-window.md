---
type: "practice"
difficulty: "Deep"
---

# How would you build an SLO burn-rate alert with multi-window multi-burn-rate PromQL and validate it won't page on transient noise?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Prometheus TSDB and Scrape Health](<../../../../01_Databases_and_SQL/NoSQL_and_NewSQL/Time_Series_Databases/Prometheus_TSDB_and_Scrape_Health.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address building an SLO (Service Level Objective) burn-rate alert using Prometheus's PromQL with multi-window, multi-burn-rate configurations, we need to ensure that the alert is robust enough to distinguish between transient noise and genuine degradation in service quality. The key idea here is to leverage PromQL effectively to dynamically adjust thresholds based on historical data patterns.

In essence, the question asks us to:

Define what SLO burn-rate means in this context.

#### Cons
truct a PromQL query that can handle different burn rates over multiple time windows.
Validate the alert system so it doesn't falsely trigger (i.e., "page") due to transient noise.

Let's break down the process step by step, starting from first principles:

#### Triaging the Question

The core of this problem is constructing a robust SLO burn-rate alert that can handle varying service levels over different time windows. We need to ensure the alert triggers only when there’s a significant deviation from expected behavior and not due to temporary noise.

#### Assumptions

We are working with a system where an SLO (e.g., 99% availability) needs monitoring.
The data is collected in Prometheus, specifically using InfluxDB as the time-series database.
We aim for multi-window alerting with different burn rates to capture both short-term and long-term trends.

#### First-Principles Solution

Define the SLO and Burn-Rates:

Let's assume we have a 95% availability SLA over a 7-day window.
We will set up two windows:
A short-term (e.g., 24 hours) window with a stricter threshold, say 90%.
A long-term (e.g., 7 days) window to capture sustained issues.

PromQL Query Construction:

PROMQL
# Short-term burn-rate check over 24 hours
short_term_burn_rate = count_over_time(
    (1 - histogram_quantile(0.95, increase("availability_metric")) == true)
    [1d]
) / time_stamps("availability_metric")

# Long-term burn-rate check over 7 days
long_term_burn_rate = count_over_time(
    (1 - histogram_quantile(0.95, increase("availability_metric")) == true)
    [7d]
) / time_stamps("availability_metric")

Dynamic Thresholds: We will use a combination of short-term and long-term burn rates to define our alerting logic:

PROMQL
# Define thresholds based on historical data (e.g., 2% deviation from average)
avg_short_term_burn_rate = mean_over_time(short_term_burn_rate[7d])
avg_long_term_burn_rate = mean_over_time(long_term_burn_rate[30d])

alert {
  condition: (short_term_burn_rate > (avg_short_term_burn_rate + 2%))
             or (long_term_burn_rate > (avg_long_term_burn_rate + 2%))
}

Noise Reduction Techniques:

Use jitter to smooth out short-term fluctuations.
Apply a moving average filter to reduce the impact of noise.

#### Closing Summary

By leveraging multi-window PromQL queries and dynamic thresholds, we can create an SLO burn-rate alert that is both sensitive enough to detect genuine issues while robust against transient noise. This approach ensures that alerts are meaningful and actionable, thus minimizing false positives.

#### Common Mistakes/Follow-Ups

Failing to account for seasonal trends or external factors (e.g., holidays) can lead to incorrect baselines.
Overlooking the importance of proper data sampling intervals can result in noisy metrics.
Not validating the alert system thoroughly before deploying it may cause unexpected false positives.

**Code:**

```

short_term_burn_rate = count_over_time( (1 - histogram_quantile(0.95, increase("availability_metric")) == true) [1d] ) / time_stamps("availability_metric")

long_term_burn_rate = count_over_time( (1 - histogram_quantile(0.95, increase("availability_metric")) == true) [7d] ) / time_stamps("availability_metric")

CODE
Copy

```promql
avg_short_term_burn_rate = mean_over_time(short_term_burn_rate[7d])
avg_long_term_burn_rate = mean_over_time(long_term_burn_rate[30d])

alert {
  condition: (short_term_burn_rate > (avg_short_term_burn_rate + 2%))
             or (long_term_burn_rate > (avg_long_term_burn_rate + 2%))
}
```

