---
type: "practice"
difficulty: "Deep"
---

# When would you choose to store a computed value like order total in a column versus calculating it every time you query?

**Difficulty:** Deep · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ Materialized CTEs](<../../../../TOOLS/PostgreSQL/PostgreSQL_Platform/Materialized_CTEs.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To decide whether to store a computed value like an order total in a database column or calculate it on-the-fly, we need to consider several factors such as performance, data consistency, and space usage. Let's break this down from first principles.

Firstly, the key idea is that storing computed values can improve query performance by reducing the load on the database engine during queries. However, doing so requires more careful consideration of the trade-offs involved in terms of storage overhead, update complexity, and potential for data inconsistencies.

**Comparison:**

```
Factors	Storing Computed Value	Calculating On-The-Fly
Performance	Higher, as it avoids the computation overhead during queries.	Lower, as computations are performed every time the query is run.
Space Usage	Uses additional storage for the computed column value.	No additional space used.
Data Consistency	Ensures consistency by having a dedicated column that is always up-to-date with the latest values.	Potential risk of inconsistency if the computation logic changes or is not correctly applied during queries.
```

**Code:**

```sql
-- Example table structure with both calculated and raw columns
CREATE TABLE orders (
    order_id INT PRIMARY KEY,
    product_id INT,
    quantity INT,
    price DECIMAL(10, 2),
    -- Calculated column (total value)
    total_value DECIMAL(15, 2) GENERATED ALWAYS AS ((quantity * price)) STORED,
    -- Raw data columns
    order_date TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    status VARCHAR(30)
);
```

**Example input / output:**

```text
-- Example query with both calculated and raw column usage
SELECT order_id, total_value, quantity * price AS on_the_fly_total FROM orders;
```

**Flow:**

1. Yes
2. Yes
3. No
4. No
5. Yes
6. No
7. Consider Performance Needs
8. Are Calculations Complex?
9. Is Storage Space Limited?
10. Store Computed Value
11. Calculate On-The-Fly
12. Assess Data Consistency Risks
13. Do Complex Queries RequirePre-Computation?
14. Store Computed Value
15. Calculate On-The-Fly

