---
type: "practice"
difficulty: "Deep"
---

# How do you handle currency conversion in a multi-country revenue fact so both reporting-currency and local-currency totals stay auditable?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Fact Dimension Integrity](<../../../../02_Data_Engineering/Modeling_and_Warehousing/Data_Warehousing/Fact_Dimension_Integrity.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To handle currency conversion in a data warehousing environment, especially when dealing with multi-country revenues where you need to maintain both reporting-currency and local-currency totals, we must ensure that all conversions are accurate, auditable, and transparent. This involves several key steps: identifying the appropriate exchange rates, applying these rates correctly, storing historical data, and ensuring traceability through the conversion process.

Firstly, let's break down what the question is really asking. The core of this problem is to maintain two sets of revenue figures—local currency (the actual transaction amount in each country’s local currency) and reporting currency (a standardized currency used for consolidated financial reports). We need a robust system that can convert between these currencies accurately, store the original and converted values, and ensure that any conversion rates and their dates are traceable.

We will assume you have historical exchange rate data available, typically from reliable sources such as central banks or financial APIs. The key concepts include:

Data Model Design: How to structure our database tables.
Conversion Logic: Applying the correct exchange rate to convert local currencies to reporting currency.
Historical Data Storage: Keeping track of original values and conversion details for audit purposes.
Auditability: Ensuring that the conversion process is transparent and can be traced back.

Let’s walk through this from first principles:

Identify Exchange Rate Sources:

We need a reliable source for exchange rates, such as historical data provided by central banks or reputable financial APIs (e.g., OpenExchangeRates).

Data Model Design:

RevenueFact Table: This table will store the local currency revenue and the reporting currency equivalent.
SQL
CREATE TABLE RevenueFact (
    id INT PRIMARY KEY,
    country_code VARCHAR(5),
    local_currency_amount DECIMAL(18, 4),
    reporting_currency_amount DECIMAL(18, 4),
    conversion_rate FLOAT,
    exchange_rate_date DATE,
    source_system_id INT
);
ExchangeRate Table: This table will store historical exchange rates.
SQL
CREATE TABLE ExchangeRate (
    id INT PRIMARY KEY,
    base_currency VARCHAR(5),
    target_currency VARCHAR(5),
    rate FLOAT,
    date DATE
);

Conversion Logic:

When a new revenue fact is recorded, we need to fetch the appropriate exchange rate for that transaction’s date and apply it.
SQL
-- Example SQL query to insert into RevenueFact
INSERT INTO RevenueFact (country_code, local_currency_amount, conversion_rate, exchange_rate_date)
SELECT 'US', 1000.00, (
    SELECT rate FROM ExchangeRate WHERE base_currency = 'USD' AND target_currency = 'EUR' AND date = '2023-09-01'
), '2023-09-01';

Historical Data Storage:

Each revenue fact should store both the local currency and reporting currency amounts, along with the conversion rate and date.
This allows us to maintain a history of all conversions.

Auditability:

Ensure that every transaction has an associated source system ID for traceability.
SQL
INSERT INTO RevenueFact (country_code, local_currency_amount, reporting_currency_amount, conversion_rate, exchange_rate_date, source_system_id)
VALUES ('US', 1000.00, 753.83, 0.75383, '2023-09-01', 1);

Data Quality and Validation:

Regularly validate the exchange rate data to ensure accuracy.
Implement validation rules to check for invalid rates or missing data.

#### Closing Summary
By structuring our database appropriately, ensuring we capture all necessary details, and maintaining a robust conversion process, we can handle currency conversions in multi-country revenue facts while keeping them auditable. This approach guarantees that the original values remain intact and that any changes are transparent and traceable.

#### Common Mistakes

Failing to store historical exchange rates, which could lead to inaccuracies if current rates are used for older transactions.
Not updating the RevenueFact table with the correct conversion rate when the transaction date is different from the exchange rate date.
Neglecting to validate the source of exchange rate data, potentially leading to using inaccurate or outdated information.

**Comparison:**

|id | country_code | local_currency_amount | reporting_currency_amount | conversion_rate | exchange_rate_date | source_system_id|
|id | base_currency | target_currency | rate | date |

**Code:**

```sql
-- SQL code for RevenueFact and ExchangeRate tables
CREATE TABLE RevenueFact (
    id INT PRIMARY KEY,
    country_code VARCHAR(5),
    local_currency_amount DECIMAL(18, 4),
    reporting_currency_amount DECIMAL(18, 4),
    conversion_rate FLOAT,
    exchange_rate_date DATE,
    source_system_id INT
);

CREATE TABLE ExchangeRate (
    id INT PRIMARY KEY,
    base_currency VARCHAR(5),
    target_currency VARCHAR(5),
    rate FLOAT,
    date DATE
);
```

**Example input / output:**

```text
-- Example input query to insert a new revenue fact and get the reporting currency amount
INSERT INTO RevenueFact (country_code, local_currency_amount, conversion_rate, exchange_rate_date, source_system_id)
VALUES ('US', 1000.00, (
    SELECT rate FROM ExchangeRate WHERE base_currency = 'USD' AND target_currency = 'EUR' AND date = '2023-09-01'
), '2023-09-01', 1);

-- Example output query to retrieve the stored data
SELECT * FROM RevenueFact WHERE id = 1;
```

**Data:**

```json
{
  "RevenueFact": [
    {
      "id": 1,
      "country_code": "US",
      "local_currency_amount": 1000.00,
      "reporting_currency_amount": 753.83,
      "conversion_rate": 0.75383,
      "exchange_rate_date": "2023-09-01",
      "source_system_id": 1
    }
  ],
  "ExchangeRate": [
    {
      "id": 1,
      "base_currency": "USD",
      "target_currency": "EUR",
      "rate": 0.75383,
      "date": "2023-09-01"
    }
  ]
}
```

**Flow:**

1. Yes
2. No
3. Transaction Recorded
4. Fetch Exchange Rate
5. Is Date of Transaction ==Exchange Rate Date?
6. Calculate Reporting CurrencyAmount
7. Use Latest Applicable Rate andAdjust Historical Record
8. Insert into RevenueFact Table
9. Update Existing Record inRevenueFact Table
10. Store Local & ReportingCurrency Amounts, ConversionDetails

