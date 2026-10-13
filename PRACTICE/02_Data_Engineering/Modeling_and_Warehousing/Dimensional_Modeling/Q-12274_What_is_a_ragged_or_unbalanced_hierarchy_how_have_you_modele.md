---
type: "practice"
difficulty: "Deep"
---

# What is a ragged or unbalanced hierarchy, how have you modeled one, and what problems does it cause for drill-down queries?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1, SDE-1/2, SDE-2/3 · [↑ Ragged Hierarchies](<../../../../02_Data_Engineering/Modeling_and_Warehousing/Dimensional_Modeling/Ragged_Hierarchies.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR

Ragged hierarchies occur in data models when different levels of granularity are present within the same dimension. These can be challenging to model because they require careful consideration to ensure consistency and performance, especially during drill-down queries. In my experience with modeling ragged hierarchies, I used a combination of star schema and nested sets for flexibility and efficient querying.

#### Triaging the Question

The question is asking about ragged hierarchies in data models, particularly how they can be modeled and the issues that arise when performing drill-down queries. The interviewer is likely probing your understanding of hierarchical structures, modeling techniques, and performance considerations.

#### Assumptions
Ragged hierarchies exist where not all levels are present at every point in a hierarchy.
We need to support efficient drill-down operations (e.g., moving from summary data to detailed data).
The model must be flexible enough to accommodate different levels of granularity dynamically.

#### First Principles Solution
Understanding the Problem

A ragged hierarchy is a challenge because traditional hierarchical structures, like those found in star schemas, assume a consistent level of detail across all dimensions. For example, if you have a customer dimension and some customers are at the "state" level while others are only at the "city" level, this creates an unbalanced structure.

Modelling Strategy

To handle ragged hierarchies effectively, we can use a combination of star schema principles and additional techniques to support dynamic hierarchies. Here’s how you might approach it:

Star Schema with Additional Fact Tables: Use the traditional star schema for basic operations but add an "optional" fact table that captures more detailed levels.
Nested Sets: Implement nested sets (or treemaps) to maintain the hierarchy in a flexible manner, allowing for different levels of detail.
Step-by-Step Solution

Define the Hierarchy Structure

Create dimension tables with all necessary attributes but allow some to be null or default.
For example, if you have customer dimensions like customer_id, state, and city, ensure that some rows might not have a city value.

Flexible Fact Tables

Design fact tables that can support different levels of granularity dynamically. This could involve using a separate fact table for detailed data (like city-level sales) while linking it back to the summary fact table.
SQL
-- Summary Fact Table
CREATE TABLE Sales_Summary (
    customer_id INT,
    state VARCHAR(100),
    total_sales DECIMAL(15, 2)
);

-- Detailed Fact Table
CREATE TABLE Sales_Detail (
    sales_id INT PRIMARY KEY,
    customer_id INT,
    city VARCHAR(100),
    sale_amount DECIMAL(15, 2)
);

Nested Sets for Hierarchy

Use nested sets to manage the hierarchy. This involves creating a table where each row has left and right values that define its position in the hierarchy.
SQL
CREATE TABLE Customer_Hierarchy (
    customer_id INT PRIMARY KEY,
    left_value BIGINT,
    right_value BIGINT,
    level INT -- 1 for state, 2 for city
);

Drill-Down Query Optimization

For efficient drill-down queries, use a combination of joins and aggregation to move from summary data to detailed data.
SQL
SELECT
    c.customer_id,
    cs.state,
    cd.city,
    SUM(s.sale_amount) AS total_sales
FROM Sales_Detail sd
JOIN Customer_Hierarchy ch ON sd.customer_id = ch.customer_id
LEFT JOIN Sales_Summary ss ON sd.customer_id = ss.customer_id AND ss.state = cd.state
GROUP BY c.customer_id, cs.state, cd.city;

#### Closing Summary
and Common Mistakes

#### Common Mistakes
Failing to account for the ragged nature of the hierarchy can lead to data inconsistencies.
Not optimizing queries properly can result in poor performance during drill-down operations.

#### Follow-Up
Questions
How do you handle changes in hierarchy levels?
What are some alternative techniques for managing ragged hierarchies?

#### Conclusion

Ragged hierarchies require a flexible and robust modeling approach to ensure both accuracy and efficient querying. Using a combination of star schema principles, nested sets, and dynamic fact tables can help manage these complexities effectively.

**Comparison:**

```
is
CUSTOMER
CUSTOMER_HIERARCHY
int
customer_id
PK
int
left_value
int
right_value
int
level
part
of
SALES_SUMMARY
int
sales_id
PK
int
customer_id
FK
string
state
decimal
total_sales
SALES_DETAIL
int
sales_id
PK
int
customer_id
FK
string
city
decimal
sale_amount
```

**Code:**

```sql
-- Example SQL for creating hierarchy tables
CREATE TABLE Sales_Summary (
    customer_id INT,
    state VARCHAR(100),
    total_sales DECIMAL(15, 2)
);

CREATE TABLE Sales_Detail (
    sales_id INT PRIMARY KEY,
    customer_id INT,
    city VARCHAR(100),
    sale_amount DECIMAL(15, 2)
);

CREATE TABLE Customer_Hierarchy (
    customer_id INT PRIMARY KEY,
    left_value BIGINT,
    right_value BIGINT,
    level INT
);
```

**Example input / output:**

```text
-- Input Query to Drill Down
SELECT
    c.customer_id,
    cs.state,
    cd.city,
    SUM(s.sale_amount) AS total_sales
FROM Sales_Detail sd
JOIN Customer_Hierarchy ch ON sd.customer_id = ch.customer_id
LEFT JOIN Sales_Summary ss ON sd.customer_id = ss.customer_id AND ss.state = cd.state
GROUP BY c.customer_id, cs.state, cd.city;

-- Output Example for customer_id 1
+----------+--------+-----------+-------------+
| customer_id | state   | city      | total_sales |
+----------+--------+-----------+-------------+
|        1    | NY      | New York  |         3000 |
+----------+--------+-----------+-------------+
```

**Data:**

```json
{
  "salesSummary": [
    {"customer_id": 1, "state": "NY", "total_sales": 5000},
    {"customer_id": 2, "state": "CA"}
  ],
  "salesDetail": [
    {"sales_id": 1, "customer_id": 1, "city": "New York", "sale_amount": 3000},
    {"sales_id": 2, "customer_id": 2, "city": "Los Angeles", "sale_amount": 2000}
  ],
  "hierarchy": [
    {"customer_id": 1, "left_value": 1, "right_value": 4, "level": 1},
    {"customer_id": 1, "left_value": 5, "right_value": 8, "level": 2},
    {"customer_id": 2, "left_value": 9, "right_value": 10, "level": 1}
  ]
}
```

**Flow:**

1. No
2. Yes
3. Start
4. Is the hierarchy ragged?
5. Use traditional star schema
6. Implement nested sets andflexible fact tables
7. Query for drill-down
8. Return detailed data asneeded
9. End

