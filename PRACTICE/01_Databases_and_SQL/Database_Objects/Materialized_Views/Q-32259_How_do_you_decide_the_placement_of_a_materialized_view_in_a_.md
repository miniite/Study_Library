---
type: "practice"
difficulty: "Deep"
---

# How do you decide the placement of a materialized view in a data-warehouse layering (staging, core, mart)?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1, SDE-1/2, SDE-2/3 · [↑ Materialized View Refresh and Maintenance](<../../../../01_Databases_and_SQL/Database_Objects/Materialized_Views/Materialized_View_Refresh_and_Maintenance.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

#### Overview

Materialized views are pre-computed and persisted query results that can significantly speed up reporting queries by reducing their execution time. In a layered data-warehouse design (with staging, core, and mart layers), placing materialized views correctly is crucial for optimizing performance while minimizing storage overhead and maintenance costs.

#### Triaging the Question

The key aspects of this question involve understanding the structure and purpose of each layer in a data-warehouse, identifying the types of queries that benefit from materialized views, and deciding which layer (staging, core, or mart) is best suited for placing them. The interviewer may be probing your knowledge of query performance optimization, understanding of data warehousing concepts, and ability to balance trade-offs between different layers.

#### Assumptions
Layered Data Warehouse: Staging Layer stores raw, unprocessed data; Core Layer processes this into normalized or semi-normalized form; Mart Layer contains denormalized data specifically for business intelligence (BI) queries.
Query Types: Reporting and analytical queries are more common in the mart layer compared to operational queries.
Storage Costs: Materialized views can consume significant storage, so their placement must balance performance gains against cost.
Step-by-Step Solution:
1. Understanding Layer Purposes

Staging Layer: Raw data is ingested here. This layer typically has no materialized views as it focuses on capturing the raw data.

Core Layer: Data is normalized and semi-normalized to support cross-functional reporting and analytics. Materialized views are not common in this layer because of potential storage overhead and normalization constraints.

Mart Layer: Denormalized data tailored for specific analytical queries. This layer benefits most from materialized views due to the high volume of complex, aggregative queries.

2. Identifying Suitable Queries

Summarization Queries: These often involve aggregation over large datasets (e.g., calculating monthly sales by department). Placing such materialized views in the mart layer can significantly speed up query execution.

Joining and Aggregating Data: Complex joins across multiple tables are common in BI. Materialized views that pre-compute these joins can greatly enhance performance.

3. Performance vs. Cost

Performance Gains: Mart layers with materialized views experience faster query times, especially for complex aggregations.

Cost Considerations: Each materialized view consumes storage space and requires maintenance (refreshing the view). Balancing these factors is critical.

Detailed Solution:
1. Staging Layer
No Materialized Views: Raw data ingestion prioritizes volume over query performance.
2. Core Layer
Limited Materialized Views: Only for small, specific use cases like summarizing raw data before it enters the mart layer.
3. Mart Layer
Primary Location for Materialized Views:
SQL
CREATE MATERIALIZED VIEW sales_summary AS
SELECT department_id, SUM(sales) as total_sales
FROM raw_sales_data
GROUP BY department_id;

#### Closing Summary
and Common Mistakes:

#### Common Mistakes

Placing materialized views in the staging or core layers, where their utility is limited.
Overestimating storage costs without considering performance benefits.

Best Practices:

Use cost-benefit analysis to determine when a materialized view adds significant value over the overhead it incurs.
Regularly review and refresh materialized views as data models evolve.

#### Example
Dry Run:
SQL
-- Example Query in Mart Layer
SELECT department_id, SUM(sales) as total_sales
FROM raw_sales_data
GROUP BY department_id;

-- Optimized with Materialized View
SELECT * FROM sales_summary;
Common Follow-Ups:
What are the maintenance strategies for materialized views?
How do you handle schema changes in the core layer affecting materialized views in the mart layer?
What tools or processes do you use to monitor and manage materialized view performance?

**Comparison:**

```
Raw Sales Data	Sales Summary
department_id	sales
101	500
102	700
103	600
```

**Code:**

```sql
-- Example Materialized View Creation
CREATE MATERIALIZED VIEW sales_summary AS
SELECT department_id, SUM(sales) as total_sales
FROM raw_sales_data
GROUP BY department_id;
```

**Example input / output:**

```text
-- Input Query (Mart Layer)
SELECT department_id, SUM(sales) as total_sales
FROM raw_sales_data
GROUP BY department_id;

-- Optimized Output with Materialized View
SELECT * FROM sales_summary;
```

**Data:**

```json
{
  "raw_sales_data": [
    {"department_id": 101, "sales": 500},
    {"department_id": 102, "sales": 700},
    {"department_id": 103, "sales": 600}
  ],
  "sales_summary": [
    {"department_id": 101, "total_sales": 500},
    {"department_id": 102, "total_sales": 700},
    {"department_id": 103, "total_sales": 600}
  ]
}
```

**Flow:**

1. No
2. Yes
3. Staging Layer
4. Core Layer
5. Mart Layer
6. Materialized View?
7. Maintain & Refresh Views

