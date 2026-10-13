---
type: "practice"
difficulty: "Deep"
---

# How do you make a data-backfill UPDATE resumable so a mid-run failure doesn't force you to restart from zero?

**Difficulty:** Deep · **Levels:** SDE-1, SDE-1/2 · [↑ Transaction Control and Prepared Transactions](<../../../../01_Databases_and_SQL/SQL_Language/SQL_Command_Types/Transaction_Control_and_Prepared_Transactions.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To make a data backfill update resumable, we need to ensure that partial updates can be resumed without losing progress after a failure. This involves handling transactions, maintaining state, and possibly using a combination of DDL (Data Definition Language), DML (Data Manipulation Language), TCL (Transaction Control Language), and even some application-level logic.

The core idea is to design the backfill process such that it can be paused at any point without corrupting data. This typically involves tracking the progress through metadata or log files, using ACID properties for transactional integrity, and ensuring that updates are idempotent where possible.

#### Triage
The question probes your understanding of transaction management, state tracking, and robust error handling in a database context. We need to ensure that even if an update fails mid-way, we can pick up from the last successful point without needing to start over.

**Code:**

```sql
-- Example schema for backfill progress tracking

CREATE TABLE backfill_progress (
    id INT PRIMARY KEY,
    table_name VARCHAR(255),
    row_id BIGINT,
    last_updated TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- DML to insert or update the progress status of a backfill operation
INSERT INTO backfill_progress (id, table_name, row_id)
VALUES (1, 'customers', 10000) ON DUPLICATE KEY UPDATE row_id = 10000;
```

**Example input / output:**

```text
Input:
- Start backfill operation on table 'customers'
- Process rows from row_id = 9000 to 20,000

Output (after a partial run):
- Progress record for 'customers' shows last updated at row_id = 15000

Dry-run after failure and restart:
- Resume backfill operation on table 'customers'
- Start from where it left off: row_id = 15000
```

**Data:**

```json
{
  "backfillProgress": [
    {
      "id": 1,
      "tableName": "customers",
      "rowId": 10000,
      "lastUpdated": "2023-10-05T14:48:00Z"
    }
  ]
}
```

