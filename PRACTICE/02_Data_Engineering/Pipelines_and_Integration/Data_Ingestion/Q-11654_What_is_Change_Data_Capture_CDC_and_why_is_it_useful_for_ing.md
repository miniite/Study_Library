---
type: "practice"
difficulty: "Intermediate"
---

# What is Change Data Capture (CDC) and why is it useful for ingestion?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ CDC vs Full Reload](<../../../../02_Data_Engineering/Pipelines_and_Integration/Data_Ingestion/CDC_vs_Full_Reload.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Change Data Capture (CDC) is a method used to efficiently ingest and process changes made to data in real-time or near-real-time. It’s particularly useful for keeping your data up-to-date without overloading your database systems with repeated queries or massive data dumps.

In the context of data ingestion, CDC allows us to capture only the updates, inserts, and deletes that occur in a source system, rather than ingesting the entire dataset every time. This approach is more efficient and helps maintain lower latency between the source and target systems.

To understand why CDC is valuable, let's break it down from first principles.

**Code:**

```python
# Example of a simple CDC scenario in Python using Airbyte
def cdc_process(source_data, current_state):
    changes = []
    for record in source_data:
        if record not in current_state:
            changes.append("INSERT " + str(record))
        else:
            old_record = current_state[record]
            if old_record != record:
                changes.append("UPDATE " + str(old_record) + " TO " + str(record))
            del current_state[record]  # Mark as processed
    for remaining in current_state.values():
        changes.append("DELETE " + str(remaining))
    return changes

# Example data
source_data = {"user1": {"name": "Alice", "email": "alice@example.com"},
               "user2": {"name": "Bob", "email": "bob@example.com"}}
current_state = {"user3": {"name": "Charlie", "email": "charlie@example.com"}}

changes = cdc_process(source_data, current_state)
print(changes)
```

**Example input / output:**

```text
Output:
['DELETE {"name": "Charlie", "email": "charlie@example.com"}', 'INSERT {"name": "Alice", "email": "alice@example.com"}', 'UPDATE {"name": "Bob", "email": "bob@example.com"} TO {"name": "Charlie", "email": "charlie@example.com"}']
```

**Flow:**

1. Yes
2. No
3. Source System
4. Changes?
5. CDC Process
6. Ingest Data to Target
7. Mark as Processed
8. Check for RemainingUnprocessed Changes
9. Dry-run CDC Process
10. Finalize Ingestion

