---
type: "practice"
difficulty: "Intermediate"
---

# How would you design and index a schema for storing user preferences as key-value pairs while keeping 'get all prefs for a user' a single fast lookup?

**Difficulty:** Intermediate · **Levels:** SDE-1, SDE-1/2 · [↑ Activity Timeline Schema](<../../../../TOOLS/MySQL/MySQL_Platform/Activity_Timeline_Schema.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To design an efficient schema in MySQL for storing user preferences as key-value pairs with fast lookups, we need to balance between simplicity and performance. The core idea is to use a normalized table structure while ensuring that retrieving all preferences for a single user is a fast operation.

The key steps are:

Create a users table to store user information.
Design a preferences table to hold the key-value pairs for each preference.
Ensure efficient indexing on the user_id and key columns of the preferences table.
Optionally, use additional queries or caches to optimize performance further.

Here’s how we can implement these steps in detail:

Users Table: Stores basic user information.
Preferences Table: Holds key-value pairs for each preference.
Indexing Strategy: Use composite indexes on the user_id and key columns to enable fast lookups.
Query Optimization: Implement efficient queries to get all preferences for a single user.

**Code:**

```sql
-- Step 1: Create Users Table
CREATE TABLE users (
    id INT AUTO_INCREMENT PRIMARY KEY,
    username VARCHAR(255) NOT NULL UNIQUE,
    -- Other fields as needed
);

-- Step 2: Create Preferences Table
CREATE TABLE preferences (
    user_id INT,
    key VARCHAR(255) NOT NULL,
    value TEXT,  -- Can be JSON or serialized string based on preference type
    PRIMARY KEY (user_id, key),
    FOREIGN KEY (user_id) REFERENCES users(id)
);
```

**Example input / output:**

```text
-- Example of a query to get all preferences for user with id=1
SELECT * FROM preferences WHERE user_id = 1;

+--------+----------+-----------------+
| user_id| key      | value           |
+--------+----------+-----------------+
|      1 | theme    | dark            |
|      1 | notifications | true          |
+--------+----------+-----------------+
```

**Data:**

```json
{
    "users": [
        {"id": 1, "username": "john_doe"},
        {"id": 2, "username": "jane_smith"}
    ],
    "preferences": [
        {"user_id": 1, "key": "theme", "value": "dark"},
        {"user_id": 1, "key": "notifications", "value": "true"},
        {"user_id": 2, "key": "language", "value": "en"}
    ]
}
```

