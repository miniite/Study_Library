---
type: "practice"
difficulty: "Intermediate"
---

# What is the difference between a full load and an incremental load in a pipeline?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Transform in Pipeline vs Warehouse](<../../../../02_Data_Engineering/Pipelines_and_Integration/Pipeline_Design/Transform_in_Pipeline_vs_Warehouse.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Full load and incremental loads are two fundamental concepts in data pipeline design, each serving different needs depending on your system's requirements. A full load involves loading all of the data from a source into a target storage system at one time, essentially performing a complete refresh. An incremental load, on the other hand, only updates the data that has changed since the last update, making it more efficient for ongoing data integration and keeping your systems up-to-date with minimal overhead.

To better understand these concepts, let's break down what each involves from first principles.

**Comparison:**

```
Full Load	Incremental Load
Loads all data at once.	Updates only changed data.
Time-consuming for large datasets.	Efficient with small changes.
Ensures no data loss.	May miss recent changes if not timely.
Useful for initial setup or system migrations.	Ideal for daily/real-time updates.
```

**Code:**

```python
# Example code to illustrate full load vs incremental load in a simplified Python context

def full_load(source_data):
    """Loads all data into the target storage."""
    # Assume source_data is a list of records
    for record in source_data:
        # Insert record into database
        pass

def incremental_load(last_updated_timestamp, source_data):
    """Updates only the data that has changed since last updated timestamp."""
    for record in source_data:
        if record['last_update_time'] > last_updated_timestamp:
            # Update or insert new record into database based on conditions
            pass
```

**Example input / output:**

```text
# Full Load Example (Input)
source_data = [
    {"id": 1, "value": 10, "last_update_time": "2023-10-05T14:30:00Z"},
    {"id": 2, "value": 20, "last_update_time": "2023-10-06T15:00:00Z"},
    {"id": 3, "value": 30, "last_update_time": "2023-10-07T16:30:00Z"}
]

# Incremental Load Example (Input)
last_updated_timestamp = "2023-10-06T15:00:00Z"

output_full_load = full_load(source_data)
output_incremental_load = incremental_load(last_updated_timestamp, source_data)
```

**Data:**

```json
{
  "source_data": [
    {"id": 1, "value": 10, "last_update_time": "2023-10-05T14:30:00Z"},
    {"id": 2, "value": 20, "last_update_time": "2023-10-06T15:00:00Z"},
    {"id": 3, "value": 30, "last_update_time": "2023-10-07T16:30:00Z"}
  ],
  "target_data": [
    {"id": 1, "value": 10, "last_update_time": "2023-10-05T14:30:00Z"},
    {"id": 2, "value": 20, "last_update_time": "2023-10-06T15:00:00Z"}
  ]
}
```

**Flow:**

1. Full Load
2. (Load all data)
3. (Database refresh)
4. Incremental Load
5. (Check for changes)
6. (Update or insert new records)

