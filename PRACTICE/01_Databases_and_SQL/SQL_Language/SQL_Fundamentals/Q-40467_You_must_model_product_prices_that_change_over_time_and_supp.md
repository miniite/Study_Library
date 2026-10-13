---
type: "practice"
difficulty: "Deep"
---

# You must model product prices that change over time and support 'price as of order date'. Compare a price-history table with valid ranges joined by date-containment vs. denormalizing the price onto each order. Trade-offs on correctness and reporting.

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Joining Type 2 Dimensions](<../../../../01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Joining_Type_2_Dimensions.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To model product prices that change over time, we can use two approaches: a price-history table with valid ranges joined by date-containment or denormalizing the price onto each order. The choice between these methods involves significant trade-offs in terms of correctness and reporting capabilities.

#### Triage

The core issue here is to understand the differences between storing historical prices separately from orders using a price-history table and embedding the latest known price directly within the order records. The interviewer will be probing your understanding of data modeling principles, normalization, and performance considerations.

#### Assumptions
We have an ongoing product inventory with varying prices over time.
Prices can change multiple times for each product on different dates.
Orders are placed at various points in time.

#### First-Principles Solution
1. Price-History Table Approach

In this approach, we maintain a prices table that tracks historical price changes and a orders table that references these prices by date.

Table Schema:

SQL
CREATE TABLE products (
    product_id INT PRIMARY KEY,
    name VARCHAR(255)
);

CREATE TABLE prices (
    price_id INT PRIMARY KEY AUTO_INCREMENT,
    product_id INT,
    effective_date DATE NOT NULL,
    price DECIMAL(10, 2),
    FOREIGN KEY (product_id) REFERENCES products(product_id)
);

Orders Table:

SQL
CREATE TABLE orders (
    order_id INT PRIMARY KEY AUTO_INCREMENT,
    product_id INT,
    order_date DATE,
    valid_from DATE,
    valid_to DATE,
    price DECIMAL(10, 2),
    FOREIGN KEY (product_id) REFERENCES products(product_id)
);

Query to get the correct price 'as of' an order date:

SQL
SELECT
    p.name AS product_name,
    COALESCE(
        (SELECT price FROM orders o WHERE o.order_id = o_id AND o.valid_from <= order_date AND (o.valid_to > order_date OR o.valid_to IS NULL) ORDER BY valid_to DESC LIMIT 1),
        (SELECT price FROM prices pr WHERE pr.product_id = p_id AND effective_date <= order_date ORDER BY effective_date DESC LIMIT 1)
    ) AS price_as_of_order
FROM
    orders o,
    products p;
2. Denormalized Prices on Each Order Approach

In this approach, the orders table stores the latest known price for each product at the time of order placement.

Table Schema:

SQL
CREATE TABLE products (
    product_id INT PRIMARY KEY,
    name VARCHAR(255)
);

CREATE TABLE orders (
    order_id INT PRIMARY KEY AUTO_INCREMENT,
    product_id INT,
    order_date DATE,
    latest_price DECIMAL(10, 2),
    FOREIGN KEY (product_id) REFERENCES products(product_id)
);

Query for 'price as of' an order date:

SQL
SELECT
    p.name AS product_name,
    o.latest_price AS price_as_of_order
FROM
    orders o,
    products p;

#### Closing Summary
and Common Mistakes/Follow-Ups

The key trade-offs involve correctness, performance, and maintenance:

Correctness: The prices table ensures that the reported prices are always accurate by referencing historical data. Denormalizing can lead to inconsistencies if prices change after an order is placed.

Performance: Queries on the orders table with a denormalized price might be simpler but could become complex when dealing with large datasets and frequent updates.

#### Common Mistakes
Overlooking Edge Cases: Failing to handle scenarios where no valid price exists before or after the order date.
Ignoring Maintenance Costs: The prices table approach requires regular updates, which can be costly in terms of time and resources.

#### Follow-Ups
Discuss the implications on data volume and query performance as the system scales.

#### Cons
ider caching strategies to optimize frequently accessed queries.
Explore indexing techniques that could improve query performance without sacrificing too much storage efficiency.

**Comparison:**

```
FLOWCHART · MERMAID
erDiagram
    PRODUCTS ||--o{ PRICES : has
    ORDERS ||--| PRICES : references
    PRICES {
        int price_id PK
        int product_id FK
        date effective_date
        decimal(10, 2) price
    }
    ORDERS {
        int order_id PK
        int product_id FK
        date order_date
        date valid_from
        date valid_to
        decimal(10, 2) price
    }
```

**Code:**

```sql
-- Price-History Table Schema
CREATE TABLE products (
    product_id INT PRIMARY KEY,
    name VARCHAR(255)
);

CREATE TABLE prices (
    price_id INT PRIMARY KEY AUTO_INCREMENT,
    product_id INT,
    effective_date DATE NOT NULL,
    price DECIMAL(10, 2),
    FOREIGN KEY (product_id) REFERENCES products(product_id)
);

-- Orders Table Schema
CREATE TABLE orders (
    order_id INT PRIMARY KEY AUTO_INCREMENT,
    product_id INT,
    order_date DATE,
    valid_from DATE,
    valid_to DATE,
    price DECIMAL(10, 2),
    FOREIGN KEY (product_id) REFERENCES products(product_id)
);
```

**Example input / output:**

```text
-- Input for 'price as of' an order date
SELECT
    p.name AS product_name,
    COALESCE(
        (SELECT price FROM orders o WHERE o.order_id = 1 AND o.valid_from <= '2023-03-01' AND (o.valid_to > '2023-03-01' OR o.valid_to IS NULL) ORDER BY valid_to DESC LIMIT 1),
        (SELECT price FROM prices pr WHERE pr.product_id = 1 AND effective_date <= '2023-03-01' ORDER BY effective_date DESC LIMIT 1)
    ) AS price_as_of_order
FROM
    orders o,
    products p;

-- Output for order with ID 1 (as of 2023-03-01)
Product Name: Laptop
Price as of Order Date: $985.99

-- Input for 'price as of' an order date
SELECT
    p.name AS product_name,
    o.latest_price AS price_as_of_order
FROM
    orders o,
    products p;

-- Output for order with ID 2 (as of 2023-02-15)
Product Name: Smartphone
Price as of Order Date: $599.99
```

**Data:**

```json
{
  "products": [
    { "product_id": 1, "name": "Laptop" },
    { "product_id": 2, "name": "Smartphone" }
  ],
  "prices": [
    { "price_id": 1, "product_id": 1, "effective_date": "2023-01-01", "price": 999.99 },
    { "price_id": 2, "product_id": 1, "effective_date": "2023-04-01", "price": 975.99 },
    { "price_id": 3, "product_id": 2, "effective_date": "2023-01-01", "price": 599.99 }
  ],
  "orders": [
    { "order_id": 1, "product_id": 1, "order_date": "2023-03-01", "valid_from": "2023-01-01", "valid_to": "2023-04-01", "price": 985.99 },
    { "order_id": 2, "product_id": 2, "order_date": "2023-02-15" }
  ]
}
```

**Flow:**

1. Product ID
2. Price History Table
3. Orders Table with Latest Price
4. Query for Correct Prices 'as of'Order Date
5. Simple Query to Return LatestKnown Price

