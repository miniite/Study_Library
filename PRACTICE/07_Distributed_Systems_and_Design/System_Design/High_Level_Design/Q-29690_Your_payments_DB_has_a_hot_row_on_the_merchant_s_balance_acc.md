---
type: "practice"
difficulty: "Deep"
---

# Your payments DB has a hot row on the merchant's balance account during peak sale hours causing lock contention — how do you redesign the balance update to remove the hotspot while keeping the running balance queryable?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Payment Ledgers](<../../../../07_Distributed_Systems_and_Design/System_Design/High_Level_Design/Payment_Ledgers.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the hot row issue in our payments database, we need to design a high-level solution that ensures both scalability and availability without compromising on transactional integrity or read consistency. The key idea is to partition the balance updates into smaller, more manageable units of work while still allowing for real-time query access.

The question primarily probes your understanding of distributed systems principles, such as sharding, caching, and eventual consistency models. We need to make several assumptions before we dive into the solution:

The database uses a row-based storage model.
There are frequent updates to merchant balances during peak hours.
Real-time balance queries should be available but with some delay tolerance.

We will reason step-by-step from first principles, starting by defining the current problem and then moving on to designing a more scalable solution.

Understand the Current Problem:

The existing system updates the merchant’s balance in a single row during each transaction.
This leads to hotspots because many concurrent transactions attempt to update the same row simultaneously.
As a result, contention arises, causing delays and potential bottlenecks during peak hours.

Design the Solution:

Sharding by Merchant ID: We can shard the merchant balances across multiple databases or tables based on the merchant's unique identifier (ID). This reduces the load on any single database.
Decompose the Update Operation: Instead of updating a single row, we can break down the balance update into smaller units. For example, each transaction could be recorded as an atomic operation in its own log or stream.
Use Eventual Consistency Model: To ensure real-time queries are still possible while maintaining performance and availability, we can use a combination of event sourcing and caching.

#### Implementation
Steps:

Step 1: Sharding by Merchant ID:

Transaction
Record in Shard
Merchant Data
Sharded DB Table

Each merchant's data is stored across multiple shards, reducing the load on any single database. This ensures that each transaction can be processed independently.

Step 2: Atomic Transaction Logging:

PYTHON
def update_balance(transaction):
    log_entry = {"timestamp": datetime.now(), "operation_type": "credit/debit", "amount": amount, "merchant_id": merchant_id}
    stream.append(log_entry)

Each transaction is recorded in a log or event stream. This allows us to maintain a history of all transactions for each merchant.

Step 3: Caching and Aggregation:

Event Source
Stream Processor
Cached Balance
Database Query
Real-Time API Response

Stream processors aggregate these log entries to update a cached balance for each merchant. This ensures that real-time queries can be served from the cache, reducing database load.

#### Closing Summary

By sharding merchants’ data and decomposing updates into atomic operations, we mitigate hotspots.
Using event sourcing and caching allows us to maintain performance during peak hours while ensuring eventual consistency in balance updates.
This design balances scalability, availability, and real-time query capabilities.

#### Common Mistakes/Follow-Ups

Failing to properly shard the data can lead to uneven distribution and continued hotspots.
Not implementing proper caching strategies might result in excessive database load during peak hours.
Ignoring eventual consistency models could introduce errors if queries are too sensitive to delays.

**Code:**

```python
def update_balance(transaction):
    log_entry = {"timestamp": datetime.now(), "operation_type": "credit/debit", "amount": transaction.amount, "merchant_id": transaction.merchant_id}
    stream.append(log_entry)
```

**Flow:**

1. Transaction
2. Record in Shard
3. Merchant Data
4. Sharded DB Table
5. Event Source
6. Stream Processor
7. Cached Balance
8. Database Query
9. Real-Time API Response

