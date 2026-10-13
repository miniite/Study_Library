---
type: "practice"
difficulty: "Deep"
---

# Explain how join semantics work with on/ignoring/group_left in PromQL and a real bug where a bad join skewed your correlation results.

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ PromQL Pitfalls](<../../../../01_Databases_and_SQL/NoSQL_and_NewSQL/Time_Series_Databases/PromQL_Pitfalls.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To explain join semantics in Prometheus Query Language (PromQL) using on, ignoring, and group_left functions, we first need to understand that these operations are used to combine time series data across different queries. This is crucial for correlation analysis, such as identifying patterns or anomalies between multiple metrics.

#### Triage
Understanding the Question

The question focuses on how join semantics work in PromQL with specific functions and illustrates a real-world bug caused by incorrect usage of these functions. The goal is to understand the behavior of these operations and identify common pitfalls that can lead to skewed results in correlation analysis.

Join Semantics in PromQL
on vs. ignoring
on(): This function ensures that the resulting time series have labels in common across both queries.
ignoring(): This function allows you to ignore certain label names during the join, meaning the resulting time series may not share all labels.

#### Example
Bug Scenario

#### Cons
ider a scenario where we are analyzing network traffic data and trying to correlate two metrics: network_received_bytes (data received) and network_sent_bytes (data sent). We suspect that there might be an issue with the network interface causing both in and out traffic to be affected similarly. Our initial query looks like this:

PROMQL
sum(network_received_bytes{interface="eth0"}) by (instance)
on () sum(network_sent_bytes{interface="eth0"}) by (instance)

However, after running this query, we notice that the results are skewed and don’t make sense.

Reasoning from First Principles
Step 1: Understanding on()

The on() function ensures that both metrics have a common set of labels. In our example, if network_received_bytes and network_sent_bytes share all instance labels, then this should be fine.

Step 2: Identifying the Bug

Upon closer inspection, we realize that network_received_bytes and network_sent_bytes might not always share the same set of labels. Specifically, there could be cases where one metric is missing an important label (e.g., job) while the other isn’t.

Step 3: Using ignoring()

To handle this scenario correctly, we should use the ignoring() function to ensure that the mismatched labels don't affect our correlation analysis. The corrected query would look like:

PROMQL
sum(network_received_bytes{interface="eth0"}) by (instance) ignoring(job)
on () sum(network_sent_bytes{interface="eth0"}) by (instance) ignoring(job)

This ensures both metrics are treated consistently, regardless of label mismatches.

#### Closing Summary
and Common Mistakes

The key takeaway is that when joining time series data in PromQL, it’s crucial to use ignoring() or on() appropriately based on the shared labels. Misusing these functions can lead to incorrect correlation results, as seen in our example where the initial query failed due to label mismatches.

#### Common mistakes
include:

Not using ignoring() when dealing with inconsistent label sets.
Overlooking the importance of common labels required by on().
Assuming that all metrics share a consistent set of labels without verification.
Common Follow-Ups
How would you debug such issues if PromQL does not provide enough information?
What other functions or strategies can be used to ensure data consistency before joining time series?

**Code:**

```
PROMQL
Copy
sum(network_received_bytes{interface="eth0"}) by (instance) ignoring(job)
on () sum(network_sent_bytes{interface="eth0"}) by (instance) ignoring(job)
```

