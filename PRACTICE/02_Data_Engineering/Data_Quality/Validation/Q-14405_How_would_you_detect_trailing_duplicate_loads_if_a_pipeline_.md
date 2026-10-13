---
type: "practice"
difficulty: "Intermediate"
---

# How would you detect trailing duplicate loads if a pipeline accidentally ran twice for the same day?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Uniqueness Checks](<../../../../02_Data_Engineering/Data_Quality/Validation/Uniqueness_Checks.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the issue of detecting trailing duplicate loads in data pipelines, we need to ensure that our system can identify and handle situations where a pipeline runs multiple times on the same day. This is crucial for maintaining data integrity and preventing redundant processing or storage.

#### Triage
& Assumptions

The core question here revolves around how to detect and manage duplicate loads within a data processing pipeline. The key assumptions are:

We have a pipeline that processes data daily.
There might be instances where the pipeline runs more than once in a single day.
Our goal is to identify these trailing duplicates efficiently.
Step-by-Step Solution from First Principles
Data Staging Layer: Implement a staging layer where raw data is stored temporarily before it undergoes further processing. This ensures that we can track which data has already been processed and prevent reprocessing.
Unique Key Identification: Identify a unique key or combination of keys for each record. This could be an auto-incrementing ID, a combination of date and primary keys, etc.
Tracking Table: Create a tracking table to log when records were last processed. This can be done using a simple database with columns like record_id, last_processed_date, and pipeline_run_id.
Duplicate Detection Logic: In the pipeline processing logic, check if any record being processed has already been marked as processed in the tracking table.
Pipeline Run Identification: Assign each pipeline run a unique identifier (e.g., timestamp or UUID) to ensure that we can track which runs are duplicates.

#### Implementation
Steps

Data Staging Layer Setup

SQL
CREATE TABLE staging_table (
  id BIGINT PRIMARY KEY AUTO_INCREMENT,
  record_data JSON NOT NULL,
  file_name VARCHAR(255),
  load_date DATE
);

Tracking Table Creation

SQL
CREATE TABLE tracking_table (
  record_id BIGINT NOT NULL,
  last_processed_date DATE NOT NULL,
  pipeline_run_id VARCHAR(36) NOT NULL,
  PRIMARY KEY (record_id, pipeline_run_id)
);

Processing Logic with Duplicate Detection

Before processing each record from the staging table, query the tracking table to check if it has already been processed.
If not, update the tracking table with the current run ID and mark the record as processed.
Sample Code

Here's a simplified example in Python using pandas for data manipulation:

PYTHON
import pandas as pd
from datetime import datetime

# Function to process records
def process_records(records_df):
    # Load existing tracking info from a file or database
    tracking_info = load_tracking_info()

    # Iterate over each record
    for _, row in records_df.iterrows():
        record_id = row['record_id']
        pipeline_run_id = str(uuid.uuid4())

        # Check if the record has already been processed
        if (record_id, pipeline_run_id) not in tracking_info:
            # Process the record
            process_record(row)

            # Update tracking info
            tracking_info.add((record_id, pipeline_run_id))
            save_tracking_info(tracking_info)

def load_tracking_info():
    # Load existing tracking data from a file or database
    pass

def save_tracking_info(tracking_info):
    # Save the updated tracking information to a file or database
    pass

def process_record(record):
    # Implement processing logic for each record
    print(f"Processing record: {record}")

# Example usage
records_df = pd.DataFrame({
    'record_id': [1, 2, 3, 4],
    'data': ['A', 'B', 'C', 'D']
})
process_records(records_df)

#### Closing Summary
and Common Mistakes

By implementing a tracking mechanism as described above, we can effectively detect trailing duplicate loads in our data pipeline. Key takeaways include the importance of having a staging layer for raw data storage and a robust system to track record processing status.

#### Common mistakes
to avoid:

Not using unique keys or identifiers for tracking.
Failing to update the tracking table after each successful process.
Ignoring edge cases where records might be partially processed before failing.

**Comparison:**

```
Staging Table
Process Record
Tracking Table
Update Status
Track Duplicate Loads
```

**Code:**

```python
import pandas as pd
from datetime import datetime
import uuid

# Function to process records
def process_records(records_df):
    # Load existing tracking info from a file or database
    tracking_info = load_tracking_info()

    # Iterate over each record
    for _, row in records_df.iterrows():
        record_id = row['record_id']
        pipeline_run_id = str(uuid.uuid4())

        # Check if the record has already been processed
        if (record_id, pipeline_run_id) not in tracking_info:
            # Process the record
            process_record(row)

            # Update tracking info
            tracking_info.add((record_id, pipeline_run_id))
            save_tracking_info(tracking_info)

def load_tracking_info():
    # Load existing tracking data from a file or database
    pass

def save_tracking_info(tracking_info):
    # Save the updated tracking information to a file or database
    pass

def process_record(record):
    # Implement processing logic for each record
    print(f"Processing record: {record}")

# Example usage
records_df = pd.DataFrame({
    'record_id': [1, 2, 3, 4],
    'data': ['A', 'B', 'C', 'D']
})
process_records(records_df)
```

**Example input / output:**

| Input DataFrame | Tracking Table Before Processing |
|-----------------|----------------------------------|
| record_id       | tracking_info                    |
| 1               | {}                               |
| 2               |                                   |
| 3               |                                   |
| 4               |                                   |
| Processed Records | Updated Tracking Table         |
|-------------------|--------------------------------|
| [{'record_id': 1, 'data': 'A'}, {'record_id': 2, 'data': 'B'}] | { (1, <pipeline_run_id_1>), (2, <pipeline_run_id_1>) } |

**Data:**

```sql
CREATE TABLE staging_table (
  id BIGINT PRIMARY KEY AUTO_INCREMENT,
  record_data JSON NOT NULL,
  file_name VARCHAR(255),
  load_date DATE
);

CREATE TABLE tracking_table (
  record_id BIGINT NOT NULL,
  last_processed_date DATE NOT NULL,
  pipeline_run_id VARCHAR(36) NOT NULL,
  PRIMARY KEY (record_id, pipeline_run_id)
);
```

**Flow:**

1. Yes
2. No
3. Load Records
4. Process Records
5. Is Record Processed?
6. Skip Processing
7. Mark As Processed
8. Track Load
9. Repeat for All Records

