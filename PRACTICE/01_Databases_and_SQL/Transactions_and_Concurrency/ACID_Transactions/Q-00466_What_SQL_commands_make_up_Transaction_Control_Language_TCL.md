---
type: "practice"
difficulty: "Basic"
---

# What SQL commands make up Transaction Control Language (TCL)?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ Commit Rollback and Savepoints](<../../../../01_Databases_and_SQL/Transactions_and_Concurrency/ACID_Transactions/Commit_Rollback_and_Savepoints.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Transaction Control Language (TCL) in SQL is used to manage transactions, ensuring data integrity and consistency. The core commands that make up TCL are BEGIN, COMMIT, and ROLLBACK. These commands allow developers to group multiple SQL statements into a single transaction, enabling the execution of these operations as a unit.

These commands help achieve ACID properties (Atomicity, Consistency, Isolation, Durability), which are crucial for reliable database management. Let's break down each command:

BEGIN: This command starts a new transaction. It marks the beginning of a sequence of SQL statements that must be treated as a unit.

COMMIT: After executing all the necessary SQL commands within a transaction, COMMIT is used to permanently save (commit) all changes made during the transaction to the database. If no errors occur, these changes become part of the permanent record.

ROLLBACK: In case an error occurs or if you decide not to commit the transaction, ROLLBACK can be used to undo all the operations performed within that transaction. The database reverts back to its state before the transaction started.

These commands are foundational for ensuring data integrity and reliability in SQL databases. Now, let's reason through how they work from first principles:

Atomicity: All changes made within a transaction must either be committed or rolled back together. There is no partial commit.

#### Cons
istency: The database moves from one valid state to another. Any operations that violate the database constraints are not allowed and will cause a rollback.
Isolation: Transactions are isolated from each other, meaning that changes made during one transaction do not interfere with the ongoing transactions.
Durability: Once a transaction is committed, its effects are permanent and cannot be undone unless explicitly rolled back.

In practice, here's how you might use these commands in a simple example:

SQL
BEGIN;
INSERT INTO employees (id, name) VALUES (102, 'Jane');
UPDATE salaries SET salary = 45000 WHERE id = 102;
COMMIT;

In this case:

The BEGIN command starts the transaction.
The INSERT and UPDATE commands modify the data within the scope of the transaction.
The COMMIT command saves these changes permanently.

If an error occurs, you could use:

SQL
BEGIN;
INSERT INTO employees (id, name) VALUES (102, 'Jane');
UPDATE salaries SET salary = 45000 WHERE id = 102;
-- Assume the next line fails due to a constraint violation
-- ROLLBACK; -- This will undo all changes made in this transaction

In this scenario:

If the ROLLBACK command is executed, it undoes both the insert and update operations.

**Code:**

```sql
BEGIN;
INSERT INTO employees (id, name) VALUES (102, 'Jane');
UPDATE salaries SET salary = 45000 WHERE id = 102;
COMMIT;

-- Example with ROLLBACK
BEGIN;
INSERT INTO employees (id, name) VALUES (103, 'John'); -- Assume a constraint violation here
ROLLBACK; -- This undoes the insert operation
```

**Example input / output:**

```

The BEGIN, COMMIT, and ROLLBACK commands are used to manage transactions. Here's a sample interaction:

SQL
Copy
-- Starting the transaction
BEGIN;

-- Inserting an employee
INSERT INTO employees (id, name) VALUES (102, 'Jane');

-- Updating the salary
UPDATE salaries SET salary = 45000 WHERE id = 102;

-- Assuming no errors occur, committing the changes
COMMIT;

-- If an error occurs during the transaction
BEGIN;
INSERT INTO employees (id, name) VALUES (103, 'John'); -- Assume a constraint violation here
ROLLBACK; -- This undoes the insert operation
```

**Data:**

```sql
CREATE TABLE employees (
    id INT PRIMARY KEY,
    name VARCHAR(50)
);

CREATE TABLE salaries (
    id INT PRIMARY KEY,
    salary DECIMAL(10, 2),
    FOREIGN KEY (id) REFERENCES employees(id)
);
```

