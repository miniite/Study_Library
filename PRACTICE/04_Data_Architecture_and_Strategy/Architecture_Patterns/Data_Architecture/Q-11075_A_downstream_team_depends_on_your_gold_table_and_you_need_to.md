---
type: "practice"
difficulty: "Intermediate"
---

# A downstream team depends on your gold table and you need to change a column. Walk me through the versioning and deprecation process you'd follow so you don't break them overnight.

**Difficulty:** Intermediate · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Layered Pipelines Bronze Silver Gold](<../../../../04_Data_Architecture_and_Strategy/Architecture_Patterns/Data_Architecture/Layered_Pipelines_Bronze_Silver_Gold/_index.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When dealing with a downstream team that relies on a gold table, changing a column can have significant implications if not handled correctly. The key steps in managing this change involve versioning and deprecation to ensure minimal disruption and maintain data integrity. Here’s a step-by-step process:

Identify the Impact: First, understand which downstream systems are consuming the gold table and what changes might affect them.
Versioning Strategy: Implement a versioned schema to track different states of the data over time.
Deprecation Plan: Create a clear deprecation timeline to alert stakeholders about the upcoming change.

**Comparison:**

| Column | Version 1 | Version 2 |
|--------|-----------|-----------|
| id     | Present   | Present   |
| name   | Present   | Depreciated |
| email  | Present   | Present   |
| newName| Absent    | Present   |

**Code:**

```python
def update_column_version(old_schema, new_schema):
    """
    Function to manage column updates by creating a new versioned schema.
    """
    # Example schemas:
    old_schema = {"columns": ["id", "name", "email"], "version": 1}
    new_schema = {"columns": ["id", "newName", "email"], "version": 2}

    return {
        "oldSchema": old_schema,
        "newSchema": new_schema
    }
```

**Example input / output:**

```text
Input:
- oldSchema = {"columns": ["id", "name", "email"], "version": 1}
- newSchema = {"columns": ["id", "newName", "email"], "version": 2}

Output:
{
  "oldSchema": {
    "columns": ["id", "name", "email"],
    "version": 1
  },
  "newSchema": {
    "columns": ["id", "newName", "email"],
    "version": 2
  }
}
```

**Data:**

```json
{
  "oldSchema": {
    "columns": ["id", "name", "email"],
    "version": 1
  },
  "newSchema": {
    "columns": ["id", "newName", "email"],
    "version": 2
  }
}
```

**Flow:**

1. Yes
2. No
3. Identify Impact
4. Does the change impactdownstream systems?
5. Document and CommunicateChanges
6. Implement Versioned Schemafor New Data
7. Continue Serving Old Schemato Downstream Systems
8. Update Only New Data inVersioned Schema
9. Serve Updated Data toDownstream Systems
10. Deprecate Old Schema after aGrace Period

