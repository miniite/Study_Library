---
type: "practice"
difficulty: "Intermediate"
---

# What is the query-time cost of a regex label matcher over millions of series, and how do you rewrite alert rules to avoid it?

**Difficulty:** Intermediate · **Levels:** SDE-1/2, SDE-2/3 · [↑ PromQL Pitfalls](<../../../../01_Databases_and_SQL/NoSQL_and_NewSQL/Time_Series_Databases/PromQL_Pitfalls.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When dealing with time-series databases like Prometheus or InfluxDB, using regex label matchers can significantly increase query times when there are millions of time series involved. The primary issue is that these matchers often perform a full-text search across all labels in the database, which becomes increasingly expensive as the number of time series grows.

#### Triage

The question aims to understand the performance implications of using regex label matchers and how to refactor alert rules to mitigate this cost. We need to assume that the system currently uses these matchers and is experiencing performance degradation due to the large volume of data. The goal is to find a more efficient way to write the alert rules.

#### First-Principles Solution
Understanding the Problem
Regex Label Matcher Usage: In time-series databases, regex label matchers are used to query multiple time series that match a certain pattern.
Performance Impact: When using regex on millions of time series, each query involves scanning all labels, leading to high CPU and memory usage.
Refactoring Alert Rules

To reduce the cost, we can adopt a few strategies:

Pre-filtering: Use more specific label matchers or additional filters to reduce the number of time series before applying regex.
Batch Processing: Process data in smaller batches instead of querying all time series at once.
Indexing: Ensure that relevant labels are indexed, which can speed up filtering operations.

#### Example
Refactoring a Query

Suppose we have alert rules like:

PROMETHEUS
alert: AlertName {
  expr: regex_replace("^[^_]*_([^_]*).*", "$1") =~ /regex_pattern/
  for: 5m
}

We can refactor this to:

PROMETHEUS
# Original query with potential high cost
alert: AlertName {
  expr: label_name =~ "pattern" and label_value =~ /regex_pattern/
  for: 5m
}

# Refactored query
alert: AlertName {
  expr: label_name =~ "specific_prefix" and regex_replace(label_value, "^.*?_([^_]*).*", "$1") =~ /regex_pattern/
  for: 5m
}
Detailed Steps
Identify Specific Patterns: Determine if there are specific prefixes or suffixes that can be used to pre-filter the time series.
Refactor Regex Expressions: Simplify the regex expressions where possible, reducing the complexity of pattern matching.
Use Label Filters: Apply additional label filters to narrow down the set of time series before applying the regex.

#### Closing Summary

By refactoring alert rules to use more specific labels and avoiding full-text regex searches on large sets of time series, we can significantly reduce query times and improve overall performance in time-series databases like Prometheus or InfluxDB. Common mistakes include not identifying effective pre-filters and over-relying on complex regex patterns.

#### Common Mistakes/Follow-Ups
Overlooking Pre-Filters: Not recognizing that certain labels could be used for more specific filtering.
Ignoring Batch Processing: Not considering the benefits of processing data in smaller, manageable chunks.
Misusing Regex Patterns: Using overly complex regex patterns without proper validation.

**Comparison:**

| Metric | Original Query | Refactored Query |
|--------|----------------|------------------|
| Complexity | High (full-text regex) | Lower (specific label + simplified regex) |
| Performance Impact | High | Low |

**Code:**

```
PROMETHEUS
Copy
# Original query with potential high cost
alert: AlertName {
  expr: regex_replace("^[^_]*_([^_]*).*", "$1") =~ /regex_pattern/
  for: 5m
}

# Refactored query
alert: AlertName {
  expr: label_name =~ "specific_prefix" and regex_replace(label_value, "^.*?_([^_]*).*", "$1") =~ /regex_pattern/
  for: 5m
}
```

**Example input / output:**

```
Example input and output:
PROMETHEUS
Copy
# Original input
expr: regex_replace("^[^_]*_([^_]*).*", "$1") =~ /regex_pattern/

# Refactored output
expr: label_name =~ "specific_prefix" and regex_replace(label_value, "^.*?_([^_]*).*", "$1") =~ /regex_pattern/
```

**Data:**

```
Sample data schema:
YAML
Copy
time_series:
  - metric: "server01_cpu_usage"
    labels: { host: "host1", instance: "server01" }
  - metric: "server02_disk_space"
    labels: { host: "host2", instance: "server02" }
```

**Flow:**

1. Yes
2. No
3. Original Alert Rule
4. Use Specific Labels?
5. Refactored with Specific Filters
6. Keep Original, Consider BatchProcessing
7. Improved Performance
8. Consider Batch Processing forBetter Performance

