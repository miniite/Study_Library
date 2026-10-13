---
type: "practice"
difficulty: "Deep"
---

# How would you implement an audit trail table that records every change to a row?

**Difficulty:** Deep · **Levels:** Fresher, SDE-1 · [↑ Schema Migration Risks](<../../../../01_Databases_and_SQL/Database_Internals/DBMS_Concepts/Schema_Migration_Risks.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To implement an audit trail table, we need to record every modification made to rows in another table. This involves tracking not only when and what changes were made but also who made those changes. The core idea is to create a separate table that logs each update, insert, or delete operation along with the old state of the row before the change.

First, let's break down the requirements:

Audit Trail Table: A dedicated table that will store information about every change.
Row-Level Tracking: Record changes at the row level rather than just capturing timestamps.
Change Information: Store details like what was changed, who made the change, and when.

#### Triaging the Question

The interviewer is probing your understanding of database transaction handling, data integrity, and how to efficiently capture historical states. You need to consider several key points:

Granularity of Changes: Whether to record changes at a column level or just the row.
Performance Impact: How recording these changes will affect write performance.
Storage Considerations: The impact on storage as each change needs to be logged.
Step-by-Step Solution

Designing the Audit Trail Table:

We need an audit table that stores information about every update, insert, or delete operation. This includes columns like id, entity_id (the ID of the row being modified), operation_type (INSERT, UPDATE, DELETE), timestamp, changed_by (user who made the change), and a state column which contains the old state of the row before modification.

Handling Inserts:

When an insert happens, no previous state exists, so the audit table will only record the new state.

Handling Updates:

Before updating a row in the main table, capture its current state and log it into the audit trail as a DELETE operation for that row ID.
After the update is made to the main table, insert another entry indicating an INSERT of the updated data.

Handling Deletes:

Before deleting a row from the main table, capture its current state and log it into the audit trail as a DELETE operation.

Concurrency Control:

Ensure that concurrent transactions do not overwrite each other's changes by using appropriate locking mechanisms or versioning techniques.

Optimizations:

Use indexes on relevant columns in the audit table to speed up queries.
Partition the audit trail if it grows too large, especially if you have many frequent updates.

#### Closing Summary
and Common Mistakes

By carefully designing the audit trail mechanism, we can maintain a comprehensive history of all changes made to our data. This approach ensures that any historical state can be reconstructed at any point in time with minimal effort. Common mistakes include:

Not Capturing Old States: Failing to log the old state before an update or delete operation.
Performance Issues: Poorly designed audit trails can significantly impact write performance if not managed correctly.
Ignoring Concurrency: Not handling concurrent modifications properly can lead to data inconsistencies.

**Comparison:**

```
Audit Trail Table
id
----
1
2
```

**Code:**

```sql
-- Example SQL for creating an audit table
CREATE TABLE audit_trail (
    id INT AUTO_INCREMENT PRIMARY KEY,
    entity_id INT NOT NULL,
    operation_type ENUM('INSERT', 'UPDATE', 'DELETE') NOT NULL,
    timestamp DATETIME DEFAULT CURRENT_TIMESTAMP,
    changed_by VARCHAR(255),
    state JSON -- Store old and new states as JSON
);
```

**Example input / output:**

```text
Input:
- Main table state before update: {"id": 42, "name": "Old Name", "value": 10}

Output (after an update):
- Audit Trail Table Entry:
  - id: 1
  - entity_id: 42
  - operation_type: DELETE
  - timestamp: 2023-10-15T12:00:00Z
  - changed_by: admin
  - state: {"old": {"name": "Old Name", "value": 10}}
- Main table state after update: {"id": 42, "name": "New Name", "value": 20}

Audit Trail Table Entry:
  - id: 2
  - entity_id: 42
  - operation_type: INSERT
  - timestamp: 2023-10-15T12:01:00Z
  - changed_by: admin
  - state: {"new": {"name": "New Name", "value": 20}}
```

**Data:**

```json
{
  "id": 1,
  "entity_id": 42,
  "operation_type": "UPDATE",
  "timestamp": "2023-10-15T12:00:00Z",
  "changed_by": "admin",
  "state": {
    "old": {"name": "Old Name", "value": 10},
    "new": {"name": "New Name", "value": 20}
  }
}
```

**Flow:**

1. Yes
2. No
3. Main Table Update
4. Check for Concurrency
5. Capture Old State
6. Audit Trail Insert DELETE
7. Update Main Table
8. Check for Concurrency
9. Audit Trail Insert UPDATE

