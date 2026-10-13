---
type: "practice"
difficulty: "Intermediate"
---

# How would you design indexes for an inbox listing that needs 'most recent thread', 'unread only', and 'search by subject' at a few million rows?

**Difficulty:** Intermediate · **Levels:** Architect/Principal, SDE-1, SDE-1/2, SDE-2/3 · [↑ Indexing Read State at Scale](<../../../../01_Databases_and_SQL/Query_Performance/Indexing/Index_Tuning_in_Practice/Indexing_Read_State_at_Scale.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To design effective indexes for an inbox listing with requirements such as 'most recent thread', 'unread only', and 'search by subject', we need to balance query performance against index overhead. The key is understanding how each type of query operates on the data.

#### Triage

The question asks us to design indexing strategies for three common queries: retrieving the most recent threads, filtering unread messages, and searching by message subject. We are dealing with a large dataset (a few million rows), so we need to ensure that these operations are efficient both in terms of speed and resource utilization.

#### Assumptions
The inbox contains a messages table.
Each message has fields such as id, thread_id, sender_id, receiver_id, subject, timestamp, read_status.
We aim to optimize these operations without creating too many indexes, which could lead to increased storage and maintenance costs.
Step-by-Step Solution:

Most Recent Thread:

The most recent thread query likely involves selecting messages from the latest threads.
The thread_id along with the timestamp would be useful here. An index on (thread_id, timestamp DESC) can help quickly fetch the latest messages in each thread.

Unread Only:

To efficiently filter unread messages, we need an index on read_status. This allows us to directly query for messages that are not read.
The read_status field could be indexed as (read_status = 0) if it uses a boolean value (unread).

Search by Subject:

For searching by subject, we need an index on the subject field. This can help in quickly filtering messages based on their content.
A B-tree or hash index could be suitable for this purpose.
Full Solution:

Based on our analysis:

Thread and Timestamp Index:

SQL
CREATE INDEX idx_thread_timestamp ON messages (thread_id, timestamp DESC);

Unread Filter:

SQL
CREATE INDEX idx_read_status ON messages (read_status);

Subject Search:

SQL
CREATE INDEX idx_subject ON messages (subject);

#### Closing Summary
and Common Mistakes:

We have designed three indexes that address the primary query requirements. However, it is important to regularly monitor index usage and performance to ensure they remain effective as data grows or changes.

#### Common Mistakes

Over-indexing: Creating too many indexes can degrade write performance and increase storage costs.
Inefficient Indexes: Not selecting the right columns or using non-optimal types of indexes (e.g., covering indexes) can limit their effectiveness.
Ignoring Query Plans: Not analyzing how the database executes queries can lead to suboptimal performance.
Final Check:

This solution covers all the required query operations and balances performance with cost efficiency. No additional fields needed, so no artifacts are provided beyond this explanation.

**Code:**

```sql
-- Thread and Timestamp Index
CREATE INDEX idx_thread_timestamp ON messages (thread_id, timestamp DESC);

-- Unread Filter
CREATE INDEX idx_read_status ON messages (read_status);

-- Subject Search
CREATE INDEX idx_subject ON messages (subject);
```

