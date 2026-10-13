---
type: "practice"
difficulty: "Intermediate"
---

# What is Monthly Active Rows (MAR) in Fivetran's pricing model?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Incremental Sync and Connector Migration](<../../../../02_Data_Engineering/Pipelines_and_Integration/Data_Ingestion/Incremental_Sync_and_Connector_Migration.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Monthly Active Rows (MAR) in Fivetran’s pricing model refers to a metric used by Fivetran for billing, specifically targeting the unique number of rows processed each month from your data sources. This metric is crucial because it helps Fivetran determine the cost based on the amount and frequency of data ingested into your destination systems.

Fivetran charges customers based on the MAR, which means you are billed according to the volume of data that flows through their platform in a given month. The key idea here is to understand how this metric is calculated and how it impacts your billing.

**Code:**

```python
# Example code snippet for calculating MAR (pseudo-code)
def calculate_mar(source_data, destination_data):
    source_rows = set()
    for row in source_data:
        source_rows.add(row['id'])  # Assuming each row has a unique identifier

    destination_rows = set()
    for row in destination_data:
        if row['id'] in source_rows:
            destination_rows.add(row['id'])

    mar = len(destination_rows)
    return mar
```

**Example input / output:**

```text
Input:
source_data = [{"id": 1, "timestamp": "2023-10-01T08:00:00Z"}, {"id": 2, "timestamp": "2023-10-01T09:00:00Z"}, {"id": 3, "timestamp": "2023-10-01T10:00:00Z"}]
destination_data = [{"id": 1, "timestamp": "2023-10-01T08:05:00Z"}, {"id": 2, "timestamp": "2023-10-01T09:05:00Z"}, {"id": 4, "timestamp": "2023-10-01T10:05:00Z"}]

Output:
MAR = 2
```

**Data:**

```json
{
  "source_data": [
    {"id": 1, "timestamp": "2023-10-01T08:00:00Z"},
    {"id": 2, "timestamp": "2023-10-01T09:00:00Z"},
    {"id": 3, "timestamp": "2023-10-01T10:00:00Z"}
  ],
  "destination_data": [
    {"id": 1, "timestamp": "2023-10-01T08:05:00Z"},
    {"id": 2, "timestamp": "2023-10-01T09:05:00Z"},
    {"id": 4, "timestamp": "2023-10-01T10:05:00Z"}
  ]
}
```

