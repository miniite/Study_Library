---
type: "practice"
difficulty: "Deep"
---

# Design the read path for a chat app's message history: load last 50 by conversation, scroll up for older, jump to a search hit. Specify the indexes and how each access pattern uses them.

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Indexing Read State at Scale](<../../../../01_Databases_and_SQL/Query_Performance/Indexing/Index_Tuning_in_Practice/Indexing_Read_State_at_Scale.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To design the read path for a chat app’s message history that loads the last 50 messages by conversation, scrolls up for older messages, and jumps to search hits, we need to carefully consider indexing strategies and query optimization techniques. We will focus on efficient retrieval of recent messages and handling scroll operations.

TL;DR

We'll implement an index on (conversation_id, message_timestamp) in descending order to efficiently fetch the last 50 messages by conversation. For scrolling up, we'll use a binary search approach with the same index. For jump-to-search hits, a full-text search index will be employed.

#### Triage
and Assumptions

The interviewer is probing our understanding of:

Index selection and usage.
Query optimization techniques for different access patterns.

#### Trade-offs
between query performance and storage requirements.

We assume:

The database is PostgreSQL or similar RDBMS supporting B-tree indexes, full-text search, and window functions.
Messages are timestamped and stored in a messages table with columns: id, conversation_id, sender_id, content, timestamp.
Step-by-Step Solution
1. Index Design

To efficiently fetch the last 50 messages by conversation, we will create an index on (conversation_id, message_timestamp DESC). This allows for fast retrieval of recent messages within each conversation.

SQL
CREATE INDEX idx_messages_by_conversation_timestamp ON messages (conversation_id, timestamp DESC);
2. Loading Last 50 Messages by Conversation

For the initial load or when scrolling down to the last 50 messages in a specific conversation, we can use:

SQL
SELECT * FROM messages WHERE conversation_id = $1 ORDER BY timestamp DESC LIMIT 50;

This query leverages our index on (conversation_id, timestamp DESC) and is efficient due to the B-tree structure.

3. Scrolling Up for Older Messages

To scroll up and fetch older messages in a conversation, we can use a binary search approach with the same index. First, find the message ID of the current cursor position:

SQL
SELECT id FROM messages WHERE conversation_id = $1 AND timestamp < $2 ORDER BY timestamp DESC LIMIT 1;

Then, query for messages older than this cursor:

SQL
SELECT * FROM messages WHERE conversation_id = $1 AND timestamp < $3 ORDER BY timestamp DESC LIMIT 50;
4. Jumping to a Search Hit

For jumping to a search hit, we can use PostgreSQL’s full-text search capabilities. Assuming the content column is indexed for full-text search:

SQL
SELECT * FROM messages WHERE conversation_id = $1 AND to_tsvector(content) @@ to_tsquery($2);

This query uses to_tsvector and @@ operator for full-text search, which can be optimized by creating a GIN index on the content column.

#### Closing Summary
+ Common Mistakes/Follow-Ups

In summary, we have designed an efficient read path with the following key points:

Index (conversation_id, timestamp DESC) for fetching last 50 messages.
Binary search approach for scrolling up using the same index.
Full-text search for jumping to a specific message.

#### Common mistakes
include:

Not creating appropriate indexes, leading to inefficient queries.
Overlooking query optimization techniques like window functions or full-text search.
Ignoring trade-offs between query performance and storage requirements.

#### Follow-up
questions might explore:

How would you handle concurrent updates to messages?
What are the implications of message deletion on our indexing strategy?

**Comparison:**

```
Messages Table
conversation_id INT NOT NULL
sender_id INT NOT NULL
content TEXT NOT NULL
timestamp TIMESTAMPTZDEFAULT NOW()
Idx Messages By ConversationTimestamp (IDX)
G
H
I
```

**Code:**

```sql
-- Index creation
CREATE INDEX idx_messages_by_conversation_timestamp ON messages (conversation_id, timestamp DESC);

-- Query for last 50 messages in a conversation
SELECT * FROM messages WHERE conversation_id = $1 ORDER BY timestamp DESC LIMIT 50;

-- Binary search approach for scrolling up
SELECT id FROM messages WHERE conversation_id = $1 AND timestamp < $2 ORDER BY timestamp DESC LIMIT 1;

-- Full-text search for jumping to a specific message
SELECT * FROM messages WHERE conversation_id = $1 AND to_tsvector(content) @@ to_tsquery($2);
```

**Example input / output:**

```text
Input:
- conversation_id = 12345
- cursor_message_timestamp = "2023-09-15 16:00:00"

Output for fetching last 50 messages in a conversation:
[
    {"id": 1, "conversation_id": 12345, "sender_id": 54321, "content": "Hello!", "timestamp": "2023-09-16 17:00:00"},
    ...
]

Output for binary search to find a cursor message:
[
    {"id": 2}
]

Output for jumping to a specific search hit:
[
    {"id": 3, "conversation_id": 12345, "sender_id": 67890, "content": "Important update!", "timestamp": "2023-09-15 15:59:59"}
]
```

**Data:**

```sql
CREATE TABLE messages (
    id SERIAL PRIMARY KEY,
    conversation_id INT NOT NULL,
    sender_id INT NOT NULL,
    content TEXT NOT NULL,
    timestamp TIMESTAMPTZ DEFAULT NOW()
);

-- Indexes
CREATE INDEX idx_messages_by_conversation_timestamp ON messages (conversation_id, timestamp DESC);
CREATE INDEX idx_content_fulltext ON messages USING GIN(to_tsvector('english', content));
```

**Flow:**

1. Load Last 50 Messages
2. Use Indexidx_messages_by_conversation_timestamp
3. ORDER BY timestamp DESCLIMIT 50
4. Scroll Up to Older Messages
5. Find Cursor Message withIndex
6. Use same index for oldermessages
7. Binary Search Approach
8. JUMP to Search Hit
9. Full-Text Search usingto_tsvector and @@
10. Create idx_content_fulltext oncontent

