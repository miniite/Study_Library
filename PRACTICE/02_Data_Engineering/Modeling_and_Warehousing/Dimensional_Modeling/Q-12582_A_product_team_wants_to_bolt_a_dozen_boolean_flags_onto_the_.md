---
type: "practice"
difficulty: "Deep"
---

# A product team wants to bolt a dozen boolean flags onto the fact table for filtering; how do you push back and where should those flags actually live?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Aggregate Fact Tables](<../../../../02_Data_Engineering/Modeling_and_Warehousing/Dimensional_Modeling/Aggregate_Fact_Tables.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the request of bolting a dozen boolean flags onto the fact table, it’s essential to consider data modeling best practices and performance implications. This scenario highlights the importance of maintaining a clean and scalable data model, especially in a data warehouse environment.

The core issue here is whether the boolean flags should reside on the fact table (as proposed) or if they should be pushed into an alternate dimension table. Let’s break this down step-by-step:

Understanding the Scenario: The product team wants to add multiple boolean flags to the fact table for filtering purposes, which suggests a need for flexible and dynamic querying capabilities.
Probing Assumptions: We assume that these boolean flags are related to specific dimensions or attributes of the products being tracked in the fact table.
First-Principles Reasoning:
Fact Table Characteristics: Fact tables typically store transactional data with a high cardinality and relatively low grain, making them efficient for aggregation operations.
Dimension Tables Characteristics: Dimension tables are used to provide context to the fact data, often including attributes like product details, customer information, or time periods.
Why Pushing Back is Necessary

Pushing back on adding boolean flags directly to the fact table involves understanding potential downsides and proposing a more scalable solution. Here are key reasons:

Cardinality Considerations: Fact tables can quickly become bloated with additional columns, reducing their effectiveness for join operations.
Query Performance Impact: Boolean flags added to a fact table might degrade query performance due to increased selectivity and the need to filter through many rows.
Maintainability and Scalability: Managing boolean flags within dimension tables improves data model maintainability and scalability.
Recommended Solution: Using Dimension Tables

To address these concerns, we should propose adding these boolean flags as additional dimensions in a star or snowflake schema. This approach keeps the fact table lean while allowing for flexible filtering through proper join operations.

Dimension Table Design:

Create a separate dimension table that maps to relevant product attributes.
Ensure this new dimension table is normalized, containing only the necessary boolean flags.

Schema Example (assuming we’re using a Snowflake schema):

SQL
CREATE TABLE Product_Attributes (
    Product_ID INT PRIMARY KEY,
    Flag_A BOOLEAN,
    Flag_B BOOLEAN,
    ...
);
Fact Table Join:
When querying, join the fact table with this dimension table to filter on boolean flags.

#### Implementation
Example (SQL)

Here’s a simple implementation:

SQL
-- Fact Table
CREATE TABLE Product_Sales (
    Sale_ID INT PRIMARY KEY,
    Date DATE,
    Product_ID INT,
    Quantity INT,
    Price DECIMAL(10, 2),
    FOREIGN KEY (Product_ID) REFERENCES Products(Product_ID)
);

-- Dimension Table
CREATE TABLE Product_Attributes (
    Product_ID INT PRIMARY KEY,
    Flag_A BOOLEAN,
    Flag_B BOOLEAN
);
Query Example:
SQL
SELECT
    PS.Sale_ID,
    PA.Flag_A,
    PA.Flag_B
FROM
    Product_Sales PS
JOIN
    Product_Attributes PA ON PS.Product_ID = PA.Product_ID
WHERE
    PA.Flag_A = TRUE AND PA.Flag_B = FALSE;

#### Closing Summary
and Common Mistakes/Follow-Ups
Common Mistake: Directly adding boolean flags to the fact table can lead to poor query performance, bloated tables, and reduced maintainability.

#### Follow-Up
Questions:
How do you handle changes in these boolean flags over time?
Are there any specific use cases where the boolean flags are frequently updated?

By sticking to a well-designed data model with appropriate dimension tables, we can ensure that our data warehouse remains performant and scalable.

**Comparison:**

| Sale ID | Date       | Product ID | Quantity | Price   |
|---------|------------|------------|----------|---------|
| 101     | 2023-10-01 | 1          | 5        | 9.99    |
| Product_ID | Flag_A | Flag_B |
|------------|--------|--------|
| 1          | true   | false  |

**Code:**

```sql
-- Example SQL for fact and dimension table creation
CREATE TABLE Product_Sales (
    Sale_ID INT PRIMARY KEY,
    Date DATE,
    Product_ID INT,
    Quantity INT,
    Price DECIMAL(10, 2),
    FOREIGN KEY (Product_ID) REFERENCES Products(Product_ID)
);

CREATE TABLE Product_Attributes (
    Product_ID INT PRIMARY KEY,
    Flag_A BOOLEAN,
    Flag_B BOOLEAN
);
```

**Example input / output:**

```text
-- Example input and output for filtering
Input:
JOIN Product_Sales PS ON PA.Product_ID = PS.Product_ID WHERE PA.Flag_A = TRUE AND PA.Flag_B = FALSE;

Output:
Sale_ID: 101, Flag_A: true, Flag_B: false
```

**Data:**

```json
{
  "Product_Sales": [
    {
      "Sale_ID": 101,
      "Date": "2023-10-01",
      "Product_ID": 1,
      "Quantity": 5,
      "Price": 9.99
    },
    ...
  ],
  "Product_Attributes": [
    {
      "Product_ID": 1,
      "Flag_A": true,
      "Flag_B": false
    },
    ...
  ]
}
```

**Flow:**

1. Fact Table
2. Dimension Table
3. Join
4. Filter by Flags
5. Result Set
6. Query Execution
7. Performance & Scalability
8. Data Integrity &Maintainability
9. Fact Table Bloat

