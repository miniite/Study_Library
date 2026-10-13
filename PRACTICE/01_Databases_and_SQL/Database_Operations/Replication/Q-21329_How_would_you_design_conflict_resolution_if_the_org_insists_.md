---
type: "practice"
difficulty: "Expert"
---

# How would you design conflict resolution if the org insists on active-active writes across regions despite your reservations, and what invariants do you protect?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Replication Topologies and Lag](<../../../../01_Databases_and_SQL/Database_Operations/Replication/Failback_After_Failover/Replication_Topologies_and_Lag.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When designing a database replication setup that supports active-active writes across multiple regions, ensuring proper conflict resolution is crucial to maintain data consistency and integrity. This scenario involves designing a master-master replication model where both databases can accept writes simultaneously. The key challenge lies in resolving conflicts when two or more nodes write to the same row at different times.

#### Triaging the Question

The question asks us to design a robust conflict resolution strategy for an active-active database setup, despite reservations about potential consistency issues. We need to protect certain invariants such as data integrity and availability while minimizing delays and ensuring that conflicting writes are handled gracefully.

#### First Principles Solution
Replication Model: We will use a master-master replication model where both databases (in different regions) act as masters.
Conflict Detection: Implement mechanisms to detect conflicts at the application level or within the database itself.
Conflict Resolution Policies:
Temporal Conflict Resolution: Use timestamps or version numbers to determine which write should take precedence based on when it was written.
Application-Level Conflict Resolution: Design a fallback strategy where the application decides how to handle conflicts.
Invariants Protection:
Ensure ACID properties are maintained, especially atomicity and consistency.
Protect against data loss or corruption.
Detailed Steps

Database Schema and Data Models

SQL
-- Example schema for a simple transaction table
CREATE TABLE transactions (
    id SERIAL PRIMARY KEY,
    user_id INT NOT NULL,
    amount DECIMAL(10, 2) NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    region VARCHAR(50)
);

Replication Setup

Use a distributed database like CockroachDB or a custom setup with Kafka for asynchronous replication.
Set up bi-directional synchronization between the two regions.

Conflict Detection Mechanisms

SQL
-- Example SQL to detect conflicts based on timestamps
CREATE OR REPLACE FUNCTION detect_conflict(transactions t)
RETURNS BOOLEAN AS $$
BEGIN
    RETURN (SELECT COUNT(*) FROM transactions WHERE user_id = transactions.user_id AND created_at > transactions.created_at) > 0;
END; $$ LANGUAGE plpgsql;

Conflict Resolution Policies

Temporal Conflict Resolution:
SQL
-- Example query to resolve conflicts based on timestamps
UPDATE transactions SET amount = new_amount WHERE id IN (
    SELECT id FROM (
        SELECT id, ROW_NUMBER() OVER (PARTITION BY user_id ORDER BY created_at DESC) as rn
        FROM transactions
        WHERE user_id = 123 AND amount = 50.00 -- Example condition
    ) t
    WHERE rn = 1
);
Application-Level Conflict Resolution:
PYTHON
def resolve_conflict(transactions):
    if len(transactions) > 1:
        # Choose the most recent write or any other strategy
        return max(transactions, key=lambda x: x['created_at'])
    else:
        return transactions[0]

Invariants Protection

Ensure that each transaction is uniquely identifiable (e.g., by user_id and region).
Use version numbers to track the state of data.
Implement strict validation checks before committing writes.

#### Closing Summary

By implementing a combination of temporal conflict resolution, application-level fallback strategies, and strict data validation, we can ensure that our active-active database setup remains robust and consistent. However, it’s important to continually monitor performance and adjust policies as needed to balance consistency with availability.

#### Common Mistakes/Follow-Ups
Not Handling Edge Cases: Ensure thorough testing of edge cases such as simultaneous writes from different regions.
Ignoring Performance Implications: Monitor the impact on database performance and optimize replication mechanisms accordingly.
Failure Modes: Develop comprehensive failure modes and recovery strategies to handle network partitions or other issues.

**Comparison:**

```
Column	Data Type	Description
id	SERIAL	Unique ID for the transaction.
user_id	INT	User performing the transaction.
amount	DECIMAL	Amount of the transaction.
created_at	TIMESTAMP	Timestamp when the transaction was recorded.
region	VARCHAR	Region where the transaction originated.
```

**Code:**

```sql
CREATE OR REPLACE FUNCTION detect_conflict(transactions t)
RETURNS BOOLEAN AS $$
BEGIN
    RETURN (SELECT COUNT(*) FROM transactions WHERE user_id = transactions.user_id AND created_at > transactions.created_at) > 0;
END; $$ LANGUAGE plpgsql;

UPDATE transactions SET amount = new_amount WHERE id IN (
    SELECT id FROM (
        SELECT id, ROW_NUMBER() OVER (PARTITION BY user_id ORDER BY created_at DESC) as rn
        FROM transactions
        WHERE user_id = 123 AND amount = 50.00 -- Example condition
    ) t
    WHERE rn = 1
);

def resolve_conflict(transactions):
    if len(transactions) > 1:
        # Choose the most recent write or any other strategy
        return max(transactions, key=lambda x: x['created_at'])
    else:
        return transactions[0]
```

**Data:**

```sql
CREATE TABLE transactions (
    id SERIAL PRIMARY KEY,
    user_id INT NOT NULL,
    amount DECIMAL(10, 2) NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    region VARCHAR(50)
);
```

**Flow:**

1. Yes
2. No
3. Transaction Write
4. Is Conflict Detected?
5. Resolve Conflicts
6. Write to Both Databases
7. Write to Primary Database
8. Update Application State

