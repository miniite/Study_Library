---
type: "practice"
difficulty: "Deep"
---

# How does Prometheus handle NaN and Inf values in float samples, and how could they silently break aggregations in your anomaly scoring?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Prometheus TSDB and Scrape Health](<../../../../01_Databases_and_SQL/NoSQL_and_NewSQL/Time_Series_Databases/Prometheus_TSDB_and_Scrape_Health.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR:

Prometheus handles NaN (Not a Number) and Inf (positive or negative infinity) values by treating them as invalid data points, which can lead to silent failures in aggregations used for anomaly detection. This behavior can silently break the accuracy of your anomaly scoring because Prometheus will simply discard these samples instead of raising an error.

#### Triaging the Question

The question probes into understanding how NaN and Inf values are handled by Prometheus and their implications on aggregation functions, specifically within the context of anomaly scoring. It also asks about potential silent failures that can arise from these handling mechanisms. Assumptions include familiarity with time-series databases and knowledge of PromQL (Prometheus Query Language) for querying data.

#### First-Principles Solution

Handling NaN and Inf:

Prometheus automatically discards any samples that contain invalid values such as NaN or Inf. This is done to ensure the integrity of the time-series data used in metrics.
When a sample contains an invalid value, Prometheus does not include it in any query results. If multiple series are being aggregated and one of them has a NaN/Inf, the result might still be valid depending on the aggregation function (e.g., sum, average) but will likely lead to unexpected behavior.

Impact on Aggregation:

In Prometheus, if any sample within an aggregation set is invalid, it can affect the results of functions like sum(), avg(), or max() in ways that might not be immediately apparent:
For avg(), if one value is Inf and all other values are finite numbers, the average could still evaluate to a valid number.
However, for min() or max(), an Inf will always dominate the result, making it difficult to detect issues.
These silent failures can significantly impact anomaly detection because Prometheus relies heavily on numerical computations and statistical functions. If these functions return unexpected results due to invalid data points, they could mask real anomalies.

#### Example Scenario
Consider a time series ts with the following values over 5 minutes:

PLAINTEXT
ts = [10.0, 20.0, NaN, Inf, -10.0]

When aggregating these using PromQL's avg() function:

The average might be calculated as (10.0 + 20.0 + Inf + -10.0) / 4, leading to a result of Inf because division by zero is not defined but results in infinity.

#### Closing Summary
and Common Mistakes/Follow-Ups:
Common Mistake: Developers often assume that Prometheus will handle invalid data gracefully, leading to silent failures without realizing the potential impact on aggregation functions.

#### Follow-Up
Questions:
How can you detect when NaN or Inf values are introduced into your time series?
What best practices should be implemented to avoid these issues in your monitoring pipeline?

Correctness and completeness confirmed.

**Comparison:**

```
Time Series	Value
t1	10.0
t2	20.0
t3	NaN
t4	Inf
t5	-10.0
```

**Code:**

```
PROMQL
Copy
# Example PromQL queries with NaN and Inf handling
avg(ts)  # Can return Inf if any value is Inf

sum_over_time(ts[5m])  # Sum might still work, but individual values could be invalid
```

**Example input / output:**

```text
- **Input:** Time series data containing `NaN` and `Inf`.
- **Output for avg(ts):** `Inf`
- **Explanation:** The presence of `Inf` causes the average to return `Inf`, even though valid values exist.
```

**Data:**

```json
{
  "timeSeries": [
    {"value": 10.0},
    {"value": 20.0},
    {"value": NaN},
    {"value": Infinity},
    {"value": -10.0}
  ]
}
```

**Flow:**

1. Yes
2. avg()
3. sum(), max(), min()
4. Receive Time Series Data
5. Contains Invalid Values?
6. Discard Invalid Samples
7. Aggregation Function Used?
8. Return Inf
9. Result Affected by Inf

