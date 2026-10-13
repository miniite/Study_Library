---
type: "practice"
difficulty: "Basic"
---

# What does the COMMIT command do in a transaction?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ Transaction Control and Prepared Transactions](<../../../../01_Databases_and_SQL/SQL_Language/SQL_Command_Types/Transaction_Control_and_Prepared_Transactions.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In the context of database operations, COMMIT is a crucial command used to finalize a transaction and make all changes permanent in the database. Essentially, it signals that the current set of changes should be saved as they are, overriding any previous state. Here’s a brief overview followed by an explanation of what COMMIT does step-by-step.

The primary purpose of the COMMIT command is to ensure data integrity and consistency. When you start a transaction with commands like BEGIN or START TRANSACTION, all subsequent changes are temporary until COMMIT is issued. Once COMMIT is executed, these changes are written into the database's permanent storage.

**Code:**

```sql
-- Example of using COMMIT in SQL
BEGIN;
INSERT INTO customers (id, name) VALUES (1, 'John Doe');
UPDATE products SET stock = 50 WHERE id = 1;
COMMIT; -- This command makes all previous changes permanent.
```

**Flow:**

1. Yes
2. No
3. Start Transaction
4. Execute INSERT/UPDATEstatements
5. Is COMMIT executed?
6. Makes changes permanent
7. Transaction remains intemporary state
8. End of transaction

