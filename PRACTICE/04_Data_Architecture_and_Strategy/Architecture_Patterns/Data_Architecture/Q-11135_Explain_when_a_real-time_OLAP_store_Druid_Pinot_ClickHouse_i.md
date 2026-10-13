---
type: "practice"
difficulty: "Intermediate"
---

# Explain when a real-time OLAP store (Druid/Pinot/ClickHouse) is worth running alongside your lakehouse versus when it's premature. What operational cost does the second store add?

**Difficulty:** Intermediate · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Real-time OLAP Serving Layer](<../../../../04_Data_Architecture_and_Strategy/Architecture_Patterns/Data_Architecture/Tiered_Lake_Storage/Real-time_OLAP_Serving_Layer.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To decide whether to run a real-time OLAP store like Druid, Pinot, or ClickHouse alongside your lakehouse (e.g., Delta Lake or Apache Iceberg), we need to consider several factors such as the query requirements, data freshness needs, and operational costs. The key is understanding that these stores are optimized for high-speed analytical queries at the cost of additional complexity and resource consumption.

#### Triage

The question asks us to determine when a real-time OLAP store is worth the added complexity versus when it might be premature. We also need to consider the operational cost this second store adds, which includes setup, maintenance, and resource overhead.

#### Assumptions
Query Requirements: Frequent ad-hoc queries that require sub-second response times.
Data Freshness Needs: Real-time insights are critical for decision-making processes.
Resource Availability: Sufficient infrastructure to support additional storage and computation resources.
Step-by-Step Solution
1. Query Requirements

Real-time OLAP stores shine when the query requirements demand low-latency responses, often under a second or even milliseconds. These systems are ideal for scenarios where users need immediate insights from large datasets, such as:

Real-time analytics in financial trading platforms.
Ad-hoc queries for marketing campaigns.
Fraud detection and anomaly analysis.
2. Data Freshness Needs

If the data needs to be queried within a few minutes of ingestion, a real-time OLAP store can provide this freshness. This is crucial in environments where timely insights are vital, such as:

Real-time monitoring of network traffic for security incidents.
Instantaneous business metrics like customer churn rates.
3. Operational Cost

The operational cost includes setup and maintenance efforts, as well as the increased resource usage (CPU, memory, storage). Consider the following factors:

Setup Complexity: These systems require careful configuration to ensure performance and reliability.
Resource Usage: They consume more resources than traditional data lakes, which can affect overall system efficiency.
Maintenance Overhead: Ongoing management of these stores is required to optimize queries and ensure data consistency.
Full Solution

To determine if running a real-time OLAP store is worth the added complexity:

Evaluate Query Requirements:

Assess whether your use case demands sub-second query response times.
Determine if ad-hoc queries are frequent enough to justify real-time analytics.

Assess Data Freshness Needs:

Evaluate how critical it is for insights to be available within minutes of data ingestion.

#### Cons
ider the impact on decision-making processes and business outcomes.

Operational Cost Analysis:

Estimate the additional resources required (CPU, memory, storage).
Assess the maintenance overhead and potential impact on other services.

#### Example Scenario

#### Cons
ider an e-commerce platform where real-time insights are crucial for stock management and inventory decisions. If the platform needs to update dashboards every few minutes with current sales data, a real-time OLAP store like Druid would be highly valuable. However, if most queries can wait up to an hour or more after data ingestion, then this setup might be premature.

#### Closing Summary

Running a real-time OLAP store is beneficial when you need sub-second query response times and real-time insights are critical for decision-making. The added operational cost in terms of resource usage and maintenance should be weighed against the value gained from immediate analytics capabilities.

#### Common Mistakes/Follow-Ups
Overlooking Query Patterns: Failing to assess whether ad-hoc queries truly require low-latency responses.
Ignoring Resource Constraints: Not considering whether your infrastructure can support additional real-time stores without performance degradation.
Premature Optimization: Implementing a complex real-time solution when simpler, batch-oriented analytics would suffice.

When in doubt, it's often prudent to start with a proof-of-concept or a small-scale implementation before committing significant resources.

**Comparison:**

| Timestamp | Total Sales |
|-----------|-------------|
| 2023-10-05T14:00:00Z | 5000.75 |
| ...         | ...        |

**Code:**

```python
# Example code for querying a Druid store (simplified)
from druid.client import QueryRunner

query_runner = QueryRunner(host='druid_host', port=8082)

def get_real_time_sales_data():
    query = {
        "queryType": "timeseries",
        "intervals": ["[now-1h/PT30M, now)"],
        "granularity": "all",
        "dataSource": "sales",
        "aggregations": [
            {"name": "total_sales", "type": "sum", "column": "price"}
        ]
    }
    result = query_runner.query(query)
    return result
```

**Example input / output:**

```text
Input:
{
  "intervals": ["[now-1h/PT30M, now)"],
  "granularity": "all",
  "dataSource": "sales",
  "aggregations": [
    {"name": "total_sales", "type": "sum", "column": "price"}
  ]
}

Output:
[
  {
    "timestamp": "2023-10-05T14:00:00Z",
    "metrics": [
      {
        "name": "total_sales",
        "value": 5000.75
      }
    ]
  },
  ...
]
```

**Data:**

```json
{
  "intervals": ["[now-1h/PT30M, now)"],
  "granularity": "all",
  "dataSource": "sales",
  "aggregations": [
    {"name": "total_sales", "type": "sum", "column": "price"}
  ]
}
```

**Flow:**

1. Yes
2. No
3. Critical?
4. Not Critical?
5. Evaluate Query Requirements
6. Ad-Hoc Queries?
7. Implement Real-Time OLAPStore
8. Consider Batch Analytics
9. Assess Data Freshness Needs
10. Real-Time Insights Crucial,Implement
11. Batch Analytics Suffice

