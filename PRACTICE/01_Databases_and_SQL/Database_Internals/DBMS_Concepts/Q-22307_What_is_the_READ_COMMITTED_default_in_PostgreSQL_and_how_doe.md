---
type: "practice"
difficulty: "Deep"
---

# What is the READ COMMITTED default in PostgreSQL and how does it differ from MySQL's REPEATABLE READ default?

**Difficulty:** Deep · **Levels:** Fresher, SDE-1 · [↑ Lock Granularity](<../../../../01_Databases_and_SQL/Database_Internals/DBMS_Concepts/Lock_Granularity.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In database systems, transaction isolation levels determine how data is visible to other transactions during their execution. PostgreSQL defaults to READ COMMITTED, which means that a transaction sees only the committed changes made before it started. In contrast, MySQL's default isolation level is REPEATABLE READ, where each transaction sees a snapshot of the database as it was at the beginning of the transaction.

To understand these differences in detail, let’s first define the key concepts and then discuss how they manifest in both PostgreSQL and MySQL.

**Comparison:**

```
Isolation Level	Description
READ COMMITTED	Reads only committed data before transaction starts.
REPEATABLE READ	Consistent snapshot of the database at the start of transaction.
```

**Code:**

```
MERMAID
Copy
graph TD
    A[Transaction Isolation Levels] --> B{READ COMMITTED}
    B --> C{Reads only committed data before transaction starts}
    A --> D{REPEATABLE READ}
    D --> E{Consistent snapshot of the database at the start of transaction}
```

**Flow:**

1. Transaction Starts
2. Read Data
3. Check if Data is Committed
4. Yes
5. Commit Read, Proceed
6. No
7. Rollback, Retry
8. ### Triaging the Question
9. The question asks to explain how PostgreSQL's READ COMMITTED isolation level differs from MySQL’s REPEATABLE READ. This requires understanding both the semantics of transaction isolation and their practical implications in these two database management systems.
10. #### Assumptions:
11. - Both PostgreSQL and MySQL use SQL-standard isolation levels.
12. - We are considering typical scenarios where transactions need to read or modify data.
13. ### First Principles Solution
14. 1. **READ COMMITTED (PostgreSQL Default)**
15. - In READ COMMITTED mode, a transaction sees only the committed changes that existed before it started executing. This means if another transaction commits new data after your transaction starts but before it reads any data, those changes will not be visible to your transaction.
16. - Example: If Transaction A inserts `new_data` at 10:00 AM and Transaction B is running from 9:55 AM to 10:15 AM, then Transaction B will see an empty state for the table. Even if `new_data` is committed at 10:02 AM, it won't be visible in Transaction B.
17. 2. **REPEATABLE READ (MySQL Default)**
18. - In REPEATABLE READ mode, a transaction sees the same snapshot of data throughout its execution. This means that once a transaction starts reading some data, any subsequent reads will always return the same values, regardless of changes committed by other transactions.
19. - Example: If Transaction A inserts `new_data` at 10:02 AM and Transaction B is running from 9:55 AM to 10:15 AM, then Transaction B will see an empty state for the table. Even if another transaction commits `new_data` at 10:03 AM, Transaction B will not see this change until it completes or retries its read operation.
20. ### Full Solution
21. Let’s delve deeper into how these isolation levels affect application behavior and potential pitfalls:
22. - **READ COMMITTED**:
23. - Advantages: Can avoid phantom reads (where a new row appears in the result set during execution) because only committed data is visible.
24. - Disadvantages: May suffer from non-repeatable reads, where the same query executed multiple times returns different results due to concurrent transactions.
25. - **REPEATABLE READ**:
26. - Advantages: Provides stronger consistency guarantees as it ensures that a transaction sees a consistent view of the database throughout its execution.
27. - Disadvantages: Can lead to higher overhead and potential deadlocks if not managed carefully, especially in scenarios with complex query patterns.
28. ### Common Mistakes and Follow-Ups
29. 1. **Misunderstanding Phantom Reads**: READ COMMITTED can indeed prevent phantom reads but does not guarantee repeatable results within a transaction.
30. 2. **Transaction Lifespan**: REPEATABLE READ ensures that the same data is always read, but it may not be up-to-date if other transactions are frequently modifying the database during its execution.
31. 3. **Performance Considerations**: Transactions running under REPEATABLE READ might need to wait for locks to release, leading to potential bottlenecks in high-concurrency environments.
32. In summary, while PostgreSQL’s READ COMMITTED and MySQL’s REPEATABLE READ serve similar purposes of maintaining data integrity, they differ significantly in how they handle read operations within transactions. The choice between these isolation levels depends on the specific application requirements for consistency versus performance.

