---
type: "practice"
difficulty: "Deep"
---

# How do you enforce referential integrity between a relational table and a JSONB column holding an array of child IDs? Discuss the trade-offs.

**Difficulty:** Deep · **Levels:** SDE-1, SDE-1/2 · [↑ Foreign Keys](<../../../../01_Databases_and_SQL/Database_Design/Constraints_and_Keys/Foreign_Keys.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To enforce referential integrity between a relational table and a JSONB column holding an array of child IDs, we need to ensure that all values in the JSONB column are valid references to existing rows in another table. This is challenging because JSONB columns don't inherently support traditional foreign key constraints. We'll explore different approaches, their trade-offs, and why one might be more suitable than others.

#### Triaging the Question

The core of the question revolves around enforcing referential integrity between a relational database table and a JSONB column containing an array of IDs. The interviewer is probing your understanding of constraints in SQL databases, the limitations of JSON data types, and potential solutions to integrate referential integrity checks.

#### Assumptions
We are working with PostgreSQL, which supports JSONB columns.
The child IDs stored in the JSONB column correspond to primary keys (PK) of another table.
The goal is to ensure that these child IDs always reference valid records in the parent table.

#### First Principles Solution

#### Approach
1: Database Triggers

One approach is to use database triggers to enforce constraints. We can create a trigger function that validates each element in the JSONB array against the primary key values of the referenced table before inserting or updating rows.

SQL
CREATE OR REPLACE FUNCTION validate_jsonb_ids()
RETURNS TRIGGER AS $$
BEGIN
    -- Assuming the JSONB column name is 'child_ids' and the parent table's PK is 'id'
    PERFORM 1 FROM jsonb_array_elements_text(NEW.child_ids)
             WHERE value::integer NOT IN (SELECT id FROM referenced_table);
    IF FOUND THEN
        RAISE EXCEPTION 'Invalid child ID in JSONB array';
    END IF;
    RETURN NEW;
END;
$$ LANGUAGE plpgsql;

-- Create a trigger that calls the function before insert or update operations
CREATE TRIGGER validate_jsonb_ids_trigger
BEFORE INSERT OR UPDATE ON main_table
FOR EACH ROW EXECUTE FUNCTION validate_jsonb_ids();

#### Approach
2: Application-Level Validation

Another approach is to perform validation at the application level. This involves writing code in your application logic to check that all JSONB values are valid before inserting or updating records.

PYTHON
def validate_jsonb_ids(ids, referenced_table):
    existing_ids = set(row['id'] for row in referenced_table)
    invalid_ids = set(ids) - existing_ids
    if invalid_ids:
        raise ValueError(f"Invalid child IDs: {invalid_ids}")

# Example usage
validate_jsonb_ids([1, 2, 3], session.query(ReferencedTable).all())

#### Approach
3: Materialized Views and Periodic Checks

For large datasets, materialized views can be used to periodically validate the JSONB column against a reference table. This approach is more suitable for less frequent updates.

SQL
CREATE MATERIALIZED VIEW jsonb_validation AS
SELECT j.id, j.child_ids FROM main_table m, LATERAL (SELECT * FROM jsonb_array_elements_text(m.child_ids)) j(id);

#### Trade-offs

Database Triggers:

#### Pros
Enforces constraints directly in the database, reducing application complexity.

#### Cons
Can impact performance due to trigger overhead and potential deadlocks.

Application-Level Validation:

#### Pros
No additional database schema changes required, easier to implement.

#### Cons
Adds extra logic in the application layer, which must be maintained and tested.

Materialized Views:

#### Pros
Reduces load on transactional tables by validating periodically.

#### Cons
Requires maintenance of materialized views and triggers to keep them up-to-date.

#### Closing Summary

To enforce referential integrity between a relational table and a JSONB column, database triggers provide the most robust solution but may introduce performance overhead. Application-level validation is simpler but requires careful handling in code. Materialized views offer a balance by reducing transactional load at the cost of additional maintenance.

#### Common Mistakes/Follow-Ups
Mistake: Ignoring the need for referential integrity in JSONB columns.

#### Follow-up
Discussing performance implications and choosing between real-time validation (triggers) or periodic checks (materialized views).

**Comparison:**

```
id	name
1	Child 1
2	Child 2
```

**Code:**

```sql
CREATE OR REPLACE FUNCTION validate_jsonb_ids()
RETURNS TRIGGER AS $$
BEGIN
    -- Assuming the JSONB column name is 'child_ids' and the parent table's PK is 'id'
    PERFORM 1 FROM jsonb_array_elements_text(NEW.child_ids)
             WHERE value::integer NOT IN (SELECT id FROM referenced_table);
    IF FOUND THEN
        RAISE EXCEPTION 'Invalid child ID in JSONB array';
    END IF;
    RETURN NEW;
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER validate_jsonb_ids_trigger
BEFORE INSERT OR UPDATE ON main_table
FOR EACH ROW EXECUTE FUNCTION validate_jsonb_ids();
```

**Example input / output:**

```text
# Python validation example
from sqlalchemy.orm import sessionmaker

Session = sessionmaker(bind=engine)
session = Session()

def validate_jsonb_ids(ids, referenced_table):
    existing_ids = set(row['id'] for row in referenced_table)
    invalid_ids = set(ids) - existing_ids
    if invalid_ids:
        raise ValueError(f"Invalid child IDs: {invalid_ids}")

# Example usage
validate_jsonb_ids([1, 2, 3], session.query(ReferencedTable).all())
```

**Data:**

```sql
-- Example referenced table schema
CREATE TABLE referenced_table (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100)
);
```

**Flow:**

1. Yes
2. No
3. Trigger Function
4. Is the ID in JSONB array valid?
5. Insert/Update allowed
6. Raise Exception

