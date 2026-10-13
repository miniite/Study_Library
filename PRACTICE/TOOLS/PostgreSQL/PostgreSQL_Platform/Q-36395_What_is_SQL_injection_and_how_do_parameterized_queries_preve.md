---
type: "practice"
difficulty: "Intermediate"
---

# What is SQL injection and how do parameterized queries prevent it in PostgreSQL?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ RETURNING Clause](<../../../../TOOLS/PostgreSQL/PostgreSQL_Platform/RETURNING_Clause.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

SQL injection is a common security vulnerability where an attacker can execute arbitrary SQL statements by injecting malicious input into a query. In PostgreSQL, this risk can be mitigated using parameterized queries, which ensure that user inputs are treated as data rather than executable code.

To understand the core of the problem and its solution, let's break down what SQL injection is and how parameterized queries work:

SQL Injection Overview: Consider a scenario where you have a login form in your application. If the backend uses string concatenation to build the query like this:

SQL
SELECT * FROM users WHERE username = 'admin' AND password = 'password';

An attacker could exploit this by injecting malicious SQL into the username field, such as:

SQL
SELECT * FROM users WHERE username = 'admin'; DROP TABLE users; -- '

This would result in a query that attempts to log in and also drops the entire users table.

Parameterized Queries: To prevent this, we should use parameterized queries where user inputs are passed as parameters, not directly concatenated into the SQL statement. For example:

SQL
PREPARE stmt AS SELECT * FROM users WHERE username = $1 AND password = $2;
EXECUTE stmt ('admin', 'password');

Here, username and password are treated as parameters that cannot be executed as part of the SQL statement. The database driver ensures they are properly escaped and sanitized.

#### Implementation
in PostgreSQL:

Prepared Statements: PostgreSQL supports prepared statements using the PREPARE, EXECUTE, and DEALLOCATE commands.
SQL
-- Prepare a statement
PREPARE select_user (TEXT, TEXT) AS SELECT * FROM users WHERE username = $1 AND password = $2;

-- Execute with parameters
EXECUTE select_user('admin', 'password');

-- Clean up the prepared statement
DEALLOCATE select_user;
Parameterized Queries with psycopg2: If you're using Python's psycopg2 library, you can easily use parameterized queries:
PYTHON
import psycopg2

conn = psycopg2.connect("dbname=test user=postgres")

cursor = conn.cursor()

# Correct usage of parameterized query
cursor.execute("SELECT * FROM users WHERE username = %s AND password = %s", ('admin', 'password'))

results = cursor.fetchall()
print(results)

# Incorrect usage (concatenation is vulnerable to SQL injection)
# cursor.execute("SELECT * FROM users WHERE username = " + admin_username + " AND password = " + admin_password)

#### Closing Summary
By using parameterized queries and prepared statements, we ensure that user inputs are treated as data rather than executable code, preventing SQL injection attacks in PostgreSQL.

#### Common Mistakes/Follow-Ups

Forgetting Prepared Statements: Always use prepared statements when dealing with dynamic SQL.
Improper Use of User Inputs: Directly concatenating user inputs into SQL statements can lead to vulnerabilities.
Not Cleaning Data Properly: Ensure that all data is sanitized and validated before being used in queries.

**Code:**

```python
import psycopg2

conn = psycopg2.connect("dbname=test user=postgres")

cursor = conn.cursor()

# Correct usage of parameterized query
cursor.execute("SELECT * FROM users WHERE username = %s AND password = %s", ('admin', 'password'))

results = cursor.fetchall()
print(results)

# Incorrect usage (concatenation is vulnerable to SQL injection)
# cursor.execute("SELECT * FROM users WHERE username = " + admin_username + " AND password = " + admin_password)
```

**Example input / output:**

```

Input:

PYTHON
Copy
cursor.execute("SELECT * FROM users WHERE username = %s AND password = %s", ('admin', 'password'))

Output:

PYTHON
Copy
[(1, 'admin', 'hashed_password')]
```

**Data:**

```
Sample data: users table with columns id, username, and password.
SQL
Copy
CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    username TEXT NOT NULL,
    password TEXT NOT NULL
);
INSERT INTO users (username, password) VALUES ('admin', 'hashed_password'), ('user', 'hashed_user');
```

