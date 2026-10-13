---
type: "practice"
difficulty: "Intermediate"
---

# Explain how you audit an aggregate report for correctness against source-of-truth: reconciliation queries, checksums, and control totals. What did you catch this way?

**Difficulty:** Intermediate · **Levels:** SDE-1, SDE-1/2 · [↑ Advanced Aggregation](<../../../../01_Databases_and_SQL/SQL_Language/Aggregation/Advanced_Aggregation.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When working as a Database Developer, one of my key responsibilities involves ensuring the integrity and accuracy of data, especially in scenarios where aggregate reports are generated from multiple sources. Auditing these reports is crucial to maintain trust in the systems that rely on them. In this context, reconciliation queries, checksums, and control totals play pivotal roles.

#### Triaging the Question

The core of the question revolves around validating whether an aggregate report generated from a database matches its source-of-truth data. The interviewer is probing your understanding of different methods to ensure data integrity. Key assumptions include:

You have access to both the aggregate report and the raw, unaggregated source data.
There are known discrepancies that need to be identified and resolved.
Step-by-Step Solution
Reconciliation Queries

Reconciliation queries involve comparing the results of the aggregate report with a derived result from the source data. Here’s how you would implement this:

Identify the Aggregates: Determine which aggregates (e.g., SUM, AVG, COUNT) are being used in the reports.
Source Data Calculation: Write SQL queries to calculate these same aggregates directly on the raw source data.
Comparison: Run both sets of queries and compare their results.

#### Example

SQL
-- Aggregate report query for total sales
SELECT SUM(sales_amount) AS total_sales FROM sales;

-- Reconciliation query using source data
SELECT SUM(item_price * quantity) AS total_sales FROM items JOIN sales ON items.item_id = sales.item_id;
Checksums

Checksums provide a simple way to verify the integrity of large datasets. A checksum is calculated for both the raw and aggregated data, and then compared.

#### Example

Calculate Checksum: Use a hashing function (e.g., MD5) on the source data and the aggregate report.
Compare Checksums: Ensure both values match.
SQL
-- Calculate checksum for source data
SELECT MD5.Aggregate(CAST(item_price * quantity AS VARCHAR)) AS checksum FROM items JOIN sales ON items.item_id = sales.item_id;

-- Calculate checksum for aggregated report
SELECT MD5.Aggregate(CAST(total_sales AS VARCHAR)) FROM (SELECT SUM(sales_amount) AS total_sales FROM sales);
Control Totals

Control totals are predefined values that should always hold true. They act as a sanity check to ensure the data is consistent.

#### Example

Define Control Total: Determine an aggregate value that should remain constant or follow specific patterns.
Verify Control Total: Compare this value with the result from your reconciliation queries.
SQL
-- Define control total
SELECT COUNT(*) AS control_total FROM sales;

-- Verify control total against aggregated report
SELECT COUNT(sales_id) AS control_total FROM (SELECT SUM(sales_amount) AS total_sales FROM sales);

#### Closing Summary
and Common Mistakes/Follow-Ups

By using a combination of reconciliation queries, checksums, and control totals, you can effectively audit the correctness of aggregate reports. This approach not only ensures data integrity but also helps in identifying discrepancies that may indicate issues such as corruption, incomplete data, or errors in the ETL pipeline.

#### Common Mistakes

Overlooking edge cases like null values or zero entries.
Not considering performance implications when running large-scale queries.
Failing to document and track changes in the validation process over time.

#### Follow-Ups

How would you handle real-time data streams?
What about dealing with schema changes in the source data?
How do you manage historical audits for legacy systems?

**Comparison:**

| Item ID | Item Price | Quantity |
|---------|------------|----------|
| 1       | 50.0       | 2        |
| 2       | 75.0       | 3        |
| Sales ID | Item ID | Quantity | Sales Amount |
|----------|---------|----------|--------------|
| 1        | 1       | 2        | 100.0        |
| 2        | 2       | 3        | 225.0        |

**Code:**

```sql
-- Example SQL queries

-- Reconciliation query for total sales
SELECT SUM(sales_amount) AS total_sales FROM sales;

-- Source data reconciliation query
SELECT SUM(item_price * quantity) AS total_sales FROM items JOIN sales ON items.item_id = sales.item_id;

-- Checksum calculation for source data
SELECT MD5.Aggregate(CAST(item_price * quantity AS VARCHAR)) AS checksum FROM items JOIN sales ON items.item_id = sales.item_id;

-- Checksum calculation for aggregated report
SELECT MD5.Aggregate(CAST(total_sales AS VARCHAR)) FROM (SELECT SUM(sales_amount) AS total_sales FROM sales);

-- Control total definition and verification
SELECT COUNT(*) AS control_total FROM sales;
SELECT COUNT(sales_id) AS control_total FROM (SELECT SUM(sales_amount) AS total_sales FROM sales);
```

**Example input / output:**

```text
Reconciliation queries:
- Total sales from source data: 325.0
- Total sales from report: 325.0

Checksums:
- Source data checksum: MD5(175.0)
- Aggregated report checksum: MD5(325.0)

Control total verification:
- Sales count (source): 2
- Sales count (report): 2
```

**Data:**

```json
{
  "items": [
    {"item_id": 1, "item_price": 50.0},
    {"item_id": 2, "item_price": 75.0}
  ],
  "sales": [
    {"sales_id": 1, "item_id": 1, "quantity": 2, "sales_amount": 100.0},
    {"sales_id": 2, "item_id": 2, "quantity": 3, "sales_amount": 225.0}
  ]
}
```

**Flow:**

1. Yes
2. Discrepancies Found
3. No
4. Yes
5. No
6. No
7. Start
8. Are there knowndiscrepancies?
9. Run reconciliation queries
10. Compare results with sourcedata
11. Audit for root cause
12. Verify checksums and controltotals
13. Are all values correct?
14. Report is valid
15. Audit for issues
16. End Audit Process

