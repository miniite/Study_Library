---
type: "practice"
difficulty: "Deep"
---

# How would you design keys and constraints for a soft-delete pattern so that a UNIQUE(email) still allows a new active row after deletion?

**Difficulty:** Deep · **Levels:** SDE-1, SDE-1/2 · [↑ Unique and Deferred Constraints](<../../../../01_Databases_and_SQL/Database_Design/Constraints_and_Keys/Unique_and_Deferred_Constraints.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the requirement of maintaining uniqueness on email while allowing reinsertion after a soft delete, we need to design keys and constraints carefully. The primary goal is to ensure that once an email is deleted (softly), it can be reused for another user without violating the unique constraint. Here’s how you would approach this:

First, let's summarize the core solution: We will implement a soft delete pattern by adding a is_active column and use a combination of primary key constraints and unique indexes to manage reinsertion of deleted emails.

**Code:**

```sql
-- Create table with soft delete logic
CREATE TABLE users (
    user_id SERIAL PRIMARY KEY,
    email VARCHAR(255) UNIQUE NOT NULL,  -- Ensure uniqueness on email
    is_active BOOLEAN DEFAULT true,      -- Soft delete flag
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Insert sample data to show the soft delete in action
INSERT INTO users (email, is_active) VALUES ('john.doe@example.com', true);
INSERT INTO users (email, is_active) VALUES ('jane.doe@example.com', true);
```

**Example input / output:**

```text
-- Soft delete a user by updating the is_active flag to false
UPDATE users SET is_active = false WHERE email = 'john.doe@example.com';

-- Check if the deleted email can be reinserted
INSERT INTO users (email, is_active) VALUES ('john.doe@example.com', true);
```

**Data:**

```json
{
  "users": [
    {
      "user_id": 1,
      "email": "john.doe@example.com",
      "is_active": true,
      "created_at": "2023-10-01T09:00:00Z"
    },
    {
      "user_id": 2,
      "email": "jane.doe@example.com",
      "is_active": true,
      "created_at": "2023-10-02T14:30:00Z"
    }
  ]
}
```

