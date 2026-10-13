---
type: "practice"
difficulty: "Intermediate"
---

# Why is it important that every row in a fact table has the same grain?

**Difficulty:** Intermediate · **Levels:** Architect/Principal, Fresher, SDE-1/2, SDE-2/3 · [↑ Fact Table Grain](<../../../../02_Data_Engineering/Modeling_and_Warehousing/Dimensional_Modeling/Fact_Table_Grain.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When designing data models, ensuring that all rows in a fact table have the same "grain" (i.e., the level of detail) is crucial to maintain consistency and integrity in your database. This practice is essential in both star and snowflake schemas used for data warehousing.

The core idea here is about maintaining uniformity in the data granularity, which ensures that all measures within a fact table are comparable and meaningful. A unified grain also simplifies query writing, maintenance, and reporting processes, making it easier to analyze and derive insights from your data.

To tackle this question step-by-step:

Understanding the Grain: The "grain" refers to the level of detail at which a particular piece of data is recorded. For example, if you are recording sales data, the grain could be per transaction, per day, or per week.

#### Cons
istency Across Rows: Every row in a fact table should represent the same granular level of detail. If you mix different grains within a single fact table, it becomes challenging to perform meaningful aggregations and comparisons.
Avoiding Data Integrity Issues: Ensuring uniformity helps prevent data integrity issues such as inconsistent or misleading calculations. For example, if one row represents daily sales while another represents monthly sales, combining them in an analysis would lead to erroneous results.

Let's break it down further:

Why Uniform Grain is Important
Simplicity and Readability: A unified grain makes the database easier to understand and work with, reducing the complexity of queries.

#### Cons
istent Aggregations: When all rows have the same granular level, you can easily aggregate data without worrying about mismatches in detail levels.
Maintaining Consistency Across Fact Tables: In a star or snowflake schema, fact tables are often linked to dimension tables. If their grain is inconsistent, it complicates linking and joining operations.

#### Example Scenario

#### Cons
ider a sales database where we have a Sales fact table with the following attributes:

CODE
SalesID | ProductID | CustomerID | Date       | SalesAmount
--------|-----------|------------|------------|------------
1       | 102       | 20456      | 2023-10-01 | 50.00
2       | 102       | 20456      | 2023-10-02 | 75.00
3       | 103       | 20456      | 2023-10-01 | 30.00

Here, all rows have a daily grain (the date column). This makes it straightforward to perform daily sales calculations or aggregate monthly totals.

#### Common Mistakes
and Follow-Ups
Mixing Granularities: One common mistake is mixing different granularities within the same fact table. For instance, using both daily and weekly data in a single Sales fact table.
Ignoring the Grain When Joining Tables: Another issue is failing to consider the grain when joining dimension tables with fact tables. For example, ensuring that the customer information (with its own granular level) aligns correctly.

In summary, maintaining uniformity in the grain of a fact table ensures data integrity and simplifies database operations, making your data model more robust and easier to work with.

**Comparison:**

```
table Sales {
    Date: Date
    ProductID: Product ID
    CustomerID: Customer ID
    SalesAmount: Sales Amount (in USD)
}
```

**Code:**

```sql
-- Example of consistent granularity in a fact table
CREATE TABLE Sales (
    SalesID INT PRIMARY KEY,
    ProductID INT,
    CustomerID INT,
    Date DATE,
    SalesAmount DECIMAL(10, 2)
);
```

**Example input / output:**

```text
Input:
SELECT Date, SUM(SalesAmount)
FROM Sales
GROUP BY Date

Output:
+------------+-------------------+
| Date       | Total Sales Amount|
+------------+-------------------+
| 2023-10-01 |                80 |
| 2023-10-02 |               75  |
+------------+-------------------+
```

**Data:**

```json
{
  "Sales": [
    { "SalesID": 1, "ProductID": 102, "CustomerID": 20456, "Date": "2023-10-01", "SalesAmount": 50.00 },
    { "SalesID": 2, "ProductID": 102, "CustomerID": 20456, "Date": "2023-10-02", "SalesAmount": 75.00 },
    { "SalesID": 3, "ProductID": 103, "CustomerID": 20456, "Date": "2023-10-01", "SalesAmount": 30.00 }
  ]
}
```

**Flow:**

1. Yes
2. No
3. Sales Data
4. Is Grain Consistent?
5. Perform Aggregations Easily
6. Misleading Results Due toInconsistent Granularity
7. Consistent Query Results
8. Data Integrity Issues

