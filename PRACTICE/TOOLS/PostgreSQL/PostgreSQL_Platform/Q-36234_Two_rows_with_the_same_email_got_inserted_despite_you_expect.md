---
type: "practice"
difficulty: "Deep"
---

# Two rows with the same email got inserted despite you expecting uniqueness; how would you prevent this at the database level going forward?

**Difficulty:** Deep · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ Adding Constraints Online](<../../../../TOOLS/PostgreSQL/PostgreSQL_Platform/Adding_Constraints_Online.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the issue of duplicate email inserts in PostgreSQL, we need to ensure that the email field is unique across all rows in a table. This can be achieved by adding a unique constraint or using a primary key on the column. Given that you've encountered this issue now, let’s walk through how to prevent it going forward.

First, it's important to understand what might have caused this issue and then discuss the best practices for handling unique constraints in PostgreSQL.

**Comparison:**

| id | name          | email                | created_at       |
|----|---------------|----------------------|------------------|
| 1  | John Doe      | john.doe@example.com | 2023-10-05 10:00 |
| 2  | Jane Smith    | jane.smith@example.com | 2023-10-06 11:00 |

**Code:**

```sql
-- Example of creating a table with a unique constraint on the email column
CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100),
    email VARCHAR(255) UNIQUE,  -- Adding a unique constraint on email
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

**Example input / output:**

```

To demonstrate how the unique constraint works:

SQL
Copy
-- Attempting to insert a duplicate email should fail with an error message
INSERT INTO users (name, email) VALUES ('Alice Johnson', 'john.doe@example.com');  -- This will raise an exception

-- Sample output of attempting the above insertion:
ERROR: duplicate key value violates unique constraint "users_email_key"
DETAIL: Key (email)=(john.doe@example.com) already exists.
```

**Data:**

```sql
-- Sample data for the table
INSERT INTO users (name, email) VALUES ('John Doe', 'john.doe@example.com');
INSERT INTO users (name, email) VALUES ('Jane Smith', 'jane.smith@example.com');
```

**Flow:**

1. No
2. Yes
3. Create Table with UniqueConstraint
4. Is email already in use?
5. Proceed with Insertion
6. Rollback Transaction andInform User
7. Insert Data Successfully

