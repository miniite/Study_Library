---
type: "practice"
difficulty: "Intermediate"
---

# How would you decide whether to model event context, such as device and app version, as a single junk dimension or as several separate small dimensions?

**Difficulty:** Intermediate · **Levels:** Architect/Principal, SDE-1, SDE-1/2, SDE-2/3 · [↑ Special Dimension Types](<../../../../02_Data_Engineering/Modeling_and_Warehousing/Dimensional_Modeling/Special_Dimension_Types.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To determine whether to use a junk dimension (single large dimension) or multiple separate small dimensions for modeling event context like device and app version in a star schema, we need to balance ease of querying with normalization benefits. Generally, using multiple small dimensions provides better normalization and easier maintenance but may complicate queries. A single junk dimension simplifies queries but can be less normalized.

The key factors to consider are:

Query complexity
Data integrity and consistency
Storage efficiency

#### Performance considerations

Let's break this down from first principles.

**Code:**

```
MERMAID
Copy
graph TD;
    A[Event Context] --> B{Single Junk Dimension};
    B --> C1[Simple Queries, Easier Maintenance];
    B --> C2[Less Normalized, Potentially Less Storage Efficient];
    A --> D{Multiple Small Dimensions};
    D --> E1[More Complex Queries, Better Normalization];
    D --> E2[Storage Efficiency, Improved Data Integrity];
```

**Example input / output:**

```text
For the single junk dimension approach:
- Input: `INSERT INTO event_context (device_id, app_version) VALUES (1, '4.3.2');`
- Output: A new record in a large table.

For multiple small dimensions:
- Input:
  - `INSERT INTO devices (id) VALUES (1);`
  - `INSERT INTO app_versions (version) VALUES ('4.3.2');`
- Output: Separate records in smaller tables.
```

**Data:**

```json
{
  "event_context": {
    "single_junk_dimension": [
      {"device_id": 1, "app_version": "4.3.2", "other_attributes": "..."}
    ],
    "multiple_dimensions": [
      {"device_id": 1},
      {"app_version": "4.3.2"}
    ]
  }
}
```

