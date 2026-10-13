---
type: "practice"
difficulty: "Deep"
---

# How do you handle currency conversion in a payments fact so reports slice by both transaction currency and a single reporting currency, and where do you store the rate?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Transaction vs Snapshot Facts](<../../../../02_Data_Engineering/Modeling_and_Warehousing/Dimensional_Modeling/Transaction_vs_Snapshot_Facts.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To handle currency conversion in a payments fact table so that reports can slice by both transaction currency and a single reporting currency, we need to carefully design our data model using either a star or snowflake schema, ensuring accuracy and efficiency. The key idea is to store the conversion rate separately from the actual payment amounts, then apply this rate during query time.

#### Triage

The question requires designing a solution for storing and converting currency in a payments fact table while supporting reporting in multiple currencies. This involves understanding how to use a star or snowflake schema effectively, managing data integrity, and optimizing performance for queries that need to slice by different currencies.

#### First-Principles Solution
Data Model Design:
Use either a star schema with a currency_conversion dimension table or a snowflake schema.
Currency Conversion Table:
Create a currency_conversion table to store historical exchange rates.
Payments Fact Table:
Store transaction amounts in the original currency.
Include a foreign key linking to the currency_conversion table.
Querying and Aggregation:
Perform real-time conversions during query execution using SQL JOINs.
Step-by-Step Solution
Currency Conversion Table (Star Schema):
SQL
CREATE TABLE currency_conversion (
    id INT PRIMARY KEY,
    base_currency VARCHAR(3),
    target_currency VARCHAR(3),
    conversion_rate DECIMAL(10, 5),
    valid_from DATE,
    valid_to DATE
);
Payments Fact Table:
SQL
CREATE TABLE payments_fact (
    transaction_id INT PRIMARY KEY,
    amount DECIMAL(19, 4),
    currency VARCHAR(3),
    conversion_rate_id INT,
    FOREIGN KEY (conversion_rate_id) REFERENCES currency_conversion(id)
);
Query to Convert and Summarize:
SQL
SELECT
    p.currency AS transaction_currency,
    c.target_currency AS reporting_currency,
    SUM(p.amount * cc.conversion_rate) AS converted_amount
FROM payments_fact p
JOIN currency_conversion cc ON p.conversion_rate_id = cc.id
GROUP BY p.currency, c.target_currency;

#### Closing Summary
and Common Mistakes/Follow-Ups

In this solution, we use a currency_conversion table to store historical exchange rates. The payments_fact table stores the actual payment amounts in their original currency along with a foreign key linking to the appropriate conversion rate. This approach ensures that reporting can be done in multiple currencies by performing conversions during query execution.

#### Common Mistakes
Storing Conversion Rates Directly in Fact Table: Storing the conversion rates directly within the payments_fact table would lead to data redundancy and potential inconsistencies.
Ignoring Historical Data: Failing to account for changes in exchange rates over time can result in inaccurate reporting.
Performance Overhead: Real-time conversions during query execution may introduce performance overhead, which needs careful consideration.

#### Follow-Ups
How do you handle bulk loading of historical data into the currency_conversion table?
What are the implications if the exchange rate changes frequently?
How would you ensure that the latest conversion rates are always used for reporting?

**Code:**

```sql
-- Currency Conversion Table (Star Schema)
CREATE TABLE currency_conversion (
    id INT PRIMARY KEY,
    base_currency VARCHAR(3),
    target_currency VARCHAR(3),
    conversion_rate DECIMAL(10, 5),
    valid_from DATE,
    valid_to DATE
);

-- Payments Fact Table
CREATE TABLE payments_fact (
    transaction_id INT PRIMARY KEY,
    amount DECIMAL(19, 4),
    currency VARCHAR(3),
    conversion_rate_id INT,
    FOREIGN KEY (conversion_rate_id) REFERENCES currency_conversion(id)
);
```

**Example input / output:**

```text
-- Example Input and Output
SELECT
    p.currency AS transaction_currency,
    c.target_currency AS reporting_currency,
    SUM(p.amount * cc.conversion_rate) AS converted_amount
FROM payments_fact p
JOIN currency_conversion cc ON p.conversion_rate_id = cc.id
GROUP BY p.currency, c.target_currency;

-- Sample Data
INSERT INTO currency_conversion (id, base_currency, target_currency, conversion_rate, valid_from, valid_to)
VALUES (1, 'USD', 'EUR', 0.85, '2023-01-01', '9999-12-31');

INSERT INTO payments_fact (transaction_id, amount, currency, conversion_rate_id)
VALUES (1, 100, 'USD', 1);

-- Expected Output
+----------------+---------------+----------------+
| transaction_currency | reporting_currency | converted_amount |
+----------------+---------------+----------------+
| USD             | EUR            | 85.00000        |
+----------------+---------------+----------------+
```

**Flow:**

1. Payment Fact Table
2. Join with Currency Conversion
3. Calculate Converted Amount
4. Aggregate by ReportingCurrency
5. Currency Conversion Table

