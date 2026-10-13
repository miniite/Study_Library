---
type: "practice"
difficulty: "Deep"
---

# You need to enforce that at most one row per user has status='active' — how do you do this at the schema level in MySQL 8 given no partial-index support?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Activity Timeline Schema](<../../../../TOOLS/MySQL/MySQL_Platform/Activity_Timeline_Schema.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To ensure that only one active row exists per user in a table, we need to enforce a unique constraint on the user_id and status columns combined with the condition where status = 'active'. This can be achieved using MySQL 8's support for JSON and expressions within constraints. Let’s break down how this is done step-by-step.

#### Triaging the Question

The core of the question is to enforce a business rule at the schema level that limits each user to only one active row in a table. The interviewer is probing your understanding of MySQL 8 features, particularly its ability to use JSON and expressions within constraints. We’ll need to make assumptions about the existing schema, such as whether there are other relevant columns.

#### First-Principles Solution
Identify the Columns: We have user_id (the user identifier) and status (which can be 'active' or some other values).
Combine Conditions in JSON: MySQL 8 allows for complex conditions within constraints using expressions.
Create a Unique Constraint: Use this expression to ensure that only one row per user with status='active' exists.
Detailed Solution

Given the schema:

SQL
CREATE TABLE users (
    id INT AUTO_INCREMENT PRIMARY KEY,
    user_id INT NOT NULL,
    name VARCHAR(100),
    status ENUM('active', 'inactive') DEFAULT 'inactive',
    -- Other columns...
);

We can enforce this constraint using a UNIQUE index with an expression:

SQL
CREATE UNIQUE INDEX unique_active_user ON users (user_id, JSON_EXTRACT(COALESCE(JSON_OBJECT('status', status), '{}'), '$.status')) WHERE status = 'active';

However, MySQL does not support direct conditionals in UNIQUE constraints as of MySQL 8. We can use a workaround by leveraging triggers or views, but for pure schema-level enforcement, we need to think differently.

Practical Approach Using Views and Triggers

Create a View: A view can be used to simulate the required constraint.

SQL
CREATE VIEW active_users AS
SELECT user_id, status FROM users WHERE status = 'active';

Enforce Integrity with a Trigger: We can use an INSTEAD OF trigger on insert or update to ensure that only one active row exists per user.

SQL
CREATE TRIGGER before_insert_user BEFORE INSERT ON users FOR EACH ROW
BEGIN
    IF NEW.status = 'active' THEN
        SIGNAL SQLSTATE '45000'
            SET MESSAGE_TEXT = 'A user can have at most one active status';
    END IF;
END;

DELIMITER //
CREATE TRIGGER before_update_user BEFORE UPDATE ON users FOR EACH ROW
BEGIN
    IF NEW.status = 'active' AND OLD.status != 'active' THEN
        SIGNAL SQLSTATE '45000'
            SET MESSAGE_TEXT = 'A user can have at most one active status';
    END IF;
END //
DELIMITER ;

#### Closing Summary
and Common Mistakes/Follow-Ups
Summary: To enforce that only one row per user has the status of 'active', we can use a combination of views, triggers, or complex unique constraints involving JSON. The exact method depends on MySQL version support.

#### Common Mistakes
Not considering alternative methods like triggers when direct constraint solutions are not possible.
Overlooking the need for integrity checks during both insert and update operations.

#### Follow-Ups
The interviewer might ask about performance implications of using views and triggers, or how to handle concurrent transactions while enforcing such constraints.

