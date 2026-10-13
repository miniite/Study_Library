---
type: "practice"
difficulty: "Deep"
---

# You need to run a data-fix that emits one summary log per statement not per row; how do statement-level triggers with transition tables (REFERENCING NEW TABLE) let you process the whole affected set at once?

**Difficulty:** Deep · **Levels:** SDE-1, SDE-1/2 · [↑ Materialized Views vs Incremental Aggregates](<../../../../TOOLS/PostgreSQL/PostgreSQL_Platform/Materialized_Views_vs_Incremental_Aggregates.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the need to run a data-fix that emits one summary log per statement, rather than per row, we can leverage PostgreSQL's statement-level triggers with transition tables (REFERENCING NEW TABLE). This approach allows us to process the entire set of affected rows in a single operation. Here’s how it works:

Firstly, let's understand what we're trying to achieve: We want to log changes to a table at the statement level rather than row by row. By using statement-level triggers and transition tables, we can capture all the changed data in one go before processing them.

To illustrate this concept, let’s assume we have a products table that tracks product inventory levels. Our goal is to create a trigger that logs all changes to the price column when any row is updated.

#### Triaging the Question

The core of the question revolves around how statement-level triggers with transition tables can help us process multiple rows in one operation, thus emitting one summary log per statement rather than per row. The key idea here is understanding the difference between row-level and statement-level triggers and leveraging REFERENCING NEW TABLE to gather all changed data.

#### First-Principles Solution
Identify the Target Table: We start by identifying the table where we want to implement our changes, which in this case is the products table.
Create a Transition Table: A transition table captures the state of the rows before and after they are modified. This allows us to process all changes as a set rather than individually.
Statement-Level Trigger: We define a trigger that fires on the UPDATE operation for the target table, using the REFERENCING NEW TABLE clause to reference the updated data.
Step-by-Step Solution

Define the Transition Table:

SQL
CREATE TABLE products_log (
    id INT,
    old_price NUMERIC,
    new_price NUMERIC,
    changed_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

Create a Statement-Level Trigger: The trigger will log all updated rows into products_log and then process them as a batch.

SQL
CREATE OR REPLACE FUNCTION update_product_prices() RETURNS TRIGGER AS $$
  BEGIN
    -- Insert the old and new prices for each row being updated
    INSERT INTO products_log (id, old_price, new_price)
    SELECT id, price, NEW.price FROM NEW TABLE;

    -- Process the log entries as a batch
    UPDATE products SET some_attribute = 'processed' WHERE id IN (SELECT id FROM products_log);

    RETURN NULL;
  END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER log_product_updates
AFTER UPDATE ON products
FOR EACH STATEMENT EXECUTE FUNCTION update_product_prices();

#### Closing Summary
and Common Mistakes/Follow-Ups
Summary: By using statement-level triggers with REFERENCING NEW TABLE, we can process multiple rows in a single operation, emitting one summary log per statement.

#### Common Mistakes
Misunderstanding the difference between row-level and statement-level triggers.
Failing to use REFERENCING NEW TABLE correctly to capture all updated data.
Not properly handling the transition table to process changes as a batch.

#### Follow-ups
could include discussing performance considerations, transaction management, and how this approach scales with large datasets.

**Code:**

```sql
CREATE TABLE products_log (
    id INT,
    old_price NUMERIC,
    new_price NUMERIC,
    changed_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE OR REPLACE FUNCTION update_product_prices() RETURNS TRIGGER AS $$
BEGIN
  -- Insert the old and new prices for each row being updated
  INSERT INTO products_log (id, old_price, new_price)
  SELECT id, price, NEW.price FROM NEW TABLE;

  -- Process the log entries as a batch
  UPDATE products SET some_attribute = 'processed' WHERE id IN (SELECT id FROM products_log);

  RETURN NULL;
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER log_product_updates
AFTER UPDATE ON products
FOR EACH STATEMENT EXECUTE FUNCTION update_product_prices();
```

**Example input / output:**

```sql
-- Example input/output
INSERT INTO products (id, price) VALUES (1, 20.5);

-- Update the product's price
UPDATE products SET price = 30.0 WHERE id = 1;

-- Check the log table for updated entries
SELECT * FROM products_log;
id	old_price	new_price	changed_at
1	20.5	30.0	2023-10-01 12:00
```

