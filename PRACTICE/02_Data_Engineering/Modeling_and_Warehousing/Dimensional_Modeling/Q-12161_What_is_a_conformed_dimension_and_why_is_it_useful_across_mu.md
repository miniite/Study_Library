---
type: "practice"
difficulty: "Deep"
---

# What is a conformed dimension and why is it useful across multiple fact tables?

**Difficulty:** Deep · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Fact and Dimension Keys](<../../../../02_Data_Engineering/Modeling_and_Warehousing/Dimensional_Modeling/Fact_and_Dimension_Keys.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

A conformed dimension in data modeling, especially within a star or snowflake schema, refers to a shared dimension table that is used by multiple fact tables. This concept is particularly valuable when dealing with large datasets where the same set of dimensions (such as customer information, product details, time periods) appears across various fact tables. By using conformed dimensions, we can ensure data consistency and reduce redundancy.

#### Triaging the Question

The question asks you to explain what a conformed dimension is and why it's useful in multiple fact tables. The core of this concept lies in the sharing of common dimensional attributes across different fact tables, thereby improving data quality and efficiency. We need to consider the implications on database design, query performance, and maintenance.

#### First-Principles Solution

A conformed dimension is a dimension table that contains attributes relevant to multiple fact tables. This approach ensures consistency by maintaining a single source of truth for shared dimensions. Here’s why this concept is useful:

Data Consistency: Using the same data across different fact tables means there's only one version of each attribute, reducing inconsistencies and ensuring accuracy.
Reduced Redundancy: By sharing common dimension tables, you avoid duplicating data in multiple places, making your database more efficient and easier to maintain.
Improved Query Performance: With a single source for shared dimensions, queries can be written more efficiently, as they don’t need to join the same attributes repeatedly.
Detailed Explanation

Let's break down why conformed dimensions are beneficial:

Data Consistency:
In a retail application, you might have multiple fact tables like sales, returns, and exchanges. If these fact tables all contain customer information (like name, address, phone number), having a single conformed dimension for customer data ensures that each record is consistent across all fact tables.
Reduced Redundancy:
Imagine you have 10 fact tables, each with its own copy of the same customer attributes. If the customer information changes (e.g., an address update), this change must be propagated to every fact table, which can become cumbersome and error-prone. With a conformed dimension, this change is made once in one place.
Improved Query Performance:
When querying across multiple fact tables that share common dimensions, you only need to join the shared dimension table once. This reduces the complexity of queries and speeds up execution time.
Practical Example

#### Cons
ider an e-commerce application with two fact tables: Sales and Returns. Both fact tables have customer information like CustomerID, Name, and Phone.

Without Conformed Dimension:

Each fact table would contain its own copy of the same attributes. Changing a customer's phone number would require updating it in both fact tables.

With Conformed Dimension:

You create a conformed dimension called Customers with columns like CustomerID, Name, and Phone. Both Sales and Returns reference this single table, ensuring that any update to customer information is reflected consistently across all related facts.

#### Closing Summary

Conformed dimensions enhance data consistency, reduce redundancy, and improve query performance. By centralizing shared attributes in a single dimension table, you simplify database maintenance and optimize query efficiency.

#### Common Mistakes
and Follow-Ups
Mistake: Failing to recognize the importance of maintaining a single source of truth for shared dimensions.

#### Follow-Up
Discussing scenarios where conformed dimensions might not be necessary or where they could lead to complexity (e.g., when dealing with highly specific, localized data).

**Comparison:**

```
table Customers {
    CustomerID (INT)
    Name (VARCHAR(100))
    Phone (VARCHAR(20))
}
table Sales {
    SaleID (INT)
    CustomerID (INT, FK -> Customers.CustomerID)
    Amount (DECIMAL)
}
```

**Code:**

```sql
-- Example SQL for creating a conformed dimension table

CREATE TABLE Customers (
    CustomerID INT PRIMARY KEY,
    Name VARCHAR(100),
    Phone VARCHAR(20)
);
```

**Example input / output:**

```text
-- Example input and output for joining fact tables with a conformed dimension

Input:
Sales Table (Fact Table)
+-------------------+------------+-----------+
| SaleID            | CustomerID | Amount    |
+-------------------+------------+-----------+
| 100               | 1          | 150.00    |
| 200               | 3          | 75.00     |
+-------------------+------------+-----------+

Customers Table (Conformed Dimension)
+-------------+--------+----------+
| CustomerID  | Name   | Phone    |
+-------------+--------+----------+
| 1           | John   | 555-1234 |
| 2           | Jane   | 555-9876 |
+-------------+--------+----------+

Output (after joining):
+-------------------+------------+-----------+---------+----------+
| SaleID            | CustomerID | Amount    | Name    | Phone    |
+-------------------+------------+-----------+---------+----------+
| 100               | 1          | 150.00    | John    | 555-1234 |
+-------------------+------------+-----------+---------+----------+
```

**Data:**

```json
{
  "Customers": [
    {"CustomerID": 1, "Name": "John Doe", "Phone": "555-1234"},
    {"CustomerID": 2, "Name": "Jane Smith", "Phone": "555-9876"}
  ]
}
```

**Flow:**

1. Sales Table
2. CustomerID
3. Conformed Dimension -Customers

