---
type: "practice"
difficulty: "Intermediate"
---

# What does the relationships test verify and how is it useful for data integrity?

**Difficulty:** Intermediate · **Levels:** Architect/Principal, Fresher, SDE-1/2, SDE-2/3 · [↑ dbt Tests](<../../../../TOOLS/dbt/dbt_Core/dbt_Tests.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

The relationships test in dbt verifies the cardinality of relationships between models, ensuring that foreign keys and primary keys are correctly maintained. This is crucial for maintaining data integrity across a data model. By verifying these relationships, dbt helps ensure that data remains consistent and accurate when dependencies exist.

To fully understand how the relationships test works and its utility, let's break down what it checks and why it matters:

What the Relationships Test Verifies:

The relationships test in dbt verifies that the foreign keys (FKS) in a child table match the primary keys (PKs) in a parent table.
It ensures that each row in the child model has exactly one corresponding row in the parent model. This is often referred to as "one-to-many" or "many-to-one" relationships.

#### Triage

The question probes your understanding of how dbt maintains data integrity and checks for referential integrity within a database.
You need to understand the concept of foreign keys, primary keys, and their importance in ensuring data consistency.
The test assumes that you have a basic understanding of relational databases and the roles of PKs and FKS.

#### First-Principles Solution

In a relationships test, dbt will run a SQL query to check if every row in the child model has a matching primary key in the parent model.
For example, if you have an orders table that depends on a customers table (assuming each order is linked to a customer), the relationships test would verify that for every order, there exists exactly one customer with whom it is associated.

Here's a simple example of how this could be implemented in SQL:

SQL
-- Example SQL query for a relationships test
SELECT
    COUNT(DISTINCT order_id) AS unique_orders,
    COUNT(DISTINCT customer_id) AS unique_customers
FROM orders
WHERE NOT EXISTS (
    SELECT 1 FROM customers WHERE id = orders.customer_id
);
The unique_orders count should match the unique_customers count to ensure referential integrity.
If there are any discrepancies, dbt will flag this as an error and report it in the test results.

#### Closing Summary
+ Common Mistakes/Follow-Ups:
The relationships test is crucial for maintaining data consistency and ensuring that your models adhere to the expected relationships.

#### Common mistakes
include not setting up primary keys and foreign keys correctly, or failing to run comprehensive tests like these in dbt.

In a real-world scenario, you might encounter more complex relationships, such as many-to-many or one-to-one. It's also important to consider performance implications when running large-scale relationships tests, ensuring that they are optimized for your specific database environment.

**Comparison:**

```
Orders Table	Customers Table
order_id	customer_id
1	2
3	4
```

**Code:**

```sql
-- Example SQL query for a relationships test
SELECT
    COUNT(DISTINCT order_id) AS unique_orders,
    COUNT(DISTINCT customer_id) AS unique_customers
FROM orders
WHERE NOT EXISTS (
    SELECT 1 FROM customers WHERE id = orders.customer_id
);
```

**Example input / output:**

```text
graph TD;
    A[Orders] --> B{Check Relationships};
    B --> C(Unique Orders = 2);
    C --> D[Customers];
    D --> E{Verify FKs};
    E --> F(All FKs match PKs);
```

**Data:**

```json
{
  "orders": [
    { "order_id": 1, "customer_id": 2 },
    { "order_id": 3, "customer_id": 4 }
  ],
  "customers": [
    { "id": 2, "name": "John" },
    { "id": 4, "name": "Jane" }
  ]
}
```

**Flow:**

1. Orders
2. Check Relationships
3. Unique Orders = 2
4. Customers
5. Verify FKs
6. All FKs match PKs

