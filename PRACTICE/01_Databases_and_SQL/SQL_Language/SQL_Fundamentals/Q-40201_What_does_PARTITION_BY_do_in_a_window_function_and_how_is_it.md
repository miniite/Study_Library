---
type: "practice"
difficulty: "Intermediate"
---

# What does PARTITION BY do in a window function, and how is it different from GROUP BY?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ GROUP BY vs PARTITION BY](<../../../../01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/GROUP_BY_vs_PARTITION_BY.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In SQL, PARTITION BY is used within window functions to divide data into partitions based on one or more expressions. This allows you to perform calculations across a subset of rows that share common values in the partitioning column(s), without affecting other partitions. In contrast, GROUP BY groups all selected columns and computes aggregate values over the entire group.

To understand this better, let’s break it down step by step from first principles.

**Comparison:**

| order_id | customer_id | quantity |
|----------|-------------|----------|
|     101  |        202   |       5  |
|     102  |        203   |       3  |
|     103  |        202   |       7  |

**Code:**

```sql
-- Example with GROUP BY
SELECT order_id, SUM(quantity) AS total_quantity
FROM orders
GROUP BY order_id;

-- Example with PARTITION BY in a window function
SELECT order_id, SUM(quantity) OVER (PARTITION BY customer_id) as total_customer_quantity
FROM orders;
```

**Example input / output:**

```

For the given data:

JSON
Copy
{
  "orders": [
    {"order_id": 101, "customer_id": 202, "quantity": 5},
    {"order_id": 102, "customer_id": 203, "quantity": 3},
    {"order_id": 103, "customer_id": 202, "quantity": 7}
  ]
}

Using GROUP BY:

SQL
Copy
SELECT order_id, SUM(quantity) AS total_quantity
FROM orders
GROUP BY order_id;

Would result in:

MARKDOWN
Copy
| order_id | total_quantity |
|----------|----------------|
|     101  |          5     |
|     102  |          3     |
|     103  |          7     |

Using PARTITION BY with a window function:

SQL
Copy
SELECT order_id, SUM(quantity) OVER (PARTITION BY customer_id) as total_customer_quantity
FROM orders;

Would result in:

MARKDOWN
Copy
| order_id | total_customer_quantity |
|----------|------------------------|
|     101  |                 12      |
|     102  |                 12      |
|     103  |                 12      |
```

**Data:**

```json
{
  "orders": [
    {"order_id": 101, "customer_id": 202, "quantity": 5},
    {"order_id": 102, "customer_id": 203, "quantity": 3},
    {"order_id": 103, "customer_id": 202, "quantity": 7}
  ]
}
```

**Flow:**

1. Yes
2. No
3. Yes
4. No
5. GROUP BY
6. Are you grouping all rows?
7. Compute aggregate for entireresult set
8. PARTITION BY -> WindowFunction
9. Do you want to calculate perpartition?
10. Perform calculations withineach partition

