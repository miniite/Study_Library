---
type: "practice"
difficulty: "Deep"
---

# You need 'threads with unread messages for a user' where unread state is per-user. Would you index a per-user read-state table, denormalize a last_read cursor, or maintain unread counts? Discuss.

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Indexing Read State at Scale](<../../../../01_Databases_and_SQL/Query_Performance/Indexing/Index_Tuning_in_Practice/Indexing_Read_State_at_Scale.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the requirement of efficiently retrieving "threads with unread messages" while considering per-user unread states, we need to carefully evaluate three approaches: indexing a per-user read-state table, denormalizing a last_read cursor, and maintaining unread counts directly in the database. Each approach has its trade-offs, which will be discussed step-by-step from first principles.

First, let's clarify what each of these options entails:

Indexing a per-user read-state table: This involves creating a separate table that tracks the read status for each user on every thread.
Denormalizing a last_read cursor: This approach uses a single value (a timestamp) to track the latest message seen by a user, allowing us to infer unread messages based on this value.
Maintaining unread counts directly in the database: This method involves updating an unread_count field whenever a new message is added or a read state changes.

#### Triaging the Question

The core of this question revolves around choosing the most efficient and scalable approach for tracking per-user unread states. The interviewer is probing your understanding of indexing, denormalization, and database optimization techniques. We'll also consider factors such as query performance, storage requirements, and write latency.

#### First-Principles Solution
1. Indexing a Per-User Read-State Table

#### Pros

Direct access to the read state can lead to efficient queries.
Easier to implement and maintain.
Scalability is generally good if indexed properly.

#### Cons

Increased storage requirements due to the additional table.
Potential performance hit during writes (inserts, updates) as every change necessitates an update in both the message thread and the read-state table.
2. Denormalizing a Last_Read Cursor

#### Pros

Reduces the number of tables involved, potentially improving write performance.
Requires less storage compared to maintaining separate read-state tables for each user.
Simplifies queries as you can use a single value (last_read timestamp) to determine unread messages.

#### Cons

Inaccuracies due to race conditions when multiple users interact with the same thread simultaneously.
Query complexity increases as you need to handle edge cases like concurrent reads and writes.
May require complex logic to ensure correctness, especially during updates and deletions.
3. Maintaining Unread Counts Directly in the Database

#### Pros

Simplified queries since you only have one table to deal with.
Potentially faster read performance due to fewer joins or lookups needed.

#### Cons

Increased complexity in maintaining accurate counts, especially during concurrent updates.
Potential for race conditions and inconsistent states if not handled carefully.

#### Closing Summary
and Common Mistakes/Follow-Ups

In conclusion, the choice between these approaches depends heavily on the specific requirements of your application. If storage is a critical concern and you can tolerate some complexity in queries, denormalizing with a last_read cursor might be the best option. For a simpler implementation that still provides good performance, maintaining an unread_count directly in the database could be effective. However, if direct access to read states is frequently required or if you're dealing with large numbers of users and threads, indexing a per-user read-state table might offer better scalability.

#### Common Mistakes

Overlooking race conditions when implementing denormalized solutions.
Neglecting proper index management in the per-user read-state approach.
Underestimating the complexity involved in maintaining accurate counts when using direct unread count fields.

#### Follow-Ups

The interviewer might ask about specific scenarios where one option would be preferable over the others, or how you would implement concurrency control mechanisms to handle race conditions in denormalized solutions. They may also probe on performance metrics and cost considerations for different database technologies (e.g., in-memory databases vs. traditional SQL databases).

**Comparison:**

```
Thread ID	Unread Count	Last Message ID	Last Message Created At
101	1	2	2023-10-01T14:05:00Z
```

**Code:**

```sql
-- Example of maintaining unread counts directly in the database

CREATE TABLE messages (
    id INT PRIMARY KEY,
    thread_id INT,
    user_id INT,
    content TEXT,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE TABLE threads (
    id INT PRIMARY KEY,
    unread_count INT DEFAULT 0,
    last_message_id INT,
    last_message_created_at TIMESTAMP
);
```

**Example input / output:**

```text
-- Example query to find threads with unread messages for a specific user

SELECT
    t.id AS thread_id,
    t.unread_count - COUNT(m.id) AS unread_message_count
FROM
    threads t
LEFT JOIN
    messages m ON t.last_message_id = m.id AND m.user_id != 502
WHERE
    t.last_message_created_at > (SELECT last_read FROM user_read_state WHERE user_id = 502)
GROUP BY
    t.id;
```

**Data:**

```json
{
  "messages": [
    {
      "id": 1,
      "thread_id": 101,
      "user_id": 502,
      "content": "Hello, how are you?",
      "created_at": "2023-10-01T14:00:00Z"
    },
    {
      "id": 2,
      "thread_id": 101,
      "user_id": 502,
      "content": "I'm doing well, thanks!",
      "created_at": "2023-10-01T14:05:00Z"
    }
  ],
  "threads": [
    {
      "id": 101,
      "unread_count": 1,
      "last_message_id": 2,
      "last_message_created_at": "2023-10-01T14:05:00Z"
    }
  ]
}
```

**Flow:**

1. User Reads Thread
2. Update Last_Read Cursor
3. New Message Added
4. Increment Unread Count inDatabase
5. Query Threads with Unreads
6. Join Messages Table toDetermine Unread Count

