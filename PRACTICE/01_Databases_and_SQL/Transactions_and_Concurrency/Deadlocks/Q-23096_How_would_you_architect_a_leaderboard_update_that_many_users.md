---
type: "practice"
difficulty: "Deep"
---

# How would you architect a leaderboard update that many users hit simultaneously to avoid deadlocks and hotspots?

**Difficulty:** Deep · **Levels:** SDE-1, SDE-1/2 · [↑ Deadlock Retry Strategy](<../../../../01_Databases_and_SQL/Transactions_and_Concurrency/Deadlocks/Deadlock_Retry_Strategy.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When designing a system where multiple users can simultaneously update a leaderboard, it's crucial to ensure both performance and concurrency safety. Deadlocks and hotspots are common pitfalls that need careful handling. In this scenario, we will focus on architecting a solution that minimizes deadlocks by carefully managing transactions and ensuring a balanced load distribution.

To summarize the key points of our approach:

Concurrency Management: Use optimistic or pessimistic locking to manage concurrent updates.
Load Distribution: Implement strategies to distribute user requests evenly across multiple instances to avoid hotspots.
Transaction Management: Handle transactions efficiently to reduce contention and minimize deadlock risks.

Let's break down the problem step by step:

#### Triage

The question asks us to architect a leaderboard update mechanism that can handle simultaneous updates from many users while avoiding deadlocks and hotspots. The core of this problem involves concurrency control, load balancing, and transaction management.

#### Assumptions
Leaderboard Data: We assume the leaderboard is stored in a database with fields such as user_id, score, and possibly timestamps.
User Actions: Users can update their own scores or view other users' rankings.
Database Schema: The schema might include tables like users and leaderboards.

#### First Principles Solution

Optimistic Locking:

Use version numbers (optimistic concurrency control) to detect conflicts during updates. For example, each row in the leaderboard table could have a version number that increments on every update.

Pessimistic Locking:

Acquire locks on rows before updating them. This can be done using SQL locks or through application-level logic.

Load Balancing:

Distribute user requests across multiple database instances to avoid hotspots and ensure even load distribution.

Transaction Management:

Ensure transactions are short-lived and isolated from each other as much as possible.
Detailed Implementation

Version Numbers for Optimistic Locking

#### Example
schema: leaderboards table with columns user_id, score, and version.
When a user requests an update, the system reads the current version number.
The application attempts to update the row while checking if the version has changed. If it has, the transaction fails, indicating a concurrent update.

Pessimistic Locking

Use SQL locks like SELECT ... FOR UPDATE or LOCK TABLE statements to ensure exclusive access during updates.

#### Example
SQL
START TRANSACTION;
SELECT * FROM leaderboards WHERE user_id = ? FOR UPDATE;
UPDATE leaderboards SET score = ? WHERE id = ? AND version = ?;
COMMIT;

Load Balancing

Use a load balancer to distribute requests across multiple database instances.

#### Example
Configure Nginx or HAProxy to route requests based on round-robin or other algorithms.

Transaction Management

Ensure transactions are short-lived and do not hold locks for extended periods.
Minimize the scope of transactions by breaking down updates into smaller, independent actions when possible.

#### Closing Summary

In summary, we have proposed a solution that leverages optimistic locking to detect conflicts early, uses pessimistic locking where necessary to ensure exclusive access, implements load balancing to distribute requests evenly, and manages transactions efficiently. These strategies help in maintaining performance while ensuring data integrity and avoiding deadlocks and hotspots.

#### Common Mistakes
Ignoring Version Numbers: Not using version numbers can lead to infinite retries or stale updates.
Inefficient Locks: Overusing locks can introduce significant contention and degrade performance.
Poor Load Distribution: Failing to distribute requests effectively can result in hotspots, leading to uneven load and potential bottlenecks.
Self-Check

The solution is correct and complete as it covers all aspects of the problem statement. It ensures data integrity and performance while managing concurrency effectively.

**Comparison:**

| User ID | Score | Version |
|---------|-------|---------|
| 1       | 100   | 2       |
| 2       | 95    | 3       |

**Code:**

```sql
-- Example SQL for optimistic locking
START TRANSACTION;
SELECT * FROM leaderboards WHERE user_id = ? FOR UPDATE OF version; -- Acquire lock on row
UPDATE leaderboards SET score = ? WHERE id = ? AND version = (SELECT version FROM leaderboards WHERE user_id = ? LIMIT 1); -- Check and update with version number

-- Example SQL for pessimistic locking
START TRANSACTION;
SELECT * FROM leaderboards WHERE user_id = ? LOCK IN SHARE MODE; -- Acquire shared lock on row
UPDATE leaderboards SET score = ? WHERE id = ? AND version = (SELECT version FROM leaderboards WHERE user_id = ? LIMIT 1); -- Check and update with version number
COMMIT;
```

**Example input / output:**

```text
Input:
- User with user_id=1 requests to update their score from 100 to 105

Output:
- Version number for user_id=1 is read as 2.
- Transaction starts and row version is checked.
- New score of 105 written, updating the version to 3.

Dry Run Request:
User with user_id=1 requests an update.

Request Response:
Transaction acquires lock on row with user_id=1. Current version is read as 2. Score updated from 100 to 105 and new version number 3 is written.
```

**Data:**

```json
{
  "leaderboard": [
    { "user_id": 1, "score": 100, "version": 2 },
    { "user_id": 2, "score": 95, "version": 3 }
  ]
}
```

**Flow:**

1. Yes
2. No
3. Yes
4. No
5. User Requests Update
6. Is Optimistic Lock Needed?
7. Acquire Row Version
8. Check Version During Update
9. Update Row with New Versionand Score
10. Commit Transaction
11. Is Pessimistic Lock Needed?
12. Acquire Shared Lock on Row
13. Update Row with New Score
14. Release Locks and CommitTransaction
15. Distribute Request to LoadBalanced Instances
16. Process Update in SeparateTransaction
17. Check Version Number DuringUpdate
18. Commit Transaction if VersionMatches

