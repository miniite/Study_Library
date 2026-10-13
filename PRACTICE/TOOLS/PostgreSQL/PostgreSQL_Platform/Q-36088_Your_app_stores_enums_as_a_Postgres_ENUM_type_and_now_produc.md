---
type: "practice"
difficulty: "Intermediate"
---

# Your app stores enums as a Postgres ENUM type and now product wants to remove a value that's still referenced; explain why removing enum values is hard, and how you'd migrate to a lookup table to gain flexibility.

**Difficulty:** Intermediate · **Levels:** SDE-1/2, SDE-2/3 · [↑ DELETE TRUNCATE and DROP](<../../../../TOOLS/PostgreSQL/PostgreSQL_Platform/DELETE_TRUNCATE_and_DROP.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When working with PostgreSQL enums, it can be challenging to modify or delete an existing value because enums are tightly integrated into your application's schema and query logic. This challenge arises from the fact that removing a value means all references to that value must be updated, which is cumbersome when dependencies exist across multiple tables.

To illustrate this, let’s assume you have an orders table with a column status that uses an enum type:

SQL
CREATE TYPE order_status AS ENUM ('pending', 'shipped', 'delivered');

If the product team wants to remove the value 'delivered', it becomes complex because other parts of your application might be using this value in various places, such as views, triggers, or even third-party integrations.

The key steps we would take to migrate from an enum to a lookup table are as follows:

Identify All Dependencies: First, identify all tables and columns where the enum value is used.
Create a Lookup Table: Create a new table that will store the values dynamically.
Migrate Existing Data: Migrate the existing data from the enum to this lookup table.
Update References: Update all references in your application schema and code to use the new lookup table instead of the enum type.
Drop the Enum Type: Once everything is migrated, you can drop the old enum type.

Let’s go through each step in detail:

Step 1: Identify All Dependencies

Run a SQL query like:

SQL
SELECT column_name, table_name FROM information_schema.columns WHERE data_type LIKE '%enum%' AND table_name IN ('orders', 'other_tables');

This will list all the tables and columns where enums are used.

Step 2: Create a Lookup Table

Create a new lookup table that stores the possible values:

SQL
CREATE TABLE order_status_lookup (
    id SERIAL PRIMARY KEY,
    status_name TEXT UNIQUE NOT NULL
);
INSERT INTO order_status_lookup (status_name) VALUES ('pending'), ('shipped');
Step 3: Migrate Existing Data

Update your orders table to use the new lookup table:

SQL
ALTER TABLE orders DROP COLUMN status;
ALTER TABLE orders ADD COLUMN status_id INTEGER REFERENCES order_status_lookup(id);
UPDATE orders SET status_id = (SELECT id FROM order_status_lookup WHERE status_name = 'delivered');
Step 4: Update References

Update all other parts of your application to use the new status_id and reference the lookup table.

Step 5: Drop the Enum Type

Once everything is migrated, you can drop the old enum type:

SQL
DROP TYPE order_status;

#### Common Mistakes
and Follow-Ups
Incomplete Migration: Not thoroughly checking all dependencies could lead to data inconsistencies or logical errors.
Performance Impact: The lookup table approach might introduce slight performance overhead for lookups compared to enums, especially if the table is large.
Data Consistency: Ensure that any transactions that might rely on the enum values are properly adjusted to handle the new lookup mechanism.

In summary, migrating from an enum to a lookup table involves identifying dependencies, creating and populating a new lookup table, updating references, and finally dropping the old enum type. This approach provides flexibility while maintaining data integrity and minimizing disruptions.

**Code:**

```

-- Step 2: Create Lookup Table CREATE TABLE order_status_lookup ( id SERIAL PRIMARY KEY, status_name TEXT UNIQUE NOT NULL );

-- Insert initial values INSERT INTO order_status_lookup (status_name) VALUES ('pending'), ('shipped');

CODE
Copy

```sql
-- Step 3: Migrate Existing Data
ALTER TABLE orders DROP COLUMN status;
ALTER TABLE orders ADD COLUMN status_id INTEGER REFERENCES order_status_lookup(id);
UPDATE orders SET status_id = (SELECT id FROM order_status_lookup WHERE status_name = 'delivered');
```

