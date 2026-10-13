---
type: "practice"
difficulty: "Deep"
---

# Design the structure behind 'resume a paginated export from where it failed' for a job that reads billions of rows; why is a keyset cursor over a stable sort key safer than a page number and what breaks under concurrent inserts?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Exactly Once Dedup Structures](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Exactly_Once_Dedup_Structures.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To design an efficient structure for resuming a paginated export from where it failed, especially in handling billions of rows, we need to focus on avoiding issues like race conditions that can arise with concurrent insertions and ensuring that our resume mechanism is robust against data changes. A keyset cursor over a stable sort key is generally safer than using a page number due to its ability to handle dynamic data well.

#### Triage
What the Question Is Really Asking

The question is asking us to design a system for resuming a paginated export operation, specifically in the context of billions of rows. The core issue here is how to maintain consistency and avoid race conditions when dealing with concurrent inserts. We need to consider why using a page number might lead to problems under such circumstances.

#### First-Principles Solution

To achieve this, we can use a keyset cursor approach combined with a stable sort key. Here’s the step-by-step reasoning:

Understanding Keyset Cursor: A keyset cursor uses the current value of one or more columns to determine where to continue a query from a previous execution.
Stable Sort Key: By ensuring that our sort key is stable, we can reliably use it as a starting point for resuming the export even if new rows are inserted in between.
Handling Concurrent Inserts: To handle concurrent inserts safely, we need to ensure that any row that has been inserted after the last exported row will not interfere with our resume process.
Why Keyset Cursor Over Stable Sort Key is Safer

Using a keyset cursor over a stable sort key is safer because:

#### Cons
istency Across Concurrency: Each time you resume from where you left off, the keyset ensures that you are always starting from an unambiguous point. If new rows are inserted between your last export and resume, they won’t affect the consistency of your resume operation.
Avoiding Race Conditions: A page number can lead to race conditions if multiple processes try to resume at the same time, especially when new data is inserted.
Why Page Number Fails

A page number approach fails under concurrent inserts because:

Race Condition: If two or more processes attempt to resume from a specific page simultaneously and rows are inserted between their last export and resume point, they might overwrite each other’s data.
Inconsistent Data: The page number mechanism assumes that the data on the page remains unchanged until the next page is requested. This assumption can be violated when new rows are inserted.
Detailed Design

Database Schema:

SQL
CREATE TABLE job_data (
    id SERIAL PRIMARY KEY,
    content TEXT NOT NULL,
    sort_key INT NOT NULL -- The stable sort key, e.g., time of insertion or ID
);

Resume Mechanism:

Store the last sort_key value from where the export was last paused.
When resuming, fetch rows where sort_key > last_sort_key.

Concurrency Handling:

Use a transaction to ensure that the resume operation is atomic and consistent.

#### Example
Resume Mechanism
SQL
-- Assume we need to resume from the last sort key 1024
SELECT * FROM job_data WHERE sort_key > 1024 ORDER BY sort_key LIMIT 50;

#### Closing Summary
+ Common Mistakes/Follow-Ups

In summary, using a keyset cursor with a stable sort key ensures that your resume operation is robust against concurrent inserts and dynamic data changes. This approach helps in maintaining the integrity of the export process.

#### Common mistakes
include:

Using page numbers without understanding their limitations.
Ignoring race conditions when dealing with concurrent operations.
Not ensuring that the sort key used for resuming remains stable across insertions.

#### Follow-ups
could include discussing additional optimizations, such as partitioning large tables or using advanced database features to handle concurrency more efficiently.

**Comparison:**

| Column  | Type      |
|---------|-----------|
| id      | SERIAL    |
| content | TEXT NOT NULL |
| sort_key | INT NOT NULL |

**Code:**

```sql
CREATE TABLE job_data (
    id SERIAL PRIMARY KEY,
    content TEXT NOT NULL,
    sort_key INT NOT NULL -- The stable sort key, e.g., time of insertion or ID
);
```

