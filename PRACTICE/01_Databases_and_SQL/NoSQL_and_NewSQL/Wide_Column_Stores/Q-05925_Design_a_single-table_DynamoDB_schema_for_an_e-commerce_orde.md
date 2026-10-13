---
type: "practice"
difficulty: "Deep"
---

# Design a single-table DynamoDB schema for an e-commerce order system that must serve: get order by id, list orders by customer sorted by date, and list orders by status. Show the PK/SK and any GSIs.

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ DynamoDB Modeling and Capacity](<../../../../01_Databases_and_SQL/NoSQL_and_NewSQL/Wide_Column_Stores/DynamoDB_Modeling_and_Capacity/_index.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To design a single-table DynamoDB schema for an e-commerce order system that can efficiently handle getting an order by ID, listing orders by customer sorted by date, and listing orders by status, we need to carefully consider the primary key (PK), sort key (SK), and global secondary indexes (GSIs). The goal is to ensure that each operation has a cost-effective and performant access pattern.

#### Triaging the Question

The question asks for a single-table schema in DynamoDB. This means we are working within the constraints of a single table, which can be challenging but also provides an opportunity to design a highly optimized schema. The operations required are:

Get order by ID: This requires fast and direct access to individual orders.
List orders by customer sorted by date: This needs to support efficient range scans on the date.
List orders by status: This also should allow for quick filtering based on the order's status.
Key Assumptions
We will assume that the primary key (PK) is orderId and the sort key (SK) is a combination of customerId and orderDate.
The global secondary index (GSI) will help us efficiently list orders by customer or by status.
For simplicity, we will use the default partitioning scheme.

#### First-Principles Solution

Primary Key (PK/SK):

PK: orderId (string)
SK: customerId#orderDate (composite key consisting of customerId and orderDate)

Global Secondary Indexes (GSIs):

GSI 1: GSIByCustomer:
Partition Key: customerId
Sort Key: orderDate
GSI 2: GSIByStatus:
Partition Key: status
Sort Key: orderId
Detailed Breakdown

PK/SK Design:

Using the orderId as the PK ensures that each order can be uniquely identified and accessed directly.
The composite SK (customerId#orderDate) allows us to first filter by customer and then sort by date, making it efficient for listing orders by a specific customer.

GSI By Customer:

This GSI is useful when we need to list all the orders of a particular customer. It ensures that we can quickly start with a partition scan on customerId and then efficiently sort by orderDate.

GSI By Status:

When listing orders by status, this GSI allows for direct filtering by the status, followed by a range scan on orderId. This is particularly useful if there are different statuses like "shipped", "cancelled", etc.

#### Closing Summary
+ Common Mistakes/Follow-Ups

#### Closing Summary

We have designed a single-table schema with a primary key and two GSIs. The PK allows for fast direct access, while the GSIs provide efficient ways to list orders by customer or status.

#### Common Mistakes

Failing to consider sorting requirements during the design of SK.
Not understanding the limitations and costs associated with GSIs, such as read/write capacity and query optimization.

#### Follow-Ups
How would you handle concurrent writes to ensure data consistency?
What if we need to support additional operations like listing orders by product or customer location? Would changes be necessary in this schema?

**Comparison:**

| PK           | SK                               | GSIByCustomer        | GSIByStatus         |
|--------------|----------------------------------|----------------------|--------------------|
| orderId      | customerId#orderDate             | customerId, orderDate| status, orderId     |
| customer_abc | 1234567890#2023-09-01T12:00:00Z | customer_abc, 2023-09-01T12:00:00Z | shipped, 1234567890 |
| customer_def | 9876543210#2023-08-31T18:00:00Z  | customer_def, 2023-08-31T18:00:00Z | shipped, 9876543210 |
| customer_ghi | 7894561230#2023-09-02T14:30:00Z  | customer_ghi, 2023-09-02T14:30:00Z | shipped, 7894561230 |

**Code:**

```python
# Example of defining the table with PK and SK
table_name = "Orders"
create_table_expression = {
    "TableName": table_name,
    "KeySchema": [
        {"AttributeName": "orderId", "KeyType": "HASH"},
        {"AttributeName": "customerId#orderDate", "KeyType": "RANGE"}
    ],
    "AttributeDefinitions": [
        {"AttributeName": "orderId", "AttributeType": "S"},
        {"AttributeName": "customerId", "AttributeType": "S"},
        {"AttributeName": "orderDate", "AttributeType": "S"}
    ]
}
gsi_definitions = [
    {
        "IndexName": "GSIByCustomer",
        "KeySchema": [
            {"AttributeName": "customerId", "KeyType": "HASH"},
            {"AttributeName": "orderDate", "KeyType": "RANGE"}
        ],
        "Projection": {"ProjectionType": "ALL"}
    },
    {
        "IndexName": "GSIByStatus",
        "KeySchema": [
            {"AttributeName": "status", "KeyType": "HASH"},
            {"AttributeName": "orderId", "KeyType": "RANGE"}
        ],
        "Projection": {"ProjectionType": "ALL"}
    }
]
```

**Example input / output:**

```text
GET ORDER BY ID:
Input: Get order by orderId = 1234567890
Output: {
    "orderId": "1234567890",
    "customerId": "customer_abc",
    "orderDate": "2023-09-01T12:00:00Z",
    "status": "shipped",
    "items": ["item_A", "item_B"]
}

LIST ORDERS BY CUSTOMER SORTED BY DATE:
Input: List orders for customerId = customer_abc
Output: [
    {
        "orderId": "1234567890",
        "customerId": "customer_abc",
        "orderDate": "2023-09-01T12:00:00Z",
        "status": "shipped",
        "items": ["item_A", "item_B"]
    },
    {
        "orderId": "9876543210",
        "customerId": "customer_abc",
        "orderDate": "2023-08-31T18:00:00Z",
        "status": "delivered",
        "items": ["item_C", "item_D"]
    }
]

LIST ORDERS BY STATUS:
Input: List orders with status = shipped
Output: [
    {
        "orderId": "1234567890",
        "customerId": "customer_abc",
        "orderDate": "2023-09-01T12:00:00Z",
        "status": "shipped",
        "items": ["item_A", "item_B"]
    },
    {
        "orderId": "7894561230",
        "customerId": "customer_def",
        "orderDate": "2023-09-02T14:30:00Z",
        "status": "shipped",
        "items": ["item_E", "item_F"]
    }
]
```

**Data:**

```json
{
    "orders_table": {
        "PK": "orderId",
        "SK": "customerId#orderDate",
        "GSIByCustomer": {
            "PK": "customerId",
            "SK": "orderDate"
        },
        "GSIByStatus": {
            "PK": "status",
            "SK": "orderId"
        }
    }
}
```

**Flow:**

1. Get Order by ID
2. Primary Key Lookup
3. Order Details
4. List Orders by CustomerSorted by Date
5. Customer ID Partition Scan
6. Date Range Query
7. Order Details
8. List Orders by Status
9. Status Filter
10. Order Details

