---
type: "practice"
difficulty: "Deep"
---

# How do you handle metric-relabeling to drop noisy labels at scrape time versus at query time, and which reduces TSDB pressure more for your ML feature store?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ PromQL Pitfalls](<../../../../01_Databases_and_SQL/NoSQL_and_NewSQL/Time_Series_Databases/PromQL_Pitfalls.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address how to manage metric relabeling in Time-Series Databases (TSDBs) like Prometheus or InfluxDB, we need to consider both scrape-time and query-time relabeling strategies. The core idea is to minimize TSDB pressure while ensuring that the data retained by your ML feature store remains clean and relevant. Let’s dive into how each approach works and their trade-offs.

#### Triage
Understanding Relabeling

Relabeling in TSDBs involves modifying metadata attached to time series based on predefined rules. Scrape-time relabeling is performed during the scrape process, while query-time relabeling happens when a query is executed. We need to determine which approach reduces TSDB pressure more effectively for our ML feature store.

#### Assumptions
and Context
ML Feature Store: A place where historical time series data is stored for training machine learning models.
TSDB Pressure: Primarily driven by the amount of storage space used, query latency, and write throughput.
Noisy Labels: Labels that provide redundant or irrelevant information, leading to unnecessary data volume.

#### First-Principles Solution
Scrape-Time Relabeling

Scrape-time relabeling is configured in the scrape_configs section of your Prometheus configuration file. It operates before the time series are ingested into the TSDB and can significantly reduce storage requirements by filtering out noisy labels early.

#### Example
Configuration:

YAML
- job_name: 'example'
  static_configs:
    - targets: ['localhost:9100']
      relabel_configs:
        - source_labels: [__name__]
          target_label: new_label
          replacement: filtered_name

Here, filtered_name would be used instead of the original metric name. This approach is effective because it reduces the amount of data that needs to be stored and queried.

Query-Time Relabeling

Query-time relabeling occurs during query execution. It involves applying transformations based on label values returned by the TSDB. While this method can still reduce noise, it incurs additional overhead due to its occurrence at runtime rather than ingestion time.

#### Example
Configuration:

PROMQL
label_replace(uptime_seconds{job="example"}, "new_label", "${1}", "instance", "(.*):(.*)")

This PromQL query replaces the instance label with a new label based on its value, but it happens only when the data is queried.

#### Trade-offs
and Performance Impact

Scrape-Time Relabeling:

#### Pros
Reduces TSDB pressure early in the process. Simplifies queries as there’s less noisy data.

#### Cons
Requires careful configuration to avoid losing necessary labels, which could impact query results or model training.

Query-Time Relabeling:

#### Pros
More flexible and can handle dynamic label values that change over time. Can be adjusted without changing the TSDB schema.

#### Cons
Increased latency due to additional processing at query time, potentially impacting performance for high-concurrency systems.
Which Reduces TSDB Pressure More?

Scrape-time relabeling is generally more effective in reducing TSDB pressure because it filters data before storage. Query-time relabeling can be useful but may not always result in the same level of data reduction due to its runtime nature and potential overhead.

For an ML feature store, it’s crucial to balance between data quality and storage efficiency. Scrape-time relabeling should be prioritized as it directly impacts how much data is stored, which is a key factor for both storage costs and query performance in the long run.

#### Closing Summary

In summary, scrape-time relabeling is more effective at reducing TSDB pressure by filtering noisy labels before ingestion. However, query-time relabeling provides flexibility and can still be valuable depending on your specific use case. By carefully configuring relabel rules, you can optimize both storage efficiency and query performance for your ML feature store.

#### Common Mistakes/Follow-Ups
Overzealous Relabeling: Be cautious not to remove essential labels that could affect query accuracy.
Inconsistent Label Sets: Ensure consistent label sets across different data sources.
Query Optimization: Even with relabeling, optimize queries to avoid unnecessary processing.

**Code:**

```
Example scrape-time relabel configuration in Prometheus

scrape_configs:

job_name: 'example' static_configs:
targets: ['localhost:9100'] relabel_configs:
source_labels: [name] target_label: new_label replacement: filtered_name
CODE
Copy
```promql
# Example query-time relabeling in PromQL
label_replace(uptime_seconds{job="example"}, "new_label", "${1}", "instance", "(.*):(.*)")
```

