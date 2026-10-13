---
type: "practice"
difficulty: "Deep"
---

# What is the difference between a snapshot of current state and a full transaction history in fact design?

**Difficulty:** Deep · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Transaction vs Snapshot Facts](<../../../../02_Data_Engineering/Modeling_and_Warehousing/Dimensional_Modeling/Transaction_vs_Snapshot_Facts.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In data modeling, particularly when designing fact tables, the choice between using a snapshot of the current state versus maintaining a full transaction history is crucial. This decision impacts the integrity, performance, and scalability of your database design.

#### Triage

The question asks us to differentiate between two approaches in fact table design: storing only the most recent state (snapshot) or keeping every transaction record that affects the fact data (full transaction history). The interviewer is probing our understanding of how these choices affect data integrity, query performance, and storage requirements. We need to consider assumptions such as the nature of the business processes being modeled, regulatory requirements, and performance constraints.

#### First-Principles Solution

Snapshot of Current State:

Definition: This approach stores only one row per fact record, representing its current state.

#### Pros
Reduced storage footprint since each dimension is stored only once.
Simpler queries as they do not need to join multiple transaction records.

#### Cons
Can lead to potential loss of historical context if the data changes over time without a rollback mechanism.
May require additional logic to handle updates, such as marking rows as “archived” or using a different table for current states.

Full Transaction History:

Definition: This approach stores every transaction that affects the fact record, providing a complete history of changes.

#### Pros
Preserves historical context and data lineage, which can be crucial for auditing, trend analysis, and business intelligence reporting.
No need to implement complex mechanisms for handling updates or rollbacks.

#### Cons
Increased storage requirements as more transaction records are stored.
More complex queries due to the need to join multiple rows representing different states over time.

#### Closing Summary

In summary, choosing between a snapshot and full transaction history depends on the specific use cases of your application. A snapshot is ideal for applications that prioritize performance and reduced storage costs but sacrifice historical context. Full transaction history, while requiring more resources, ensures data integrity and provides detailed insights into how facts change over time.

#### Common Mistakes/Follow-Ups
Mistake: Ignoring the long-term implications of choosing a snapshot approach when regulatory or auditing requirements demand historical data.

#### Follow-Up
How do you handle updates in a transaction history model without causing duplication issues? Consider using techniques like surrogate keys, versioning, and tombstone rows.

**Comparison:**

| Fact Table Design          | Snapshot of Current State               | Full Transaction History                     |
|---------------------------|-----------------------------------------|---------------------------------------------|
| Data Storage              | One row per fact, representing the current state | One row per transaction, preserving historical context |
| Storage Footprint         | Lower due to fewer rows                  | Higher due to more detailed records          |
| Query Complexity          | Simpler as it does not require joins     | More complex due to multiple join operations|
| Historical Integrity      | Potentially compromised without rollback mechanisms | Maintained by storing all changes            |

**Code:**

```sql
-- Example SQL for full transaction history fact table

CREATE TABLE sales_history (
    sale_id INT PRIMARY KEY,
    product_id INT NOT NULL,
    customer_id INT NOT NULL,
    quantity INT NOT NULL,
    price DECIMAL(10, 2) NOT NULL,
    sale_date TIMESTAMP NOT NULL,
    transaction_type VARCHAR(50) CHECK (transaction_type IN ('sale', 'return'))
);

-- Example SQL for snapshot fact table

CREATE TABLE current_sales (
    product_id INT PRIMARY KEY REFERENCES products(id),
    customer_id INT PRIMARY KEY REFERENCES customers(id),
    total_value DECIMAL(18, 2)
);
```

**Example input / output:**

```text
Input for full transaction history:

INSERT INTO sales_history (sale_id, product_id, customer_id, quantity, price, sale_date, transaction_type) VALUES
(1, 101, 501, 2, 9.99, '2023-07-01', 'sale'),
(2, 102, 502, 4, 19.99, '2023-07-01', 'return');

Output:

SELECT * FROM sales_history;

+---------+------------+-----------+----------+--------+---------------------+-------------+
| sale_id | product_id | customer_id | quantity | price  | sale_date           | transaction_type |
+---------+------------+-----------+----------+--------+---------------------+-------------+
|       1 |        101 |         501 |        2 |    9.99 | 2023-07-01 00:00:00 | sale          |
|       2 |        102 |         502 |        4 |   19.99 | 2023-07-01 00:00:00 | return         |
+---------+------------+-----------+----------+--------+---------------------+-------------+

Input for snapshot:

INSERT INTO current_sales (product_id, customer_id, total_value) VALUES
(101, 501, 19.98),
(102, 502, 79.96);

Output:

SELECT * FROM current_sales;

+------------+-----------+----------+
| product_id | customer_id | total_value |
+------------+-----------+----------+
|        101 |         501 |       19.98 |
|        102 |         502 |       79.96 |
+------------+-----------+----------+
```

**Data:**

```json
{
  "sales_history": [
    {
      "sale_id": 1,
      "product_id": 101,
      "customer_id": 501,
      "quantity": 2,
      "price": 9.99,
      "sale_date": "2023-07-01",
      "transaction_type": "sale"
    },
    {
      "sale_id": 2,
      "product_id": 102,
      "customer_id": 502,
      "quantity": 4,
      "price": 19.99,
      "sale_date": "2023-07-01",
      "transaction_type": "return"
    }
  ],
  "current_sales_snapshot": [
    {
      "product_id": 101,
      "customer_id": 501,
      "total_value": 19.98
    },
    {
      "product_id": 102,
      "customer_id": 502,
      "total_value": 79.96
    }
  ]
}
```

**Flow:**

1. Yes
2. No
3. Transaction Occurs
4. Is it a critical transaction?
5. Store in Full TransactionHistory
6. Store in Snapshot of CurrentState
7. Ensure Historical IntegrityMechanisms
8. Monitor for Data Changes

