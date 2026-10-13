---
type: "practice"
difficulty: "Intermediate"
---

# How do transactions work in Snowflake and what does COMMIT versus ROLLBACK do?

**Difficulty:** Intermediate · **Levels:** Architect/Principal, Fresher, SDE-1/2, SDE-2/3 · [↑ Snowflake Loading and Pipelines](<../../../../TOOLS/Snowflake/Snowflake_Platform/Snowflake_Loading_and_Pipelines.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To understand how transactions work in Snowflake, we first need to break down the core concept of transactions and then explore how they are managed in this cloud-based data warehousing platform. A transaction is a sequence of operations treated as a single unit of work. In Snowflake, transactions are essential for ensuring data consistency and integrity during database operations. The key terms here—COMMIT and ROLLBACK—are used to control the state of these transactions.

In this explanation, I'll provide an overview suitable for someone with intermediate experience, followed by a detailed breakdown from first principles.

**Code:**

```sql
-- Example Snowflake SQL Statements
INSERT INTO sales (id, date, amount) VALUES (1, '2023-09-01', 100);
UPDATE sales SET amount = 200 WHERE id = 1;
DELETE FROM sales WHERE id = 1;
```

**Example input / output:**

```text
-- Example Transaction Flow
BEGIN TRANSACTION; -- Start a transaction

INSERT INTO sales (id, date, amount) VALUES (1, '2023-09-01', 100);

-- Suppose an error occurs here:
DELETE FROM sales WHERE id = 1; -- This would fail and ROLLBACK is needed

COMMIT; -- If no errors occurred
ROLLBACK; -- To revert the changes if an error did occur
```

**Flow:**

1. Begin Transaction
2. INSERT INTO sales
3. Error in DELETE Operation
4. Rollback Transaction
5. No Error, Proceed withCOMMIT
6. Commit Transaction

