---
type: "practice"
difficulty: "Deep"
---

# If your warehouse table drifts out of sync with the source how would you resync it with CDC?

**Difficulty:** Deep · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ CDC Fundamentals and Ordering](<../../../../02_Data_Engineering/Pipelines_and_Integration/Change_Data_Capture/CDC_Fundamentals_and_Ordering.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In this scenario, we need to address a situation where our warehouse table has drifted out of sync with its source system. To resynchronize them using Change Data Capture (CDC), we must implement a robust solution that tracks and applies changes from the source in an efficient manner. Let's break down the steps required for a thorough understanding.

**Code:**

```python
# Example Python function to illustrate CDC logic
def apply_cdc_changes(source_db, warehouse_table):
    # Fetch new or updated records from the source system
    new_records = fetch_new_records_from_source(source_db)

    # Apply changes to the warehouse table
    update_warehouse_table(warehouse_table, new_records)
```

**Example input / output:**

```text
Input:
- Source system contains new order data with IDs [101, 202, 303]
- Warehouse table has existing orders with IDs [101]

Output:
- Updated warehouse table includes all three new records (IDs 101, 202, 303)
```

**Data:**

```json
{
  "source_system": {
    "id": "12345",
    "name": "Sales System"
  },
  "warehouse_table": {
    "id": "67890",
    "name": "Orders"
  }
}
```

