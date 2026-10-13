---
type: "practice"
difficulty: "Basic"
---

# How would you write a query to add a computed load_timestamp column so every row in the warehouse records when it was ingested?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Timezones and Timestamps](<../../../../01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Timezones_and_Timestamps.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To solve this problem, we need to write an SQL query that adds a load_timestamp column to each record in our warehouse table. This timestamp will capture the exact moment of ingestion, which is crucial for tracking data freshness and auditing purposes. The core idea here is to use a built-in function or setting in your database management system (DBMS) to automatically insert the current timestamp whenever a new row is inserted.

**Comparison:**

```
Before Insert	After Insert
product_id	101
product_name	Widget
load_timestamp	NULL
```

**Code:**

```

-- Assuming we are using PostgreSQL as our DBMS, which provides an auto_now_insert feature through the CURRENT_TIMESTAMP function

INSERT INTO warehouse_table (column1, column2, ..., load_timestamp) VALUES ('value1', 'value2', ..., CURRENT_TIMESTAMP);

CODE
Copy
Note that if your DBMS does not have a built-in function for capturing current timestamp on insert, we can use a trigger or an alternative approach. However, the exact method will vary depending on the database system.

Here is how you might implement this using a trigger in PostgreSQL:

```sql
-- Create a trigger to automatically set load_timestamp when inserting data

CREATE TRIGGER set_load_timestamp
BEFORE INSERT ON warehouse_table
FOR EACH ROW
EXECUTE FUNCTION set_load_timestamp_function();

-- Function definition (PostgreSQL specific)
CREATE OR REPLACE FUNCTION set_load_timestamp_function()
RETURNS TRIGGER AS $$
BEGIN
    NEW.load_timestamp := CURRENT_TIMESTAMP;
    RETURN NEW;
END;
$$ LANGUAGE plpgsql;
```

**Example input / output:**

```

For example, if we insert a new row into the warehouse_table:

SQL
Copy
INSERT INTO warehouse_table (product_id, product_name, load_timestamp)
VALUES (101, 'Widget', CURRENT_TIMESTAMP);

The output will include the load_timestamp populated with the current timestamp when the row was inserted.
```

**Flow:**

1. Yes
2. No
3. New Row Inserted
4. Is DBMS with auto_now_insert?
5. Use CURRENT_TIMESTAMP inVALUES
6. Create Trigger ->set_load_timestamp_function()
7. Insert Row withload_timestamp set byfunction

