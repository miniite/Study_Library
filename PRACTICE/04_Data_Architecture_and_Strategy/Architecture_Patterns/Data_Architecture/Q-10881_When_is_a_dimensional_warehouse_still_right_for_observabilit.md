---
type: "practice"
difficulty: "Deep"
---

# When is a dimensional warehouse still right for observability analytics (SLO reporting, MTTR trends) versus a wide event store versus a mesh of team telemetry products? What determined your answer at your last org?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Centralized vs Mesh Observability Data](<../../../../04_Data_Architecture_and_Strategy/Architecture_Patterns/Data_Architecture/Layered_Pipelines_Bronze_Silver_Gold/Centralized_vs_Mesh_Observability_Data.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When deciding between a dimensional warehouse, a wide event store, or a mesh of team telemetry products for observability analytics like SLO reporting and MTTR trends, it’s crucial to consider the specific requirements of your organization, such as data volume, query complexity, real-time needs, and the trade-offs in terms of cost and operational complexity. At my last organization, we opted for a dimensional warehouse because our primary requirement was generating high-quality reports that required complex aggregations over long time periods.

Firstly, let’s triage the question: The key is understanding which type of data architecture best supports SLO reporting (e.g., SLIs, SLOs) and MTTR trends. A dimensional warehouse excels in complex aggregation and historical analysis, a wide event store is ideal for real-time processing and anomaly detection, while a mesh of team telemetry products offers flexibility but might struggle with consistent query performance.

Now, let’s dive into the first-principles reasoning:

Dimensional Warehouse: This architecture is highly optimized for reporting and complex analytics over historical data. It uses star or snowflake schemas to store facts (metrics) alongside dimensions (attributes). For SLO reporting, we need detailed insights like calculating SLIs across different time periods, which a dimensional warehouse can handle efficiently.

Wide Event Store: This architecture is designed for real-time processing and anomaly detection. Events are stored in wide tables with columns representing attributes and values. While it supports high write throughput, its structure makes complex reporting more challenging without additional ETL processes to aggregate data into a form suitable for reporting.

Mesh of Team Telemetry Products: Each team has their own telemetry product, which can lead to inconsistent query performance and difficulty in integrating across teams. However, this setup is flexible and allows each team to define its schema based on specific needs.

Given these points, at my last organization, we chose a dimensional warehouse for SLO reporting because:

Complex Aggregations: Reporting dashboards required complex calculations over time periods (e.g., 30-day moving averages).
Historical Data Analysis: We needed to analyze historical data for trend analysis and root cause investigations.

#### Cons
istent Query Performance: A well-designed dimensional warehouse can provide consistent query performance, which is crucial for generating reliable reports.

However, we also implemented a wide event store as an adjunct system for real-time anomaly detection and alerting. This setup allowed us to have both worlds: the flexibility of real-time processing from the event store and the deep insights provided by the dimensional warehouse.

#### Closing Summary
In summary, the choice between these architectures depends on the specific use cases and requirements. A dimensional warehouse is best suited for complex reporting and historical analysis, a wide event store for real-time anomaly detection, and a mesh of team telemetry products for flexibility across teams. At my last organization, we tailored our architecture to leverage the strengths of each system based on their respective roles in the observability stack.

#### Common Mistakes/Follow-Ups
A common mistake is overlooking the need for consistent query performance when choosing an architecture. Another key point to explore further would be how data consistency and latency requirements influence your choice.

**Comparison:**

| Metric ID | Timestamp         | Value  | Dimension1 | Dimension2 |
|-----------|------------------|--------|------------|------------|
| 1         | 2023-06-01T12:00:00| 95.0   | prod       | us-west    |
| 2         | 2023-07-01T14:00:00| 89.5   | qa         | eu-central |

**Code:**

```sql
-- Example of a simple dimensional warehouse schema
CREATE TABLE metrics (
    metric_id INT PRIMARY KEY,
    timestamp TIMESTAMP NOT NULL,
    value DOUBLE PRECISION NOT NULL,
    dimension1 VARCHAR(255),
    dimension2 VARCHAR(255)
);

-- Query example for SLO reporting
SELECT
    AVG(value) as avg_value,
    dimension1,
    dimension2
FROM metrics
WHERE timestamp BETWEEN '2023-01-01' AND '2023-07-01'
GROUP BY dimension1, dimension2;
```

**Example input / output:**

```text
Input:
- Metrics data with dimensions for SLO reporting

Output:
- Aggregated metrics over time periods (e.g., daily, weekly)
- Calculation of SLIs and SLOs
- Trends analysis for MTTR
```

**Data:**

```json
{
  "metrics": [
    {
      "metric_id": 1,
      "timestamp": "2023-06-01T12:00:00",
      "value": 95.0,
      "dimension1": "prod",
      "dimension2": "us-west"
    },
    {
      "metric_id": 2,
      "timestamp": "2023-07-01T14:00:00",
      "value": 89.5,
      "dimension1": "qa",
      "dimension2": "eu-central"
    }
  ]
}
```

**Flow:**

1. Yes
2. No
3. Data Source
4. Is the data complex andhistorical?
5. Dimensional Warehouse
6. Wide Event Store
7. Complex Aggregations &Historical Analysis
8. Real-Time Anomaly Detection& Alerting

