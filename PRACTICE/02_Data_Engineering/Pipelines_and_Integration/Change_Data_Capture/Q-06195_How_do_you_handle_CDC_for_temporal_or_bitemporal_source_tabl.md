---
type: "practice"
difficulty: "Deep"
---

# How do you handle CDC for temporal or bitemporal source tables where the source itself already tracks valid-time versions?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ CDC Fundamentals and Ordering](<../../../../02_Data_Engineering/Pipelines_and_Integration/Change_Data_Capture/CDC_Fundamentals_and_Ordering.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When dealing with Change Data Capture (CDC) for temporal or bitemporal source tables, it's essential to understand that these tables are designed to track events over time with two dimensions of validity: the valid-time (when an event occurred) and the system-time (when the record was created or last updated). Handling CDC in such cases involves capturing both the temporal aspects and ensuring consistency between source and target systems. The key is to properly manage the source’s timestamps and ensure they are preserved during the CDC process.

To summarize, we need a robust CDC mechanism that accurately captures and propagates valid-time and system-time information from the source tables to the target, maintaining data integrity over time.

**Comparison:**

```
Source Table	Target Table
unique_id, column1, column2, valid_time, system_time	unique_id, column1, column2, valid_time, system_time
```

**Code:**

```python
# Example of a Python function for CDC with temporal data
def capture_cdc_changes(source_table, target_table):
    # Fetch recent changes from the source table based on system-time and valid-time
    changes = fetch_recent_changes(source_table)

    # Insert or update records in the target table with appropriate timestamps
    for change in changes:
        if is_record_new(change['valid_time']):
            insert_record_into_target(change, target_table)
        else:
            update_record_in_target(change, target_table)

def fetch_recent_changes(table):
    """Fetches records that have changed since the last CDC run."""
    return table.execute("SELECT * FROM {} WHERE valid_time > :last_valid_time AND system_time > :last_system_time".format(table.name),
                         {"last_valid_time": last_valid_time, "last_system_time": last_system_time})

def insert_record_into_target(record, target):
    """Inserts a new record into the target table with proper timestamps."""
    target.execute("INSERT INTO target_table (column1, column2, valid_time, system_time) VALUES (:val1, :val2, :valid_time, :system_time)",
                   {"val1": record['val1'], "val2": record['val2'], "valid_time": record['valid_time'], "system_time": record['system_time']})

def update_record_in_target(record, target):
    """Updates an existing record in the target table with proper timestamps."""
    target.execute("UPDATE target_table SET column1 = :new_val1, column2 = :new_val2, valid_time = :valid_time, system_time = :system_time WHERE unique_id = :unique_id",
                   {"new_val1": record['val1'], "new_val2": record['val2'], "valid_time": record['valid_time'], "system_time": record['system_time'], "unique_id": record['unique_id']})
```

**Example input / output:**

```text
# Example input/output for fetching and updating records
last_valid_time = "2023-06-10T00:00:00Z"
last_system_time = "2023-06-15T00:00:00Z"

changes = fetch_recent_changes(source_table, last_valid_time, last_system_time)

for change in changes:
    if is_record_new(change['valid_time']):
        insert_record_into_target(change, target_table)
    else:
        update_record_in_target(change, target_table)

# Target table after processing
{
  "records": [
    {"unique_id": 1, "column1": "A", "column2": "B", "valid_time": "2023-06-01T00:00:00Z", "system_time": "2023-06-05T00:00:00Z"},
    {"unique_id": 2, "column1": "X", "column2": "Y", "valid_time": "2023-06-10T00:00:00Z", "system_time": "2023-06-08T00:00:00Z"}
  ]
}
```

**Data:**

```json
{
  "source_table": {
    "columns": ["unique_id", "column1", "column2", "valid_time", "system_time"],
    "records": [
      {"unique_id": 1, "column1": "A", "column2": "B", "valid_time": "2023-06-01T00:00:00Z", "system_time": "2023-06-05T00:00:00Z"},
      {"unique_id": 2, "column1": "X", "column2": "Y", "valid_time": "2023-06-10T00:00:00Z", "system_time": "2023-06-08T00:00:00Z"}
    ]
  },
  "target_table": {
    "columns": ["unique_id", "column1", "column2", "valid_time", "system_time"],
    "records": [
      {"unique_id": 1, "column1": "A", "column2": "B", "valid_time": "2023-06-01T00:00:00Z", "system_time": "2023-06-05T00:00:00Z"}
    ]
  }
}
```

**Flow:**

1. Yes
2. No
3. Fetch Recent Changes
4. Fetch Records
5. Is Record New?
6. Insert Record
7. Update Record

