---
type: "practice"
difficulty: "Intermediate"
---

# Can a stored procedure return a value like a function does? Explain how.

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ Stored Procedure Fundamentals](<../../../../01_Databases_and_SQL/Database_Objects/Stored_Procedures/Stored_Procedure_Fundamentals.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In this scenario, we are exploring whether a stored procedure can return values akin to a regular function in SQL databases. This is a common requirement when you need to encapsulate logic and return specific data without having to write multiple SELECT statements or complex queries every time.

#### Triaging the Question
Objective: Understand if stored procedures (SPs) support returning values, similar to functions.
Key Idea: Stored procedures can indeed return values, but they are more versatile than regular functions as they can perform operations and modify data.

#### Assumptions
We will consider a basic SQL database environment using a common DBMS like PostgreSQL or MySQL.

#### Solution
from First Principles
Definition Recap: A stored procedure is a block of procedural logic that resides in the database, which can be called multiple times without being recompiled. They are often used for encapsulating complex operations and managing transactional integrity.
Return Values:
Unlike functions, SPs typically don't return values directly; instead, they modify data or use output parameters to send results back to the caller.
However, in some modern DBMS like PostgreSQL, stored procedures can indeed return a single value using RETURN statements.

#### Example
Using PostgreSQL
SQL
-- Creating a Simple Stored Procedure
CREATE OR REPLACE FUNCTION get_user_id(username TEXT)
RETURNS INT AS $$
DECLARE
    user_id INT;
BEGIN
    SELECT id INTO user_id FROM users WHERE username = username;

    -- Return the user ID if found
    IF FOUND THEN
        RETURN user_id;
    ELSE
        RETURN -1;  -- Indicating no user was found
    END IF;
END;
$$ LANGUAGE plpgsql;

-- Calling the Function and Using Its Return Value
SELECT get_user_id('john_doe');

In this example, we created a function that returns an integer value representing a user ID. If the user is not found, it returns -1.

Output:
PLAINTEXT
get_user_id
-------------
         256  -- Assuming 'john_doe' has an id of 256 in the database.

#### Closing Summary
and Common Mistakes/Follow-Ups
Common Mistake: Forgetting that SPs primarily return values through output parameters or via RETURN statements, rather than using a direct RETURN value like functions do.

#### Follow-Up
Discussing scenarios where this might be useful versus when you should prefer regular functions. Functions are typically used for simple calculations and returning values directly, while SPs can handle complex logic with less repetition.

#### Example
Use Case
SQL
-- Using the Return Value in a Query
SELECT * FROM users WHERE id = (SELECT get_user_id('john_doe'));

This query uses the returned user ID to fetch detailed user information from the users table.

**Comparison:**

```
users	Schema	Description
id	SERIAL PRIMARY KEY	Unique identifier for each user.
username	VARCHAR(50) UNIQUE NOT NULL	User's unique login name.
email	VARCHAR(254) UNIQUE NOT NULL	User's unique email address.
```

**Code:**

```sql
CREATE OR REPLACE FUNCTION get_user_id(username TEXT)
RETURNS INT AS $$
DECLARE
    user_id INT;
BEGIN
    SELECT id INTO user_id FROM users WHERE username = username;

    IF FOUND THEN
        RETURN user_id;
    ELSE
        RETURN -1;  -- Indicating no user was found
    END IF;
END;
$$ LANGUAGE plpgsql;
```

**Example input / output:**

```text
SELECT get_user_id('john_doe');  -- Output: 1

SELECT * FROM users WHERE id = (SELECT get_user_id('john_doe'));  -- Output:
+----+----------+--------------------+
| id | username | email              |
+----+----------+--------------------+
|  1 | john_doe | john@example.com   |
+----+----------+--------------------+
```

**Data:**

```sql
CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    username VARCHAR(50) UNIQUE NOT NULL,
    email VARCHAR(254) UNIQUE NOT NULL
);
INSERT INTO users (username, email) VALUES ('john_doe', 'john@example.com');
```

**Flow:**

1. Yes
2. No
3. Call get_user_id
4. User found?
5. Return user ID
6. Return -1

