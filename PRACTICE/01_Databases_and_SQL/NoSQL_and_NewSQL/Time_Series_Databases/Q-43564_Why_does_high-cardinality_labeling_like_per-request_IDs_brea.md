---
type: "practice"
difficulty: "Deep"
---

# Why does high-cardinality labeling (like per-request IDs) break Prometheus, and how do you decide what belongs as a label versus a separate metric for an AIOps telemetry pipeline?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ PromQL Pitfalls](<../../../../01_Databases_and_SQL/NoSQL_and_NewSQL/Time_Series_Databases/PromQL_Pitfalls.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Summary:

High-cardinality data in time-series databases like Prometheus can lead to scalability issues due to excessive memory and storage consumption, making it impractical to include them as labels. The decision between using a label versus a separate metric depends on the cardinality of the data and its relevance to the overall telemetry pipeline.

#### Triage

The question is asking about why high-cardinality labels are problematic in time-series databases like Prometheus and how to decide whether to use them as labels or as separate metrics. This involves understanding the impact of label cardinality, the design principles of time-series databases, and practical considerations for AIOps telemetry pipelines.

#### First-Principles Solution
Understanding High-Cardinality Data:

High-cardinality data refers to a large number of unique values that can appear in a label or metric. In Prometheus, each combination of label keys and values forms a series, which is stored and queried independently. For example, if you have per-request IDs as labels, the number of series could grow exponentially with the number of requests, leading to significant performance degradation.

Why High-Cardinality Labels Break Prometheus:
Memory and Storage: Each unique combination of label values requires storage space in memory or on disk. With high-cardinality data, the number of these combinations can become very large, overwhelming the database's resources.
Query Performance: Querying for metrics with a large number of labels involves scanning through many series, which is time-consuming and can lead to slow response times.
Decision Criteria:
Cardinality Thresholds:
Prometheus recommends keeping label cardinality below 10,000 unique values per metric.
Relevance to the Problem:

#### Cons
ider whether the data has a direct impact on the metrics being measured or if it can be derived from existing labels.

#### Example
s of When to Use Labels vs Separate Metrics:
Labels: Use when the data is directly relevant to the metric and does not significantly increase cardinality. For example, user_id for monitoring user-specific metrics.
Separate Metrics: Use when the data is high-cardinality or irrelevant to the core metric. For example, using a separate metric like request_duration_by_request_id instead of including request ID as a label.
Practical Considerations:
Sampling and Aggregation:
Implement sampling techniques to reduce cardinality where possible.
External Storage:
Use external storage solutions for high-cardinality data that can be queried separately but linked back to the main metrics.
Custom Label Handling:
Design custom label handling strategies, such as bucketing or grouping, to manage high-cardinality more efficiently.

#### Closing Summary
and Common Mistakes/Follow-Ups:

In summary, while Prometheus is powerful for monitoring time-series data, its performance can be severely impacted by high-cardinality labels. The key decisions revolve around the cardinality of the data and its relevance to the metrics being measured. To manage this, consider using separate metrics where appropriate and employing best practices like sampling and external storage.

#### Common mistakes
include not understanding the implications of high-cardinality or ignoring performance impacts when designing telemetry pipelines. Follow-ups might involve discussing specific use cases, alternative database choices (like InfluxDB), and advanced techniques for managing high-cardinality data in AIOps environments.

**Comparison:**

```
Metric Name	Labels	Cardinality
request_duration	user_id, app_version	Low
error_rate	service_name, error_type	Low
High-Cardinality Label Decision
Use separate metric
```

**Code:**

```python
# Example Python code to demonstrate label handling
def handle_labels(labels):
    if len(labels) > 10000:
        raise ValueError("Label cardinality too high")
    # Implement custom logic to manage labels efficiently
```

**Example input / output:**

```text
Input:
- Metric name: request_duration
- Labels: user_id (10,000 unique values), request_id (high cardinality)

Output:
- Decision: Use separate metric for request_id and keep other labels as is.
```

**Data:**

```json
{
  "metrics": [
    {
      "name": "request_duration",
      "labels": {"user_id": 12345, "request_id": "abcde-123", "app_version": "v1.0"}
    },
    {
      "name": "error_rate",
      "labels": {"service_name": "backend-api", "error_type": "timeout"}
    }
  ]
}
```

**Flow:**

1. Yes
2. No
3. Start
4. Is label high-cardinality?
5. Use separate metric
6. Keep as a label

