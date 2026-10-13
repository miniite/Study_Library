---
type: "practice"
difficulty: "Deep"
---

# What happens if two fact rows have different grains but are stored in the same fact table?

**Difficulty:** Deep · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Fact Table Grain](<../../../../02_Data_Engineering/Modeling_and_Warehousing/Dimensional_Modeling/Fact_Table_Grain.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When dealing with a star or snowflake schema, it's essential to understand how data is aggregated and analyzed. If two fact rows have different grains (i.e., they represent different levels of detail) but are stored in the same fact table, this can lead to logical inconsistencies and potential misinterpretations during analysis.

#### Triage

The question is probing your understanding of data granularity in a star or snowflake schema and how it impacts data consistency. You need to consider the implications for query results, data integrity, and overall database design. Assumptions include that you are familiar with fact tables and the concept of grain in data modeling.

#### First-Principles Solution
Understanding Fact Tables: In a star or snowflake schema, fact tables store measurements (facts) related to specific events or transactions. Each fact typically has multiple dimensions (related to attributes like time, location, etc.) that provide context.
Grain Definition: Grain refers to the level of detail in a fact table. For example, if grain is daily sales, each row might represent one day's worth of sales data. If it's hourly sales, each row would represent one hour’s worth of sales.
Impact of Mixed Grains: Storing two fact rows with different grains in the same fact table can lead to ambiguity and incorrect analysis:
Queries aggregating across different grain levels might produce misleading results.
It may also violate data integrity rules, leading to inconsistent data storage.
Detailed Solution

#### Example Scenario
Consider a sales fact table where one row represents daily sales (grain: day), and another represents hourly sales (grain: hour).
Logical Issues:
If you attempt to aggregate hourly sales with daily totals, the granularity mismatch can result in incorrect summation.
For instance, if you sum up all hours for a single day, it might not accurately reflect the total daily sales due to potential overlaps or missing data points.
Step-by-Step Solution
Identify the Grain: Determine the appropriate grain level for your fact table based on business requirements and analysis needs.
Data Normalization: Ensure that all fact rows within a single fact table share the same grain. This can be achieved by:
Creating separate fact tables for different grains if needed.
Using dimension tables to store granular data, which can then be joined with higher-grain fact tables.

#### Closing Summary
and Common Mistakes

To avoid issues related to mixed grain levels in a star or snowflake schema:

Maintain Consistency: Ensure that all rows within the same fact table have the same grain.
Use Separate Tables: If different grains are necessary, consider using separate fact tables for each level of detail.
Proper Aggregation Logic: Develop robust aggregation logic to handle data from multiple granular levels.

#### Common Mistakes
Ignoring Grain Consistency: Failing to ensure that all rows within a single fact table share the same grain can lead to incorrect analysis and data inconsistencies.
Overcomplication: Creating overly complex schemas with unnecessary tables without clear business justification can complicate database management and query performance.
Summary

Maintaining consistent grain levels in fact tables is crucial for accurate data modeling, especially in star or snowflake schema designs. Proper design practices ensure that queries yield meaningful results, maintaining the integrity and reliability of your data.

**Comparison:**

| Date       | Total Sales |
|------------|-------------|
| 2023-01-01 | 500.00      |
| 2023-01-02 | 600.00      |
| Timestamp   | Total Sales |
|-------------|-------------|
| 2023-01-01T09:00:00Z | 50.00       |
| 2023-01-02T10:00:00Z | 60.00       |

**Code:**

```sql
-- Example SQL to demonstrate grain inconsistency issues

CREATE TABLE daily_sales (
    sales_id INT PRIMARY KEY,
    date DATE,
    total_sales DECIMAL(10, 2)
);

INSERT INTO daily_sales (sales_id, date, total_sales) VALUES
(1, '2023-01-01', 500.00),
(2, '2023-01-02', 600.00);

CREATE TABLE hourly_sales (
    sales_id INT PRIMARY KEY,
    timestamp TIMESTAMP,
    total_sales DECIMAL(10, 2)
);

INSERT INTO hourly_sales (sales_id, timestamp, total_sales) VALUES
(1, '2023-01-01 09:00:00', 50.00),
(2, '2023-01-02 10:00:00', 60.00);
```

**Example input / output:**

```text
Input:
SELECT SUM(total_sales) FROM daily_sales

Output:
500.00 + 600.00 = 1100.00 (Correct)

Incorrect Input/Output Example:
SELECT SUM(total_sales) FROM hourly_sales

Output:
50.00 + 60.00 = 110.00 (Incorrect due to grain mismatch)
```

**Data:**

```json
{
  "daily_sales": [
    {"sales_id": 1, "date": "2023-01-01", "total_sales": 500.00},
    {"sales_id": 2, "date": "2023-01-02", "total_sales": 600.00}
  ],
  "hourly_sales": [
    {"sales_id": 1, "timestamp": "2023-01-01T09:00:00Z", "total_sales": 50.00},
    {"sales_id": 2, "timestamp": "2023-01-02T10:00:00Z", "total_sales": 60.00}
  ]
}
```

**Flow:**

1. No
2. Yes
3. Check Data Granularity
4. Are all rows in the same tableconsistent?
5. Create separate fact tables fordifferent granularities
6. Verify aggregation logic tohandle mixed grains correctly

