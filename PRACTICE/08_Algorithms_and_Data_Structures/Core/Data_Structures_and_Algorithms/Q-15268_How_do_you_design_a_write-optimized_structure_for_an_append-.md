---
type: "practice"
difficulty: "Deep"
---

# How do you design a write-optimized structure for an append-only audit log that still supports fast 'events between time A and B for entity X' queries?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Stream Join Data Structures](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Stream_Join_Data_Structures.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To design a write-optimized structure for an append-only audit log while supporting fast "events between time A and B for entity X" queries, we need to balance the trade-offs between writing performance and query efficiency. The key is to leverage a combination of a simple file-based storage mechanism with an inverted index that allows quick range queries.

#### Triage
Understanding the Problem

The core challenge here involves two main aspects:

Write Performance: Since the log is append-only, we want to minimize the overhead associated with each write operation.
Query Efficiency: The system must be able to quickly retrieve events for a specific entity within a given time range.

#### Assumptions
and Constraints
The audit log records are timestamped.
Entities can be uniquely identified by an identifier (e.g., user ID, session ID).
The query is frequently between two timestamps and involves filtering on the entity ID.
We need to support both write and read operations in a high-performance manner.
Step-by-Step Solution
1. Append-Only File Storage

For write optimization, we can use a simple file-based storage mechanism where each new event is appended to the end of a log file. This approach minimizes the overhead associated with each write operation since no data needs to be moved or reorganized.

Yes
No
Append Event
Is File Open?
Write to File
Open New File, Write to NewFile
Close File
2. Inverted Index for Fast Queries

To support fast range queries, we can create an inverted index that maps each entity ID to a list of timestamps where events occurred for that entity.

Inverted Index Structure
Entity
Timestamps
Timestamp1
File Offset

The Entity node is linked to multiple Timestamp nodes, each pointing to the offset in the log file where the event occurred.

3. Query Execution

When a query requests events for a specific entity within a time range, we:

Use the inverted index to get all timestamps associated with the entity.
Filter these timestamps to find those that fall within the specified range.
Use the offsets from the filtered timestamps to read the corresponding records from the log files.
Full Solution Overview
Append Events: Each event is appended to a log file, ensuring minimal write overhead.
Maintain Inverted Index: For each entity ID, keep track of all events with their respective timestamps and file offsets.
Range Query Execution:
Retrieve the relevant timestamps using the inverted index.
Filter by the time range.
Read the corresponding records from the log files.

#### Closing Summary
+ Common Mistakes/Follow-Ups

The proposed solution effectively balances write performance with query efficiency by leveraging a simple append-only storage mechanism and an efficient inverted index. However, there are potential pitfalls to consider:

Index Size: The inverted index can grow significantly if entities generate many events.
Memory Usage: Storing timestamps in memory might be inefficient for very high volumes of data.
Concurrency: Handling concurrent writes and reads efficiently without introducing bottlenecks is crucial.

#### Follow-ups
could include discussing optimizations such as using a more compact representation for the log entries, implementing caching strategies, or exploring distributed storage solutions to handle larger scales.

