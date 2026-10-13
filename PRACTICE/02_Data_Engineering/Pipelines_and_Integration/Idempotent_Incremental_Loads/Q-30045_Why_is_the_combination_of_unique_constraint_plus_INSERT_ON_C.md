---
type: "practice"
difficulty: "Deep"
---

# Why is the combination of unique constraint plus INSERT ... ON CONFLICT a common idempotency pattern and where does it fail at scale?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Idempotent Incremental Loads](<../../../../02_Data_Engineering/Pipelines_and_Integration/Idempotent_Incremental_Loads/Idempotent_Incremental_Loads.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address why the combination of a unique constraint plus INSERT ... ON CONFLICT is commonly used in idempotent pipelines, we need to understand its core benefits and limitations. This pattern ensures that duplicate records are not inserted multiple times, which is crucial for maintaining data integrity and preventing redundant processing. However, at scale, this approach can fail due to several factors including the overhead of conflict detection and resolution, transactional costs, and potential performance bottlenecks.

The INSERT ... ON CONFLICT SQL pattern works by first checking if a unique constraint on one or more columns is violated during an insert attempt. If it does (indicating that the record already exists), the conflicting rows are identified, and specific actions can be taken (e.g., update another column). This mechanism ensures idempotency because identical requests result in no change to the database state.

However, at scale, this pattern faces several challenges:

Conflict Detection Overhead: As the number of records increases, so does the likelihood that a conflict will occur. The overhead of checking for and resolving conflicts can become significant.
Transaction Costs: Each ON CONFLICT insert operation requires a transaction to manage the concurrent execution and ensure data consistency. At scale, managing these transactions efficiently is challenging.
Performance Bottlenecks: In high-throughput systems, frequent conflict checks can lead to performance degradation. This is especially true when multiple conflicting operations are in play simultaneously.

To illustrate these points, consider an example where we use INSERT ... ON CONFLICT to append user activity records into a database. If many users perform the same action (e.g., logging into an app) within a short period, frequent conflicts can occur, leading to increased load on the system and potential performance issues.

**Code:**

```sql
-- Example SQL using ON CONFLICT for idempotent insertions
CREATE TABLE user_activity (
    user_id INT,
    activity_timestamp TIMESTAMP,
    action VARCHAR(50),
    PRIMARY KEY (user_id, activity_timestamp) -- Ensures uniqueness based on user and timestamp
);

INSERT INTO user_activity (user_id, activity_timestamp, action)
VALUES (1, '2023-10-01 10:00:00', 'login')
ON CONFLICT (user_id, activity_timestamp)
DO NOTHING; -- Do nothing if the record already exists

-- This pattern ensures that identical logins are not recorded multiple times
```

**Example input / output:**

```text
Input 1:
- User ID: 1
- Activity Timestamp: 2023-10-01 10:00:00
- Action: login

Output 1:
- No new record is inserted because the exact combination of (user_id, activity_timestamp) already exists.

Input 2:
- User ID: 1
- Activity Timestamp: 2023-10-01 10:05:00
- Action: login

Output 2:
- A new record is inserted with the updated timestamp.
```

**Flow:**

1. Yes
2. No
3. User Activity
4. Is Record Unique?
5. Do Nothing
6. Insert New Record
7. Check Next User
8. Mark as Processed

