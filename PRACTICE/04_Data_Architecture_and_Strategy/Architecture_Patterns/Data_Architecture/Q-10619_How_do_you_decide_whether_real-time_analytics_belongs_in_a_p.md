---
type: "practice"
difficulty: "Deep"
---

# How do you decide whether real-time analytics belongs in a purpose-built OLAP store (Druid/Pinot/ClickHouse) vs your lakehouse's streaming tables? What determines the split and what's the cost of running both?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Real-time OLAP Serving Layer](<../../../../04_Data_Architecture_and_Strategy/Architecture_Patterns/Data_Architecture/Tiered_Lake_Storage/Real-time_OLAP_Serving_Layer.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To decide whether real-time analytics belongs in a purpose-built OLAP store (Druid, Pinot, or ClickHouse) versus your lakehouse's streaming tables, we need to consider several factors that influence performance, cost, and data consistency. The key determinants include query patterns, required latency, data freshness requirements, and the overall architecture of your data ecosystem.

First, let’s break down what each option offers:

Druid/Pinot/ClickHouse: These systems are optimized for real-time analytics and can provide low-latency queries suitable for dashboards and reporting.
Lakehouse (e.g., Delta Lake): These platforms store raw, semi-processed data, allowing for complex transformations before analysis.

#### Triage

The question is essentially asking how to balance the trade-offs between a specialized OLAP system and a general-purpose lakehouse. The decision hinges on understanding the specific needs of the analytics workload.

Key Assumptions:
Query Complexity: Real-time queries often involve simpler, more straightforward aggregations compared to complex joins or transformations.
Data Freshness: Real-time analytics requires data that is as fresh as possible, which might not be feasible in a lakehouse with batch processing.
Storage Costs: OLAP systems are optimized for low-latency queries at the cost of storage efficiency.

#### First-Principles Solution
Query Latency:
For real-time analytics, Druid/Pinot/ClickHouse can provide sub-second response times, making them ideal for dashboards and high-frequency monitoring.
Data Transformation:
If complex transformations are needed before analysis (e.g., complex joins, aggregations), a lakehouse might be more appropriate as it allows batch processing of data.
Cost Analysis
Druid/Pinot/ClickHouse: These systems are designed for high-performance analytics and can handle large volumes of real-time data efficiently. However, they come with higher operational costs due to their specialized nature.
Lakehouse: While more flexible and capable of handling complex transformations, the lakehouse might introduce additional latency in query processing.
Data Freshness
Real-time analytics requires minimal delay between data ingestion and analysis. OLAP systems are better suited for this requirement as they ingest and serve data almost instantaneously.

#### Trade-offs
Performance vs Flexibility: Druid/Pinot/ClickHouse excel in performance but lack the flexibility of a lakehouse.
Cost vs Complexity: Lakehouses offer more cost-effective storage solutions, but their complexity can increase operational overhead due to batch processing requirements.

#### Common Mistakes
and Follow-Ups
Over-optimizing for Real-Time: Failing to consider complex transformations needed before analytics can lead to suboptimal results.
Ignoring Cost Implications: Not accounting for the additional costs of operating specialized OLAP systems versus a general-purpose lakehouse.

#### Closing Summary

In summary, real-time analytics should be placed in purpose-built OLAP stores like Druid/Pinot/ClickHouse when the workload requires minimal latency and simple, high-frequency queries. For more complex transformations and batch processing needs, a lakehouse like Delta Lake is a better fit. The key is to balance performance, cost, and data freshness requirements.

#### Common Mistakes/Follow-Ups
Ignoring Query Patterns: Not understanding whether real-time analytics truly requires low-latency response times.
Overlooking Cost Implications: Failing to consider the operational costs associated with specialized OLAP systems versus a general-purpose lakehouse.

**Comparison:**

| Column | Type   |
|--------|--------|
| timestamp | timestamp |
| metric1 | double |
| metric2 | int     |

**Code:**

```
MERMAID
Copy
graph TD;
    A[Real-Time Analytics] --> B{Query Complexity};
    B -->|Simple| C[Druid/Pinot/ClickHouse];
    B -->|Complex| D[Lakehouse];
```

**Example input / output:**

```text
{
  "query": {
    "type": "select",
    "fields": ["sum(metric1)", "avg(metric2)"],
    "filter": {
      "op": "=",
      "left": "timestamp",
      "right": "2023-10-01"
    }
  },
  "response": {
    "results": [
      { "sum(metric1)": 54678, "avg(metric2)": 99.32 },
      { "sum(metric1)": 54678, "avg(metric2)": 99.32 }
    ]
  }
}
```

**Data:**

```yaml
# Example schema for a lakehouse table (Delta Lake)
data_source: "s3://example-data-lake/source_data.csv"
table_name: "analytics_streaming_table"
storage_format: "delta"
columns:
  - {name: "timestamp", type: "timestamp"}
  - {name: "metric1", type: "double"}
  - {name: "metric2", type: "int"}
```

**Flow:**

1. Simple
2. Complex
3. Data Ingest
4. Query Complexity
5. Real-Time OLAP
6. Lakehouse Processing
7. High-Frequency Dashboards
8. Batch Processing & Analysis

