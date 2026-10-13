---
type: "practice"
difficulty: "Basic"
---

# How do you check what version of MySQL a server is running from the SQL prompt?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ Online Schema Changes](<../../../../TOOLS/MySQL/MySQL_Platform/Online_Schema_Changes.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To check the version of MySQL running on a server, you can use a simple SQL query from the command line or any tool that connects to your database. The core idea is to look at system variables that store this information.

The key query involves using the SELECT statement with specific system variables like version() and @@version. This approach leverages MySQL's built-in functions designed to provide such metadata.

**Code:**

```sql
-- Check MySQL version using version() function
SELECT version();

-- Alternatively, use @@version system variable
SHOW VARIABLES LIKE 'version%';
```

**Example input / output:**

```

When you run these queries in the SQL prompt or a database client like MySQL Workbench:

For SELECT version();, you might get an output like:

CODE
Copy
+------------+
| version()  |
+------------+
| 8.0.23      |
+------------+

Running SHOW VARIABLES LIKE 'version%'; could return a more detailed information, including additional related variables.
```

