---
type: "practice"
difficulty: "Deep"
---

# How would you write a backward-compatible migration to rename a column without breaking a running application?

**Difficulty:** Deep · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Online Schema Changes](<../../../../TOOLS/MySQL/MySQL_Platform/Online_Schema_Changes.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To write a backward-compatible migration that renames a column in a MySQL database, we need to ensure that any application using the old schema can continue functioning seamlessly even after the renaming is done. This involves creating a new column with the desired name, copying data from the old column to the new one, and then safely removing or updating references to the old column. Let’s break this down step-by-step.

#### Triage

The question asks us to write a backward-compatible migration for renaming a column in MySQL while ensuring that a running application continues to function correctly. This involves understanding how to handle schema changes without causing downtime or data loss, and ensuring that the transition is smooth for both the database and the application.

#### First Principles Solution
Add a new column: Introduce a new column with the desired name in your table.
Copy data: Populate the new column with data from the old column.
Update references: Gradually update queries and stored procedures to use the new column where appropriate.
Drop old column: Once all dependencies are updated, safely remove the old column.
Step-by-Step Solution

Create a New Column:

SQL
ALTER TABLE users ADD COLUMN email VARCHAR(255);

Copy Data from Old to New Column:

SQL
UPDATE users SET email = old_email;

Update Application Code: Modify all queries and stored procedures to start using email instead of the deprecated old_email.

Mark Old Column for Deletion: As a best practice, you can mark the old column as NOT NULL with a default value or set its data type to BLOB/TEXT, preventing it from being used until fully removed.

SQL
ALTER TABLE users MODIFY old_email BLOB;

Gradual Removal: Once your application code is updated, you can remove the old column:

SQL
ALTER TABLE users DROP COLUMN old_email;

#### Closing Summary
and Common Mistakes/Follow-ups

In summary, renaming a column in MySQL without breaking existing applications involves careful planning to ensure that data integrity and application functionality are maintained. Key steps include creating a new column, copying data from the old one, updating all relevant code, and then safely removing the old column.

#### Common mistakes
include:

Failing to update all dependent queries and stored procedures.
Not ensuring that both columns coexist during the transition period.
Ignoring potential performance issues that might arise due to redundant storage of data in two columns temporarily.

#### Follow-up
questions could involve discussing how this approach handles concurrent transactions, ensuring no data corruption, and strategies for rolling back changes if something goes wrong.

**Comparison:**

| Column Name | Data Type  |
|-------------|-----------|
| id          | INT(10)   |
| old_email   | VARCHAR(255) or BLOB (temporarily) |
| email       | VARCHAR(255) |

**Code:**

```sql
-- Step 1: Add a new column
ALTER TABLE users ADD COLUMN email VARCHAR(255);

-- Step 2: Copy data from old to new column
UPDATE users SET email = old_email;

-- Step 3: Mark old column for deletion
ALTER TABLE users MODIFY old_email BLOB;

-- Step 4: Drop the old column after ensuring all dependencies are updated
ALTER TABLE users DROP COLUMN old_email;
```

**Example input / output:**

```text
Input:
- Existing table with column "old_email"

Output:
- Table updated to include "email" column with data from old email
- Old column marked as BLOB and eventually dropped
```

**Data:**

```sql
CREATE TABLE IF NOT EXISTS `users` (
    `id` INT(10) unsigned NOT NULL AUTO_INCREMENT,
    `old_email` VARCHAR(255),
    PRIMARY KEY (`id`)
);
```

**Flow:**

1. FLOWCHART · MERMAID
2. graph LR;
3. A[Start] --> B{Does the application use "old_email"?}
4. B -- No --> C[Add new column "email"]
5. C --> D{Copy data from "old_email" to "email"}
6. D --> E[Mark "old_email" as BLOB]
7. E --> F{Is all code updated?}
8. F -- Yes --> G[Drop "old_email" column]
9. F -- No --> H[Update code and retry]
10. B -- Yes --> I[Notify developers to update code]
11. I --> G

