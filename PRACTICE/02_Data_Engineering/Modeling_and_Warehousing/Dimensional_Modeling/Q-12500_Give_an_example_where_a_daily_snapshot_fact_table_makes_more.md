---
type: "practice"
difficulty: "Intermediate"
---

# Give an example where a daily snapshot fact table makes more sense than a transaction one.

**Difficulty:** Intermediate · **Levels:** Architect/Principal, Fresher, SDE-1/2, SDE-2/3 · [↑ Transaction vs Snapshot Facts](<../../../../02_Data_Engineering/Modeling_and_Warehousing/Dimensional_Modeling/Transaction_vs_Snapshot_Facts.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In data warehousing, a daily snapshot fact table is particularly useful when dealing with time-series data that needs to be aggregated at regular intervals but also maintains historical context. For instance, in an e-commerce setting, tracking daily sales figures can provide valuable insights without capturing every transaction detail.

#### Triage
Understanding the Question

The question asks for a scenario where using a daily snapshot fact table is more appropriate than a transactional fact table. A daily snapshot fact table summarizes data at specific points in time (e.g., end of each day), whereas a transactional fact table stores every single transaction. The key here is to identify cases where summarizing the entire day's activity provides sufficient granularity for reporting and analysis.

#### Assumptions
We are dealing with an e-commerce platform.
The system needs to capture daily sales metrics but does not require detailed transaction logs.
Performance and storage costs must be considered in the design decision.

#### First Principles Solution
Scenario: Daily Sales Summary

Imagine a company that operates 24/7, selling products across various categories. They want to track their daily sales performance for reporting purposes, such as monthly sales reports, trend analysis, and inventory management. In this case, using a daily snapshot fact table makes more sense than storing every transaction.

Schema Design:

Facts Table (Daily Sales Summary):
SQL
CREATE TABLE daily_sales_summary (
    date DATE,
    product_category VARCHAR(50),
    total_units_sold INT,
    total_revenue DECIMAL(18, 2)
);
Dimensions:
date: The day when the summary was generated.
product_category: Different categories of products (e.g., electronics, clothing).

Fact Table Population: Each night, a process aggregates all transactions from that day and populates the daily sales summary table with the following metrics:

Total units sold for each product category.
Total revenue generated.

Benefits:

Storage Efficiency: Only one record per day is stored, significantly reducing storage requirements.
Performance: Faster querying as fewer records need to be scanned.
Simplicity: Easier to manage and maintain compared to a transactional fact table with millions of rows.

#### Example
Workflow
Every night at 12 AM UTC, the system runs a daily summary job:
SQL
INSERT INTO daily_sales_summary (date, product_category, total_units_sold, total_revenue)
SELECT
    CURRENT_DATE(),
    p.category,
    SUM(t.units),
    SUM(t.revenue)
FROM transactions t
JOIN products p ON t.product_id = p.id
GROUP BY p.category;
This process efficiently aggregates all transactions for the day and stores a single record in the fact table.

#### Closing Summary
+ Common Mistakes/Follow-Ups

In scenarios where detailed transactional data is not needed, daily snapshot fact tables provide an optimal balance between storage efficiency and query performance. However, it's crucial to ensure that this approach does not compromise on critical reporting needs. For example, if real-time analysis of individual transactions is required, a more granular transactional fact table might be necessary.

#### Common Mistakes
Overlooking the necessity for detailed transaction logs in scenarios requiring real-time or granular analysis.
Ignoring storage and query performance when designing the data model.

#### Follow-Ups
Real-Time vs. Historical Reporting: How would you balance the need for real-time processing with historical reporting requirements?
Data Granularity Trade-offs: In what other situations might daily snapshot fact tables be less appropriate?

**Comparison:**

```
date	product_category	total_units_sold	total_revenue
2023-10-05	Electronics	500	75000.00
```

**Code:**

```sql
CREATE TABLE daily_sales_summary (
    date DATE,
    product_category VARCHAR(50),
    total_units_sold INT,
    total_revenue DECIMAL(18, 2)
);

INSERT INTO daily_sales_summary (date, product_category, total_units_sold, total_revenue)
SELECT
    CURRENT_DATE(),
    p.category,
    SUM(t.units),
    SUM(t.revenue)
FROM transactions t
JOIN products p ON t.product_id = p.id
GROUP BY p.category;
```

**Example input / output:**

```text
Input:
Transaction data for the day (e.g., from a transactions table)

Output:
A single record in the daily_sales_summary table with aggregated sales metrics.
```

**Data:**

```json
{
  "date": "2023-10-05",
  "product_category": "Electronics",
  "total_units_sold": 500,
  "total_revenue": 75000.00
}
```

**Flow:**

1. Yes
2. No
3. Daily Transaction Data
4. Is it end of day?
5. Aggregate data
6. Daily Sales Summary Table
7. Wait for next batch

