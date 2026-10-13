---
type: "practice"
difficulty: "Deep"
---

# A service intermittently sees "Duplicate entry for key PRIMARY" on an INSERT it believes is unique — how do you trace whether it's a retry, replication, or app-side ID reuse?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Deadlocks in MySQL](<../../../../TOOLS/MySQL/MySQL_Platform/Deadlocks_in_MySQL.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To diagnose why a Duplicate entry for key PRIMARY error occurs during what your service believes to be a unique INSERT operation, we need to systematically investigate potential causes such as retries, replication lags, or application-side ID reuse. This issue hints at underlying problems with data consistency and transactional integrity that require careful tracing and troubleshooting.

The core of the problem is understanding where the inconsistency arises. Let’s break it down step by step:

#### Triage
What Are We Probing?
Retry Mechanism: The service might be retrying an operation due to a transient error, leading to multiple attempts.
Replication Lag: In a distributed system with MySQL replication, there might be a lag between writes on the master and slaves, causing conflicts.
Application-Side ID Generation: The application could generate duplicate IDs for records.

#### Assumptions
We assume the service uses auto-increment or a unique identifier strategy (like UUIDs) for primary keys.
MySQL’s default behavior is to enforce uniqueness constraints strictly.
Step-by-Step Solution

Service-Level Analysis:

Log Review: Check logs from the application that generated the insert operation. Look for any retry logic and timestamps of attempts.
Database Operations: Ensure the service handles retries gracefully, possibly with exponential backoff or circuit breaker patterns.

Replication Lag Investigation:

Primary Key Consistency Across Replicas: Use SHOW SLAVE STATUS to check if the master is ahead of all replicas.
Binlog Position: Check binlog positions and timestamps on both master and replica databases to see if there are significant lags.

Application-Side ID Generation:

ID Generation Algorithm: Examine the application’s ID generation logic, especially in scenarios where auto-increment is used or custom UUIDs are generated.
Concurrency Control: Ensure that the service manages concurrent requests properly to avoid duplicate IDs being generated simultaneously.
Detailed Steps

Enable Debug Logging: Add detailed logging around the INSERT operation. Log primary key values before and after the attempt, along with timestamps. This will help trace whether multiple attempts are happening.

Review Retry Logic: If retry logic is in place, ensure it’s correctly configured to handle transient errors without accidentally duplicating operations.

Replication Lag Check:

Use SHOW SLAVE STATUS to monitor the replication status and check for any significant lags.
Compare timestamps of transactions on master and replicas using tools like pt-table-checksum.

Application-Side ID Uniqueness Verification:

For auto-increment, ensure there are no gaps or reuse patterns in primary keys over time.
For custom IDs (UUIDs), verify the distribution and uniqueness across your application’s operations.

#### Closing Summary

By systematically tracing potential sources of duplicates—service retries, replication lags, or application-side ID generation—we can pinpoint the exact cause. This involves detailed logging, understanding retry mechanisms, verifying replication status, and examining ID generation algorithms.

#### Common Mistakes/Follow-Ups
Misconfigured Retries: Ensuring retries aren't inadvertently causing duplicate operations.
Replication Misconfigurations: Incorrect setup leading to lags or out-of-sync issues.
ID Generation Bugs: Unseen bugs in the application logic generating non-unique IDs.

**Comparison:**

| Key      | Value                  |
|----------|------------------------|
| Retry    | Enabled, configured    |
| Replication Lag | Not significant  |
| UUID Generation   | Unique per request     |

**Code:**

```sql
-- Example SQL commands for debugging
SHOW SLAVE STATUS;
```

**Example input / output:**

```text
Input:
Service attempts to insert record with primary key 1234, but encounters a 'Duplicate entry for key PRIMARY' error.

Expected Output:
- Check logs for retry patterns.
- Verify replication status and binlog positions.
- Inspect application-side ID generation logic.
```

**Data:**

```json
{
  "logEntries": [
    {
      "timestamp": "2023-10-05 14:00:00",
      "operation": "INSERT",
      "primary_key": 1234,
      "success": false
    },
    {
      "timestamp": "2023-10-05 14:01:00",
      "operation": "RETRY INSERT",
      "primary_key": 1234,
      "success": true
    }
  ]
}
```

**Flow:**

1. Log Review
2. Check for Retries
3. Review Retry Logic
4. Replication Check
5. Verify Replication Status
6. ID Generation Analysis
7. Examine UUID Distribution

