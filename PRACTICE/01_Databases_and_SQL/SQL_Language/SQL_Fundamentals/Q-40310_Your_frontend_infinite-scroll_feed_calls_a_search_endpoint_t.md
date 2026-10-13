---
type: "practice"
difficulty: "Intermediate"
---

# Your frontend infinite-scroll feed calls a search endpoint that gets slower as users scroll. Explain, from a full-stack view, how deep OFFSET kills the DB and how you'd redesign the API contract for keyset paging.

**Difficulty:** Intermediate · **Levels:** SDE-1/2, SDE-2/3 · [↑ Large Values and TOAST](<../../../../01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Large_Values_and_TOAST.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR: This problem arises when an infinite-scroll feed uses a large OFFSET in SQL queries to fetch data, which can significantly degrade database performance as users scroll down. We need to redesign the API contract using keyset paging, where the backend returns a cursor or offset value along with each page of results.

#### Triage
The question is probing your understanding of infinite-scrolling pagination mechanisms and how they impact both frontend and backend performance. Key assumptions are that the database has a large number of rows, the frontend fetches data in chunks, and we aim to maintain good user experience while minimizing server load.

#### Solution

Understanding Infinite Scroll with OFFSET: When using an infinite scroll, each request sends an OFFSET value to fetch more records from the database starting from that point. However, as users scroll down, the offset increases exponentially, leading to slower query performance because SQL needs to scan through a growing number of rows.

Performance Impact Analysis:

Time Complexity: The time complexity for each SQL query is O(n), where n is the number of rows between the current and next page.
Resource Utilization: As more data is requested, the database server's CPU usage and memory consumption increase.
Network Latency: Slower queries lead to increased network latency, affecting overall user experience.

Redesigning API Contract with Keyset Paging:

Database Schema: Ensure the schema supports efficient lookups. For example, use a created_at timestamp or an auto-incremented ID.
API Response Structure:
JSON
{
  "items": [
    {"id": 123, "title": "Item Title"},
    ...
  ],
  "nextPageCursor": "cursor_value_123"
}

#### Implementation
Steps: a. Initial Request: On the first request, the frontend sends null or an initial cursor. b. Backend Logic:
SQL
SELECT * FROM items WHERE id > :last_id LIMIT 10;
c. Frontend Handling: The frontend uses the returned nextPageCursor to fetch the next page of data. d. Edge Case Handling: Implement logic to handle edge cases like no more results or invalid cursors.

#### Example
Code:

JAVASCRIPT
async function fetchData(cursor) {

#### cons
t response = await fetch(`/api/items?page_cursor=${cursor}`);

#### cons
t data = await response.json();
    return { items: data.items, nextPageCursor: data.nextPageCursor };
}

#### Closing Summary
By switching to keyset paging, we ensure that each request only needs to scan a fixed number of rows, significantly improving performance and reducing the load on the database server.

#### Common Mistakes/Follow-ups

Failing to consider edge cases where no more results are available.
Not optimizing cursor-based queries for large datasets.
Ignoring potential issues with network latency due to increased request volume.

**Code:**

```javascript
// Example frontend code using keyset paging
async function fetchData(cursor) {
    const response = await fetch(`/api/items?page_cursor=${cursor}`);
    const data = await response.json();
    return { items: data.items, nextPageCursor: data.nextPageCursor };
}
```

