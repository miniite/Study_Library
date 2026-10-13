---
type: "practice"
difficulty: "Deep"
---

# Describe how you'd add a foreign key constraint to an existing large table without a long validation lock.

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1, SDE-1/2, SDE-2/3 · [↑ Concurrent Index Builds](<../../../../01_Databases_and_SQL/Database_Operations/Migrations/Concurrent_Index_Builds/_index.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When adding a foreign key constraint to an existing large table, the primary concern is minimizing downtime and ensuring data consistency. The solution involves multiple steps from first principles to ensure minimal impact on the database's operational performance. This approach leverages temporary tables, constraints, and online DDL operations to add the constraint efficiently.

Firstly, let’s understand what we are trying to achieve: adding a foreign key constraint without locking the table for extended periods. This is critical in production environments where data modification can significantly impact application performance and availability.

We'll break down the process into several steps:

Identify the Source Table: Determine which table contains the primary key that will be referenced by the new foreign key.
Create a Temporary Table: Populate a temporary table with the relevant data from the source table to avoid locking issues during data validation.
Add Constraints Temporarily on the Target Table: Create the foreign key constraint but mark it as deferred initially to avoid immediate checking.
Validate Data: Ensure that all rows in the target table match the referenced primary keys in the temporary table.
Drop and Re-add the Constraint: Once validation passes, drop the deferred constraint and re-add it with non-deferred status.

Now, let’s go through these steps in detail:

Identify the Source Table:

First, identify the source table that contains the primary key. For example, users is our source table.

Create a Temporary Table:

SQL
CREATE TABLE temp_users AS SELECT * FROM users;

Add Constraints Temporarily on the Target Table: Assume we have a target table called orders. We add a foreign key constraint temporarily:

SQL
ALTER TABLE orders ADD CONSTRAINT fk_order_user DEFERRABLE INITIALLY DEFERRED REFERENCES temp_users (user_id);

Validate Data: Check if all rows in the orders table have valid references to the temp_users table:

SQL
DO $$
DECLARE
  v_row record;
  v_user_id integer;
BEGIN
  FOR v_row IN SELECT * FROM orders LOOP
    SELECT user_id INTO v_user_id FROM temp_users WHERE id = v_row.user_id;
    IF NOT FOUND THEN
      RAISE EXCEPTION 'Invalid reference: Order with ID % has an invalid user', v_row.id;
    END IF;
  END LOOP;
END $$;

Drop and Re-add the Constraint: Once validation passes, we can remove the deferred constraint and add a non-deferred version:

SQL
ALTER TABLE orders DROP CONSTRAINT fk_order_user;
ALTER TABLE orders ADD CONSTRAINT fk_order_user REFERENCES temp_users (user_id) INITIALLY IMMEDIATE NOT DEFERRABLE;

**Code:**

```sql
-- Step 2: Create a temporary table
CREATE TABLE temp_users AS SELECT * FROM users;

-- Step 3: Add constraints temporarily on the target table
ALTER TABLE orders ADD CONSTRAINT fk_order_user DEFERRABLE INITIALLY DEFERRED REFERENCES temp_users (user_id);

-- Step 4: Validate data
DO $$
DECLARE
  v_row record;
  v_user_id integer;
BEGIN
  FOR v_row IN SELECT * FROM orders LOOP
    SELECT user_id INTO v_user_id FROM temp_users WHERE id = v_row.user_id;
    IF NOT FOUND THEN
      RAISE EXCEPTION 'Invalid reference: Order with ID % has an invalid user', v_row.id;
    END IF;
  END LOOP;
END $$;

-- Step 5: Drop and re-add the constraint
ALTER TABLE orders DROP CONSTRAINT fk_order_user;
ALTER TABLE orders ADD CONSTRAINT fk_order_user REFERENCES temp_users (user_id) INITIALLY IMMEDIATE NOT DEFERRABLE;
```

**Example input / output:**

```sql
-- Input: Adding a foreign key to the 'orders' table referencing 'users'
CREATE TABLE temp_users AS SELECT * FROM users;

ALTER TABLE orders ADD CONSTRAINT fk_order_user DEFERRABLE INITIALLY DEFERRED REFERENCES temp_users (user_id);

DO $$
DECLARE
  v_row record;
  v_user_id integer;
BEGIN
  FOR v_row IN SELECT * FROM orders LOOP
    SELECT user_id INTO v_user_id FROM temp_users WHERE id = v_row.user_id;
    IF NOT FOUND THEN
      RAISE EXCEPTION 'Invalid reference: Order with ID % has an invalid user', v_row.id;
    END IF;
  END LOOP;
END $$;

ALTER TABLE orders DROP CONSTRAINT fk_order_user;
ALTER TABLE orders ADD CONSTRAINT fk_order_user REFERENCES temp_users (user_id) INITIALLY IMMEDIATE NOT DEFERRABLE;

-- Output: Successfully added the foreign key constraint without locking the 'orders' table for long periods.
```

**Flow:**

1. Identify Source Table
2. Create Temp Table
3. Add Temp Constraint
4. Validate Data
5. Drop & Re-add FinalConstraint
6. Constraint Added Successfully

