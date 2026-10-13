---
type: "practice"
difficulty: "Intermediate"
---

# What is the difference between DEFAULT now() and DEFAULT CURRENT_TIMESTAMP for a column?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Date and Timestamp Types](<../../../../TOOLS/PostgreSQL/PostgreSQL_Platform/Date_and_Timestamp_Types.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When working with PostgreSQL, DEFAULT now() and DEFAULT CURRENT_TIMESTAMP are both used to set default values for columns based on the current timestamp. However, there is a subtle but important difference between them that can affect how your data is stored and queried.

#### Triage

The question is asking about the distinction between these two default value functions in PostgreSQL. The interviewer is probing your understanding of SQL defaults and the specifics of how now() and CURRENT_TIMESTAMP behave differently over time.

#### First-Principles Solution
DEFAULT now(): This function is evaluated at the point when a new row is inserted into the table, regardless of when that row is actually committed to the database.
DEFAULT CURRENT_TIMESTAMP: This function returns the current timestamp only when the query is executed and is not re-evaluated during transaction processing.

#### Example Scenario

#### Cons
ider an example where you have a created_at column in your users table:

SQL
CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    username VARCHAR(50),
    created_at TIMESTAMP DEFAULT now()
);

In this case, the created_at timestamp would be set to the exact moment of insertion, even if there are multiple statements or transactions occurring within the same session.

Now consider:

SQL
CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    username VARCHAR(50),
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

Here, created_at will always reflect the current timestamp at the moment of the query execution. If you insert a row and then execute a SELECT statement immediately after, both timestamps would be the same. However, if you execute another SELECT statement later, it could return a different timestamp.

#### Common Mistakes
and Follow-Ups

Mistake: Thinking that now() and CURRENT_TIMESTAMP are interchangeable.

Correctness: They are not. The difference lies in when and how the current timestamp is evaluated.

#### Follow-Up
Question: What happens if you have multiple statements within a transaction?

Answer: In the case of DEFAULT now(), each statement would receive its own timestamp, whereas with DEFAULT CURRENT_TIMESTAMP, only one timestamp (the last execution) would be used for all rows in the transaction.

Additional Example:

SQL
-- Inserting a row with DEFAULT now()
INSERT INTO users (username) VALUES ('john_doe');

-- Selecting the created_at column immediately after insertion
SELECT created_at FROM users WHERE username = 'john_doe';

This will show you the exact timestamp of when john_doe was inserted.

SQL
-- Inserting a row with DEFAULT CURRENT_TIMESTAMP
INSERT INTO users (username) VALUES ('jane_doe');

-- Selecting the created_at column immediately after insertion
SELECT created_at FROM users WHERE username = 'jane_doe';

This will show you the timestamp of when SELECT was executed, not necessarily the exact moment of insertion.

#### Closing Summary

Understanding these differences is crucial for managing time-stamped data accurately in PostgreSQL. While both functions serve similar purposes, they behave differently under various scenarios involving transactions and multiple statements. The choice between them should be made based on your specific requirements.

**Code:**

```

CREATE TABLE users ( id SERIAL PRIMARY KEY, username VARCHAR(50), created_at TIMESTAMP DEFAULT now() );

INSERT INTO users (username) VALUES ('john_doe');

SELECT created_at FROM users WHERE username = 'john_doe';

CODE
Copy

```sql
CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    username VARCHAR(50),
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

INSERT INTO users (username) VALUES ('jane_doe');

SELECT created_at FROM users WHERE username = 'jane_doe';
```

**Example input / output:**

```text
-- Output for DEFAULT now()
created_at: 2023-10-01T12:00:00.000

-- Output for DEFAULT CURRENT_TIMESTAMP
created_at: 2023-10-01T12:05:00.000
```

