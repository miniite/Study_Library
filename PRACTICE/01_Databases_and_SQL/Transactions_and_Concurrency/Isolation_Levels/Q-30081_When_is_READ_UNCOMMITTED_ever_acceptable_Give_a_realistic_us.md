---
type: "practice"
difficulty: "Basic"
---

# When is READ UNCOMMITTED ever acceptable? Give a realistic use case and explain the risk you accept.

**Difficulty:** Basic · **Levels:** Architect/Principal, SDE-1, SDE-1/2, SDE-2/3 · [↑ Setting Isolation Levels](<../../../../01_Databases_and_SQL/Transactions_and_Concurrency/Isolation_Levels/Setting_Isolation_Levels/_index.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR:

READ UNCOMMITTED isolation level should almost never be used in a production environment due to its high risk of reading uncommitted data, which can lead to inconsistent results and dirty reads. However, there are very specific use cases where the performance benefits might outweigh these risks.

#### Triaging the Question

The question asks when READ UNCOMMITTED is acceptable and requires us to provide an example scenario while also acknowledging the associated risks. The key aspects here are understanding what READ UNCOMMITTED does, identifying its advantages in certain contexts, and discussing the trade-offs.

Isolation Levels Explained:
Read Uncommitted (RU): Allows a transaction to read uncommitted data changes made by other transactions that have not yet been committed.
Read Committed (RC): Ensures that data is only visible after it has been committed. This level provides stronger consistency but can still result in phantom reads or non-repeatable reads under certain conditions.
Repeatable Read (RR): Guarantees repeatable read operations, which means that the same query run multiple times during a transaction will return the same results.
Serializable: Ensures serializability by preventing dirty reads, non-repeatable reads, and phantom reads.
Scenario: In-Memory Processing in E-commerce

Imagine an e-commerce platform experiencing extremely high traffic during Black Friday sales. The system needs to process large volumes of orders quickly, which can result in a flood of write operations (orders being placed). During this period, we may want to accept the risk of READ UNCOMMITTED to improve performance.

Risk Acknowledgment:

By using READ UNCOMMITTED, the system risks reading partially written data. For example, if an order is being processed but not yet committed, another transaction might read that partial data, leading to inconsistent billing or inventory counts.

#### Example
Use Case:

#### Cons
ider a scenario where we need to calculate real-time sales analytics by aggregating orders from multiple databases. Each database represents different geographical regions and updates its order data frequently during peak hours. To avoid blocking other transactions, we could use READ UNCOMMITTED to quickly summarize the partially updated datasets without waiting for all transactions to commit.

SQL
-- Example Query Using READ UNCOMMITTED
SELECT SUM(order_total)
FROM orders WITH (NOLOCK)  -- NOLOCK hint is a SQL Server equivalent of READ UNCOMMITTED
WHERE order_date >= '2023-11-24'
Potential Risks:
Dirty Reads: A transaction might read data that has been partially updated and then roll back, leading to inaccurate analytics.
Non-repeatable Reads: Data retrieved by a query may change between the time it is first read and when an attempt is made to re-read it.

#### Closing Summary
and Common Mistakes:

While READ UNCOMMITTED can provide significant performance benefits in certain high-throughput scenarios like real-time processing, its usage should be extremely cautious. The primary risk of reading uncommitted data must always be weighed against the potential for inconsistent results and dirty reads.

#### Common Mistakes/Follow-Ups
Failing to Consider Consistency: Ignoring the implications of non-repeatable reads and dirty reads.
Overusing in Production: Using READ UNCOMMITTED outside of controlled, low-criticality environments where performance is the primary concern.
Lack of Monitoring: Not setting up adequate monitoring to detect and mitigate issues caused by using this isolation level.

