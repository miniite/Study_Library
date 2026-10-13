---
type: "practice"
difficulty: "Deep"
---

# Why can an ALTER TABLE block ongoing reads and writes on that table?

**Difficulty:** Deep · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ MVCC](<../../../../01_Databases_and_SQL/Transactions_and_Concurrency/Locking/MVCC.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When you execute an ALTER TABLE statement in a relational database, it's essentially asking the database to modify the underlying schema or structure of the table. This operation can have significant implications for ongoing reads and writes on that table due to its nature. The primary reason is the need to ensure data integrity and consistency during the change process.

To understand why ALTER TABLE blocks ongoing reads and writes, we must first delve into the core concepts of locking and concurrency control in databases. Concurrency control mechanisms are designed to manage multiple transactions accessing shared resources (like tables) concurrently without causing issues like inconsistencies or deadlocks. When an ALTER TABLE operation is initiated, it needs to lock the table to prevent any modifications or reads from interfering with the ongoing changes.

Here’s a step-by-step breakdown of why this happens:

Schema Changes and Data Integrity: Altering a table can involve changing column definitions, adding or removing indexes, splitting or merging partitions, etc. These operations require the database to lock the table to ensure that no other transaction is writing data at the same time. If writes were allowed, they could potentially overwrite or corrupt the process of schema changes.

#### Cons
istency and Isolation Levels: The isolation levels defined by ACID properties (Atomicity, Consistency, Isolation, Durability) play a crucial role here. At higher isolation levels like Serializable, transactions are isolated from other concurrent operations to ensure data consistency. An ALTER TABLE operation typically requires a high level of isolation, meaning it blocks all ongoing reads and writes until the change is complete.

Transaction Management: During an ALTER TABLE, the database might need to perform operations that require exclusive access to the table, such as dropping indexes or creating new ones. This process must be atomic; if reads or writes are allowed, they could interfere with these critical steps, leading to inconsistent states or even data corruption.

Let's walk through a simple example to illustrate this:

Suppose we have an employees table and someone initiates an ALTER TABLE employees ADD COLUMN department_id INT. The database will need to lock the employees table to ensure that no writes or reads are happening while it performs the necessary operations, such as creating new columns and updating existing rows.

SQL
-- Attempting to alter a table during ongoing transactions
BEGIN TRANSACTION;

-- Imagine some data is being written in this transaction
INSERT INTO employees (name, salary) VALUES ('John Doe', 50000);

-- ALTER TABLE operation starts here
ALTER TABLE employees ADD COLUMN department_id INT;

In the above example, the ALTER TABLE statement will block any ongoing insertions or updates to the employees table until it is completed. This ensures that when the changes are applied, they do not conflict with any current transactions.

To summarize, an ALTER TABLE operation blocks ongoing reads and writes because of the critical nature of schema modifications. Ensuring data integrity, consistency, and atomicity during such operations requires exclusive access to the table, hence the blocking mechanism in place.

