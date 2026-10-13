---
type: "practice"
difficulty: "Basic"
---

# What is an orphan record and why is it a data quality problem?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Data Quality Fundamentals](<../../../../02_Data_Engineering/Data_Quality/Validation/Data_Quality_Fundamentals.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

An orphan record in the context of data quality refers to a piece of data that exists independently without corresponding relationships, often leading to inconsistencies or incomplete datasets. Understanding why orphan records are problematic requires delving into the basics of data integrity and how it impacts downstream processes.

#### Triaging the Question

The interviewer is probing your understanding of key data quality principles such as referential integrity and the importance of maintaining consistent and accurate data relationships. You need to explain what orphan records are, their causes, and why they pose a risk to data reliability.

Step-by-Step Solution from First Principles
Definition

Orphan records occur when a record in one dataset does not have a corresponding entry in another related dataset. For instance, consider an Order table and a Customer table. An order might exist without a matching customer ID, making the order data incomplete or unreliable.

Causes
Data Ingestion Issues: Data may be ingested from multiple sources at different times, leading to inconsistencies.
Manual Errors: During manual entry processes, records might be created in one dataset without being reflected in another.
System Glitches: Technical glitches can cause records to be created or lost during data synchronization.
Impact on Data Quality
Incomplete Insights: Orphan records can lead to skewed analysis and inaccurate insights. For example, if a customer's order is missing their name, financial reports might misrepresent the total revenue.
Operational Hiccups: Incomplete data can cause issues in downstream systems that rely on comprehensive information for operations such as billing or inventory management.
Legal & Compliance Issues: In regulated industries, incomplete records could lead to non-compliance and potential legal repercussions.

#### Closing Summary

Orphan records are a critical issue in maintaining high-quality datasets because they introduce inconsistencies and gaps. Proper data validation and cleansing processes should be implemented to ensure referential integrity and consistency across related tables.

#### Common Mistakes
and Follow-Ups
Failing to Consider All Tables: Often, developers focus only on the main table and overlook other related tables that might contain orphan records.
Not Implementing Data Validation: Basic validation checks can help prevent or detect orphan records early in the data pipeline. For example, using primary keys and foreign key constraints in relational databases.
Overlooking Historical Data: Orphan records can persist over time unless historical datasets are regularly cleaned.

**Comparison:**

```
Order Table
Customer Table
Orphan Record
```

**Code:**

```python
# Example Python code for basic validation
def validate_records(order_data, customer_data):
    # Assuming order_data is a list of dictionaries and customer_data is a DataFrame
    orphans = []
    for order in order_data:
        if order['customer_id'] not in customer_data.index:
            orphans.append(order)
    return orphans

# Sample data
order_data = [
    {'id': 1, 'product': 'Laptop', 'customer_id': 10},
    {'id': 2, 'product': 'Mouse', 'customer_id': None}, # Orphan record here
]

customer_data = pd.DataFrame({
    'id': [10, 20],
    'name': ['Alice', 'Bob']
})

orphans = validate_records(order_data, customer_data)
print(orphans)  # Output: [{'id': 2, 'product': 'Mouse', 'customer_id': None}]
```

**Example input / output:**

```text
Input:
order_data = [{"id": 1, "product": "Laptop", "customer_id": 10}, {"id": 2, "product": "Mouse", "customer_id": null}]
customer_data = pd.DataFrame({"id": [10, 20], "name": ["Alice", "Bob"]})

Output:
orphans = [{"id": 2, "product": "Mouse", "customer_id": null}]
```

**Data:**

```json
{
  "order_data": [
    {"id": 1, "product": "Laptop", "customer_id": 10},
    {"id": 2, "product": "Mouse", "customer_id": null}
  ],
  "customer_data": {
    "id": [10, 20],
    "name": ["Alice", "Bob"]
  }
}
```

**Flow:**

1. digraph {
2. rankdir=LR;
3. subgraph cluster_order {
4. style=filled;
5. color=lightgrey;
6. "Order Data" [shape=record, label="<i>Order Data</i>|{id: 1 | product: Laptop | customer_id: 10}<br/>|{id: 2 | product: Mouse | customer_id: null}"];
7. }
8. subgraph cluster_customer {
9. style=filled;
10. color=lightgrey;
11. "Customer Data" [shape=record, label="<i>Customer Data</i>|{id: 10 | name: Alice}<br/>|{id: 20 | name: Bob}"];
12. }
13. OrderData->CustomerData
14. }

