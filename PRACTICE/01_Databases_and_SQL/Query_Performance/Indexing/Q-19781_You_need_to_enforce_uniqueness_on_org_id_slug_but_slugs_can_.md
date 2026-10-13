---
type: "practice"
difficulty: "Deep"
---

# You need to enforce uniqueness on (org_id, slug) but slugs can be reused after archival. Design the partial unique index and explain the concurrency edge cases on re-creation.

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Adding Unique Indexes Safely](<../../../../01_Databases_and_SQL/Query_Performance/Indexing/Adding_Unique_Indexes_Safely.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To design a database indexing strategy that enforces uniqueness on (org_id, slug) while allowing re-use of slugs after they are archived, we need to carefully craft a partial unique index and handle concurrency issues effectively. This solution involves understanding the trade-offs between performance and data integrity.

#### Triage

The core question is about designing a database schema for enforcing partial uniqueness with re-usage capabilities. The interviewer is probing your ability to balance data consistency and concurrency in a complex scenario. Assumptions include using SQL databases, like PostgreSQL or MySQL, which support unique indexes. The key challenge lies in handling concurrency during the archiving process.

#### First-Principles Solution
Index Design: We need a partial unique index on (org_id, slug) but allow re-use after archiving.
Concurrency Handling: Ensure that concurrent operations do not interfere with each other or violate data integrity during the archival and re-use process.
Step-by-Step Solution

Database Schema

SQL
CREATE TABLE content (
    id BIGINT PRIMARY KEY,
    org_id INT NOT NULL,
    slug VARCHAR(255) NOT NULL,
    status ENUM('active', 'archived') DEFAULT 'active',
    -- other fields...
);

Partial Unique Index

SQL
CREATE UNIQUE INDEX idx_unique_slug ON content (org_id, slug) WHERE status = 'active';

This index ensures that (org_id, slug) pairs are unique only when the status is 'active'.

Concurrency Handling

Archiving: When archiving a record, mark its status as 'archived'.
Re-usage Validation: Before inserting or updating a new record with an existing (org_id, slug), check if the slug is not currently in use by any active records.
Detailed Implementation

Archiving Existing Record

SQL
UPDATE content SET status = 'archived' WHERE org_id = ? AND slug = ?;

Inserting or Updating a New Record

SQL
INSERT INTO content (org_id, slug, ...other fields...)
VALUES (?, ?, ...)
ON DUPLICATE KEY UPDATE
    -- handle any necessary updates...

Concurrency Control

Use transactions to ensure atomicity.
Employ locking mechanisms like row-level locking or pessimistic locking.

#### Closing Summary
and Common Mistakes
Ensure the unique index is correctly defined with a WHERE clause to account for archiving status.
Handle concurrency issues by using proper transaction management and locking strategies.

#### Common mistakes
include not properly defining the unique constraint, overlooking transactional integrity during concurrent operations, and failing to handle edge cases like partial updates.

**Comparison:**

|Field       |Description|
|------------|-----------|
|org_id      |Unique ID for organization|
|slug        |Uniquely identifying string within an organization|
|status      |Record status (active/archived)|

**Code:**

```sql
-- Example SQL statements
CREATE TABLE content (
    id BIGINT PRIMARY KEY,
    org_id INT NOT NULL,
    slug VARCHAR(255) NOT NULL,
    status ENUM('active', 'archived') DEFAULT 'active'
);

CREATE UNIQUE INDEX idx_unique_slug ON content (org_id, slug) WHERE status = 'active';

UPDATE content SET status = 'archived' WHERE org_id = 1 AND slug = 'example-slug';

INSERT INTO content (org_id, slug, ...other fields...)
VALUES (?, ?, ...)
ON DUPLICATE KEY UPDATE
    -- handle any necessary updates...
```

**Example input / output:**

```text
Input:
- Update org_id=101, slug='example-slug' to archived.

Output:
- SQL statement executed successfully.
- Status of (org_id, 'example-slug') pair updated to 'archived'.

Input:
- Insert new record with org_id=101, slug='example-slug'.

Output:
- Unique index check passes since the original record is now archived.
- New record inserted into database.
```

**Data:**

```json
{
    "id": 1,
    "org_id": 101,
    "slug": "example-slug",
    "status": "active"
}
```

**Flow:**

1. Yes
2. No
3. Check if slug is active
4. Is the slug active?
5. Mark record as archived
6. Insert new record with uniquecheck

