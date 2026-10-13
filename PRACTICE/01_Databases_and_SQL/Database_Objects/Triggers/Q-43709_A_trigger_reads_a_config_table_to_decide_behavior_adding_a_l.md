---
type: "practice"
difficulty: "Deep"
---

# A trigger reads a config table to decide behavior, adding a lookup per row; how do you cache or precompute that config to avoid per-row overhead?

**Difficulty:** Deep · **Levels:** SDE-1, SDE-1/2 · [↑ Triggers Behavior Performance and Alternatives](<../../../../01_Databases_and_SQL/Database_Objects/Triggers/Triggers_Behavior_Performance_and_Alternatives.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the issue of reading a config table in triggers and avoiding per-row overhead, we need to understand how triggers work and their limitations. Triggers are database objects designed to automatically execute when a specific event occurs on a particular table, such as an insert, update, or delete operation. However, these operations can become inefficient if they require complex logic involving multiple rows from another table. To mitigate this, we can use caching or precomputation strategies.

First, let's break down the problem: our trigger needs to read a config table to decide its behavior on every row insert/update/delete operation. This leads to repeated lookups and potentially significant overhead, especially if the config table is large or frequently updated. By caching or precomputing this configuration data, we can reduce the per-row overhead.

Now, let's explore different strategies for implementing this solution:

In-Trigger Caching: We can cache the necessary configuration data within a single trigger by storing it in a temporary variable or a session-specific memory.
Materialized View: Create a materialized view of the config table that precomputes and stores the required configurations, reducing the need for repeated lookups.
Database-Level Caching: Utilize database caching mechanisms like Redis or Memcached to store frequently accessed configuration data.
In-Trigger Caching

In this approach, we would cache the necessary configuration within the trigger itself. Here’s an example:

SQL
CREATE OR REPLACE FUNCTION my_trigger_function()
RETURNS TRIGGER AS $$
DECLARE
    config_data record;
BEGIN
    -- Simulate caching or precomputing data here
    SELECT * INTO config_data FROM config_table WHERE id = NEW.config_id;

    -- Use the cached data to decide behavior
    IF config_data.active THEN
        -- Perform some action based on the configuration
    ELSE
        -- Handle inactive case
    END IF;

    RETURN NEW;  -- or OLD, depending on the operation
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER my_trigger
AFTER INSERT ON my_table
FOR EACH ROW EXECUTE FUNCTION my_trigger_function();
Materialized View

Creating a materialized view can significantly reduce the overhead of repeated lookups. Here’s an example:

SQL
-- Create the materialized view
CREATE MATERIALIZED VIEW config_view AS
SELECT id, active, setting1, setting2, ...
FROM config_table;

-- Optionally refresh the materialized view periodically
REFRESH MATERIALIZED VIEW config_view;

Then, in your trigger, you can query this materialized view:

SQL
CREATE OR REPLACE FUNCTION my_trigger_function()
RETURNS TRIGGER AS $$
DECLARE
    config_data record;
BEGIN
    -- Query the materialized view for configuration data
    SELECT * INTO config_data FROM config_view WHERE id = NEW.config_id;

    -- Use the cached data to decide behavior
    IF config_data.active THEN
        -- Perform some action based on the configuration
    ELSE
        -- Handle inactive case
    END IF;

    RETURN NEW;  -- or OLD, depending on the operation
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER my_trigger
AFTER INSERT ON my_table
FOR EACH ROW EXECUTE FUNCTION my_trigger_function();
Database-Level Caching (e.g., Redis)

Using an external caching solution like Redis can be highly effective for frequent lookups. Here’s how you might implement it:

Set up a Redis server and install the necessary database client.
Cache configuration data in Redis: When a row is updated or inserted, cache the relevant configuration to Redis.
SQL
-- Example of caching in Redis using PL/PythonU extension (PostgreSQL)
CREATE OR REPLACE FUNCTION cache_config(config_id integer) RETURNS void AS $$
import redis

r = redis.Redis(host='localhost', port=6379, db=0)

# Fetch config from the database
config_data = select_config_from_table(config_id)

# Cache in Redis
r.set(f'config_{config_id}', str(config_data))
$$ LANGUAGE plpythonu;

Then, modify your trigger to check Redis before querying the database:

SQL
CREATE OR REPLACE FUNCTION my_trigger_function()
RETURNS TRIGGER AS $$
DECLARE
    config_data record;
BEGIN
    -- Check if data is cached in Redis
    IF EXISTS (SELECT 1 FROM redis.cache WHERE key='config_' || NEW.config_id) THEN
        SELECT * INTO config_data FROM redis.cache WHERE key='config_' || NEW.config_id;

        -- Use the cached data to decide behavior
        IF config_data.active THEN
            -- Perform some action based on the configuration
        ELSE
            -- Handle inactive case
        END IF;
    ELSE
        -- Query the database for configuration data (as a fallback)
        SELECT * INTO config_data FROM config_table WHERE id = NEW.config_id;

        -- Cache in Redis and use it to decide behavior
        cache_config(NEW.config_id);

        -- Use the cached data to decide behavior
        IF config_data.active THEN
            -- Perform some action based on the configuration
        ELSE
            -- Handle inactive case
        END IF;
    END IF;

    RETURN NEW;  -- or OLD, depending on the operation
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER my_trigger
AFTER INSERT ON my_table
FOR EACH ROW EXECUTE FUNCTION my_trigger_function();

#### Closing Summary
and Common Mistakes

By caching or precomputing configuration data, we can significantly reduce per-row overhead in triggers. The most common mistakes include:

Inadequate Cache Management: Failing to refresh the cache when necessary.
Overlooking Database-Specific Caching Mechanisms: Not leveraging built-in database features like materialized views.
Ignoring External Caches: Forgetting external caching solutions, which can offer even better performance.

Each of these strategies has its trade-offs and is suitable for different use cases. The choice depends on the specific requirements and constraints of your application.

**Comparison:**

| ID | Active | Setting1 | Setting2 |
|----|--------|----------|---------|
| 1  | true   | value1   | 42      |
| 2  | false  | value2   | 84      |

**Code:**

```

-- In-Trigger Caching Example CREATE OR REPLACE FUNCTION my_trigger_function() RETURNS TRIGGER AS $$ DECLARE config_data record; BEGIN -- Simulate caching or precomputing data here SELECT * INTO config_data FROM config_table WHERE id = NEW.config_id;

CODE
Copy
-- Use the cached data to decide behavior
IF config_data.active THEN
    -- Perform some action based on the configuration
ELSE
    -- Handle inactive case
END IF;

RETURN NEW;  -- or OLD, depending on the operation

END; $$ LANGUAGE plpgsql;

CREATE TRIGGER my_trigger AFTER INSERT ON my_table FOR EACH ROW EXECUTE FUNCTION my_trigger_function();

CODE
Copy

```sql
-- Materialized View Example
CREATE MATERIALIZED VIEW config_view AS
SELECT id, active, setting1, setting2, ...
FROM config_table;

REFRESH MATERIALIZED VIEW config_view;
SQL
Copy
-- Database-Level Caching in Redis (PL/PythonU)
CREATE OR REPLACE FUNCTION cache_config(config_id integer) RETURNS void AS $$
import redis

r = redis.Redis(host='localhost', port=6379, db=0)

# Fetch config from the database
config_data = select_config_from_table(config_id)

# Cache in Redis
r.set(f'config_{config_id}', str(config_data))
$$ LANGUAGE plpythonu;
```

**Example input / output:**

```text
-- Example Input/Output for In-Trigger Caching
CREATE OR REPLACE FUNCTION my_trigger_function()
RETURNS TRIGGER AS $$
DECLARE
    config_data record;
BEGIN
    -- Simulate caching or precomputing data here
    SELECT * INTO config_data FROM config_table WHERE id = NEW.config_id;

    -- Use the cached data to decide behavior
    IF config_data.active THEN
        RAISE NOTICE 'Active configuration found';
    ELSE
        RAISE NOTICE 'Inactive configuration found';
    END IF;

    RETURN NEW;  -- or OLD, depending on the operation
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER my_trigger
AFTER INSERT ON my_table
FOR EACH ROW EXECUTE FUNCTION my_trigger_function();

-- Example Input/Output for Materialized View
REFRESH MATERIALIZED VIEW config_view;

SELECT * INTO config_data FROM config_view WHERE id = 1;
RAISE NOTICE 'Config data: %', config_data;
```

**Data:**

```sql
-- Sample Data for Materialized View
INSERT INTO config_table (id, active, setting1, setting2) VALUES (1, true, 'value1', 42);
INSERT INTO config_table (id, active, setting1, setting2) VALUES (2, false, 'value2', 84);
```

**Flow:**

1. Yes
2. No
3. Active
4. Inactive
5. Active
6. Inactive
7. Check Redis Cache
8. Cache Present?
9. Use Cached Data
10. Query Database forConfiguration
11. Perform Action Based onConfig
12. Action Based on Active Config
13. Handle Inactive Case
14. Cache in Redis and Use It toDecide Behavior
15. Perform Action Based onConfig
16. Action Based on Active Config
17. Handle Inactive Case

