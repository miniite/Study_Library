---
type: "practice"
difficulty: "Intermediate"
---

# How would you use SQL to check referential integrity between a fact and its dimension after a load?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Referential Integrity Checks](<../../../../01_Databases_and_SQL/Database_Design/Normalization/Referential_Integrity_Checks.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To ensure referential integrity between a fact table and its dimensions in a data warehouse, we need to verify that all foreign keys in the fact table are valid references to primary keys in their respective dimension tables. This is crucial for maintaining data consistency and accuracy.

The question asks how to use SQL to check this relationship after loading new data into the fact table. We will cover the steps from first principles: identifying the relevant tables, writing the SQL queries, and handling potential edge cases like missing or invalid references.

**Comparison:**

```
Dimension Table
Fact Table
customer_id
fact_orders
dim_customer
```

**Code:**

```

-- Sample schema for illustration CREATE TABLE dim_customer ( customer_id INT PRIMARY KEY, name VARCHAR(100), email VARCHAR(255) );

CREATE TABLE fact_orders ( order_id INT PRIMARY KEY, customer_id INT, product_id INT, quantity INT, FOREIGN KEY (customer_id) REFERENCES dim_customer(customer_id) );

CODE
Copy

```sql
-- Query to check referential integrity in the fact table
SELECT
    f.order_id,
    c.name AS customer_name,
    f.product_id,
    CASE
        WHEN c.customer_id IS NULL THEN 'Invalid Reference'
        ELSE 'Valid Reference'
    END AS ref_integrity_status
FROM
    fact_orders f
LEFT JOIN
    dim_customer c ON f.customer_id = c.customer_id;
SQL
Copy
-- Example of a faulty insert that would violate referential integrity
INSERT INTO fact_orders (order_id, customer_id, product_id, quantity) VALUES (101, 999, 201, 5);

-- Query to identify faulty records after the faulty insert
SELECT
    f.order_id,
    c.name AS customer_name,
    f.product_id,
    CASE
        WHEN c.customer_id IS NULL THEN 'Invalid Reference'
        ELSE 'Valid Reference'
    END AS ref_integrity_status
FROM
    fact_orders f
LEFT JOIN
    dim_customer c ON f.customer_id = c.customer_id;
```

**Example input / output:**

```text
-- Example input data after a faulty insert
fact_orders:
INSERT INTO fact_orders (order_id, customer_id, product_id, quantity) VALUES (101, 999, 201, 5);

-- Expected output to identify invalid references
+------------+-------------+-----------+----------------+
| order_id   | customer_name| product_id| ref_integrity_status |
+------------+-------------+-----------+----------------+
|       100  | Alice        |      201  | Valid Reference     |
|       101  | (null)      |      201  | Invalid Reference   |
+------------+-------------+-----------+----------------+
```

**Data:**

```json
{
  "fact_orders": [
    {"order_id": 100, "customer_id": 101, "product_id": 201, "quantity": 5},
    {"order_id": 101, "customer_id": 999, "product_id": 201, "quantity": 5}
  ],
  "dim_customer": [
    {"customer_id": 101, "name": "Alice", "email": "alice@example.com"},
    {"customer_id": 201, "name": "Bob", "email": "bob@example.com"}
  ]
}
```

**Flow:**

1. Yes
2. No
3. Start
4. Is the foreign key infact_orders a valid primary keyin dim_customer?
5. Record is Valid
6. Record is Invalid
7. Mark record as 'ValidReference'
8. Mark record as 'InvalidReference'
9. Check next row

