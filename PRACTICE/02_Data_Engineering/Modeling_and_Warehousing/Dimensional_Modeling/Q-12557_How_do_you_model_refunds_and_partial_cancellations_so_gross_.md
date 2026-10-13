---
type: "practice"
difficulty: "Deep"
---

# How do you model refunds and partial cancellations so gross revenue, net revenue, and refunded amount all reconcile from a single sales fact without negative-row confusion?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Additive and Allocated Measures](<../../../../02_Data_Engineering/Modeling_and_Warehousing/Dimensional_Modeling/Additive_and_Allocated_Measures.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Summary:

When modeling refunds and partial cancellations in a data warehouse, it's crucial to ensure that metrics like gross revenue, net revenue, and refunded amount are accurately calculated across all records without introducing negative-row confusion. This can be achieved using a star or snowflake schema with appropriate granular detail tables.

#### Triaging the Question

The interviewer is probing how you handle complex business scenarios in data modeling, specifically focusing on maintaining consistency between different financial metrics. The key aspect here is ensuring that the data model accurately reflects the true state of transactions while avoiding double-counting or negative values.

#### First-Principles Solution

Understanding the Problem:

Gross revenue is the total amount of money from sales.
Net revenue is gross revenue minus refunds and cancellations.
Refunds are partial or full returns of money to customers.

Key Assumptions:

We have a Sales fact table with detailed transaction records.
Each transaction can be marked as either complete, refunded, or partially canceled.
We need to ensure that the sum of all refunds and cancellations is subtracted from gross revenue to get net revenue.

Modeling the Data:

Create a Sales fact table with columns for order ID, customer ID, date, amount, transaction type (e.g., sale, refund, partial cancellation), and status.
For each transaction, update the Sales table directly or use a separate Refunds and Cancellations detail tables linked to the Sales table.

Implementing the Solution:

Step 1: Create a Sales fact table:

SQL
CREATE TABLE Sales (
    sale_id INT PRIMARY KEY,
    customer_id INT,
    order_date DATE,
    amount DECIMAL(10, 2),
    transaction_type VARCHAR(50),
    status VARCHAR(50)
);

Step 2: For each transaction type:

Sales: Mark the amount as positive.
Refunds: Mark the amount as negative to reflect the refund.
Partial Cancellations: Mark the amount as a fraction of the original sale, indicating partial cancellation.

Step 3: Create summary tables for gross revenue and net revenue:

SQL
CREATE TABLE GrossRevenueSummary (
    period_start DATE,
    period_end DATE,
    total_revenue DECIMAL(10, 2)
);

CREATE TABLE NetRevenueSummary (
    period_start DATE,
    period_end DATE,
    total_revenue DECIMAL(10, 2)
);

Step 4: Populate summary tables using SQL queries:

SQL
INSERT INTO GrossRevenueSummary (period_start, period_end, total_revenue)
SELECT order_date::DATE AS period_start,
       order_date::DATE + INTERVAL '1 day' - INTERVAL '1 millisecond' AS period_end,
       SUM(CASE WHEN transaction_type = 'sale' THEN amount ELSE 0 END) AS total_revenue
FROM Sales
GROUP BY order_date;

INSERT INTO NetRevenueSummary (period_start, period_end, total_revenue)
SELECT GRS.period_start,
       GRS.period_end,
       GR.total_revenue - COALESCE(RR.total_refunds, 0) - COALESCE(CC.total_cancellations, 0) AS net_revenue
FROM GrossRevenueSummary GRS
LEFT JOIN (
    SELECT order_date::DATE AS period_start,
           order_date::DATE + INTERVAL '1 day' - INTERVAL '1 millisecond' AS period_end,
           SUM(amount) AS total_refunds
    FROM Sales
    WHERE transaction_type = 'refund'
    GROUP BY order_date
) RR ON GRS.period_start = RR.period_start AND GRS.period_end = RR.period_end
LEFT JOIN (
    SELECT order_date::DATE AS period_start,
           order_date::DATE + INTERVAL '1 day' - INTERVAL '1 millisecond' AS period_end,
           SUM(amount) AS total_cancellations
    FROM Sales
    WHERE transaction_type = 'partial cancellation'
    GROUP BY order_date
) CC ON GRS.period_start = CC.period_start AND GRS.period_end = CC.period_end;

Ensuring Data Integrity:

Use constraints and triggers to ensure that the amount values are correctly marked for each transaction type.
Implement periodic reconciliation checks between the summary tables and the underlying detail tables.

#### Closing Summary

By using a detailed schema with granular records in the Sales fact table, we can accurately model refunds and partial cancellations. Summary tables provide an aggregated view of gross revenue and net revenue, ensuring that all transactions are accounted for without introducing negative-row confusion. This approach ensures consistency and accuracy in financial metrics.

#### Common Mistakes/Follow-Ups
Negative-Row Confusion: Ensure that refunded amounts are correctly marked as negative to avoid double-counting.

