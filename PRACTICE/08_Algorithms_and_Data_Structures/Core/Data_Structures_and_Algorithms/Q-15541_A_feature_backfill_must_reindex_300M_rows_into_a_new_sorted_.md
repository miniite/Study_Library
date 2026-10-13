---
type: "practice"
difficulty: "Deep"
---

# A feature backfill must reindex 300M rows into a new sorted layout with zero downtime; how do you sequence build, dual-write, and cutover, and what invariant must hold so no feature values are lost?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Stream Join Data Structures](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Stream_Join_Data_Structures.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To reindex 300 million rows into a new sorted layout with zero downtime in an ML engineering context, we need to carefully sequence build, dual-write, and cutover processes while ensuring no feature values are lost. The key idea is to leverage distributed systems principles and data structures like balanced trees or heaps to maintain the invariant that all feature values are updated without disrupting service.

The question primarily probes our understanding of:

Distributed systems concepts
Data structure choices for efficient reindexing
Strategies for zero-downtime database migrations
Invariants and consistency models

We assume a distributed environment with multiple nodes, possibly using a distributed database or a message queue system like Kafka to facilitate the transition smoothly.

Let’s break this down step-by-step:

Preparation Phase:

Create a new table structure for the sorted layout.
Ensure both old and new tables are initially in sync.

Build New Layout:

Use a balanced tree data structure (e.g., AVL, Red-Black) to insert rows in sorted order.
This ensures efficient insertion and retrieval operations during the build phase.

Dual Write Phase:

Simultaneously write new feature values into both old and new tables.
Ensure that for each row being updated, it is written to both databases atomically or as part of a transaction.

Cutover Phase:

Once the build phase is complete, perform a coordinated cutover where all read operations are directed to the new table.
This transition must be seamless and ensure no downtime.
Detailed Steps

Preparation Phase

SQL
CREATE TABLE new_table AS SELECT * FROM old_table;

Build New Layout (Dual Write)

Implement a worker service that reads from the source (e.g., Kafka topic, database) and inserts into both tables.
Use distributed locks or transactions to ensure atomicity.

Cutover Phase

Coordinate with all services to start reading from new_table.
Ensure all read operations are directed to new_table post-cutover.
Invariants

To maintain the invariant that no feature values are lost, we must ensure:

All writes are mirrored.
No reads can occur during the transition period.
The final state is consistent with the new layout.

#### Common Mistakes
and Follow-Ups

Mistakes:

Forgetting to write to both tables in a transaction.
Not coordinating services for cutover.
Ignoring distributed failure modes (e.g., node failures, network partitions).

#### Follow-Ups

How would you handle partial builds or retries?
What about consistency models like eventual vs. strong consistency?
How do you manage memory and compute resources during the dual-write phase?

**Comparison:**

```
Row ID	Old Data (Feature Values)
1	{ "featureA": 0.5, "featureB": 1 }
...	...
300M	{ "featureX": 0.2, "featureY": 0.8 }
```

**Code:**

```python
# Example of a worker service for dual writing in Python using asyncio and distributed locks

import asyncio
from distributed_lock import acquire_lock, release_lock

async def process_row(row_id):
    # Simulate reading from source
    old_data = await read_source(row_id)

    # Acquire lock before performing writes
    async with acquire_lock(f'lock_{row_id}'):
        new_data = transform(old_data)  # Transform data as needed

        # Dual write
        await write_to_old_table(row_id, old_data)
        await write_to_new_table(row_id, new_data)

async def main():
    rows = get_all_rows()  # Simulate getting all rows to process
    tasks = [process_row(row) for row in rows]
    await asyncio.gather(*tasks)

# Example async functions
async def read_source(row_id):
    pass

async def write_to_old_table(row_id, data):
    pass

async def write_to_new_table(row_id, data):
    pass

asyncio.run(main())
```

**Example input / output:**

```text
Input:
- Source rows with old data

Output:
- Both new and old tables updated simultaneously
- New table ready for cutover post-build
- Zero downtime during transition
```

**Data:**

```json
{
  "rows": [
    { "row_id": 1, "old_data": { /* feature values */ } },
    { "row_id": 2, "old_data": { /* feature values */ } },
    ...
    { "row_id": 300000000, "old_data": { /* feature values */ } }
  ]
}
```

**Flow:**

1. Yes
2. No
3. Read from source
4. Is row ready?
5. Write to old and new tables
6. Await row processing
7. Update status flag
8. Cutover to new table
9. All read operations redirected

