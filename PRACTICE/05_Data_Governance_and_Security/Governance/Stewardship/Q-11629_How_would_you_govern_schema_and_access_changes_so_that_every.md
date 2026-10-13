---
type: "practice"
difficulty: "Deep"
---

# How would you govern schema and access changes so that every DDL on a sensitive table is reviewed, logged, and tied to a change ticket automatically?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Governing Real-time Features](<../../../../05_Data_Governance_and_Security/Governance/Stewardship/Governing_Real-time_Features.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To ensure robust data governance and stewardship in our database environment, we need a system that can review, log, and tie every Data Definition Language (DDL) operation on sensitive tables to a corresponding change ticket. This requires a multi-layered approach involving automated scripts, logging mechanisms, and workflow integration.

The core idea is to create an automated pipeline that intercepts DDL operations before they are executed, reviews them against predefined policies, logs the changes, and integrates with our existing ticketing system to ensure accountability. The steps involve:

Intercepting DDL Operations: Using triggers or stored procedures.
Review Mechanism: Ensuring compliance with data governance rules.
Logging Changes: Recording detailed information about the change.
Integrating with Change Management System: Linking the changes to a formal ticket.

**Comparison:**

| Column Name | Description                      |
|-------------|----------------------------------|
| table_name  | Name of the affected sensitive table. |
| operation_type | Type of DDL operation (CREATE, ALTER). |
| user_name   | Username of the DB user who performed the operation. |
| timestamp   | Timestamp when the DDL operation was logged. |

**Code:**

```sql
-- Example SQL Trigger for Interception
CREATE OR REPLACE TRIGGER log_ddl_operations
BEFORE CREATE OR ALTER ON SCHEMA_NAME
FOR EACH STATEMENT
DECLARE
    v_sql VARCHAR2(32767);
BEGIN
    -- Log the DDL statement and relevant metadata
    INSERT INTO ddl_log (table_name, operation_type, user_name, timestamp)
    VALUES (
        (SELECT object_name FROM dba_objects WHERE object_id = SYS_CONTEXT('USERENV', 'CURRENT_SCHEMA_ID')),
        CASE
            WHEN UPPER(SQL_TEXT) LIKE '%CREATE%' THEN 'CREATE'
            WHEN UPPER(SQL_TEXT) LIKE '%ALTER%' THEN 'ALTER'
            ELSE NULL
        END,
        USER,
        CURRENT_TIMESTAMP
    );
END;
/
```

**Example input / output:**

```text
Input:
User issues the following DDL command on a sensitive table:

CREATE TABLE SENSITIVE_TABLE (id INT, data VARCHAR(255));

Expected Output:
- The `ddl_log` table is updated with the relevant metadata.
- A ticket is generated in the change management system linked to the DDL operation.

Actual Output:
- Log entry created: { "table_name": "SENSITIVE_TABLE", "operation_type": "CREATE", "user_name": "DB_ADMIN", "timestamp": "2023-10-01T14:30:00Z" }
- Ticket #12345 generated and linked to the DDL operation.
```

**Data:**

```json
{
  "ddl_log": {
    "table_name": "SENSITIVE_TABLE",
    "operation_type": "CREATE",
    "user_name": "DB_ADMIN",
    "timestamp": "2023-10-01T14:30:00Z"
  }
}
```

**Flow:**

1. No
2. Yes
3. User Issues DDL Command
4. Is Table Sensitive?
5. DDL Command is ExecutedDirectly
6. Trigger Logs DDL Operationand Creates Ticket
7. Ticket Linked to DDLOperation in ChangeManagement System

