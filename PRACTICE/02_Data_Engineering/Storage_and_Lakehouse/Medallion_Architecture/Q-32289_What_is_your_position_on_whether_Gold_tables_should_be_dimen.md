---
type: "practice"
difficulty: "Deep"
---

# What is your position on whether Gold tables should be dimensional (star schema) or wide denormalized OBTs, and how does the consumption pattern change that decision at org scale?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Medallion Layer Quality](<../../../../02_Data_Engineering/Storage_and_Lakehouse/Medallion_Architecture/Medallion_Layer_Quality.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address this question, we need to delve into the core principles of Medallion Architecture and consider the trade-offs between dimensional (star schema) Gold tables and wide denormalized OBTs (Operational Business Tables). The decision hinges on the consumption pattern at an organizational scale. Let's break down these concepts step by step.

#### Triaging the Question

The question is asking for a strategic perspective on choosing between two types of Gold tables in Medallion Architecture—dimensional and wide denormalized OBTs—and how the consumption behavior influences this decision. We need to consider factors such as query performance, data freshness, schema complexity, and storage efficiency.

Key Assumptions
Medallion Architecture: A layered approach where Bronze (raw), Silver (aggregated), and Gold (analyzed) layers are used.
Gold Tables: Represent the final analysis results for reporting and analytics.

#### Cons
umption Pattern: The way different parts of the organization interact with these tables.

#### First-Principles Solution
Dimensional vs Wide Denormalized OBTs

Dimensional (Star Schema) Gold Tables:

#### Pros
Easier to understand and maintain.
Efficient for ad-hoc queries due to pre-aggregated facts and dimensions.
Good for reporting and analytics.

#### Cons
More complex schema, which can lead to increased storage costs if not optimized.
Can be slower for certain types of queries that require joining multiple tables.

Wide Denormalized OBTs:

#### Pros
Simpler schema with fewer joins required for complex queries.
Potentially faster query performance due to reduced join operations.
Suitable for scenarios where data freshness is critical and the denormalization reduces complexity.

#### Cons
More challenging to maintain and update, especially if business rules change frequently.
Risk of redundancy and increased storage costs.
Organizational Consumption Patterns
Reporting and Analytics Teams:
Typically benefit from dimensional Gold tables because they perform ad-hoc queries that require pre-aggregated data for quick insights.
Operational Teams:
May prefer wide denormalized OBTs due to their need for real-time or near-real-time access to the latest data, reducing latency in complex queries.
Detailed Analysis

Storage and Query Performance:

For organizations with large datasets and a high volume of reporting tasks, dimensional Gold tables are often preferred because they optimize query performance through pre-aggregation.
In contrast, wide denormalized OBTs can offer better performance for operational teams that require real-time data access.

Maintenance and Complexity:

Dimensional models require less maintenance as the schema is fixed, making them easier to manage.
Wide denormalized tables are more complex to maintain but may be necessary in scenarios where query performance is critical and data freshness takes precedence over schema simplicity.

Scalability and Cost Management:

Over time, the cost of maintaining dimensional models can increase due to storage requirements for multiple dimensions and facts.
Wide denormalized tables can reduce costs by minimizing join operations but may require more frequent updates to keep data fresh.

#### Closing Summary

Given the strategic importance of balancing query performance, data freshness, schema complexity, and maintenance in Medallion Architecture, organizations must carefully consider their consumption patterns. For reporting and analytics teams, dimensional Gold tables provide optimal query performance with minimal overhead. For operational teams requiring real-time access, wide denormalized OBTs are often more suitable despite the increased complexity.

#### Common Mistakes/Follow-Ups
Ignoring Organizational Needs: Failing to tailor the architecture to specific business requirements can lead to suboptimal performance and maintenance issues.
Overly Complex Schema: A overly complex schema for dimensional models without proper optimization can result in unnecessary storage costs.
Neglecting Data Freshness: Overemphasis on query performance at the expense of data freshness can undermine the usefulness of analytical outputs.

To make an informed decision, organizations should conduct a thorough analysis considering their specific use cases and consult with both reporting and operational teams to understand their unique needs.

**Comparison:**

| Bronze Layer | Raw Data                |
|--------------|-------------------------|
|              | - timestamp             |
|              | - event                  |
| Silver Layer | Aggregated Data         |
|--------------|-------------------------|
|              | - date                   |
|              | - totalPurchases         |
|              | - totalReturns           |
| Gold Layer Dimensional | Dimension Data          |
|------------------------|-------------------------|
|                        | - date                   |
|                        | - category               |
|                        | - amount                 |
| Gold Layer Wide Denormalized | Joined Data             |
|------------------------------|-------------------------|
|                              | - timestamp              |
|                              | - event                  |
|                              | - totalAmount            |

**Code:**

```
MERMAID
Copy
graph LR;
    A[Raw Data] --> B(Bronze Layer);
    B --> C(Silver Layer);
    C --> D(Gold Layer Dimensional);
    C --> E(Gold Layer Wide Denormalized);
    F(Reporting) --> D;
    G[Operational] --> E;
```

**Example input / output:**

```text
Input for Reporting:
- Query to calculate total purchases by category on October 1, 2023.

Output:
- {"category": "Electronics", "amount": 100}

Input for Operational:
- Query to get real-time total amount of all transactions within the last hour.

Output:
- {"timestamp": "2023-10-03T14:00:00Z", "totalAmount": 150}
```

**Data:**

```json
{
  "BronzeLayer": {
    "rawData": [
      {"timestamp": "2023-10-01", "event": "Purchase"},
      {"timestamp": "2023-10-02", "event": "Return"}
    ]
  },
  "SilverLayer": {
    "aggregatedData": [
      {"date": "2023-10-01", "totalPurchases": 10},
      {"date": "2023-10-02", "totalReturns": 5}
    ]
  },
  "GoldLayerDimensional": {
    "dimensionData": [
      {"date": "2023-10-01", "category": "Electronics", "amount": 100},
      {"date": "2023-10-02", "category": "Clothing", "amount": 50}
    ]
  },
  "GoldLayerWideDenormalized": {
    "joinedData": [
      {"timestamp": "2023-10-01", "event": "Purchase", "totalAmount": 100},
      {"timestamp": "2023-10-02", "event": "Return", "totalAmount": 50}
    ]
  }
}
```

**Flow:**

1. Raw Data
2. Bronze Layer
3. Silver Layer
4. Gold Layer Dimensional
5. Gold Layer Wide Denormalized
6. Reporting
7. Operational

