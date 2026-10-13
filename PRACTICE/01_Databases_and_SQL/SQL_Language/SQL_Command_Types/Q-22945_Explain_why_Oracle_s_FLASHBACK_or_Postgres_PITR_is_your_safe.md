---
type: "practice"
difficulty: "Deep"
---

# Explain why Oracle's FLASHBACK or Postgres PITR is your safety net for irreversible DML, and how you'd rehearse the recovery.

**Difficulty:** Deep · **Levels:** SDE-1, SDE-1/2 · [↑ Transaction Control and Prepared Transactions](<../../../../01_Databases_and_SQL/SQL_Language/SQL_Command_Types/Transaction_Control_and_Prepared_Transactions.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Flashback and Point-in-Time Recovery (PITR) are powerful safety nets in Oracle and PostgreSQL respectively, providing a way to recover from irreversible Data Manipulation Language (DML) operations such as DELETE or UPDATE without having explicit backups. These features allow you to roll back changes made by DML commands within a specified time frame, making data recovery straightforward and efficient.

To understand the core of this question, we need to focus on why these mechanisms are essential for database developers and how one would rehearse a recovery process. We will also cover assumptions and step-by-step reasoning from first principles to arrive at the full solution.

**Comparison:**

```
Time	Action	Result
2023-10-01	DELETE Employee	Employee record was deleted
2023-10-01	Flashback Query	Employee record returned
```

**Code:**

```sql
-- Example Oracle Flashback Query
SELECT * FROM employees VERSIONS BETWEEN TIMESTAMP '2023-10-01 14:00:00' AND '2023-10-01 15:00:00';

-- Example PostgreSQL PITR Command
RESTART荻AL TO '2023-10-01 14:00:00';
```

**Example input / output:**

```text
User Input:
RESTART荻AL TO '2023-10-01 14:00:00';
System Response:
Server started from backup point and applied all transaction logs up to the specified time, reverting any changes made after that point.
```

**Data:**

```json
{
  "backup_location": "/var/backups/postgres/",
  "flashback_time": "2023-10-01T14:00:00Z",
  "transaction_log_file": "/path/to/redo_log.000001"
}
```

**Flow:**

1. Rehearse Recovery
2. Define Recovery Point
3. Select Time or Log File
4. Start PITR/Flashback
5. Roll Back Changes
6. Verify Data Integrity

