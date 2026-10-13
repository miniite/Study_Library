---
type: "practice"
difficulty: "Deep"
---

# Explain how you'd keep dozens of derived stores (search indexes, caches, read models) in sync with primaries via CDC and detect drift org-wide. Design the reconciliation and drift-alerting as a shared platform capability rather than each team reinventing it.

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Keeping Indexes in Sync via CDC](<../../../../07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Clock_Skew_and_Time_Windows/Keeping_Indexes_in_Sync_via_CDC.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To keep dozens of derived stores (search indexes, caches, read models) in sync with primaries via Change Data Capture (CDC) and detect drift org-wide, we need a robust platform-level solution that ensures consistency across all systems. The primary challenge is to design this capability as a shared service rather than duplicating efforts across different teams. Here’s how I would approach it:

#### Overview
This involves setting up a CDC pipeline to capture changes from the primary sources and apply them to derived stores in real-time or near-real-time. We need to build a resilient system that can handle drift, which is any unintended divergence between derived stores and their primaries. The solution should be modular and scalable.

#### Triage
The question is essentially about designing a distributed systems architecture with focus on consistency, fault tolerance, and operational efficiency. Key assumptions include the use of reliable CDC mechanisms like Debezium or Apache Kafka Connect, and that we have access to the database change logs. We need to ensure that drift detection and reconciliation are efficient and minimal in terms of overhead.

#### Solution

Architecture: Use a centralized platform for CDC processing and drift detection.

Centralized CDC Processor: This component will consume change events from primary sources (databases, message queues).
Derived Store Updaters: These components apply the changes to each derived store based on the event type. For example, updates might trigger cache invalidation or re-indexing operations.
Drift Detector: A service that periodically checks for consistency between primaries and derived stores. It can use techniques like checksums, hashes, or full data comparison.

Reconciliation Mechanism: Implement a strategy to reconcile any detected drift. This could involve manually reviewing changes or automatically triggering re-indexing/cache invalidation processes.

Conflict Resolution: Decide on the precedence rules for handling conflicting updates (e.g., prioritize primary source over derived stores).
Logging and Monitoring: Ensure that all changes are logged, which can help in auditing and troubleshooting.

Shared Platform Service: To avoid reinventing the wheel, build this as a shared service with clear APIs. This ensures consistency and reduces operational burden across teams.

APIs for Configuration and Status: Allow teams to configure their derived stores, specify CDC sources, and monitor drift status.
Automated Deployment: Use CI/CD pipelines to ensure consistent setup of the platform components.

#### Closing Summary
The solution involves a centralized CDC processor that applies changes to derived stores and detects drift using a combination of real-time processing and periodic checks. By building this as a shared service, we can ensure consistency while reducing redundancy and operational overhead across teams.

#### Common Mistakes/Follow-Ups
Common pitfalls include not handling edge cases properly (e.g., partial updates), failing to account for network latency, or overcomplicating the reconciliation process. In follow-ups, one might be asked about specific technologies used, performance optimizations, or how to scale this solution.

**Comparison:**

```
Component	Functionality
CDC Processor	Consumes changes from primary sources
Derived Store Updater	Applies changes to derived stores
Drift Detector	Detects inconsistencies between primaries and derived stores
```

**Code:**

```python
# Example CDC Consumer in Python using Debezium
import json

def consume_change_event(event):
    change_type = event['changeType']
    table_name = event['tableSchema'] + '.' + event['tableName']

    if change_type == 'INSERT':
        # Handle insert operation
        print(f"New record inserted into {table_name}")
    elif change_type == 'UPDATE':
        # Handle update operation
        print(f"Record updated in {table_name}")
    elif change_type == 'DELETE':
        # Handle delete operation
        print(f"Record deleted from {table_name}")

# Example drift detector function
def detect_drift(primary_data, derived_data):
    if primary_data != derived_data:
        return True
    return False
```

**Example input / output:**

```text
Input:
- Change event from primary database: {'changeType': 'UPDATE', 'tableSchema': 'schema', 'tableName': 'table_name'}

Output:
- Derived store updater processes the change and updates derived data.
```

**Data:**

```json
{
  "primaryData": {"key1": "value1", "key2": "value2"},
  "derivedData": {"key1": "value1", "key2": "value3"}
}
```

**Flow:**

1. Yes
2. Yes
3. No
4. Centralized CDC Processor
5. Change Event?
6. Apply Changes
7. Drift Detector
8. Detect Drift?
9. Reconciliation Mechanism
10. Log and Monitor
11. Next Change Event

