---
type: "practice"
difficulty: "Intermediate"
---

# What is change data capture (CDC) and why is it useful for keeping a warehouse in sync with a source database?

**Difficulty:** Intermediate · **Levels:** Fresher · [↑ Schema Migration Risks](<../../../../01_Databases_and_SQL/Database_Internals/DBMS_Concepts/Schema_Migration_Risks.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Change Data Capture (CDC) is a technique used to track and capture changes made to data in real-time or near-real-time, enabling these changes to be replicated to other systems, such as data warehouses. In simpler terms, CDC allows us to see the "delta" between two database versions—essentially what has changed since the last update. This mechanism is invaluable for maintaining a synchronized and up-to-date data warehouse because it ensures that only current and relevant data flows from the source database to the target system.

The question essentially probes your understanding of how CDC works, its benefits, and why it's necessary in certain scenarios, particularly when dealing with large-scale data integration projects. Key assumptions are that you know about databases, understand what a data warehouse is, and can discuss real-time versus batch processing contexts.

**Comparison:**

```
Change Type	Description
Insert	New record added to the database.
Update	Existing record values have changed.
Delete	Record is marked as deleted in the source database but may still need to be handled.
```

**Code:**

```python
# This is an example of how CDC might be implemented using Python pseudocode.
def capture_changes(source_db, target_db):
    # Step 1: Identify the last known state in the target database
    last_known_state = get_last_snapshot(target_db)

    # Step 2: Compare this with the current state in the source database
    changes = find_differences(last_known_state, source_db)

    # Step 3: Apply these changes to the target database
    apply_changes(changes, target_db)

# Pseudocode functions for illustration:
def get_last_snapshot(target_db):
    return fetch_last_snapshot_info(target_db)

def find_differences(last_known_state, source_db):
    return identify_records_with_diffs(last_known_state, source_db)

def apply_changes(changes, target_db):
    update_target_database(changes)
```

**Example input / output:**

```text
Input:
- Source Database: `orders` table with updates
- Target Database: Currently at snapshot time `2023-10-01T14:30:00Z`

Output:
- Target Database now updated with new and changed records, e.g., order ID 1 has been shipped, and the amount for order ID 2 has increased to $400.
```

**Data:**

```json
{
  "last_snapshot": {
    "table_name": "orders",
    "timestamp": "2023-10-01T14:30:00Z"
  },
  "changes": [
    {
      "id": 1,
      "action": "insert",
      "new_values": {"order_amount": 150, "status": "shipped"}
    },
    {
      "id": 2,
      "action": "update",
      "old_values": {"order_amount": 300},
      "new_values": {"order_amount": 400}
    }
  ]
}
```

**Flow:**

1. Yes
2. No
3. Identify Last Snapshot
4. Fetch Changes from Source
5. Changes?
6. Apply Changes to Target DB
7. No Changes, Sync Complete

