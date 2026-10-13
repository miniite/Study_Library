---
type: "practice"
difficulty: "Intermediate"
---

# Your infinite-scroll feed uses OFFSET and users on page 500 see slow loads. Walk through migrating to keyset pagination while the frontend still needs a 'jump to page' affordance.

**Difficulty:** Intermediate · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Offset Pagination Pitfalls](<../../../../01_Databases_and_SQL/Query_Performance/Indexing/Offset_Pagination_Pitfalls.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

#### Overview

When dealing with infinite-scroll feeds that experience performance degradation as users scroll further, a common issue is the use of OFFSET in SQL queries. While OFFSET works for fetching records starting from a certain point, it can be inefficient when you need to jump to a specific page or handle large result sets. Migrating to keyset pagination addresses these issues by using a unique identifier (key) from the previous record as a starting point. However, we also need to maintain a user experience that allows them to jump to any page quickly.

#### Triaging the Question

The core of this question is migrating from OFFSET-based pagination to keyset pagination while ensuring that users can still navigate directly to any page. This involves understanding how OFFSET works and its limitations, comprehending the concept of keyset pagination, and adapting both backend and frontend logic accordingly.

#### Assumptions
The current implementation uses OFFSET for infinite-scroll.
Users on page 500 are experiencing slow loads due to the inefficiency of OFFSET.
We need a way to provide users with a "jump to page" functionality while switching to keyset pagination.

#### First-Principles Solution
Understanding OFFSET

The OFFSET clause in SQL is used for fetching records starting from a specific point, often combined with a LIMIT. It can be slow when you have a large number of records and need to skip many rows. For example:

SQL
SELECT * FROM items LIMIT 10 OFFSET 4990;

This query fetches the first 10 records starting from record 5000.

Keyset Pagination

Keyset pagination works by using a unique identifier (e.g., id, created_at, etc.) of the last seen record as a starting point for fetching subsequent records. For example, if you are on page 499 and the last item's ID is 5003, your next query would be:

SQL
SELECT * FROM items WHERE id > 5003 LIMIT 10;

This approach avoids having to skip large numbers of rows.

Migrating from OFFSET to Keyset Pagination

Frontend Changes:

Modify the frontend to send the key (e.g., ID) along with the request for the next set of items.
Implement a "jump to page" feature that allows users to input a specific page number and fetch records from there.

Backend Changes:

Adjust the SQL query logic to use id > last_seen_id instead of OFFSET.
Ensure the backend can handle both pagination requests (next set) and jump-to-page requests.
Implement caching mechanisms to improve performance, especially for frequently accessed records.

Maintaining User Experience:

Provide clear navigation controls in the UI.
Cache keyset IDs to avoid unnecessary database hits.
Optimize the database schema and indexes to ensure fast retrieval of keyset values.

#### Example
Implementation

Here is a simplified example using Node.js and SQL:

JAVASCRIPT
// Backend: Fetching Next Set of Items with Keyset Pagination

#### cons
t fetchNextSet = async (lastSeenId) => {

#### cons
t items = await db.query(`SELECT * FROM items WHERE id > ? LIMIT 10`, [lastSeenId]);
  return items;
};

// Backend: Jump to Page Functionality

#### cons
t jumpToPage = async (page, sortByField, sortDirection) => {

#### cons
t offset = (page - 1) * 10; // Calculate the offset for this page

#### cons
t items = await db.query(`SELECT * FROM items ORDER BY ${sortByField} ${sortDirection} LIMIT 10 OFFSET ?`, [offset]);
  return items;
};

// Frontend: Example Query Parameters

#### cons
t getNextSetParams = (lastSeenId) => {
  return { id: lastSeenId };
};

#### cons
t getJumpToPageParams = (page, sortByField, sortDirection) => {
  return { page, sortByField, sortDirection };
};

#### Closing Summary
and Common Mistakes

By migrating to keyset pagination, we can significantly improve the performance of our infinite-scroll feed. Key considerations include ensuring that the database schema supports efficient retrieval of keyset values, optimizing queries, and providing a seamless user experience with both next-set and jump-to-page functionalities.

#### Common Mistakes
Forgetting to implement caching mechanisms.
Not properly handling edge cases like reaching the end of the dataset.
Misunderstanding the implications of different sorting strategies on performance.
Final Thoughts

This approach ensures that we can scale our application efficiently while maintaining user experience. It is crucial to test thoroughly and consider all possible scenarios, especially when dealing with large datasets.

**Comparison:**

| Page Number | Items Fetched |
|-------------|---------------|
| 50          | 10            |
| 60          | 10            |
| 70          | 10            |

**Code:**

```javascript
const fetchNextSet = async (lastSeenId) => {
  const items = await db.query(`SELECT * FROM items WHERE id > ? LIMIT 10`, [lastSeenId]);
  return items;
};

const jumpToPage = async (page, sortByField, sortDirection) => {
  const offset = (page - 1) * 10; // Calculate the offset for this page
  const items = await db.query(`SELECT * FROM items ORDER BY ${sortByField} ${sortDirection} LIMIT 10 OFFSET ?`, [offset]);
  return items;
};
```

**Example input / output:**

```text
Input for fetchNextSet with lastSeenId = 5003:
- Response: [ { id: 5004, name: 'Item D' }, ... ]

Input for jumpToPage with page=100, sortByField='created_at', sortDirection='DESC':
- Response: [ { id: 9999, created_at: '2023-01-01' }, ... ]
```

**Data:**

```json
{
  "items": [
    { "id": 1, "name": "Item A" },
    { "id": 2, "name": "Item B" },
    { "id": 3, "name": "Item C" }
  ]
}
```

**Flow:**

1. Yes
2. No
3. User Loads Page
4. Is Last Seen ID Provided?
5. Fetch Next Set of Items
6. Jump To Page Request
7. Fetch Items for Jumped Page
8. Display Items to User
9. End

