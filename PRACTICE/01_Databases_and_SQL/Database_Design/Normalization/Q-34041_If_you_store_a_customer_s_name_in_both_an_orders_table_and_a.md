---
type: "practice"
difficulty: "Basic"
---

# If you store a customer's name in both an orders table and a customers table what problem can arise?

**Difficulty:** Basic · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Redundancy and Anomalies](<../../../../01_Databases_and_SQL/Database_Design/Normalization/Redundancy_and_Anomalies.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To understand why storing a customer's name in both an orders table and a customers table could be problematic, let’s start with a brief overview. Storing the same data redundantly across multiple tables can lead to inconsistencies and increased maintenance costs. In this case, if we store a customer’s name in two places, any changes made to one place might not be reflected in the other, leading to outdated or conflicting information.

#### Triaging the Question

This question is asking you to identify potential issues arising from redundancy in database design. Specifically, it highlights how storing the same data in multiple tables (in this case, orders and customers) can lead to inconsistencies and difficulties in maintaining a single source of truth.

#### First Principles Solution

Understanding the Problem:

Storing customer names redundantly means that any changes made to one name will not automatically propagate to the other.

Identifying the Consequences:

Inconsistent data across tables can lead to confusion and errors, such as different orders being associated with a customer under two different names.

Proper Normalization Approach:

To avoid these issues, it’s best practice to store customer information in one table (e.g., customers) and ensure that other related tables (orders, invoices, etc.) reference this single source of truth via foreign keys.

Denormalization Considerations:

While denormalization can improve performance by reducing joins, it should be done carefully to balance the need for speed against the risk of data inconsistencies.

#### Closing Summary
and Common Mistakes
Storing customer names redundantly is a common mistake that leads to data inconsistency.
The primary solution is to store all relevant information in one normalized table (customers), while other tables (orders, etc.) should reference this single source via foreign keys.
Denormalization can be used for performance optimization but requires careful management to avoid introducing more issues.

#### Common Mistakes
and Follow-Ups
Inconsistent Data: Not updating both tables when a customer’s name changes, leading to outdated or conflicting data.
Maintenance Overhead: Frequent updates in multiple places increase the risk of human error and maintenance overhead.
Performance vs. Consistency: Discussing whether denormalization is necessary for performance and understanding the trade-offs involved.

**Comparison:**

| Customers Table | Orders Table |
|-----------------|--------------|
| customer_id     | order_id     |
| first_name      | customer_id  |
| last_name       |              |

**Code:**

```sql
-- Example schema
CREATE TABLE customers (
    customer_id INT PRIMARY KEY,
    first_name VARCHAR(50),
    last_name VARCHAR(50)
);

CREATE TABLE orders (
    order_id INT PRIMARY KEY,
    customer_id INT,
    order_date DATE,
    FOREIGN KEY (customer_id) REFERENCES customers(customer_id)
);
```

**Example input / output:**

```text
Input:
UPDATE customers SET first_name = 'Johnny' WHERE customer_id = 1;

Expected Output after the update in `customers` table only:
- customers: [{"customer_id": 1, "first_name": "Johnny", "last_name": "Doe"}]
- orders: [{"order_id": 101, "customer_id": 1, "order_date": "2023-06-01"}, {"order_id": 102, "customer_id": 1, "order_date": "2023-06-05"}]

Actual Output if `orders` table was updated manually:
- customers: [{"customer_id": 1, "first_name": "Johnny", "last_name": "Doe"}]
- orders: [{"order_id": 101, "customer_id": 1, "order_date": "2023-06-01"}, {"order_id": 102, "customer_id": 1, "order_date": "2023-06-05"}]

Desired Output (correct):
- customers: [{"customer_id": 1, "first_name": "Johnny", "last_name": "Doe"}]
- orders: [{"order_id": 101, "customer_id": 1, "order_date": "2023-06-01"}, {"order_id": 102, "customer_id": 1, "order_date": "2023-06-05"}]
```

**Data:**

```json
{
  "customers": [
    { "customer_id": 1, "first_name": "John", "last_name": "Doe" },
    { "customer_id": 2, "first_name": "Jane", "last_name": "Smith" }
  ],
  "orders": [
    { "order_id": 101, "customer_id": 1, "order_date": "2023-06-01" },
    { "order_id": 102, "customer_id": 1, "order_date": "2023-06-05" }
  ]
}
```

**Flow:**

1. Yes
2. No
3. Customers Table
4. Update Customer Name?
5. Update Customers Table
6. Update Orders Table viaForeign Key
7. Do Nothing

