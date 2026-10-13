---
type: "practice"
difficulty: "Deep"
---

# Explain a case where you chose a snowflake schema over a star schema in production and how it affected join cost and BI usability.

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Distribution Keys](<../../../../02_Data_Engineering/Modeling_and_Warehousing/Data_Warehousing/Distribution_Keys.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When dealing with complex data models, choosing between a Snowflake Schema and a Star Schema often comes down to specific use cases and performance requirements. In one instance, I encountered a scenario where we opted for a Snowflake Schema over a Star Schema in production due to the need for more flexible query patterns and reduced join costs.

In this particular project, our dataset was highly normalized with many dimension tables that had complex relationships across them. The primary goal of the system was to support detailed financial reporting while also providing flexibility for advanced analytics and ad-hoc queries. A Star Schema would have worked well for basic reports but would struggle with more intricate analyses involving multiple levels of joins.

First, let's review the core concepts:

Star Schema: Typically used in data warehouses where fact tables are joined with dimension tables. It simplifies the structure by flattening out many-to-many relationships.
Snowflake Schema: A variation of the Star Schema that normalizes dimensions into smaller, related tables to reduce redundancy and improve data integrity.

Given our use case:

We needed a high level of query flexibility to handle complex reporting scenarios.
The system had to support both batch processing for generating detailed reports and real-time analytics.
There were extensive relationships between dimensions that required multiple levels of joins, which could be costly in a Star Schema.

#### Triaging the Question

The interviewer is probing your understanding of when to use each schema type and how they affect join costs and BI usability. Key assumptions include:

The data model has complex many-to-many relationships.
Performance and query flexibility are critical.
Real-time analytics and batch processing need to coexist efficiently.

#### First-Principles Solution

We chose a Snowflake Schema because it allowed us to maintain normalized dimension tables while minimizing the number of joins required for reporting. Here’s how this decision impacted join cost and BI usability:

Join Cost Reduction: In our Star Schema, we would have had many-to-many relationships that would translate into multiple levels of joins between fact and dimension tables. This could lead to performance degradation as more joins increase query latency. By normalizing dimensions, the Snowflake Schema reduced the number of necessary joins, improving overall query performance.

BI Usability: The normalized structure in a Snowflake Schema meant that dimension tables were smaller and more focused on specific attributes. This made it easier to manage large datasets without bloating any single table. Additionally, using partitioning strategies and proper indexing could further optimize queries for both reporting and analytics.

#### Example Scenario

#### Cons
ider a scenario where we have multiple dimensions such as Customer, Product, and Time. A Star Schema might look like this:

CODE
Fact_Sales -> Customer Dim -> Product Dim -> Time Dim

In contrast, the Snowflake Schema would normalize these relationships into smaller, related tables:

CODE
Fact_Sales -> (Customer Dim) -> Subdimension1 (e.g., Geography) -> Subdimension2 (e.g., Demographics)
            -> (Product Dim) -> Subdimension3 (e.g., Category) -> Subdimension4 (e.g., Supplier)
            -> Time Dim

#### Closing Summary
and Common Mistakes/Follow-Ups

In this case, the Snowflake Schema provided better performance for complex queries by reducing join costs while maintaining data integrity. It also improved usability by allowing more targeted indexing and partitioning strategies.

#### Common mistakes
in choosing a schema:

Over-normalizing to an extent that it complicates query execution.
Underestimating the impact of joins on performance, especially with complex relationships.
Failing to consider the trade-offs between normalization and query flexibility.

For follow-ups, you might explore how different indexing strategies could further enhance performance or discuss the potential downsides of a fully normalized Snowflake Schema in terms of data redundancy and complexity.

**Comparison:**

```
Fact_Sales
Customer_Dim
Product_Dim
Time_Dim
Fact_Sales
Customer_Dim2
Subdimension1
Subdimension2
Subdimension3
Subdimension4
```

**Code:**

```
MERMAID
Copy
graph TD;
    Fact_Sales(Fact Sales) --> Customer Dim(Customer Dimension);
    Fact_Sales --> Product Dim(Product Dimension);
    Fact_Sales --> Time Dim(Time Dimension);

    Customer Dim --> Subdimension1(Subdimension 1 - Geography);
    Customer Dim --> Subdimension2(Subdimension 2 - Demographics);

    Product Dim --> Subdimension3(Subdimension 3 - Category);
    Product Dim --> Subdimension4(Subdimension 4 - Supplier);
```

**Example input / output:**

| Fact_Sales Query | Star Schema Joins | Snowflake Schema Joins |
|-----------------|------------------|-----------------------|
| SELECT * FROM Fact_Sales WHERE customer_id = 123 AND product_id = 456 AND time_id = 789; | 1 Join to Customer Dim, 1 Join to Product Dim, 1 Join to Time Dim | 3 Joins (to Subdimension tables) |

**Data:**

```yaml
# Example Data Schema for Star and Snowflake Schemas

star_schema:
  tables:
    - name: Fact_Sales
      fields:
        - fact_id: integer
        - customer_id: integer
        - product_id: integer
        - time_id: integer
    - name: Customer_Dim
      fields:
        - dim_id: integer
        - first_name: string
        - last_name: string
        - address: string

snowflake_schema:
  tables:
    - name: Fact_Sales
      fields:
        - fact_id: integer
        - customer_id: integer
        - product_id: integer
        - time_id: integer
    - name: Customer_Dim
      fields:
        - dim_id: integer
        - first_name: string
        - last_name: string
        - address: string
    - name: Subdimension1
      fields:
        - subdim_id: integer
        - geography_name: string
    - name: Subdimension2
      fields:
        - subdim_id: integer
        - demographic_name: string
```

**Flow:**

1. Fact_Sales
2. Customer_Dim
3. Product_Dim
4. Time_Dim
5. Fact_Sales
6. Customer_Dim2
7. Subdimension1
8. Subdimension2
9. Subdimension3
10. Subdimension4

