---
type: "practice"
difficulty: "Deep"
---

# A trigger that writes to an audit table on every UPDATE is slowing down a hot path; how would you measure the trigger's cost and decide between async logging, logical decoding, or accepting the trigger overhead?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Materialized Views vs Incremental Aggregates](<../../../../TOOLS/PostgreSQL/PostgreSQL_Platform/Materialized_Views_vs_Incremental_Aggregates.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When dealing with performance issues in PostgreSQL, especially around triggers, it’s crucial to understand that every update operation incurs additional overhead due to the trigger execution. The specific scenario described here involves an audit table being updated on every UPDATE, which can become a bottleneck if not managed properly. To address this issue, we need to measure the cost of the current trigger implementation and then decide whether to switch to asynchronous logging, logical decoding, or accept some performance overhead.

#### Triaging the Question

The question is essentially asking us to:

Quantify the impact of the current trigger on the hot path.
Evaluate different alternatives, including asynchronous logging and logical decoding.
Decide on an appropriate solution based on the measured costs and the application's requirements.

#### Assumptions
The audit table is being updated in a synchronous manner, meaning that every UPDATE operation on the main table waits for the trigger to finish before proceeding.
We are using PostgreSQL 12 or later, as these versions support logical decoding natively.
The hot path is critical and needs minimal latency.

#### First Principles Solution
Step 1: Measure the Current Cost

To measure the cost of the current implementation:

SQL
EXPLAIN ANALYZE UPDATE table SET column = value WHERE condition;

This will provide us with a detailed execution plan showing the time taken for the UPDATE operation, including any waits or blocks due to the trigger.

Step 2: Analyze Trigger Performance

We need to understand why the current trigger is slow. Common causes include:

Complexity: The logic inside the trigger might be complex.
Locking: Triggers can block other transactions if they hold locks for too long.
Data Volume: If the audit table has a large volume of data, each update can take longer.
Step 3: Consider Asynchronous Logging

Asynchronous logging involves processing updates in a separate background process rather than blocking the main transaction. This can be achieved using:

SQL
LISTEN and NOTIFY

or by implementing an external script that listens for changes via pg_logical_decoding.

#### Implementation
We would need to set up a notification system where the trigger sends notifications to the logging service.
Cost: This introduces additional complexity but reduces blocking.
Step 4: Evaluate Logical Decoding

Logical decoding allows us to replicate only the necessary data changes rather than full rows. This is particularly useful for audit logs:

SQL
CREATE PUBLICATION my_publication FOR TABLE audit_table;

Then use logical replication to subscribe and process these changes.

#### Pros
Reduces overhead by processing only relevant changes.

#### Cons
Requires setting up subscriptions, which can add complexity.
Step 5: Compare Costs

Compare the performance of:

Current synchronous trigger.
Asynchronous logging with notifications.
Logical decoding for audit logs.

#### Closing Summary

Based on our analysis and measurements, we can decide whether to:

Keep the current trigger if it is not significantly impacting performance.
Implement asynchronous logging if there are frequent updates and blocking issues.
Use logical decoding for more complex scenarios where only relevant data changes need to be logged.

#### Common Mistakes/Follow-Ups
Not measuring: Always measure before deciding on a solution.
Ignoring complexity: Synchronous triggers can simplify debugging but might introduce blocking issues.
Over-engineering: Logical decoding is powerful but adds complexity, so it should only be used when necessary.

**Comparison:**

```
Hot Path Queries	Execution Time (ms)	Wait Time for Trigger (ms)
1	50	40
```

**Code:**

```sql
-- Example of an audit trigger
CREATE OR REPLACE FUNCTION update_audit_trigger()
RETURNS TRIGGER AS $$
BEGIN
    -- Insert logic to log changes into the audit table
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER trg_audit
AFTER UPDATE ON main_table
FOR EACH ROW EXECUTE PROCEDURE update_audit_trigger();
```

**Example input / output:**

```text
Input:
- Current query performance metrics showing significant overhead due to triggers.

Output:
- Measured execution times and waits.
- Decisions on whether to move to asynchronous logging or logical decoding.
```

**Data:**

```json
{
  "hot_path_queries": [
    {
      "query": "UPDATE orders SET status = 'shipped' WHERE order_id = 123",
      "execution_time": 50ms,
      "wait_time_for_trigger": 40ms
    }
  ]
}
```

**Flow:**

1. No
2. Yes
3. Measure Current Cost
4. Is the current trigger abottleneck?
5. Keep the current trigger
6. Synchronize withasynchronous logging
7. Implement notifications andlogging service
8. Evaluate logical decoding formore complex scenarios

