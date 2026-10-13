---
type: "practice"
difficulty: "Intermediate"
---

# You need to model soft-deletable rows but a foreign key from another table still points at them; how do you keep referential integrity working while hiding deleted rows from normal queries?

**Difficulty:** Intermediate · **Levels:** SDE-1, SDE-1/2 · [↑ DELETE TRUNCATE and DROP](<../../../../TOOLS/PostgreSQL/PostgreSQL_Platform/DELETE_TRUNCATE_and_DROP.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In PostgreSQL, when dealing with soft deletable rows where there's a foreign key pointing to these rows, we need to ensure that the referential integrity is maintained while still being able to hide deleted rows from regular queries. This involves creating a mechanism for marking rows as deleted without actually removing them and ensuring that any query that should see "deleted" rows will be filtered out.

The core idea here is to use a boolean flag in the table to indicate whether a row has been deleted, and then modify the query logic to exclude these rows from views or specific queries. Additionally, we'll need to adjust foreign key constraints so they respect this flag.

#### Triage
Understanding the Problem
What the question asks: How to implement soft deletable rows in PostgreSQL while keeping referential integrity intact.

#### Assumptions
The table with the foreign key references exists and contains actual data. The deletion process should not physically delete the row but simply mark it as deleted.
Key probe: Ensuring that a foreign key constraint is respected even when a referenced row is marked as soft-deleted.

#### First Principles Solution

Add a Soft Delete Column:

Add a is_deleted column to the table where you need soft delete functionality. This boolean flag will be used to mark rows for deletion.

Create Views or Custom Queries:

Use views or custom queries that exclude rows marked as deleted when returning data.

Adjust Foreign Key Constraints:

Modify foreign key constraints to ignore rows where is_deleted is true.

Modify Insert/Update/Delete Logic:

When deleting a row, update the is_deleted flag instead of physically removing the row from the table.

Ensure Data Integrity:

Make sure that any new insertions or updates to foreign key-referencing tables do not violate the soft delete condition.
Detailed Steps

Schema Definition:

SQL
ALTER TABLE your_table
ADD COLUMN is_deleted BOOLEAN NOT NULL DEFAULT FALSE;

Create a View for Soft Delete:

Create a view that excludes deleted rows.
SQL
CREATE OR REPLACE VIEW active_rows_view AS
SELECT * FROM your_table WHERE is_deleted = FALSE;

Modify Foreign Key Constraints:

Ensure foreign key constraints use a WHERE clause to exclude soft-deleted rows.
SQL
ALTER TABLE referencing_table
ADD CONSTRAINT fk_active_rows FOREIGN KEY (your_column)
REFERENCES your_table(your_column) MATCH SIMPLE
ON UPDATE NO ACTION
ON DELETE CASCADE
DEFERRABLE INITIALLY DEFERRED;

Insertion and Update Logic:

Modify the logic in your application to update is_deleted instead of deleting rows.
SQL
-- Example: Mark a row as deleted
UPDATE your_table SET is_deleted = TRUE WHERE id = 123;

-- Example: Insert with a check for non-deleted row
INSERT INTO your_table (column1, column2) VALUES ('value1', 'value2')
ON CONFLICT (your_column) DO NOTHING;

Testing and Validation:

Validate that foreign key constraints respect the is_deleted flag.
SQL
-- Test insert with a deleted row
INSERT INTO your_table (column1, column2) VALUES ('value1', 'value2')
ON CONFLICT (your_column) DO NOTHING;  -- Should fail if there's an active reference

-- Test delete and insertion
UPDATE your_table SET is_deleted = TRUE WHERE id = 456;
INSERT INTO referencing_table (reference_id, ...) VALUES (456, ...);  -- Should succeed now

#### Closing Summary
and Common Mistakes
Summary: By adding a is_deleted column and adjusting queries to exclude deleted rows, we can maintain referential integrity while soft deleting data.

#### Common Mistakes
Forgetting to adjust foreign key constraints to ignore deleted rows.
Not properly validating the deletion process in your application logic.
Misconfiguring views or custom queries that should exclude deleted rows.

**Comparison:**

| Column | Type     |
|--------|----------|
| id     | SERIAL   |
| name   | VARCHAR  |
| is_deleted | BOOLEAN  |

**Code:**

```sql
-- Adding a soft delete column
ALTER TABLE your_table ADD COLUMN is_deleted BOOLEAN NOT NULL DEFAULT FALSE;

-- Creating a view to filter out deleted rows
CREATE OR REPLACE VIEW active_rows_view AS
SELECT * FROM your_table WHERE is_deleted = FALSE;

-- Modifying foreign key constraint
ALTER TABLE referencing_table
ADD CONSTRAINT fk_active_rows FOREIGN KEY (your_column)
REFERENCES your_table(your_column) MATCH SIMPLE
ON UPDATE NO ACTION
ON DELETE CASCADE
DEFERRABLE INITIALLY DEFERRED;
```

**Example input / output:**

```text
-- Example input and output for a soft delete operation
INSERT INTO your_table (name, is_deleted) VALUES ('Example', FALSE);

SELECT * FROM active_rows_view;  -- Returns the above row

UPDATE your_table SET is_deleted = TRUE WHERE id = 1;

SELECT * FROM active_rows_view;  -- No rows returned now
```

**Data:**

```sql
-- Sample data schema
CREATE TABLE your_table (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100),
    is_deleted BOOLEAN NOT NULL DEFAULT FALSE
);

CREATE TABLE referencing_table (
    id SERIAL PRIMARY KEY,
    reference_id INTEGER REFERENCES your_table(id) MATCH SIMPLE ON UPDATE NO ACTION ON DELETE CASCADE DEFERRABLE INITIALLY DEFERRED
);
```

**Flow:**

1. Yes
2. No
3. Mark Row as Deleted
4. Is row referenced?
5. Delete from referencing table
6. Update `is_deleted` flag toTRUE
7. Mark Referencing rows asdeleted if necessary

