---
type: "practice"
difficulty: "Deep"
---

# How does MySQL decide whether to use an index or do a full table scan?

**Difficulty:** Deep · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ MySQL Optimizer Behavior](<../../../../TOOLS/MySQL/MySQL_Platform/MySQL_Optimizer_Behavior.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When deciding between using an index or performing a full table scan, MySQL employs a cost-based optimizer (CBO) that evaluates multiple factors to determine the most efficient execution plan. This decision is critical because it can significantly impact query performance.

The CBO considers several key aspects:

The size of the table.
The selectivity of the columns used in the index or where clause.
The cardinality (distinct values) of the columns being considered.
Whether a full table scan might be more efficient due to factors like fragmented data or low indexed column usage.

To walk through this step-by-step, let's start with some foundational concepts and then dive into how MySQL makes these decisions.

**Comparison:**

```
order_id	customer_id	product_id	order_date	quantity
1	2001	3001	2023-01-01	5

-- Index on customer_id and product_id CREATE INDEX idx_customer_product ON Orders (customer_id, product_id);
```

**Code:**

```sql
-- Example table schema for demonstration purposes
CREATE TABLE Orders (
    order_id INT PRIMARY KEY,
    customer_id INT NOT NULL,
    product_id INT NOT NULL,
    order_date DATE NOT NULL,
    quantity INT NOT NULL
);

-- Sample data inserted into the Orders table INSERT INTO Orders (order_id, customer_id, product_id, order_date, quantity) VALUES (1, 2001, 3001, '2023-01-01', 5), (2, 2002, 3001, '2023-02-01', 3), (3, 2003, 3002, '2023-03-01', 7);
```

**Example input / output:**

```text
-- Query using an indexed column to find orders for a specific customer_id and product_id
SELECT * FROM Orders WHERE customer_id = 2001 AND product_id = 3001;
```

**Flow:**

1. Yes
2. No
3. Yes
4. No
5. Start
6. Is the index selective enough?
7. Use index
8. Is a full table scan moreefficient?
9. Perform full table scan
10. Use covering index if available

