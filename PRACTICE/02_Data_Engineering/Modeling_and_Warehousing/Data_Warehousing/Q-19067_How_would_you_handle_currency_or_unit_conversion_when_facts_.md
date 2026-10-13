---
type: "practice"
difficulty: "Deep"
---

# How would you handle currency or unit conversion when facts come from multiple countries?

**Difficulty:** Deep · **Levels:** Architect/Principal, Fresher, SDE-1/2, SDE-2/3 · [↑ Fact Dimension Integrity](<../../../../02_Data_Engineering/Modeling_and_Warehousing/Data_Warehousing/Fact_Dimension_Integrity.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR:

Currency or unit conversion in a data warehousing environment is crucial to ensure consistency and accuracy across different regions and units of measurement. The challenge lies in efficiently handling these conversions, especially when facts come from multiple countries with varying currency systems and units of measure.

#### Triage

The question asks how to handle currency or unit conversion when data originates from multiple countries. This involves understanding the data schema, identifying the source and target currencies/units, ensuring accuracy through reliable exchange rates, and efficiently implementing these conversions in a data warehousing system.

#### First-Principles Solution

Understand Data Sources: Identify the countries and regions where the facts originate. For each country, determine the primary currency used (e.g., USD for the USA, EUR for Europe).

Schema Design:

Create dimensions for currencies (CurrencyDim) with fields like CountryCode, Name, ISOCode.
Similarly, create a dimension for units of measure (UnitsOfMeasureDim).
Include fact tables where these conversions will be applied.

Exchange Rate Management:

Use an exchange rate table (ExchangeRateDim), which tracks the conversion rates between different currencies over time.

#### Cons
ider using a reference date to manage historical and real-time conversion rates.

Unit Conversion Logic:

For unit conversions, maintain tables or dimensions that map units from one system to another (e.g., meters to feet).
Implement logic in ETL pipelines to dynamically convert units based on the source data's context.

ETL Pipelines:

Develop an ETL pipeline that reads raw data, applies currency/unit conversion rules using lookups from dimensions.
Ensure the pipeline is robust by handling null values and edge cases (e.g., no available exchange rate).

#### Performance Considerations

Use efficient indexing on dimensions to speed up queries involving large datasets.
Implement caching strategies for frequently accessed conversion rates.

Data Quality Checks:

Validate the accuracy of conversions through integrity checks in the ETL process and data quality rules.
Monitor the system's performance and make adjustments as needed.

Scalability and Maintenance:

Design the solution to handle future countries and units gracefully, possibly through modular code or dynamic configuration.
Plan for regular updates of exchange rates and unit conversion tables.

#### Closing Summary
+ Common Mistakes/Follow-Ups:

#### Common Mistakes

Failing to account for historical data when implementing real-time conversions.
Not properly handling null values in the source data, leading to incorrect calculations.
Overlooking edge cases such as zero or one-to-one unit conversions.

#### Follow-Ups

How would you handle bulk updates of exchange rates?
What strategies do you have for ensuring the integrity and accuracy of your conversion logic?

**Comparison:**

```
CountryCode	Name	ISOCode
US	United States	USD
DE	Germany	EUR
Date	CurrencyFrom	CurrencyTo	Rate
2023-10-01	USD	EUR	0.85
2023-10-05	EUR	USD	1.18
```

**Code:**

```python
# Example ETL Code Snippet for Currency Conversion
import pandas as pd

def convert_currency(amount, source_currency, target_currency, exchange_rates):
    if source_currency != target_currency:
        rate = exchange_rates[source_currency][target_currency]
        return amount * rate
    else:
        return amount  # No conversion needed

# Example Exchange Rate Data
exchange_rates = {
    'USD': {'EUR': 0.85, 'JPY': 110},
    'EUR': {'USD': 1.18, 'JPY': 130}
}

# Sample Data
data = pd.DataFrame({
    'Country': ['USA', 'Germany'],
    'Amount': [100, 200],
    'Currency': ['USD', 'EUR']
})

# Apply Conversion Logic
converted_data = data.apply(lambda row: (row['Amount'], convert_currency(row['Amount'], row['Currency'], 'EUR', exchange_rates)), axis=1)
```

**Example input / output:**

```text
Input:
Country: US, Amount: 100, Currency: USD

Output:
Converted Currency: EUR, Converted Amount: 85.0
```

**Data:**

```json
{
  "CurrencyDim": {
    "CountryCode": ["US", "DE"],
    "Name": ["United States", "Germany"],
    "ISOCode": ["USD", "EUR"]
  },
  "ExchangeRateDim": {
    "Date": ["2023-10-01", "2023-10-05"],
    "CurrencyFrom": ["USD", "EUR"],
    "CurrencyTo": ["EUR", "USD"],
    "Rate": [0.85, 1.18]
  }
}
```

**Flow:**

1. Yes
2. No
3. Yes
4. Yes
5. No
6. No
7. ETL Start
8. Is Source Currency Different?
9. Convert Amount
10. Use Historical Data?
11. Fetch Real-Time Rate
12. Apply Conversion
13. Apply Historical Rate
14. Store Converted Data
15. Is Unit Conversion Needed?
16. Convert Units
17. Store Converted Units
18. End ETL Process

