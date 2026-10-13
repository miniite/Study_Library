---
type: "practice"
difficulty: "Deep"
---

# How do you cache paginated data so that inserts/deletes don't cause items to be skipped or repeated across pages?

**Difficulty:** Deep · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Stale While Revalidate](<../../../../07_Distributed_Systems_and_Design/System_Design/Caching/Stale_While_Revalidate.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Caching paginated data is a common requirement in web applications, especially when dealing with large datasets and frequent user interactions. The key challenge here is ensuring that cache updates do not result in items being skipped or repeated across pages. To address this, we need to carefully consider the caching strategy, update mechanisms, and synchronization techniques. In this solution, I'll explain a detailed approach from first principles.

#### Triaging the Question

The core of the question revolves around handling paginated data in a way that maintains consistency and correctness when performing CRUD operations (Create, Read, Update, Delete). Specifically, we need to ensure that cached pages do not exhibit issues such as missing items or duplicate entries. This requires understanding how caching works, how updates are propagated, and how cache coherence is maintained.

#### Assumptions
and Context
Pagination: We use a standard pagination mechanism with offset-based queries.
Caching Strategy: We employ a local cache on the server side to reduce database load.
Database Schema: A simple table items with fields: id, name, created_at.
Operations: Insertions, deletions, and updates are frequent.

#### Solution
Cache Structure: Design the cache as a mapping of pagination tokens (offsets) to data slices.
Cache Update Mechanism: Implement a mechanism that ensures cache updates are atomic and consistent across all relevant pages.
Synchronization Techniques: Use versioning or timestamp-based synchronization to handle concurrent updates.
Step 1: Cache Structure

To manage cached paginated data, we can use a hash map where the key is a unique token (e.g., offset) and the value is the corresponding slice of data. For example:

Token
Yes
No
Cache
Token Exists?
Cached Data Slice
Load Fresh Data
Update Cache with New Slice
Step 2: Cache Update Mechanism

When an update occurs (insert, delete, or update), we need to ensure that the cache is updated atomically. This can be achieved by:

Versioning: Each data slice in the cache has a version number.
Timestamps: Updates include timestamps which allow us to track changes.
Step 3: Synchronization Techniques

To handle concurrent updates, we use distributed locking mechanisms or optimistic concurrency controls like MVCC (Multi-Version Concurrency Control).

#### Implementation
Details

Insert Operation:

Load the relevant data slice from cache.
Insert the new item in the correct position.
Update the version number and timestamp of the cache entry.

Delete Operation:

Locate the item to be deleted.
Mark it as deleted in the local cache.
Update the version number and timestamp.

Update Operation:

Load the relevant data slice from cache.
Find the item to update.
Apply changes and update the version number and timestamp.
Code Example
PYTHON
class CacheManager:
    def __init__(self):
        self.cache = {}
        self.version = 0

    def get_data_slice(self, offset, size):
        if offset in self.cache:
            return self.cache[offset], self.version
        else:
            # Load fresh data from database
            data_slice, new_version = load_from_db(offset, size)
            self.cache[offset] = data_slice
            self.version = new_version + 1
            return data_slice, self.version

    def update_data(self, offset, updated_items):
        if offset not in self.cache:
            raise KeyError(f"Cache entry for {offset} does not exist")

        old_data, version = self.get_data_slice(offset, size)
        # Apply updates and ensure consistency
        new_data = apply_updates(old_data, updated_items)

        self.cache[offset] = new_data
        self.version += 1

def load_from_db(offset, size):
    # Simulate database operation
    return [f"Item {i}" for i in range(offset, offset + size)], 42

def apply_updates(old_data, updates):
    updated_data = old_data.copy()
    for update in updates:
        index = find_index_by_id(updated_data, update["id"])
        if index is not None:
            updated_data[index] = update
    return updated_data

#### Closing Summary
and Common Mistakes/Follow-Ups

In summary, caching paginated data requires a robust strategy that ensures consistency across multiple pages. By using versioning, timestamps, and atomic updates, we can maintain correctness even with frequent CRUD operations.

#### Common Mistakes

Failing to update cache entries atomically.
Not handling concurrent updates properly.
Ignoring edge cases like partial page refreshes or large deletions.

#### Follow-Ups

How would you handle very large datasets?
What about the impact on performance and memory usage?
Can you discuss how this solution scales with increasing traffic?

**Flow:**

1. Get Slice
2. Yes
3. No
4. Cache Manager
5. Token Exists?
6. Cached Data
7. Load Fresh Data
8. Update Cache with New Slice