#### Performance Considerations
For large datasets, consider partitioning the Sales table by date or customer ID to improve query performance.
Data Consistency Checks: Implement periodic checks to verify that summary tables reconcile with detailed records.

**Comparison:**

```
Yes
No
Yes
No
Sales
Is Refund?
Mark as Negative
Is Partial Cancel?
Mark as Fraction of OriginalSale
Mark as Positive (Sale)
```

**Code:**

```sql
CREATE TABLE Sales (
    sale_id INT PRIMARY KEY,
    customer_id INT,
    order_date DATE,
    amount DECIMAL(10, 2),
    transaction_type VARCHAR(50),
    status VARCHAR(50)
);

INSERT INTO GrossRevenueSummary (period_start, period_end, total_revenue)
SELECT order_date::DATE AS period_start,
       order_date::DATE + INTERVAL '1 day' - INTERVAL '1 millisecond' AS period_end,
       SUM(CASE WHEN transaction_type = 'sale' THEN amount ELSE 0 END) AS total_revenue
FROM Sales
GROUP BY order_date;

INSERT INTO NetRevenueSummary (period_start, period_end, total_revenue)
SELECT GRS.period_start,
       GRS.period_end,
       GR.total_revenue - COALESCE(RR.total_refunds, 0) - COALESCE(CC.total_cancellations, 0) AS net_revenue
FROM GrossRevenueSummary GRS
LEFT JOIN (
    SELECT order_date::DATE AS period_start,
           order_date::DATE + INTERVAL '1 day' - INTERVAL '1 millisecond' AS period_end,
           SUM(amount) AS total_refunds
    FROM Sales
    WHERE transaction_type = 'refund'
    GROUP BY order_date
) RR ON GRS.period_start = RR.period_start AND GRS.period_end = RR.period_end
LEFT JOIN (
    SELECT order_date::DATE AS period_start,
           order_date::DATE + INTERVAL '1 day' - INTERVAL '1 millisecond' AS period_end,
           SUM(amount) AS total_cancellations
    FROM Sales
    WHERE transaction_type = 'partial cancellation'
    GROUP BY order_date
) CC ON GRS.period_start = CC.period_start AND GRS.period_end = CC.period_end;
```

**Example input / output:**

```text
-- Sample Data for Sales Table
INSERT INTO Sales (sale_id, customer_id, order_date, amount, transaction_type, status) VALUES
(1, 101, '2023-01-01', 50.00, 'sale', 'completed'),
(2, 102, '2023-01-01', 75.00, 'refund', 'processed'),
(3, 103, '2023-01-02', 45.00, 'partial cancellation', 'in progress');

-- Summary Data for GrossRevenueSummary Table
SELECT order_date::DATE AS period_start,
       order_date::DATE + INTERVAL '1 day' - INTERVAL '1 millisecond' AS period_end,
       SUM(CASE WHEN transaction_type = 'sale' THEN amount ELSE 0 END) AS total_revenue
FROM Sales
GROUP BY order_date;

-- Summary Data for NetRevenueSummary Table
SELECT GRS.period_start,
       GRS.period_end,
       GR.total_revenue - COALESCE(RR.total_refunds, 0) - COALESCE(CC.total_cancellations, 0) AS net_revenue
FROM GrossRevenueSummary GRS
LEFT JOIN (
    SELECT order_date::DATE AS period_start,
           order_date::DATE + INTERVAL '1 day' - INTERVAL '1 millisecond' AS period_end,
           SUM(amount) AS total_refunds
    FROM Sales
    WHERE transaction_type = 'refund'
    GROUP BY order_date
) RR ON GRS.period_start = RR.period_start AND GRS.period_end = RR.period_end
LEFT JOIN (
    SELECT order_date::DATE AS period_start,
           order_date::DATE + INTERVAL '1 day' - INTERVAL '1 millisecond' AS period_end,
           SUM(amount) AS total_cancellations
    FROM Sales
    WHERE transaction_type = 'partial cancellation'
    GROUP BY order_date
) CC ON GRS.period_start = CC.period_start AND GRS.period_end = CC.period_end;
```

**Data:**

```sql
CREATE TABLE Sales (
    sale_id INT PRIMARY KEY,
    customer_id INT,
    order_date DATE,
    amount DECIMAL(10, 2),
    transaction_type VARCHAR(50),
    status VARCHAR(50)
);

CREATE TABLE GrossRevenueSummary (
    period_start DATE,
    period_end DATE,
    total_revenue DECIMAL(10, 2)
);

CREATE TABLE NetRevenueSummary (
    period_start DATE,
    period_end DATE,
    total_revenue DECIMAL(10, 2)
);
```

**Flow:**

1. Yes
2. No
3. Yes
4. No
5. Sales
6. Is Transaction a Refund?
7. Record as Negative in SalesTable
8. Is Transaction a PartialCancellation?
9. Record as Fraction of OriginalSale Amount
10. Record as Positive (Sale) inSales Table

