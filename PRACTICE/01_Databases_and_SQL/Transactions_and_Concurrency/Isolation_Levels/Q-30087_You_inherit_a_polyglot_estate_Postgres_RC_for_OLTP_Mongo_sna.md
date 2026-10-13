---
type: "practice"
difficulty: "Expert"
---

# You inherit a polyglot estate: Postgres RC for OLTP Mongo snapshot for docs and DynamoDB for sessions. How do you reason about end-to-end isolation for a workflow that spans all three?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Isolation with Read Replicas](<../../../../01_Databases_and_SQL/Transactions_and_Concurrency/Isolation_Levels/Setting_Isolation_Levels/Isolation_with_Read_Replicas.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In this scenario, we're dealing with an architecture that spans multiple databases with varying isolation levels (Postgres RC, Mongo snapshot, and DynamoDB), which introduces complexity in ensuring end-to-end consistency across these systems. The key idea is to understand the implications of each isolation level on read-write operations and how they interact within a distributed workflow.

First, let's clarify what we mean by "isolation" in this context:

Read Committed (RC): Ensures that dirty reads are not allowed; you cannot see uncommitted writes from other transactions.
Serializable: Enforces the highest level of isolation, preventing any form of non-repeatable reads or phantom reads.

Next, we need to consider how each database's isolation level affects our workflow. We'll start with understanding the implications for each:

Postgres RC:

Ensures that read operations do not see uncommitted writes.
May lead to dirty reads if a transaction is committed just after it starts.

Mongo Snapshot:

Provides snapshot isolation, meaning all read operations within a session see the same snapshot of data.
However, this snapshot is taken at the start of the session and does not reflect any changes made by other transactions.

DynamoDB:

DynamoDB uses eventual consistency by default but supports strong consistency through strongly consistent reads.
This means that if you perform a strongly consistent read immediately after writing, you will see the updated data.

Given these isolation levels, we need to reason about how to manage transactions across different databases and ensure that our workflow maintains the desired level of isolation. Here’s a step-by-step approach:

Step 1: Define Transaction Boundaries
Identify clear transaction boundaries for each database operation.
Use database-specific transaction management features (e.g., Postgres's BEGIN, COMMIT, ROLLBACK).
Step 2: Ensure Consistency Across Databases
For Postgres, use serializable transactions to enforce the highest level of isolation.
For Mongo, ensure that snapshot reads are used consistently within a session.
For DynamoDB, use strongly consistent reads where necessary.
Step 3: Handle Data Synchronization
Use message queues or event sourcing to manage data synchronization between databases.
Ensure that writes to one database trigger corresponding updates in others using idempotent operations and retries.
Step 4: Implement Compensating Actions
Define compensating actions for scenarios where a transaction fails, ensuring data consistency is maintained even if the workflow is interrupted.

#### Example
Workflow:

#### Cons
ider an order placement workflow where we need to update inventory in Postgres, create a document record in Mongo, and store session data in DynamoDB:

Start Transaction in Postgres:

SQL
BEGIN;
-- Perform read operations to check inventory availability
-- If available, perform the write operation to update inventory
UPDATE inventory SET quantity = quantity - 1 WHERE product_id = 'XYZ';

Create Document Record in Mongo:

JAVASCRIPT
// Start a session with snapshot isolation
db.startSession({ readConcern: { level: "snapshot" } });
try {

#### cons
t doc = {
    orderId: '12345',
    productId: 'XYZ'
  };
  db.orders.insertOne(doc);
} finally {
  db.endSession();
}

Store Session Data in DynamoDB:

PYTHON
import boto3

dynamodb = boto3.resource('dynamodb')
table = dynamodb.Table('sessions')

response = table.put_item(
    Item={
        'session_id': '12345',
        'user_id': '67890'
    }
)

#### Closing Summary

To ensure end-to-end isolation for a workflow spanning Postgres RC, Mongo snapshot, and DynamoDB, we must carefully define transaction boundaries and use appropriate isolation levels within each database. By leveraging database-specific features (serializable transactions in Postgres, snapshot reads in Mongo, and strongly consistent reads in DynamoDB), we can manage data consistency and synchronization effectively.

#### Common Mistakes

Failing to properly handle transactions across databases.
Not using snapshot or serializable isolation where needed.
Ignoring the implications of eventual vs. strong consistency.

**Comparison:**

| Database | Isolation Level | Example Operation |
|----------|----------------|------------------|
| Postgres  | RC             | Update inventory |
| Mongo     | Snapshot       | Insert document   |
| DynamoDB  | Strong Consistency | Store session data |

**Code:**

```

-- Example Postgres RC transaction BEGIN; SELECT quantity FROM inventory WHERE product_id = 'XYZ'; UPDATE inventory SET quantity = quantity - 1 WHERE product_id = 'XYZ' AND quantity > 0; COMMIT;

CODE
Copy

```javascript
// Example Mongo snapshot session
db.startSession({ readConcern: { level: "snapshot" } });
try {
  const doc = {
    orderId: '12345',
    productId: 'XYZ'
  };
  db.orders.insertOne(doc);
} finally {
  db.endSession();
}
PYTHON
Copy
# Example DynamoDB put_item with strongly consistent read
import boto3

dynamodb = boto3.resource('dynamodb')
table = dynamodb.Table('sessions')

response = table.put_item(
    Item={
        'session_id': '12345',
        'user_id': '67890'
    }
)
```

**Example input / output:**

```text
Input:
- Inventory check for product 'XYZ'
- Insert order record in Mongo with snapshot isolation
- Store session data in DynamoDB

Output:
- Postgres transaction commits successfully
- Document inserted into MongoDB with snapshot isolation
- Session data stored in DynamoDB using strongly consistent read
```

**Data:**

```json
{
  "inventory": [
    {"product_id": "XYZ", "quantity": 100}
  ],
  "orders": [
    {"orderId": "12345", "productId": "XYZ"}
  ],
  "sessions": [
    {"session_id": "12345", "user_id": "67890"}
  ]
}
```

**Flow:**

1. Available
2. Commit
3. Start Transaction in Postgres
4. Check Inventory
5. Update Inventory
6. Commit/Retry
7. Create Document Record inMongo
8. End Session
9. Store Session Data inDynamoDB

