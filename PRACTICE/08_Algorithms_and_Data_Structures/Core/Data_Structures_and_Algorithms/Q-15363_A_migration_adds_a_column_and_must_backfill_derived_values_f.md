---
type: "practice"
difficulty: "Intermediate"
---

# A migration adds a column and must backfill derived values for 300M rows without long locks; how do you chunk by primary-key ranges and reason about the ordering so retries are idempotent?

**Difficulty:** Intermediate · **Levels:** SDE-1/2, SDE-2/3 · [↑ Stream Join Data Structures](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Stream_Join_Data_Structures.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address this migration problem, we need to design a strategy that allows us to efficiently update 300 million rows in a database without causing long locks or race conditions during the backfilling process. The key idea is to chunk the primary keys into smaller ranges and handle them in a way that ensures idempotency upon retries.

Triaging the Problem

The question essentially asks how to partition a large set of records (300 million rows) such that we can update them in batches without causing long-term locks. Additionally, it requires ensuring that if any batch fails and needs to be retried, the operations are idempotent—meaning they produce the same result even if executed multiple times.

#### Assumptions
We have a primary key (PK) that uniquely identifies each row.
The migration involves adding or updating a column with derived values.
Locking mechanisms are in place but need to be optimized.
The system has some means of tracking which rows have been processed and which haven’t.

#### First-Principles Solution
Partition the Data: Break down the 300 million rows into smaller, manageable chunks based on their primary keys. For example, we could partition by range (e.g., every 10,000 PK values).
Process in Batch: Process each batch independently to avoid long locks.
Idempotent Processing: Ensure that even if a batch fails and needs to be retried, the same operations produce the same result.
Step-by-Step Solution

Identify Ranges:

Calculate the total number of ranges needed for 300 million rows, given each range contains 10,000 rows.
This gives us ( \frac{300,000,000}{10,000} = 30,000 ) ranges.

Assign Ranges to Workers:

Assign these ranges to worker threads or processes for concurrent processing. Ensure each range is processed by a single thread to avoid conflicts.

Backfilling Strategy:

For each range, fetch the rows and calculate the derived values.
Update the database in small batches (e.g., 1000 at a time) to reduce lock times.
Use transactions where necessary to ensure atomicity during updates.

Idempotent Handling:

Track which rows have been processed using a separate table or flag in the row itself.
If a batch fails, re-check this tracking before processing the same range again.

Retry Mechanism:

Implement exponential backoff for retries to avoid overwhelming the system with repeated failures.
Log errors and failed batches to analyze issues post-migration if needed.

#### Closing Summary

By chunking the data into smaller ranges, we can process large datasets efficiently without causing long locks or race conditions. The idempotent design ensures that any failed batch can be retried safely without affecting other processed rows.

#### Common Mistakes
and Follow-Ups
Not Handling Idempotency Properly: Forgetting to track processed rows can lead to duplicated work.
Insufficient Error Handling: Not implementing robust error handling or exponential backoff can cause unnecessary retries or system overload.
Poor Partitioning Strategy: Poor range partitioning (e.g., too many small ranges) can increase the number of worker threads and overhead.

To address these, consider refining your partition strategy to balance load distribution and ensure efficient processing. Additionally, continuously monitor and adjust the retry mechanism based on real-world performance data.

