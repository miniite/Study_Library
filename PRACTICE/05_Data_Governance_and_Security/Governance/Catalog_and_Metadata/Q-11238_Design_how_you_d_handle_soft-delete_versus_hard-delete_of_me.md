---
type: "practice"
difficulty: "Deep"
---

# Design how you'd handle soft-delete versus hard-delete of metadata — a table is dropped, then recreated a week later. Should lineage history persist, and how do you avoid resurrecting stale metadata or losing audit trail?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Catalog Consistency Model](<../../../../05_Data_Governance_and_Security/Governance/Catalog_and_Metadata/Catalog_Consistency_Model.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To design an effective system for handling soft-delete versus hard-delete of metadata in a data catalog, we need to ensure that lineage history is preserved while avoiding the resurrection of stale metadata and maintaining a robust audit trail. This involves a multi-layered approach considering both technical implementation and business requirements.

#### Triaging the Question

The core of this question revolves around how to manage the lifecycle of metadata within a data catalog when deletions occur, ensuring historical lineage is retained without introducing risks such as resurrecting outdated or incorrect metadata. The primary concerns are:

Lineage History: Retaining the history of changes and transformations.
Avoiding Stale Metadata: Ensuring that deleted metadata does not reappear in active use.
Audit Trail: Maintaining a comprehensive record of all actions performed on data assets.

#### Assumptions
We are using a modern data catalog system with capabilities for soft-deletes, hard-deletes, and versioning.
The system supports lineage tracking through metadata governance policies.
There is an existing audit trail mechanism in place to log deletions and recreations.

#### First Principles Solution
Step 1: Soft-Delete Implementation

Soft-delete involves marking the metadata as logically deleted but not physically removing it from the catalog. This can be done by:

SQL
UPDATE metadata SET is_deleted = TRUE WHERE table_id = 'example_table';

This ensures that the metadata remains in the system, preserving lineage and audit trail information.

Step 2: Hard-Delete Handling

Hard-delete involves fully removing the metadata from the system to prevent any accidental resurrection. However, we need a mechanism to ensure that historical data is not lost:

SQL
DELETE FROM metadata WHERE table_id = 'example_table';

For hard-deletes, we should also log this action in an audit trail.

Step 3: Recreating Metadata

When recreating the metadata after a week (or any other defined period), we need to ensure that lineage information is preserved. This can be achieved by:

Retrieving Historical Data: Reconstructing the metadata from historical logs.
Versioning: Implementing versioned metadata to distinguish between different states of the table.
Step 4: Lineage Preservation

To maintain accurate lineage, we should capture all operations performed on the data asset before and after its recreation:

PYTHON
def create_lineage_log(table_id):
    with open('lineage_logs.txt', 'a') as log_file:
        log_file.write(f"Recreated {table_id} at {datetime.now()}\n")
Step 5: Audit Trail Management

Ensure that all deletion and recreation actions are logged in an audit trail to maintain accountability and traceability.

SQL
INSERT INTO audit_trail (action, table_id, timestamp) VALUES ('RECREATED', 'example_table', NOW());

#### Closing Summary
+ Common Mistakes/Follow-Ups

In summary, the key is to balance between maintaining lineage history and avoiding stale metadata. By implementing soft-deletes with logs, versioning, and robust audit trails, we can effectively manage deletions in a way that preserves necessary historical context without introducing operational risks.

#### Common Mistakes

Failing to log all deletion and recreation actions.
Not properly distinguishing between soft and hard deletes.
Losing lineage information during metadata recreations.

#### Follow-up
questions might include:

How do you ensure data integrity during the recreation process?
What are the implications of different retention periods for metadata?
How would you handle concurrent modifications or deletions?

**Comparison:**

| Metadata Table                | Is Deleted | Last Updated   |
|-------------------------------|------------|----------------|
| example_table                 | TRUE       | 2023-10-01     |
| Audit Trail                   | Action    | Table ID       | Timestamp           |
|-------------------------------|-----------|----------------|
| DELETED                       | example_table | 2023-10-05T14:30:00Z |
| RECREATED                     | example_table | 2023-10-12T16:00:00Z |

**Code:**

```sql
-- Soft-delete implementation
UPDATE metadata SET is_deleted = TRUE WHERE table_id = 'example_table';

-- Hard-delete implementation
DELETE FROM metadata WHERE table_id = 'example_table';

-- Creating lineage log
def create_lineage_log(table_id):
    with open('lineage_logs.txt', 'a') as log_file:
        log_file.write(f"Recreated {table_id} at {datetime.now()}\n")

-- Audit trail management
INSERT INTO audit_trail (action, table_id, timestamp) VALUES ('RECREATED', 'example_table', NOW());
```

**Example input / output:**

```text
Input:
- Soft-delete: UPDATE metadata SET is_deleted = TRUE WHERE table_id = 'example_table';
- Hard-delete: DELETE FROM metadata WHERE table_id = 'example_table';

Output:
- Lineage log entry: "Recreated example_table at 2023-10-12T16:00:00Z"
- Audit trail entry: "RECREATED, example_table, 2023-10-12T16:00:00Z"
```

**Data:**

```json
{
  "metadata": {
    "table_id": "example_table",
    "is_deleted": true,
    "last_updated": "2023-10-01T12:00:00Z"
  },
  "audit_trail": [
    {
      "action": "DELETED",
      "table_id": "example_table",
      "timestamp": "2023-10-05T14:30:00Z"
    },
    {
      "action": "RECREATED",
      "table_id": "example_table",
      "timestamp": "2023-10-12T16:00:00Z"
    }
  ]
}
```

**Flow:**

1. Yes
2. No
3. Metadata Soft-Deleted
4. Is Table Recreated?
5. Log Recreate Action
6. Version Metadata
7. Create Lineage Log
8. Audit Trail Updated

