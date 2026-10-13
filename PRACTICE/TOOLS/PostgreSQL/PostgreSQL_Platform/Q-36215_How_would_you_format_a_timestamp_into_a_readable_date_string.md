---
type: "practice"
difficulty: "Intermediate"
---

# How would you format a timestamp into a readable date string like 'DD-Mon-YYYY' using to_char?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ Date and Timestamp Types](<../../../../TOOLS/PostgreSQL/PostgreSQL_Platform/Date_and_Timestamp_Types.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To format a timestamp in PostgreSQL into a readable date string such as 'DD-Mon-YYYY', we will use the to_char function, which allows us to specify various formatting options. This is a common requirement when displaying dates in user-facing applications or logs. Let’s break down the process step by step.

**Code:**

```sql
-- Example usage of to_char with the desired format
SELECT to_char(your_timestamp_column, 'DD-Mon-YYYY') AS formatted_date;
```

**Example input / output:**

```

Example input:

SQL
Copy
CREATE TABLE events (
    id SERIAL PRIMARY KEY,
    event_name VARCHAR(255),
    timestamp_column TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

INSERT INTO events (event_name) VALUES ('Sample Event');
SELECT to_char(timestamp_column, 'DD-Mon-YYYY') AS formatted_date FROM events;

Example output:

PLAINTEXT
Copy
  formatted_date
-----------------
  01-Jan-2023
(1 row)
```

