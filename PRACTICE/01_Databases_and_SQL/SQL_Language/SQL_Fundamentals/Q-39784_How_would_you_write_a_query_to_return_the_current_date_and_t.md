---
type: "practice"
difficulty: "Basic"
---

# How would you write a query to return the current date and time from the database server?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ Timezones and Timestamps](<../../../../01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Timezones_and_Timestamps.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To retrieve the current date and time from the database server, we need to use SQL (Structured Query Language) which is designed for managing data in relational database management systems. The specific function used depends on the database system you're working with, as different databases implement this feature differently.

The core idea here is simple: we will select a built-in function that returns the current timestamp. For most SQL databases, such as MySQL, PostgreSQL, and SQLite, this can be achieved using CURRENT_TIMESTAMP or similar functions. For example:

SQL
SELECT CURRENT_TIMESTAMP AS "Current DateTime";

This query will return the server's current date and time in a format that includes both date and time components.

**Code:**

```sql
-- Example SQL query to get the current timestamp
SELECT CURRENT_TIMESTAMP AS "Current DateTime";
```

**Example input / output:**

```

Example input:

SQL
Copy
SELECT CURRENT_TIMESTAMP AS "Current DateTime";

Expected output (server-dependent):

PLAINTEXT
Copy
+------------------------+
| Current DateTime       |
+------------------------+
| 2023-11-29 14:45:36.123|
+------------------------+
```

