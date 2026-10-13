---
type: "practice"
difficulty: "Deep"
---

# You're designing pagination for a feed where items are inserted constantly; explain why offset pagination breaks and how a cursor built on a sorted key avoids skipped or duplicated rows.

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Skip Lists](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Skip_Lists.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When designing a pagination system, one of the most common scenarios involves handling a large number of items that are frequently being inserted into a database. In this context, we often encounter two main approaches: offset pagination and cursor-based pagination. The key difference between these methods is how they handle the state of the data over time, with offset pagination tending to break down in certain situations, while cursor-based pagination remains robust.

Offset pagination works by fetching a fixed number of items starting from an offset index within the result set. However, this method can fail when new items are inserted between the initial fetch and subsequent requests. This leads to either skipping or duplicating rows in the feed, which is unacceptable for most user experiences.

In contrast, cursor-based pagination uses a sorted key (such as created_at or id) to identify the position of the next set of results. By leveraging this key, you can ensure that each request fetches items strictly after the last known item from previous requests, avoiding both skipped and duplicated rows.

#### Triaging the Question

The core of the question revolves around understanding why offset pagination is problematic in dynamic data environments and how cursor-based pagination offers a more robust solution. The interviewer is likely probing your ability to explain fundamental concepts about data handling, consistency, and efficient query strategies.

#### Assumptions
Items are being inserted into the database at an unpredictable rate.
The goal is to provide a consistent and uninterrupted feed of items to users.
Both offset pagination and cursor-based pagination are viable methods for handling large datasets.

#### First Principles Solution

Understanding Offset Pagination:

In an offset-based system, each request fetches a fixed number of items starting from an arbitrary point determined by the offset value.
If new items are inserted between requests, they may shift the offset range, causing rows to be skipped or duplicated.

#### Example Scenario
with Offset Pagination:

Suppose you start fetching 10 items at offset 0. Then, before the next request (still at offset 0), another 5 items are inserted.
The next request might return only 5 items instead of 10 because some items have been inserted in between.

Understanding Cursor-Based Pagination:

A cursor-based system maintains a pointer to the last item seen during the previous fetch.
By using this pointer (cursor) as a starting point for each new request, you ensure that all subsequent requests are strictly after the last known item.

#### Example Scenario
with Cursor-Based Pagination:

Assume you start fetching 10 items and get an id of the last item (cursor = id_of_last_item).
In your next request, you use this cursor to fetch the next batch, ensuring that no rows are skipped or duplicated.

#### Closing Summary
and Common Mistakes

In summary, offset pagination is inherently prone to issues in dynamic environments due to its reliance on fixed offsets. Cursor-based pagination, however, provides a more reliable mechanism by leveraging sorted keys to maintain consistency across requests. Key mistakes often arise from not fully considering the implications of data mutations between requests.

Common Follow-Ups:
How would you handle cases where multiple cursors are needed?
What performance considerations should be taken into account with cursor-based pagination?

**Comparison:**

```
Item ID	Created At
1	2023-10-01T09:00:00Z
2	2023-10-01T09:05:00Z
3	2023-10-01T09:10:00Z
```

**Code:**

```python
# Example Python code for offset pagination and cursor-based pagination

import sqlite3

# Offset Pagination (Broken in dynamic environments)
def fetch_items_offset(conn, offset=0, limit=10):
    cursor = conn.cursor()
    cursor.execute(f"SELECT * FROM items LIMIT {limit} OFFSET {offset}")
    return cursor.fetchall()

# Cursor-Based Pagination
def fetch_items_cursor(conn, last_item_id, limit=10):
    cursor = conn.cursor()
    cursor.execute(f"SELECT * FROM items WHERE id > ? ORDER BY id ASC LIMIT ?", (last_item_id, limit))
    return cursor.fetchall()
```

**Example input / output:**

```text
# Example input/output for cursor-based pagination

Input:
- Initial fetch: `fetch_items_cursor(conn, last_item_id=None)`

Output:
- Items: [{"id": 1}, {"id": 2}]

Next request with a cursor (last_item_id=2):
- Input: `fetch_items_cursor(conn, last_item_id=2)`
- Output: [{"id": 3}]
```

**Data:**

```json
{
  "items": [
    {"id": 1, "created_at": "2023-10-01T09:00:00Z"},
    {"id": 2, "created_at": "2023-10-01T09:05:00Z"},
    {"id": 3, "created_at": "2023-10-01T09:10:00Z"}
  ]
}
```

**Flow:**

1. Yes
2. Initial Fetch
3. Items: [1, 2]
4. New Item?
5. Add New Item
6. Cursor: id=2
7. Fetch Next Items (id > 2)
8. Items: [3]

