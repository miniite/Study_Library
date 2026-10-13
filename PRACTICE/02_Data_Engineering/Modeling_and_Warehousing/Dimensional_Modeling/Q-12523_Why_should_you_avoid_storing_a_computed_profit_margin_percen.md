---
type: "practice"
difficulty: "Intermediate"
---

# Why should you avoid storing a computed 'profit margin percent' directly in the fact table?

**Difficulty:** Intermediate · **Levels:** Architect/Principal, Fresher, SDE-1/2, SDE-2/3 · [↑ Aggregate Fact Tables](<../../../../02_Data_Engineering/Modeling_and_Warehousing/Dimensional_Modeling/Aggregate_Fact_Tables.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In data warehousing, storing a computed value like profit margin percent directly in the fact table can lead to several issues that compromise data integrity and efficiency. Let's delve into why this is generally avoided by considering the principles of data modeling.

TL;DR:

Storing computed values such as profit margin percent directly in the fact table violates the principles of normalization, increases storage overhead, and complicates data maintenance and query performance.

#### Triage

The question is probing your understanding of fundamental data modeling concepts, specifically why computed values are not typically stored in fact tables. The key assumptions here include the nature of a fact table (typically used for transactional or metric-focused data) and the implications of storing derived values within it.

#### First-Principles Solution
Normalization: Fact tables should contain only atomic, granular facts that are directly measurable from source systems. Storing computed values can lead to redundancy and violate the principle of normalization.
Data Integrity: Computed values like profit margin percent might change as underlying data changes (e.g., if costs or sales figures fluctuate). Direct storage would require frequent updates, risking inconsistencies in the database.
Query Performance: Storing computed values can negatively impact query performance because these values need to be recalculated whenever a new fact is inserted or updated.
Detailed Solution:

Normalization Principle:

Fact tables are designed to store raw data that is derived directly from transactional systems (e.g., sales, inventory). By storing profit margin percent, we introduce a dependency on other fields (cost and revenue) which are stored in dimensions.
This violates the third normal form (3NF), where each non-key attribute must be fully functionally dependent on the primary key.

Data Integrity:

If the cost or revenue changes, the profit margin percent value would need to be recalculated and updated in the fact table. This introduces maintenance overhead.
In contrast, storing raw data in dimensions allows for easy recalculation using a simple SQL query whenever needed.

Query Performance:

Storing computed values directly in the fact table can lead to slower queries because these values would need to be recalculated every time they are accessed.
Using star or snowflake schemas, we can leverage pre-calculated summary tables (e.g., a daily profit margin summary) that are updated less frequently.

#### Example

Assume we have a fact table SalesFact with columns [date, product_id, quantity_sold, revenue]. If we store profit_margin_percent, it would mean every time there is a change in the revenue or cost (stored in dimensions), we need to update this value. Instead, we can create a summary table that calculates the profit margin for each date and product combination:

SQL
CREATE TABLE DailyProfitMarginSummary (
    date DATE,
    product_id INT,
    total_revenue DECIMAL(10, 2),
    total_cost DECIMAL(10, 2),
    profit_margin_percent DECIMAL(5, 2)
);

This summary table can be updated less frequently (e.g., daily) and used for reporting purposes.

#### Closing Summary

In conclusion, storing computed values like profit margin percent directly in the fact table is generally avoided because it violates normalization principles, compromises data integrity, and negatively impacts query performance. Instead, these values should be calculated on-the-fly from raw data stored in dimensions or through summary tables.

#### Common Mistakes/Follow-Ups
Mistake 1: Assuming that computed values are more efficient to store than calculate.
Mistake 2: Ignoring the need for frequent updates and recalculations when using direct storage.

#### Follow-Up
Discuss how this decision can impact reporting, ETL processes, and the overall architecture of your data warehouse.

**Comparison:**

```
SUM(revenue)
COALESCE(SUM(cost), 0)
profit_margin_percent =(total_revenue - total_cost) /total_revenue * 100
SalesFact
ProfitMarginSummary
CostFact
ReportingQueries
```

**Code:**

```sql
-- Example of creating a summary table
CREATE TABLE DailyProfitMarginSummary (
    date DATE,
    product_id INT,
    total_revenue DECIMAL(10, 2),
    total_cost DECIMAL(10, 2),
    profit_margin_percent DECIMAL(5, 2)
);

INSERT INTO DailyProfitMarginSummary (date, product_id, total_revenue, total_cost, profit_margin_percent)
SELECT
    s.date,
    s.product_id,
    SUM(s.revenue) AS total_revenue,
    COALESCE(SUM(d.cost), 0) AS total_cost,
    ((SUM(s.revenue) - COALESCE(SUM(d.cost), 0)) / SUM(s.revenue)) * 100 AS profit_margin_percent
FROM SalesFact s
LEFT JOIN CostFact d ON s.date = d.date AND s.product_id = d.product_id
GROUP BY s.date, s.product_id;
```

**Example input / output:**

```text
Input:
SalesFact = [
    {"date": "2023-10-01", "product_id": 1, "quantity_sold": 100, "revenue": 5000.00},
    {"date": "2023-10-01", "product_id": 1, "quantity_sold": 50, "revenue": 2500.00}
]
CostFact = [
    {"date": "2023-10-01", "product_id": 1, "cost": 3000.00}
]

Output:
DailyProfitMarginSummary = [
    {"date": "2023-10-01", "product_id": 1, "total_revenue": 7500.00, "total_cost": 3000.00, "profit_margin_percent": 60.00}
]
```

**Data:**

```json
{
    "SalesFact": {
        "date": "2023-10-01",
        "product_id": 1,
        "quantity_sold": 100,
        "revenue": 5000.00
    },
    "CostFact": {
        "date": "2023-10-01",
        "product_id": 1,
        "cost": 3000.00
    }
}
```

**Flow:**

1. SalesFact Input
2. Aggregate Revenue
3. CostFact Input
4. Join Tables
5. Calculate Profit Margin
6. Insert into SummaryTable
7. Query Summary Table forReporting

