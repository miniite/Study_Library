---
type: "practice"
difficulty: "Deep"
---

# How do you keep a cache and the database consistent when a transaction commits, avoiding stale reads?

**Difficulty:** Deep · **Levels:** SDE-1, SDE-1/2 · [↑ ACID Durability and Consistency](<../../../../01_Databases_and_SQL/Transactions_and_Concurrency/ACID_Transactions/ACID_Durability_and_Consistency.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In this scenario, we're dealing with ensuring that our in-memory cache remains synchronized with the database after a transaction commit to avoid any stale reads. This is a classic problem known as maintaining data consistency between distributed systems, often addressed through mechanisms like read-through caching and write-back caching.

#### Triaging the Question

The question essentially asks how to manage transactions so that changes made by committed transactions are reflected in both the database and the cache in real-time or with minimal delay. This involves understanding ACID properties (Atomicity, Consistency, Isolation, Durability), especially focusing on consistency and isolation. The key assumptions include:

There is a caching layer over the database.
Transactions can modify data in both the cache and the database.
We want to avoid stale reads where cached data might not reflect recent updates.

#### First-Principles Solution

To ensure that our cache and database stay consistent, we need to implement a strategy that propagates changes from the transaction log or database directly to the cache. Here’s a step-by-step solution:

Database Transaction Commit: When a transaction commits in the database, it writes its changes to the transaction log.
Transaction Log Propagation: The transaction log is monitored by a cache update service that detects committed transactions and updates the cache accordingly.
Cache Update Service: This service processes the transaction logs and ensures that the cache reflects the latest state of the data.
Detailed Workflow

Transaction Commit:

A client application initiates a write operation (e.g., insert, update) on the database.
The database system performs the necessary operations and writes the changes to its transaction log.
Once the commit is successful, the transaction log contains all necessary information for the cache update.

Transaction Log Monitoring:

A dedicated service or thread monitors the transaction logs in real-time.
This service reads each committed transaction from the log and extracts relevant data needed to update the cache.

Cache Update Service:

The service processes the extracted data and updates the corresponding entries in the cache.
For operations like insert or update, it ensures the cache reflects the new state.
For delete operations, it removes the outdated entry from the cache.
Handling Edge Cases
Concurrency: If multiple transactions commit simultaneously, ensure that the cache service can handle them without causing conflicts. One approach is to use atomic updates and ensure thread safety.
Performance: Monitoring transaction logs in real-time might add overhead. Consider optimizing this by batching log reads or using a queue system for processing.
Failure Handling: Implement robust error handling and retry mechanisms in case the cache update fails due to network issues or other errors.

#### Closing Summary

By implementing a separate service that monitors database transaction logs, we can ensure that our cache always reflects the latest state of data. This approach leverages ACID properties for consistency while maintaining high availability by caching frequently accessed data.

#### Common Mistakes
and Follow-Ups:

Failing to properly monitor the transaction logs or using an inefficient method.
Not implementing robust error handling, leading to data inconsistencies during failures.
Overlooking concurrency issues that could cause race conditions in updating the cache.

**Comparison:**

```
Operation	Cache Action
INSERT	Insert value
UPDATE	Update value
DELETE	Delete entry
```

**Code:**

```python
# Example Cache Update Service Code

class CacheUpdateService:
    def __init__(self, transaction_log_queue):
        self.transaction_log_queue = transaction_log_queue

    def process_transactions(self):
        while True:
            # Poll for new transactions from the queue
            transaction = self.transaction_log_queue.get()

            if transaction.type == 'INSERT':
                self.cache.insert(transaction.key, transaction.value)
            elif transaction.type == 'UPDATE':
                self.cache.update(transaction.key, transaction.value)
            elif transaction.type == 'DELETE':
                self.cache.delete(transaction.key)

    def start(self):
        threading.Thread(target=self.process_transactions).start()
```

**Example input / output:**

```text
Input:
- Transaction log containing insert, update, and delete operations.

Output:
- Cache entries updated to reflect the latest state of the database.
```

**Data:**

```json
{
  "transaction_log": [
    {"type": "INSERT", "key": "user123", "value": {"name": "Alice"}},
    {"type": "UPDATE", "key": "user123", "value": {"age": 30}},
    {"type": "DELETE", "key": "user456"}
  ]
}
```

**Flow:**

1. Yes
2. No
3. Transaction Commits
4. Monitored?
5. Update Cache Service
6. Process Transaction Log
7. Cache Updated
8. Continue Monitoring

