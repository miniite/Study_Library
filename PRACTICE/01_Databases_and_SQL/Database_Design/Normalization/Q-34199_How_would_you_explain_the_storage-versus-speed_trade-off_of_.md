---
type: "practice"
difficulty: "Intermediate"
---

# How would you explain the storage-versus-speed trade-off of denormalization using a concrete pipeline example?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Denormalization Trade-offs and Patterns](<../../../../01_Databases_and_SQL/Database_Design/Normalization/Denormalization_Trade-offs_and_Patterns/_index.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Denormalization in database design is often used to improve query performance by reducing the number of joins required, but this comes at the cost of increased storage requirements and potential data redundancy. Let's explore a concrete pipeline example to illustrate this trade-off.

First, let’s clarify what we mean by denormalization versus normalization:

Normalization aims to reduce redundancy and improve data integrity.
Denormalization, on the other hand, introduces redundancy for performance gains.

Now, consider an E-commerce application with a simplified schema. We have two tables: Orders and Order_Items.

Original Normalized Schema
SQL
CREATE TABLE Orders (
    order_id INT PRIMARY KEY,
    customer_id INT NOT NULL,
    order_date DATE NOT NULL,
    total_amount DECIMAL(10, 2) NOT NULL
);

CREATE TABLE Order_Items (
    order_item_id INT PRIMARY KEY,
    order_id INT NOT NULL,
    product_id INT NOT NULL,
    quantity INT NOT NULL,
    price DECIMAL(10, 2) NOT NULL,
    FOREIGN KEY (order_id) REFERENCES Orders(order_id)
);
Denormalized Schema

To denormalize this schema for better performance, we can add the product_id and price fields directly into the Orders table. This reduces the need to join with the Order_Items table when retrieving order details.

SQL
CREATE TABLE Orders (
    order_id INT PRIMARY KEY,
    customer_id INT NOT NULL,
    order_date DATE NOT NULL,
    total_amount DECIMAL(10, 2) NOT NULL,
    product_id INT NOT NULL,   -- Denormalized field
    quantity INT NOT NULL     -- Denormalized field
);

#### Trade-Offs
Storage vs. Speed
Increased Storage Requirements

By denormalizing the schema as shown above, we are storing the same data (product ID and quantity) in every row of the Orders table. This can significantly increase storage requirements, especially if orders often contain many items.

For example, let’s assume each order contains 5 products on average:

Without denormalization, a single row in Order_Items would suffice for each order.
With denormalization, we store 5 product IDs and quantities per order, leading to increased storage.
Improved Query Performance

However, this trade-off is often worth it because queries can now be executed faster. For instance, when retrieving the details of an order, there's no need to perform a join with Order_Items, which can significantly speed up the query execution time.

#### Example Scenario

Imagine we have 10,000 orders and each order on average contains 5 products:

Original Normalized Schema: 10,000 rows in Orders + 50,000 rows in Order_Items
Denormalized Schema: 10,000 rows in Orders, but each row now stores product ID and quantity.

If we need to fetch order details for a specific customer:

In the normalized schema: SELECT * FROM Orders JOIN Order_Items ON ... (potentially complex query).
In the denormalized schema: A simple SELECT * FROM Orders WHERE customer_id = X.

#### Closing Summary

Denormalization trades increased storage requirements for improved query performance. While this approach can be beneficial in certain high-performance scenarios, it’s crucial to weigh these trade-offs based on specific use cases and application needs.

#### Common mistakes
include over-denormalizing without considering the long-term impact on data integrity and storage costs. Always validate your denormalized schema with proper testing to ensure that the benefits outweigh the drawbacks.

**Comparison:**

| Order ID | Customer ID | Order Date       | Total Amount | Product ID | Quantity |
|----------|-------------|-----------------|--------------|------------|---------|
| 1        | 101         | 2023-10-01      | 15.99        | 456789     | 3       |
| 2        | 102         | 2023-10-02      | 9.99         | 987654     | 2       |

**Code:**

```sql
-- Original Normalized Schema
CREATE TABLE Orders (
    order_id INT PRIMARY KEY,
    customer_id INT NOT NULL,
    order_date DATE NOT NULL,
    total_amount DECIMAL(10, 2) NOT NULL
);

CREATE TABLE Order_Items (
    order_item_id INT PRIMARY KEY,
    order_id INT NOT NULL,
    product_id INT NOT NULL,
    quantity INT NOT NULL,
    price DECIMAL(10, 2) NOT NULL,
    FOREIGN KEY (order_id) REFERENCES Orders(order_id)
);

-- Denormalized Schema
CREATE TABLE Orders (
    order_id INT PRIMARY KEY,
    customer_id INT NOT NULL,
    order_date DATE NOT NULL,
    total_amount DECIMAL(10, 2) NOT NULL,
    product_id INT NOT NULL,
    quantity INT NOT NULL
);
```

**Example input / output:**

```text
-- Normalized Query Example
SELECT o.order_id, c.customer_name, i.product_id, i.quantity
FROM Orders o
JOIN Customers c ON o.customer_id = c.customer_id
JOIN Order_Items i ON o.order_id = i.order_id
WHERE o.order_id = 1;

-- Denormalized Query Example
SELECT order_id, customer_id, order_date, total_amount, product_id, quantity
FROM Orders
WHERE order_id = 1;
```

**Data:**

```sql
INSERT INTO Orders (order_id, customer_id, order_date, total_amount, product_id, quantity)
VALUES
(1, 101, '2023-10-01', 15.99, 456789, 3),
(2, 102, '2023-10-02', 9.99, 987654, 2);
```

**Flow:**

1. FLOWCHART · MERMAID
2. graph LR
3. A[Query Execution]
4. B(Join Operation)
5. C(Denormalized Query)
6. D[Direct Row Fetch]
7. A -->|For Normalized Schema| B -->|Execution Time|
8. A -->|For Denormalized Schema| C --> D -->|Execution Time (Faster)|

