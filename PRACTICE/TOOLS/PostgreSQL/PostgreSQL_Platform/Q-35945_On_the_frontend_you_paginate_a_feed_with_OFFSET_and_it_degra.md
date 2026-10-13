---
type: "practice"
difficulty: "Intermediate"
---

# On the frontend you paginate a feed with OFFSET and it degrades as users scroll deep; how do you migrate to keyset pagination while keeping stable sort and handling ties?

**Difficulty:** Intermediate · **Levels:** SDE-1/2, SDE-2/3 · [↑ Materialized Views vs Incremental Aggregates](<../../../../TOOLS/PostgreSQL/PostgreSQL_Platform/Materialized_Views_vs_Incremental_Aggregates.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address this issue, we need to transition from using OFFSET in our queries to implementing keyset pagination (also known as token-based pagination). This approach will ensure more efficient data fetching and stable sorting, even when users scroll deep into the feed.

#### Triage
of the Question

The core question here is how to migrate away from OFFSET pagination while preserving stable sorting and handling ties. The interviewer is probing your understanding of SQL query optimization, database indexing, and pagination strategies. We will assume that the current implementation uses OFFSET with a fixed size per page.

#### First-Principles Solution

Understand Current Implementation: The current system fetches data in chunks using OFFSET. As users scroll deeper, more offsets are required, leading to slower performance.

Identify Key Issues:

Inconsistent sorting: If the underlying data changes between queries, the order of items can become unstable.
Handling ties: When multiple records have the same key or value, they might not be returned in a stable order.

Keyset Pagination Approach: We will use an additional column (such as created_at or id) to fetch subsequent pages based on a unique identifier rather than relying on offsets.

Maintain Stable Sorting:

Ensure the chosen key is sorted consistently with the desired display order.
Use appropriate indexing to optimize query performance.
Step-by-Step Solution
1. Schema and Indexing

Assume our table feed_items has columns: id, user_id, content, created_at.

SQL
CREATE INDEX idx_feed_items_created_at ON feed_items(created_at DESC);

This index ensures that when we fetch subsequent pages, the data is sorted by created_at in descending order.

2. Implement Keyset Pagination

We need to pass a key (e.g., id) from the previous page to fetch the next set of items.

SQL
-- Example SQL query for fetching the next page using keyset pagination
SELECT * FROM feed_items WHERE id < $key ORDER BY created_at DESC LIMIT 10;

Here, $key is the last item's ID from the previous page. This ensures that we always fetch the correct set of items.

3. Handling Ties and Stable Sorting

To handle ties, we can add a tie-breaking column (e.g., id). For example:

SQL
SELECT * FROM feed_items WHERE id < $key AND created_at = $created_at ORDER BY id ASC LIMIT 10;

This query ensures that even if multiple items have the same created_at value, they are still ordered by their id.

4. Migrate Frontend Implementation

In the frontend, maintain state of the last item's ID and pass it to the backend with each request.

JAVASCRIPT
// Example frontend code
let lastItemId = // retrieved from previous page or initial state
fetch('/api/feed-items', {
  method: 'GET',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({ key: lastItemId })
})
.then(response => response.json())
.then(data => {

#### cons
t newItems = data.items;
  // Update UI with new items and set the ID of the last item for the next request
  lastItemId = newItems[newItems.length - 1].id;
});

#### Closing Summary

By migrating from OFFSET to keyset pagination, we can ensure more efficient and stable sorting. Key considerations include proper indexing, handling ties, and ensuring consistent state between frontend and backend.

#### Common Mistakes/Follow-Ups
Not Handling Ties: Failing to account for multiple items with the same key value can lead to inconsistent display.
Ignoring Indexing: Poorly chosen keys or missing indexes can degrade performance significantly.
Improper State Management: Losing track of the last item's ID in the frontend can result in incorrect pagination.

**Comparison:**

| Column | Data Type | Description |
|--------|-----------|-------------|
| id     | INTEGER   | Unique identifier for each item |
| user_id| INTEGER   | ID of the user who created the item |
| content| TEXT      | Content of the feed item |
| created_at| TIMESTAMP | Timestamp when the item was created |

**Code:**

```sql
-- Example SQL index creation
CREATE INDEX idx_feed_items_created_at ON feed_items(created_at DESC);

-- Example SQL query for fetching next page with keyset pagination
SELECT * FROM feed_items WHERE id < $key ORDER BY created_at DESC LIMIT 10;

-- Example SQL query handling ties
SELECT * FROM feed_items WHERE id < $key AND created_at = $created_at ORDER BY id ASC LIMIT 10;
```

**Example input / output:**

```text
// Initial request to get the first page of feed items
{
  "key": null
}

// Response with pagination token (last item's ID)
{
  "items": [
    {"id": 34, "user_id": 123, "content": "First item", "created_at": "2023-10-01T12:00:00Z"},
    // ... more items
  ],
  "nextKey": 35
}
```

**Data:**

```json
{
  "items": [
    {"id": 34, "user_id": 123, "content": "First item", "created_at": "2023-10-01T12:00:00Z"},
    {"id": 35, "user_id": 456, "content": "Second item", "created_at": "2023-10-01T12:00:00Z"},
    // ... more items
  ]
}
```

**Flow:**

1. User scrolls down
2. Maintain state of last item's ID
3. Pass key to backend API
4. Fetch new items using keysetpagination
5. Update UI with new items andset nextKey for the frontend

