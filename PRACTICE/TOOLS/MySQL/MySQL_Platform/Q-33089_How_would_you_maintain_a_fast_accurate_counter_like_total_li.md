---
type: "practice"
difficulty: "Deep"
---

# How would you maintain a fast, accurate counter (like total likes) without running COUNT(*) on every read?

**Difficulty:** Deep · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ MySQL Performance Diagnosis](<../../../../TOOLS/MySQL/MySQL_Platform/MySQL_Performance_Diagnosis.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To maintain a fast and accurate counter such as "total likes" in MySQL without performing COUNT(*) operations on every read, we can leverage a combination of counters and caching strategies. The core idea is to use a denormalized approach where the counter is stored separately from the actual data and updated incrementally. This ensures that reads are fast and writes are efficient while maintaining accuracy.

#### Triage

The question is essentially asking for an optimization technique to reduce read operations on the database, specifically when dealing with a frequently accessed aggregate metric like "total likes". The interviewer might be probing your understanding of how MySQL handles caching, indexing, and denormalization. We need to make assumptions about the frequency of updates versus reads and consider different trade-offs.

#### Solution
Separate Counter Table: Create a separate table that stores the counter value.
Incremental Updates: Update this counter incrementally when likes are added or removed.
Caching: Cache the counter in memory to reduce database hits during read operations.
Replication and Consistency: Ensure consistent updates across replicas if using a distributed system.
Step-by-Step Solution

Design the Counter Table:

SQL
CREATE TABLE likes_counter (
    id INT PRIMARY KEY,
    total_likes BIGINT DEFAULT 0
);

Incremental Updates:

When a user likes an item, increment the counter in the likes_counter table.
This can be done using a single SQL statement with appropriate transaction isolation levels to ensure atomicity and consistency.

Caching:

Cache the current value of total_likes in memory (e.g., Redis) when it is updated.
Provide an API endpoint that reads the cached value instead of querying the database directly.

Replication and Consistency:

Use MySQL replication to keep a secondary or multiple copies of the counter table up-to-date.
Synchronize the cache with these replicas to ensure consistency across all nodes.

#### Closing Summary

By maintaining a separate counter table, incrementally updating it, caching the result in memory, and ensuring consistent updates through replication, we can significantly reduce the load on the database while keeping the "total likes" metric up-to-date. This approach strikes a balance between performance and accuracy, making it suitable for high-traffic applications.

#### Common Mistakes/Follow-Ups
Ignoring Replication: Failing to use proper replication mechanisms might lead to inconsistent counters across nodes.
Not Considering Cache Eviction: Without proper eviction policies in the cache, stale data could be served during read operations.
Over-Reliance on Database Caching: While MySQL does have caching mechanisms (like query cache), relying solely on this can introduce complexities and performance issues.

**Code:**

```sql
-- Example SQL for updating the likes counter
 UPDATE likes_counter SET total_likes = total_likes + 1 WHERE id = 1;
```

**Example input / output:**

```text
Input: Increment like count for item with ID 1
 Output: Success, likes_counter.total_likes updated to new value

 Input: Get current like count for item with ID 1 from cache
 Response: Current total_likes (e.g., 500)
```

**Flow:**

1. User Likes Item
2. Increment likes_counter Table
3. Update Cache with New Count
4. Read Like Count from Cache
5. Return Current Count (e.g.,501)

