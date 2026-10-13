---
type: "practice"
difficulty: "Deep"
---

# Two concurrent SELECT ... FOR UPDATE statements on overlapping rows in different orders deadlock; how would you enforce a deterministic locking order via ORDER BY in the SELECT that acquires locks?

**Difficulty:** Deep · **Levels:** SDE-1, SDE-1/2 · [↑ Deadlocks and Lock Ordering](<../../../../TOOLS/PostgreSQL/PostgreSQL_Platform/Deadlocks_and_Lock_Ordering.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the issue of deadlocks caused by two concurrent SELECT ... FOR UPDATE statements on overlapping rows, we can enforce a deterministic locking order using an appropriate ORDER BY clause in our SQL query. This ensures that both transactions acquire locks in the same sequence, thus preventing deadlocks.

#### Triage

The question is probing for understanding of how to manage concurrency issues in PostgreSQL by ensuring deterministic lock acquisition. The key assumptions are:

We have two concurrent transactions.
These transactions execute SELECT ... FOR UPDATE statements on overlapping rows but in different orders.
Deadlocks occur due to the non-deterministic nature of the order in which locks are acquired.

#### First Principles Solution

To enforce a deterministic locking order, we can use an ORDER BY clause with a stable sort key that is common across all transactions. Here’s how you can achieve this:

Identify Common Attributes: Find attributes that uniquely identify each row and ensure these attributes are used in the ORDER BY clause.
Use a Stable Order: Ensure the order by which rows are selected (and thus locked) is consistent across all transactions.

Let's assume we have a table named orders with columns such as order_id, customer_id, etc. Here’s an example of how you can write your query:

SQL
BEGIN;
SELECT * FROM orders WHERE customer_id = 1 FOR UPDATE ORDER BY order_id ASC;

-- Concurrent Transaction 2:
BEGIN;
SELECT * FROM orders WHERE customer_id = 1 FOR UPDATE ORDER BY order_id ASC;

In this case, both transactions will always lock the rows in the same order based on order_id. If there are no overlapping rows or if the ORDER BY clause doesn’t affect the row selection, you can consider adding more attributes.

#### Closing Summary

By using an appropriate ORDER BY clause with a stable sort key across all transactions, we ensure that both statements acquire locks in the same order. This approach prevents deadlocks by making the locking process deterministic.

#### Common Mistakes
and Follow-Ups
Not Using ORDER BY: Failing to use ORDER BY can lead to non-deterministic lock acquisition.
Inconsistent Order By: If you change the ORDER BY clause in one transaction but not another, deadlocks might still occur.
Complex Queries: For more complex scenarios, consider using a more sophisticated locking strategy or additional constraints.

For further optimization and handling edge cases, consider:

Using advisory locks for fine-grained control over locking.
Implementing retry logic to handle transient issues.
Reviewing transaction isolation levels to ensure they align with your application requirements.

**Code:**

```sql
BEGIN;
SELECT * FROM orders WHERE customer_id = 1 FOR UPDATE ORDER BY order_id ASC;

-- Concurrent Transaction 2:
BEGIN;
SELECT * FROM orders WHERE customer_id = 1 FOR UPDATE ORDER BY order_id ASC;
```

**Flow:**

1. Transaction 1
2. SELECT * FROM orders WHEREcustomer_id = 1 FOR UPDATEORDER BY order_id ASC
3. Lock rows in ascending orderby order_id
4. Transaction 2
5. SELECT * FROM orders WHEREcustomer_id = 1 FOR UPDATEORDER BY order_id ASC
6. Lock rows in ascending orderby order_id

