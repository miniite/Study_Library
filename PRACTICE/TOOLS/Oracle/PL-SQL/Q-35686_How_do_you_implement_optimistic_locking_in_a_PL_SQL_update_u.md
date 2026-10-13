---
type: "practice"
difficulty: "Deep"
---

# How do you implement optimistic locking in a PL/SQL update using a version column or ORA_ROWSCN, and how do you detect a lost update?

**Difficulty:** Deep · **Levels:** SDE-1, SDE-1/2 · [↑ PL-SQL in Production](<../../../../TOOLS/Oracle/PL-SQL/PL-SQL_in_Production.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Optimistic locking is a technique used to prevent conflicts during concurrent updates by tracking changes made to records in a database. In PL/SQL, we can implement optimistic locking using either a version column or ORA_ROWSCN. The key idea here is that before performing an update, you first read the current state of the record (including the version or SCN), make your modifications, and then verify if another user has changed the record in between.

To detect a lost update, we check the version or SCN value post-update to see if it differs from the one read at the beginning. If there's a mismatch, it means that another transaction updated the row before our current transaction did, and thus, we have a lost update scenario.

**Comparison:**

```
Yes
No
Read Record
Check Version
Update Record
Version Mismatch?
Rollback and Notify LostUpdate
Commit Transaction
```

**Code:**

```

-- Example of optimistic locking using a version column

DECLARE l_old_version NUMBER; l_new_version NUMBER := 1; -- Initial version number for demonstration purposes BEGIN -- Read the current state of the record including the version number SELECT id, data, version INTO l_id, l_data, l_old_version FROM my_table WHERE id = 12345 AND version = 1;

-- Make modifications to the data l_data := 'Updated Data';

-- Try to update the record with the new data and check the version BEGIN UPDATE my_table SET data = l_data, version = l_new_version + 1 WHERE id = l_id AND version = l_old_version;

CODE
Copy
COMMIT; -- The transaction is committed only if no conflict was detected

EXCEPTION WHEN OTHERS THEN ROLLBACK; -- Rollback the transaction in case of a conflict DBMS_OUTPUT.PUT_LINE('Lost update detected. New data not applied.'); END; END; /

CODE
Copy

```plsql
-- Example using ORA_ROWSCN for optimistic locking

DECLARE
  l_old_scn NUMBER;
BEGIN
  -- Read the current state of the record including the SCN number
  SELECT id, data, ora_rowscn INTO l_id, l_data, l_old_scn
  FROM my_table
  WHERE id = 12345;

  -- Make modifications to the data
  l_data := 'Updated Data';

  -- Try to update the record with the new data and check the SCN
  BEGIN
    UPDATE my_table
    SET data = l_data, ora_rowscn = ora_rowscn + 1
    WHERE id = l_id AND ora_rowscn = l_old_scn;

    COMMIT; -- The transaction is committed only if no conflict was detected
  EXCEPTION
    WHEN OTHERS THEN
      ROLLBACK; -- Rollback the transaction in case of a conflict
      DBMS_OUTPUT.PUT_LINE('Lost update detected. New data not applied.');
  END;
END;
/
```

**Example input / output:**

```text
-- Input (Example of optimistic locking using a version column)
DECLARE
  l_id NUMBER := 12345;
  l_data VARCHAR2(100) := 'Initial Data';
  l_old_version NUMBER;

BEGIN
  -- Read the current state of the record including the version number
  SELECT id, data, version INTO l_id, l_data, l_old_version FROM my_table WHERE id = l_id AND version = 1;

  -- Make modifications to the data
  l_data := 'Updated Data';

  -- Try to update the record with the new data and check the version
  BEGIN
    UPDATE my_table SET data = l_data, version = l_old_version + 1 WHERE id = l_id AND version = l_old_version;
    COMMIT; -- The transaction is committed only if no conflict was detected
  EXCEPTION
    WHEN OTHERS THEN
      ROLLBACK; -- Rollback the transaction in case of a conflict
      DBMS_OUTPUT.PUT_LINE('Lost update detected. New data not applied.');
  END;

END;
/

-- Output
PL/SQL procedure successfully completed.

DBMS_OUTPUT:
'Lost update detected. New data not applied.'
```

**Data:**

```sql
-- Sample table schema
CREATE TABLE my_table (
  id NUMBER PRIMARY KEY,
  data VARCHAR2(100),
  version NUMBER DEFAULT 1,
  ora_rowscn BINARY_INTEGER -- This column is optional and used for ORA_ROWSCN-based optimistic locking
);
```

**Flow:**

1. Both match
2. Mismatch
3. Read record
4. Check version number or SCN
5. Update record and incrementversion/SCN
6. Commit transaction
7. Rollback transaction and notifylost update

