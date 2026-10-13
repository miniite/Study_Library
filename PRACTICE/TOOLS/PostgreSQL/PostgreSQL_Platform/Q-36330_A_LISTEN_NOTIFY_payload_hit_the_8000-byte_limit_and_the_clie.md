---
type: "practice"
difficulty: "Intermediate"
---

# A LISTEN/NOTIFY payload hit the 8000-byte limit and the client got truncated data; how would you restructure to send only an id in the payload and fetch the row, and why is that more robust?

**Difficulty:** Intermediate · **Levels:** SDE-1, SDE-1/2 · [↑ Replica Lag and Read Your Writes](<../../../../TOOLS/PostgreSQL/PostgreSQL_Platform/Replica_Lag_and_Read_Your_Writes.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When dealing with a LISTEN/NOTIFY payload that exceeds 8000 bytes, truncation can occur, leading to corrupted data at the client end. The goal here is to design a system where only an ID is sent in the notification payload, and the relevant row details are fetched from the database whenever a notification is received. This approach enhances robustness by ensuring that no critical data is lost during transmission.

#### Triage

The core of the problem involves restructuring the NOTIFY payload to contain only the minimal information required—specifically an ID—and then fetching the full row details using this ID. The interviewer might be probing your understanding of PostgreSQL's messaging system, your ability to handle edge cases, and how you would implement such a design robustly.

#### Assumptions
You have a table with rows that need to be updated via notifications.
The ID is unique for each row in the table.
You can use a separate query to fetch the full row details based on this ID.

#### Solution
From First Principles

Define a Table: Let's assume you have a simple users table where you want to notify when certain updates occur.

SQL
CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    name VARCHAR(255),
    email VARCHAR(255)
);

Set Up Listen/Notify: You need to listen for notifications on a specific channel, say user_updates.

Send Notification with ID Only: When an update occurs, you notify the client with only the ID.

SQL
-- Example: Update user 10
UPDATE users SET name = 'John Doe' WHERE id = 10;

-- Notify clients about this update
PERFORM pg_notify('user_updates', '10');

Listen for Notifications: On the client side, listen to the user_updates channel and handle incoming notifications.

SQL
LISTEN user_updates;

-- Client-side trigger function or query
CREATE OR REPLACE FUNCTION notify_handler()
  RETURNS event_trigger AS $$
BEGIN
    -- Fetch full row details using the ID from the notification payload
    RETURN QUERY EXECUTE 'SELECT * FROM users WHERE id = ' || pg_event_data();
END;
$$ LANGUAGE plpgsql;

-- Register the trigger on the database server
CREATE EVENT TRIGGER notify_trigger ON sql_drop
INDEPENDENTLY NOT DISABLE
EXECEDURE notify_handler();

-- Alternatively, if you want to listen for specific updates
LISTEN user_updates;

-- Client-side query to handle notifications
SELECT pg_event_data() FROM pg_event_trigger('notify_trigger');

Fetching Full Row Details: When the client receives a notification, it can use the ID in the payload to fetch the full row details from the database.

#### Closing Summary
and Common Mistakes/Follow-Ups

This design ensures that only necessary data is transmitted over the wire, reducing the risk of truncation issues. By fetching the full row details using the ID, you avoid sending potentially large amounts of data. However, this introduces an additional round-trip to the database, which might impact performance in scenarios with high-frequency updates.

#### Common Mistakes

Forgetting to handle edge cases where notifications are missed due to client disconnection.
Not considering concurrent transactions that could update the same row while a notification is being processed.
Overlooking potential issues with SQL injection if dynamic SQL construction isn't carefully handled.

#### Follow-Ups

How would you optimize this setup for scenarios with very high update rates?
Can you implement a more efficient mechanism to ensure notifications are not missed in case of disconnection?

**Code:**

```sql
-- Example notification payload handling
CREATE OR REPLACE FUNCTION notify_handler()
  RETURNS event_trigger AS $$
BEGIN
    -- Fetch full row details using the ID from the notification payload
    RETURN QUERY EXECUTE 'SELECT * FROM users WHERE id = ' || pg_event_data();
END;
$$ LANGUAGE plpgsql;

-- Register the trigger on the database server
CREATE EVENT TRIGGER notify_trigger ON sql_drop
INDEPENDENTLY NOT DISABLE
EXECUTE FUNCTION notify_handler();

-- Client-side query to handle notifications
LISTEN user_updates;

SELECT pg_event_data() FROM pg_event_trigger('notify_trigger');
```

**Example input / output:**

```text
-- Example input and output
UPDATE users SET name = 'John Doe' WHERE id = 10;

PERFORM pg_notify('user_updates', '10');

-- Client-side response handling
SELECT * FROM users WHERE id = 10;
```

**Data:**

```json
{
  "users": [
    {
      "id": 1,
      "name": "Alice",
      "email": "alice@example.com"
    },
    {
      "id": 2,
      "name": "Bob",
      "email": "bob@example.com"
    }
  ]
}
```

