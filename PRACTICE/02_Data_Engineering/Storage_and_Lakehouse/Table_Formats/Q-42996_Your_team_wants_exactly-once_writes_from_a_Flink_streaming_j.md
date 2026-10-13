---
type: "practice"
difficulty: "Expert"
---

# Your team wants exactly-once writes from a Flink streaming job into Iceberg. How does the Iceberg Flink sink achieve this and where can duplicates still creep in?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Table Format Internals](<../../../../02_Data_Engineering/Storage_and_Lakehouse/Table_Formats/Table_Format_Internals.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To ensure exactly-once writes from a Flink streaming job into Iceberg, you need to understand how the Iceberg Flink sink operates and its limitations. This involves leveraging transactional semantics and understanding the potential pitfalls that could lead to duplicate data.

#### Triage
Understanding the Question

The question is probing your knowledge of exactly-once semantics in streaming contexts, specifically with Iceberg as the storage layer. The key points are:

How does the Iceberg Flink sink ensure exactly-once writes?
Where can duplicates still occur despite the use of the Iceberg Flink sink?

#### First Principles Solution

The Iceberg Flink sink leverages a combination of transactional semantics, checkpointing, and commit logs to achieve exactly-once writes. However, there are potential areas where duplicates might still arise due to the asynchronous nature of stream processing and distributed systems.

Iceberg Flink Sink Mechanism
Transaction Management: The Iceberg Flink sink uses Flink's transactional semantics. When a write request is sent to Iceberg, it first creates a new transaction. Once all records are successfully written, the commit log entry for this transaction is created.
Checkpointing and Failures: Checkpoints help in recovering from failures by ensuring that state can be restored. However, if a failure occurs between checkpointing and committing the write operation, it might result in duplicate writes or missed writes.
Commit Logs: The commit log ensures idempotency of operations by maintaining a record of all transactions. This helps in preventing reprocessing of already committed data.
Potential Duplicates

Despite these mechanisms, duplicates can still occur due to several reasons:

Late Arrival Records: In stream processing, late arriving records might be processed multiple times if the window is not properly configured.
Network Latency and Failures: If there are network delays or system failures, a record might be written twice before the transaction is committed.
Concurrency Issues: Concurrent writes to Iceberg can lead to conflicts, especially in distributed systems where coordination is challenging.

#### Closing Summary
+ Common Mistakes

To summarize, the Iceberg Flink sink ensures exactly-once writes by combining transactional semantics with checkpointing and commit logs. However, duplicates can still occur due to late arrival records, network latency, and concurrency issues.

#### Common mistakes
include not configuring windowing properly, ignoring network delays, or failing to handle concurrent writes effectively. Ensuring robust error handling and retry mechanisms are crucial in production environments.

Common Follow-Ups
How would you configure Flink's checkpointing for this use case?
What measures can be taken to minimize the impact of late arrival records?
How do you ensure idempotency in distributed systems when using Iceberg with Flink?

**Comparison:**

```
Transaction State	Description
In Progress	Records are being written.
Committed	Write operation is complete.
Failed	Write operation has failed.
```

**Code:**

```python
# Example configuration snippet for Flink's checkpointing
checkpoint_options = {
    'enable': True,
    'configuration': {'state.checkpoints.dir': 's3://your-checkpoint-dir',
                      'execution.checkpointing-interval': '5000 ms'}
}
```

**Example input / output:**

```text
Input:
- Flink streaming job writes records into Iceberg table

Output:
- Exactly-once semantics ensured through transactional operations and commit logs, but potential duplicates due to late arrivals or network issues.
```

**Data:**

```json
{
  "iceberg_table_schema": {
    "fields": [
      {"name": "id", "type": "INT"},
      {"name": "value", "type": "STRING"}
    ],
    "parameters": {
      "format-version": "2",
      "write-behavior": "MERGE Conflict"
    }
  }
}
```

**Flow:**

1. Yes
2. No
3. Record Received
4. Transaction Started
5. Transaction Confirmed?
6. Write to Iceberg Table
7. Commit Log Entry Created
8. Retry or Fail
9. Checkpoint Triggered
10. State Saved
11. Failure Occurs
12. Fall Back to Checkpoint

