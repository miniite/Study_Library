---
type: "practice"
difficulty: "Intermediate"
---

# How do you decide between a materialized view and a physical aggregate fact table for speeding up a recurring executive dashboard?

**Difficulty:** Intermediate · **Levels:** Architect/Principal, SDE-1, SDE-1/2, SDE-2/3 · [↑ Aggregate Fact Tables](<../../../../02_Data_Engineering/Modeling_and_Warehousing/Dimensional_Modeling/Aggregate_Fact_Tables.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When deciding between using a materialized view versus a physical aggregate fact table to speed up a recurring executive dashboard, we need to consider several factors such as query performance requirements, data update frequency, maintenance overhead, and cost trade-offs. Let's break down these considerations step by step.

First, we should clarify that both methods aim to precompute results from the source data to reduce runtime in queries executed by the dashboard. A materialized view is a database object that stores the result of a query as a table, while a physical aggregate fact table directly contains aggregated data. The choice between these depends on specific use cases and constraints.

**Comparison:**

| sales_data                | sales_aggregates            | sales_summary (materialized view) |
|--------------------------|----------------------------|----------------------------------|
| product_category          | total_sales                 | product_category, total_sales    |
| sales_amount              |                            |                                  |

**Code:**

```sql
-- Example SQL for creating a materialized view
CREATE MATERIALIZED VIEW sales_summary AS
SELECT product_category, SUM(sales_amount) as total_sales
FROM sales_data
GROUP BY product_category;

-- Example creation of an aggregate fact table
CREATE TABLE sales_aggregates (
    product_category VARCHAR(50),
    total_sales DECIMAL,
    PRIMARY KEY (product_category)
);
```

**Example input / output:**

```text
No input-output examples required here since we're focusing on the conceptual decision process.
```

**Data:**

```text
No specific data blocks are needed for this explanation, but assume `sales_data` contains fields such as `product_category`, `sales_amount`, and potentially other dimensions relevant to the dashboard.
```

