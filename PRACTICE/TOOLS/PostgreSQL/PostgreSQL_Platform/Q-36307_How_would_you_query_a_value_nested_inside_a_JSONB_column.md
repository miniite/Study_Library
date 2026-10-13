---
type: "practice"
difficulty: "Intermediate"
---

# How would you query a value nested inside a JSONB column?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ FIRST_VALUE and LAST_VALUE](<../../../../TOOLS/PostgreSQL/PostgreSQL_Platform/FIRST_VALUE_and_LAST_VALUE.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When querying a value from a JSONB column in PostgreSQL, we need to use specific functions and operators provided by PostgreSQL. This task involves understanding how JSON data is stored, using the appropriate SQL syntax to extract the required information, and considering performance implications.

The core idea here is that you must utilize JSONB-specific functions such as -> for simple key lookups or jsonb_path_exists/jsonb_path_query for more complex queries. We'll also discuss how these methods impact performance and provide an example of a practical query.

To start, let's assume we have a table named users, which contains a JSONB column called user_data. This column stores user information in the form of key-value pairs. Our goal is to extract the value associated with the key "email" from this JSONB column.

**Code:**

```sql
-- Example table schema
CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    user_data JSONB NOT NULL
);

-- Sample data insertion
INSERT INTO users (user_data) VALUES ('{"name": "John Doe", "email": "johndoe@example.com"}');
```

**Example input / output:**

```

-- Query to retrieve the email from JSONB column SELECT user_data->'email' AS email FROM users WHERE id = 1;

CODE
Copy

Expected Output:
```plaintext
+---------------------+
| email               |
+---------------------+
| johndoe@example.com |
+---------------------+
```

**Data:**

```text
---
| id | user_data                     |
|----|-------------------------------|
| 1  | {"name": "John Doe", "email": "johndoe@example.com"} |
```

