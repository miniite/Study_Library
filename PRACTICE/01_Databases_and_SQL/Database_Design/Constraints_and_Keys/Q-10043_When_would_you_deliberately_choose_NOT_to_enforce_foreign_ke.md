---
type: "practice"
difficulty: "Expert"
---

# When would you deliberately choose NOT to enforce foreign keys at the database level in a high-throughput system, and what safeguards replace them?

**Difficulty:** Expert · **Levels:** SDE-1, SDE-1/2 · [↑ Foreign Keys](<../../../../01_Databases_and_SQL/Database_Design/Constraints_and_Keys/Foreign_Keys.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When designing a high-throughput system, it's common to prioritize performance over strict data integrity constraints like foreign keys (FKs). However, there are specific scenarios where you might choose not to enforce FKs at the database level. These situations often involve real-time systems with stringent latency requirements or distributed architectures that cannot afford the overhead of maintaining referential integrity across multiple nodes.

**Code:**

```sql
-- Example SQL schema without FK constraints
CREATE TABLE orders (
    id INT PRIMARY KEY,
    product_id INT, -- No FK on this column
    customer_id INT, -- No FK on this column
    order_date TIMESTAMP
);
```

