---
type: "practice"
difficulty: "Deep"
---

# What is a write skew anomaly in simple terms?

**Difficulty:** Deep · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ Range Locks and Retries](<../../../../01_Databases_and_SQL/Transactions_and_Concurrency/Locking/Range_Locks_and_Retries.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Write skew anomalies occur when multiple transactions access and modify different parts of a database, leading to inconsistent states that violate the integrity rules designed by the DBMS. In simpler terms, imagine you have an online store with products in stock, and several customers are trying to purchase items simultaneously. If one customer’s order is processed before others, it might lead to an incorrect inventory state, as some transactions might not see the latest data.

#### Triage

The question asks for a simple explanation of write skew anomalies. The interviewer is probing your understanding of how concurrent writes can lead to inconsistent states in a database. We need to assume that the database is managing multiple concurrent transactions and discussing how these can cause issues if not handled properly.

#### First-Principles Solution

Let's consider a scenario where an online store has products with limited stock, and customers are trying to purchase items concurrently. Suppose we have two customers, Alice and Bob, both wanting to buy the same product. The database uses simple row-level locking (SRL) for managing concurrency.

Alice Starts Her Transaction: She reads the inventory table and sees that there is 1 unit of a particular product.
Bob Starts His Transaction Independently: He also reads the same inventory table and sees that there is still 1 unit available.
Alice Completes Her Purchase: She updates the database to decrement the stock by one, leaving zero units left.
Bob Completes His Purchase After Alice’s Commit: Even though both transactions read the initial state of the product having one unit, Bob's transaction sees a different state after Alice commits her changes.

As a result, when Bob completes his purchase and also decrements the inventory, it incorrectly shows that there are no units left for this product. This inconsistency is an example of a write skew anomaly.

#### Closing Summary
+ Common Mistakes/Follow-Ups

In summary, write skew anomalies occur in databases where multiple transactions modify different parts of the data without proper coordination or visibility to other concurrent operations. To avoid such issues, it's crucial to implement advanced concurrency control techniques like two-phase locking (2PL), multiversion concurrency control (MVCC), or using distributed systems with global consistency guarantees.

A common mistake is assuming that simple row-level locks are sufficient in all scenarios, which can lead to unexpected and inconsistent states. Follow-up questions might explore how different concurrency control mechanisms handle such anomalies or how sharding could affect the occurrence of write skew issues.

**Comparison:**

```
product_id	quantity
1	1
Transaction	Operation	Result
Alice	Read Lock	Quantity = 1
	Update	Quantity = 0
Bob	Read Lock (after Alice commits)	Quantity = 0
```

**Code:**

```sql
-- Example SQL for reading and updating inventory with simple locking
BEGIN TRANSACTION;

SELECT product_id, quantity FROM products WHERE product_id = 1 FOR UPDATE; -- Row-level lock

UPDATE products SET quantity = quantity - 1 WHERE product_id = 1;

COMMIT;
```

**Example input / output:**

```text
- **Input**: Alice and Bob start transactions to buy a product.
  - Alice reads `quantity = 1`.
  - Bob reads `quantity = 1` independently.

- **Output**: Both Alice and Bob commit their transactions, but the final state shows no units left (`quantity = 0`), leading to an anomaly.

- **Follow-Up**: Discuss how distributed databases or global transaction managers could handle such issues.
```

**Data:**

```json
{
    "products": [
        {
            "product_id": 1,
            "quantity": 1
        }
    ]
}
```

**Flow:**

1. Transaction Starts
2. Read Product
3. Read Lock
4. Update Product
5. Commit
6. Transaction Starts
7. Read Product
8. Read Lock (after Alicecommits)
9. Update Product
10. Commit

