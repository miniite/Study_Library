---
type: "practice"
difficulty: "Deep"
---

# Your denormalized wide gold table becomes an update nightmare when a dimension changes. Where's the line between a defensible 'one big table' and a maintenance disaster, and how do you refresh it incrementally?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Layered Pipelines Bronze Silver Gold](<../../../../04_Data_Architecture_and_Strategy/Architecture_Patterns/Data_Architecture/Layered_Pipelines_Bronze_Silver_Gold/_index.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In data engineering, the tension between maintaining a single source of truth through a wide gold table and ensuring maintainability when dimensions change is a common challenge. The line between a defensible 'one big table' and a maintenance disaster lies in how you design your architecture to handle changes efficiently while minimizing operational overhead. In this scenario, we need to balance the benefits of a unified view with the practicalities of real-time data updates.

Firstly, let's clarify what we're dealing with: a wide gold table is essentially a denormalized fact table that includes all necessary dimensions and metrics. When dimensions change frequently, updating this single table can become a bottleneck, leading to performance degradation and increased complexity. The question at hand is how to strike the right balance between maintaining the simplicity of a unified view and ensuring efficient updates.

To solve this problem, we need to consider several factors:

Frequency of Dimension Changes: How often do dimensions change? If changes are infrequent, you might be able to live with periodic full table refreshes.
Impact on Performance: Frequent writes can degrade performance, especially in large tables. We need a strategy that minimizes the impact on read performance while handling write operations effectively.
Data Freshness Requirements: How critical is real-time data? If near-real-time updates are required, traditional full refresh methods won't cut it.
Step-by-Step Solution
1. Understand the Problem

We need to design a solution that allows us to efficiently update specific columns in our wide gold table when dimensions change, without having to rewrite the entire table. This can be achieved through incremental updates or by using a hybrid approach where we maintain multiple smaller tables and reconcile them periodically.

2. Design a Hybrid Approach

One effective way is to use a hybrid architecture that combines both a wide gold table and a set of more specialized tables:

Wide Gold Table: The primary source of truth, containing all necessary dimensions and metrics.
Dimension Tables: Separate tables for each dimension, updated independently when changes occur.
Incremental Update Mechanism: A process to reconcile the changes in the dimension tables back into the wide gold table.
3. Incremental Update Process
Capture Changes: Use an event sourcing or change data capture (CDC) mechanism to detect changes in dimensions.
Update Dimension Tables: When a change is detected, update the corresponding row(s) in the relevant dimension table.
Reconcile Changes: Periodically run a process that reads these changes and applies them to the wide gold table.
4. Implementation Example

Let's assume we have a wide gold table sales_data with dimensions like customer_id, product_id, and date. We also have separate dimension tables for each of these:

Dimension Tables:
customers
products
dates

When a customer changes their address, the customers table is updated. The incremental update process then triggers to reflect this change in the wide gold table.

5. Architecture Diagram
Sales Data
Customer Dim
Product Dim
Date Dim
Update Process

#### Closing Summary
and Common Mistakes

The key is to design a system that can handle incremental updates while maintaining the integrity of the wide gold table. Over-engineering this process, however, can lead to increased complexity and operational overhead.

#### Common mistakes
include:

Failing to implement an effective change detection mechanism.
Not periodically reconciling changes, leading to data inconsistencies.
Ignoring performance implications during updates.

By adopting a hybrid approach that leverages separate dimension tables for incremental updates, we can maintain the benefits of a wide gold table while ensuring efficient and robust maintenance.

**Comparison:**

| Customer Dim | Product Dim | Date Dim |
|--------------|-------------|----------|
| id | address     |         | id      | name       |        | id  | date            |
| 1    | New York    |          | 1      | Product A  |        | 1   | 2023-04-05     |
| 2    | Los Angeles |          | 2      | Product B  |        | 2   | 2023-04-06     |

**Code:**

```python
# Example Python code to capture and reconcile changes

def update_customer(dim_table, customer_id, new_address):
    # Update the customer's address in the dim_table
    pass

def reconcile_changes(gold_table, dim_tables):
    for table in dim_tables:
        # Read changes from each dim_table and apply them to gold_table
        pass
```

**Example input / output:**

```text
Input:
- Customer with id=1 has their address updated to 'Chicago'.
Output:
- The `customers` table is updated.
- The reconciliation process runs and updates the `sales_data` wide gold table accordingly.

Request:
- A customer's address change notification.
Response:
- The `customers` table is updated, and a reconciliation task starts.
```

**Data:**

```json
{
  "sales_data": {
    "customer_id": 1,
    "product_id": 2,
    "date": "2023-04-05",
    "amount": 100.00
  },
  "customers": [
    {"id": 1, "address": "New York"},
    {"id": 2, "address": "Los Angeles"}
  ],
  "products": [
    {"id": 1, "name": "Product A"},
    {"id": 2, "name": "Product B"}
  ]
}
```

**Flow:**

1. Sales Data
2. Customer Dim
3. Product Dim
4. Date Dim
5. Update Process
6. Change Detected
7. Update Customer Dim
8. Reconcile Changes
9. Apply to Sales Data

