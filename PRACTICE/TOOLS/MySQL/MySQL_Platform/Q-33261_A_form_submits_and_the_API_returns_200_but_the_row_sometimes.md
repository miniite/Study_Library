---
type: "practice"
difficulty: "Intermediate"
---

# A form submits and the API returns 200 but the row sometimes isn't visible on the next page load — how do you debug this write-then-read-replica consistency gap end to end?

**Difficulty:** Intermediate · **Levels:** SDE-1/2, SDE-2/3 · [↑ Read Your Writes in MySQL](<../../../../TOOLS/MySQL/MySQL_Platform/Read_Your_Writes_in_MySQL.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To diagnose and resolve a write-then-read-replica consistency issue in MySQL, where a row is sometimes not visible on subsequent page loads after an API submit, we need to methodically understand the system's architecture, explore potential causes, and validate our findings. This involves examining the database replication setup, transaction isolation levels, network latency, and caching mechanisms.

First, let's clarify what the question is probing:

Understanding of MySQL Replication: How the write-to-replica process works.
Transaction Isolation Levels: What level might be causing this issue.

#### Cons
istency Models: The differences between eventual consistency and strong consistency.
Debugging Techniques: Steps to identify where the problem lies.

#### Triage

The key idea here is that the write operation is succeeding but not immediately visible on read. This suggests a potential gap in replication or transactional consistency. We need to check if the issue is due to replication lag, transaction isolation settings, or other factors such as caching or network latency.

#### Assumptions

The application uses MySQL.
There are write and read replicas for scalability and availability.
Network and caching layers could potentially impact visibility.
The problem occurs consistently but not every time.

#### Solution
Step 1: Verify Replication Setup

Ensure that the replication is correctly configured. Check if there's any delay in applying writes to the replica:

SQL
SHOW SLAVE STATUS\G

This command provides details about the status of the slave (replica) including Seconds_Behind_Master. If this value is significantly large, it indicates a delay.

Step 2: Examine Transaction Isolation Levels

Check the transaction isolation level being used. MySQL supports several levels like READ UNCOMMITTED, READ COMMITTED, REPEATABLE READ, and SERIALIZABLE:

SQL
SELECT @@tx_isolation;

For most production systems, REPEATABLE READ is commonly used but might not be the best choice for write-heavy operations. Consider changing it to READ COMMITTED, which can reduce replication lag.

Step 3: Test Replication Lag

Use tools like pt-heartbeat or custom scripts to test if writes are being applied to replicas in a timely manner:

SH
# Example script snippet (pseudo-code)
while true; do
    echo "Writing to master"
    mysql -e "INSERT INTO table VALUES (...)"
    sleep 1 # Wait for replication to kick in
    echo "Reading from replica"
    mysql -h <replica_host> -e "SELECT * FROM table WHERE id = <last_insert_id>"
done
Step 4: Network and Latency Checks

Check network latency between the master and replicas using tools like ping or traceroute. High latency can cause delays in replication.

Step 5: Caching Mechanisms

Ensure that caching layers (like Redis) are not storing stale data. Verify cache invalidation policies to ensure they are correctly updated after writes.

#### Closing Summary

To debug the issue, start by verifying the MySQL replication setup and transaction isolation level. Check for consistent lag using monitoring tools or custom scripts. Ensure network latency is minimal and that caching mechanisms are up-to-date with recent changes.

#### Common Mistakes

Overlooking network latency as a factor.
Not testing writes directly on replicas to confirm they are being applied correctly.
Ignoring the impact of transaction isolation levels on consistency.

By following these steps, you can systematically identify the root cause and resolve the inconsistency gap between write operations and subsequent reads.

**Flow:**

1. Yes
2. No
3. Yes
4. Start
5. Verify Replication Setup
6. Mysql `SHOW SLAVE STATUS`
7. Test Replication Lag
8. Check Transaction IsolationLevel
9. Change to Read Committed
10. Test Writes on Replica
11. Verify Network Latency
12. Check Caching Mechanisms
13. Apply Changes and Test
14. Network Diagnostics
15. Mysql `SHOW ENGINEINNODB STATUS` for moredetails

