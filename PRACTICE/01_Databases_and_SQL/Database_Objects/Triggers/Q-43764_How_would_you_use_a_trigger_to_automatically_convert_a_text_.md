---
type: "practice"
difficulty: "Intermediate"
---

# How would you use a trigger to automatically convert a text field to uppercase before storing it?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ Triggers Behavior Performance and Alternatives](<../../../../01_Databases_and_SQL/Database_Objects/Triggers/Triggers_Behavior_Performance_and_Alternatives.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address this scenario, we'll first need to understand what triggers are and how they can be used in database systems like MySQL or PostgreSQL. Triggers are special types of stored procedures that automatically execute when a specific event occurs on a table (like an INSERT or UPDATE). In this case, we want to use a trigger to ensure that any text input for a particular column is converted to uppercase before it gets inserted into the database.

#### Triaging the Question

The question asks us to implement a trigger that converts a given text field to uppercase. The key points here are understanding when and how to apply this transformation, ensuring data consistency, and handling edge cases such as null values or empty strings.

#### First-Principles Solution
Identify the Table and Column: Determine which table and column we need to monitor for changes.
Define the Trigger Type: Choose between BEFORE (to modify data before insertion) or AFTER (to log or audit changes).
Write the SQL Statement: Create a trigger that listens for specific events on the column, converts the input to uppercase, and updates the row accordingly.
Handle Edge Cases: Ensure null values are handled appropriately.

Here’s an example using PostgreSQL:

SQL
-- Step 1: Identify the table and column
CREATE TABLE example_table (
    id SERIAL PRIMARY KEY,
    name TEXT NOT NULL
);

-- Step 2: Define the trigger type (BEFORE INSERT)
DO $$
BEGIN
    CREATE OR REPLACE FUNCTION uppercase_name_trigger()
    RETURNS TRIGGER AS $$
    BEGIN
        NEW.name := UPPER(NEW.name);
        RETURN NEW;
    END;
    $$ LANGUAGE plpgsql;

    -- Create the trigger that uses this function
    CREATE TRIGGER before_insert_example_table
    BEFORE INSERT ON example_table
    FOR EACH ROW EXECUTE FUNCTION uppercase_name_trigger();
END $$;

#### Closing Summary
and Common Mistakes/Follow-Ups
Summary

We have successfully implemented a trigger in PostgreSQL to convert any text input into uppercase. This ensures that the name field is always stored in uppercase, maintaining data consistency.

#### Common Mistakes
and Follow-Ups
Not Handling Null Values: Ensure you handle cases where the name might be null or empty.

#### Performance Considerations
Be mindful of performance implications with large tables, as triggers can add overhead.
Security Concerns: Make sure that any changes made by the trigger do not violate data integrity rules.
Additional Questions
How would you modify this approach if you needed to handle multiple columns?
What are the potential impacts on other database operations (e.g., UPDATE operations)?
Are there any performance optimization techniques you could apply?

**Code:**

```sql
-- SQL code for creating a trigger in PostgreSQL
DO $$
BEGIN
    CREATE OR REPLACE FUNCTION uppercase_name_trigger()
    RETURNS TRIGGER AS $$
    BEGIN
        NEW.name := UPPER(NEW.name);
        RETURN NEW;
    END;
    $$ LANGUAGE plpgsql;

    -- Create the trigger that uses this function
    CREATE TRIGGER before_insert_example_table
    BEFORE INSERT ON example_table
    FOR EACH ROW EXECUTE FUNCTION uppercase_name_trigger();
END $$;
```

**Example input / output:**

```text
INSERT INTO example_table (name) VALUES ('john doe');
SELECT * FROM example_table;
-- Output:
-- id |  name
-- ---|--------
-- 1  | JOHN DOE
```

**Data:**

```sql
-- Sample data schema for example_table
CREATE TABLE example_table (
    id SERIAL PRIMARY KEY,
    name TEXT NOT NULL
);
```

