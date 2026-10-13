---
type: "practice"
difficulty: "Intermediate"
---

# Explain how you'd grant an application the ability to read a sequence and insert rows but never truncate or alter the table.

**Difficulty:** Intermediate · **Levels:** Architect/Principal, SDE-1, SDE-1/2, SDE-2/3 · [↑ Column-level Access](<../../../../01_Databases_and_SQL/Database_Operations/Security_and_Access/Column-level_Access/_index.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address this requirement, we need to ensure that our database permissions are finely tuned so that the application can perform specific operations on a table while preventing unauthorized actions like truncation or alteration of the underlying schema. The key steps involve setting up appropriate roles and privileges, leveraging SQL policies if available, and possibly using database triggers for additional enforcement.

The core idea is to grant minimal necessary permissions to the application role, ensuring it can only read from and insert into the sequence and table in question. This approach minimizes the risk of accidental or malicious modifications.

**Code:**

```sql
-- Step 1: Create an Application Role
CREATE ROLE app_role;

-- Step 2: Grant SELECT and INSERT permissions on specific columns or the entire table
GRANT SELECT, INSERT ON sequence_table TO app_role;

-- Step 3: Optionally, use SQL POLICIES for additional security (PostgreSQL)
CREATE OR REPLACE FUNCTION enforce_read_only()
RETURNS TRIGGER AS $$
BEGIN
    IF TG_OP = 'INSERT' THEN
        RETURN NULL; -- Prevent row insertions directly through this policy
    END IF;
    RETURN NEW;
END;
$$ LANGUAGE plpgsql;

-- Apply the policy on sequence_table to prevent truncation and column modification
CREATE POLICY read_only_policy ON sequence_table FOR SELECT TO app_role
USING (TRUE);

CREATE TRIGGER enforce_read_only_trigger BEFORE INSERT OR DELETE OR UPDATE ON sequence_table
FOR EACH ROW EXECUTE FUNCTION enforce_read_only();
```

