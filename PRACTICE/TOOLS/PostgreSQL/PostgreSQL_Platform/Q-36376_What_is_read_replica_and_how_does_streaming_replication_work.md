---
type: "practice"
difficulty: "Deep"
---

# What is read replica and how does streaming replication work in PostgreSQL?

**Difficulty:** Deep · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Replica Lag and Read Your Writes](<../../../../TOOLS/PostgreSQL/PostgreSQL_Platform/Replica_Lag_and_Read_Your_Writes.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Read replicas are an essential feature of PostgreSQL that enhance database scalability, availability, and performance by allowing multiple copies of a primary database to be maintained across different servers. Streaming replication is one of the mechanisms used to keep these replicas up-to-date with the primary database in real-time.

To understand streaming replication in PostgreSQL, let's break down its core concepts, how it works from first principles, and some common pitfalls.

**Code:**

```sql
-- Example setup for a read replica
-- Assuming we have a primary server and a standby server

CREATE REPLICATION SLOT my_slot ON source_db;
```

**Flow:**

1. Primary Server
2. New Data
3. Change Streams
4. Replication Slot
5. Standby Server
6. Apply Changes

