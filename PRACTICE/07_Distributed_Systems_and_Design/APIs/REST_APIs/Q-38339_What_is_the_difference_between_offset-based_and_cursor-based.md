---
type: "practice"
difficulty: "Deep"
---

# What is the difference between offset-based and cursor-based pagination? When would you prefer cursor-based?

**Difficulty:** Deep · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Consistent Pagination](<../../../../07_Distributed_Systems_and_Design/APIs/REST_APIs/Consistent_Pagination.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

#### Overview

Pagination is a crucial aspect of building APIs, especially when dealing with large datasets that need to be fetched in smaller chunks. The two primary methods of implementing pagination are offset-based and cursor-based paging. Offset-based pagination relies on offsets (starting points) and limits to fetch records, while cursor-based pagination uses unique identifiers or tokens to navigate through a dataset more efficiently. In this answer, we'll delve into the differences between these approaches and discuss when you would prefer cursor-based pagination.

#### Triaging the Question

The core of this question revolves around understanding the mechanisms used in different types of pagination and determining under what conditions one approach is preferable over another. The interviewer might be looking for insights on scalability, performance, edge cases, and the overall architecture implications of each method.

Key Assumptions:
We are considering a REST API context where data retrieval needs to handle large datasets efficiently.
Both offset-based and cursor-based pagination are viable options in many applications but have distinct advantages and disadvantages.

#### First-Principles Solution
Offset-Based Pagination

Mechanism: Offset-based pagination works by specifying the starting point (offset) and a limit to fetch a subset of records from a database. For example, if you want to retrieve 10 items starting from the 20th item, you would set offset = 20 and limit = 10.

#### Example
Request:

HTTP
GET /items?offset=20&limit=10

Advantages:

Simplicity: Easier to implement as it relies on basic SQL queries.
Performance: Can be efficient if the database is well-indexed and caching mechanisms are in place.

Disadvantages:

Inefficient for large datasets: Fetching a subset of records can lead to additional I/O operations and increased latency, especially with large offsets.
Unreliable cursor management: If items are updated or deleted between requests, the offset-based pagination may return incorrect data.
Cursor-Based Pagination

Mechanism: Cursor-based pagination uses unique identifiers (cursors) to navigate through a dataset. A cursor typically points to the last item retrieved in the previous request and is used to fetch subsequent records.

#### Example
Request:

HTTP
GET /items?cursor=last_seen_cursor_value

Advantages:

Efficient for large datasets: By using unique identifiers, it avoids the need to traverse through unnecessary data.

#### Cons
istent results: Even if items are updated or deleted, cursor-based pagination ensures consistent retrieval of subsequent records.

Disadvantages:

More complex implementation: Requires additional logic to generate and manage cursors.
Potential overhead: Managing state across requests can introduce some complexity in the backend architecture.
When to Prefer Cursor-Based Pagination

Cursor-based pagination is generally preferred when:

Handling Large Datasets: When dealing with large datasets, cursor-based pagination can be more efficient as it avoids unnecessary I/O operations and reduces latency.
Maintaining Consistency: If your dataset frequently changes (e.g., due to updates or deletions), cursor-based pagination ensures that subsequent requests return consistent results.
Scalability: For highly scalable applications where performance is critical, cursor-based pagination can be a better choice as it scales more gracefully with increasing data volume.

#### Closing Summary

In summary, offset-based and cursor-based pagination are two distinct approaches to handling large datasets in REST APIs. While offset-based pagination is simpler to implement, it may not perform well or return consistent results in all scenarios, particularly when dealing with frequent updates. Cursor-based pagination, on the other hand, offers better performance and consistency but requires more complex implementation.

#### Common Mistakes/Follow-Ups
Mistake: Relying too heavily on offset-based pagination for large datasets without considering its limitations.

#### Follow-Up
Discussing additional techniques like server-side cursors or combining both methods (e.g., using a combination of offset and cursor).

**Comparison:**

```
Pagination Type	Mechanism	Advantages	Disadvantages
Offset-Based	Specified offset + limit	Simplicity, performance with well-indexed databases	Inefficient for large datasets, unreliable cursor management
Cursor-Based	Unique identifier	Efficient for large datasets, consistent results	More complex implementation, potential overhead
```

**Code:**

```python
# Example of offset-based pagination in Python (simplified)
def get_items(offset, limit):
    # Simulate database query
    items = db.query("SELECT * FROM items LIMIT %d OFFSET %d" % (limit, offset))
    return items

# Example of cursor-based pagination in Python (simplified)
def fetch_next_items(cursor):
    next_cursor = None  # Placeholder for actual logic to generate next cursor
    items = db.query("SELECT * FROM items WHERE id > %s ORDER BY id LIMIT 10" % cursor)
    if items:
        next_cursor = items[-1].id
    return (items, next_cursor)
```

**Example input / output:**

```text
Example input:
- `GET /items?offset=20&limit=10`

Example output:
- Response containing the next 10 items from offset 20.

Example input (for cursor-based):
- `GET /items?cursor=last_seen_cursor_value`

Example output (for cursor-based):
- Response containing the next 10 items and a new cursor value.
```

**Data:**

```json
{
  "offset": 20,
  "limit": 10,
  "cursor": "last_seen_cursor_value"
}
```

**Flow:**

1. Yes
2. No
3. Fetch Items
4. Database Query
5. Is Next Item?
6. Return Items

