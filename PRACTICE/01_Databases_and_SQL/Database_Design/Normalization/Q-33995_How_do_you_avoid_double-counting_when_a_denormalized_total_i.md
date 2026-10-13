---
type: "practice"
difficulty: "Expert"
---

# How do you avoid double-counting when a denormalized total is updated by both a trigger and application code during a migration window?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-1, SDE-1/2, SDE-2/3 · [↑ Denormalization Trade-offs and Patterns](<../../../../01_Databases_and_SQL/Database_Design/Normalization/Denormalization_Trade-offs_and_Patterns/_index.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In this scenario, we face a common challenge in database migrations where a denormalized total might be updated twice—once via an application update and once through a trigger handling real-time updates or batch processing. To avoid double-counting, we need to ensure that the update operations are synchronized and consistent. Here’s how we can approach this problem from first principles.

Triaging the Problem

The core issue is ensuring consistency in updating denormalized totals during migration. The key points to address include:

Synchronization: Ensuring that both the application and trigger use a single source of truth.
Atomicity: Making sure updates are atomic to prevent partial state changes.
Race Conditions: Handling potential race conditions where multiple update operations might occur simultaneously.

#### Assumptions
The denormalized total is stored in a single column or field.
Both the application and trigger can read and write to this field.
We have control over both the application code and the database schema/triggers.
The migration window involves concurrent updates from multiple sources.

#### Solution

To avoid double-counting, we will implement a mechanism that ensures only one of the two update operations is processed at any given time. This can be achieved using a combination of transactions and optimistic locking techniques.

Transaction Isolation Levels:

Use a transaction isolation level that allows us to lock the row(s) being updated for as long as necessary without blocking other read-only queries.

Optimistic Locking:

Implement versioning or timestamp fields in the table to detect and handle concurrency issues.

Synchronization Mechanism:

Use a mutex (lock) mechanism, such as a row-level lock or a distributed lock like Redis, to ensure only one update operation is active at any time.

Application Code Update:

SQL
BEGIN TRANSACTION;
UPDATE totals SET total = total + new_value WHERE id = @id AND version = @version;
-- Check if the update was successful; if not, retry or log an error
COMMIT;

Trigger for Real-Time Updates:

The trigger should also check the version before performing the update to avoid conflicts.
SQL
CREATE TRIGGER real_time_update
ON totals
FOR UPDATE
AS
BEGIN
    IF EXISTS (SELECT 1 FROM inserted WHERE id = (SELECT id FROM deleted) AND version = (SELECT version FROM deleted))
    BEGIN
        -- Perform the update only if the row was updated and the version matches
        UPDATE totals SET total = total + new_value FROM inserted WHERE id = inserted.id AND version = inserted.version;
    END
END;

Handling Race Conditions:

If a race condition occurs, we can implement exponential backoff or retry logic to handle retries gracefully.

#### Closing Summary
and Common Mistakes/Follow-Ups

By using transaction isolation levels, optimistic locking, and synchronization mechanisms like mutexes, we ensure that only one of the two update operations is processed. This approach avoids double-counting and maintains data consistency during the migration window.

#### Common Mistakes

Not considering race conditions: Failing to account for simultaneous updates can lead to inconsistent states.
Poor transaction management: Incorrect isolation levels or lack of retries can result in partial state changes.
Inadequate versioning: Without proper version checks, conflicts are likely to occur during concurrent updates.

#### Follow-Ups
How would you handle large-scale migrations with multiple tables and complex dependencies?
Can you discuss the trade-offs between using pessimistic locking versus optimistic locking in this context?

**Comparison:**

```
id	total	version
1	50	1
```

**Code:**

```sql
-- Example Application Code Update
BEGIN TRANSACTION;
UPDATE totals SET total = total + 10 WHERE id = 1 AND version = 1; -- Assuming versioning is used
-- Check if the update was successful
IF @@ROWCOUNT = 0
BEGIN
    ROLLBACK;
    RAISERROR('Update failed due to concurrency issues.', 16, 1);
END
ELSE
BEGIN
    COMMIT;
END;

-- Example Trigger for Real-Time Updates
CREATE TRIGGER real_time_update
ON totals
FOR UPDATE
AS
BEGIN
    IF EXISTS (SELECT 1 FROM inserted WHERE id = (SELECT id FROM deleted) AND version = (SELECT version FROM deleted))
    BEGIN
        -- Perform the update only if the row was updated and the version matches
        UPDATE totals SET total = total + 10 FROM inserted WHERE id = inserted.id AND version = inserted.version;
    END
END;
```

**Example input / output:**

```text
**Input (Application Code)**:
- Initial total: 50, new value to add: 10, version: 1

**Output (Successful Transaction)**:
- Final total: 60

**Input (Trigger Update)**:
- Trigger runs after the application update and finds no change in version.

**Output (No Change in Total by Trigger)**:
- Final total remains 60
```

**Data:**

```json
{
  "id": 1,
  "total": 50,
  "version": 1
}
```

**Flow:**

1. Yes
2. No
3. Yes
4. No
5. Start Transaction
6. Update Total in ApplicationCode
7. Is Version Match?
8. Commit Transaction
9. Retry with Exponential Backoff
10. Trigger Update
11. Version Match?
12. Update Total
13. No Change in Total

