---
type: "practice"
difficulty: "Intermediate"
---

# How would you design a schema to model soft-deleted records while keeping foreign keys, unique constraints, and query ergonomics sane? What are the partial-unique-index tricks involved?

**Difficulty:** Intermediate · **Levels:** SDE-1/2, SDE-2/3 · [↑ Foreign Keys and Referential Integrity](<../../../../01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Foreign_Keys_and_Referential_Integrity.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To design a schema that models soft-deleted records while preserving foreign key relationships, maintaining unique constraints, and ensuring query ergonomics, we need to carefully balance these requirements. The core idea is to use a deleted_at timestamp or boolean flag in the main entity table and implement additional mechanisms for handling the soft deletes without breaking any of the database integrity rules.

#### Triaging the Question

The question requires designing a schema that:

Soft Deletes Records: Allows marking records as deleted instead of physically removing them.
Preserves Foreign Key Relationships: Ensures that even when a record is marked as deleted, its reference in foreign key relationships remains valid.
Maintains Unique Constraints: Ensures that unique constraints are not violated due to soft-deleted records.
Query Ergonomics: Provides an efficient and user-friendly way of querying both active and deleted records.

#### First-Principles Solution
1. Soft Delete Column in the Main Entity Table

We introduce a deleted_at timestamp column or a is_deleted boolean flag to mark records as soft-deleted. This approach keeps the primary table structure simple while allowing for easy identification of deleted records.

SQL
CREATE TABLE users (
    id INT PRIMARY KEY,
    username VARCHAR(255) NOT NULL UNIQUE,
    email VARCHAR(255) NOT NULL UNIQUE,
    password_hash VARCHAR(255),
    deleted_at TIMESTAMP NULL, -- Soft delete timestamp
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP
);
2. Handling Foreign Key Relationships

To ensure that foreign key relationships remain valid, we need to consider the implications of soft deletes on these relationships. One common approach is to add a is_active flag in the related tables or handle deletions through cascading operations.

Option A: Using a is_active Flag

In the related table, introduce an is_active flag to indicate whether a record should be considered active.

SQL
CREATE TABLE user_roles (
    id INT PRIMARY KEY,
    user_id INT NOT NULL REFERENCES users(id),
    role_id INT NOT NULL REFERENCES roles(id),
    is_active BOOLEAN DEFAULT TRUE -- True if the user is still active
);

This approach keeps the foreign key relationships intact while allowing for soft deletes.

Option B: Cascading Deletions

Alternatively, you can use cascading delete rules to automatically remove related records when a parent record is deleted.

SQL
ALTER TABLE user_roles ADD CONSTRAINT fk_user ON DELETE CASCADE;
3. Maintaining Unique Constraints

To maintain unique constraints while allowing for soft deletes, we need to ensure that the uniqueness checks are applied only to active records.

Using Views or Triggers

One way to achieve this is by using a view that filters out deleted records.

SQL
CREATE VIEW active_users AS SELECT * FROM users WHERE deleted_at IS NULL;

Or, you can use triggers to enforce unique constraints on active records.

SQL
CREATE TRIGGER enforce_unique_on_insert BEFORE INSERT ON users FOR EACH ROW
WHEN (NEW.deleted_at IS NOT NULL)
BEGIN
    SIGNAL SQLSTATE '45000' SET MESSAGE_TEXT = 'Cannot insert a deleted record';
END;

#### Closing Summary
and Common Mistakes/Follow-Ups
Mistake 1: Forgetting to update foreign keys when a parent record is soft-deleted.
Mistake 2: Violating unique constraints by not filtering out deleted records in unique checks.

#### Follow-Up
Discuss how to optimize queries for active and deleted records. For example, using indexes on deleted_at or creating partial unique indexes.
Partial Unique Index Tricks

A partial unique index can be used to enforce uniqueness only on non-deleted records. This ensures that the unique constraint is maintained without affecting performance.

SQL
CREATE UNIQUE INDEX idx_unique_username ON users(username) WHERE deleted_at IS NULL;

**Comparison:**

| User     | Username   | Email               | Deleted At        |
|----------|-----------|---------------------|-------------------|
| User 1   | user1     | user1@example.com   | NULL              |
| User 2   | user2     | user2@example.com   | NULL              |
| Role    | ID    | Name      |
|---------|-------|----------|
| Admin   | 1     | admin     |
| User    | 2     | user      |

**Code:**

```sql
-- Example of a soft delete query
UPDATE users SET deleted_at = CURRENT_TIMESTAMP WHERE id = 1;

-- Example of an active records view
CREATE VIEW active_users AS SELECT * FROM users WHERE deleted_at IS NULL;

-- Example of a trigger to enforce unique constraints on insert
CREATE TRIGGER enforce_unique_on_insert BEFORE INSERT ON users FOR EACH ROW
WHEN (NEW.deleted_at IS NOT NULL)
BEGIN
    SIGNAL SQLSTATE '45000' SET MESSAGE_TEXT = 'Cannot insert a deleted record';
END;
```

**Example input / output:**

```text
-- Insert a new user
INSERT INTO users (username, email) VALUES ('testuser', 'test@example.com');

-- Soft delete an existing user
UPDATE users SET deleted_at = CURRENT_TIMESTAMP WHERE id = 1;

-- Query active and deleted users
SELECT * FROM users;
```

**Data:**

```json
{
  "users": [
    {
      "id": 1,
      "username": "user1",
      "email": "user1@example.com",
      "password_hash": "$2b$12$q4Xzv8Kj3Tn0mR5B9xYfZu.",
      "deleted_at": null
    },
    {
      "id": 2,
      "username": "user2",
      "email": "user2@example.com",
      "password_hash": "$2b$12$q4Xzv8Kj3Tn0mR5B9xYfZu.",
      "deleted_at": null
    }
  ],
  "roles": [
    {
      "id": 1,
      "name": "admin"
    },
    {
      "id": 2,
      "name": "user"
    }
  ]
}
```

**Flow:**

1. Yes
2. No
3. Soft Delete Query
4. Is Record Deleted?
5. Return Deleted Record
6. Return Active Record
7. Log or Audit Record Deletion

