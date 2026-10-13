---
type: "practice"
difficulty: "Deep"
---

# How would you design an ETL job to be restartable from where it left off rather than the beginning?

**Difficulty:** Deep · **Levels:** Architect/Principal, Fresher, SDE-1/2, SDE-2/3 · [↑ Safe Re-runs of Failed Jobs](<../../../../02_Data_Engineering/Pipelines_and_Integration/ETL_ELT/Safe_Re-runs_of_Failed_Jobs.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To design a restartable ETL (Extract, Transform, Load) job, we need to ensure that each step in the process can pick up where it left off if interrupted or fail. This is crucial for maintaining data integrity and minimizing downtime during failures. We'll cover a systematic approach from first principles, suitable for both intermediate and advanced levels.

#### Triaging the Question

The core of this question is about making an ETL job robust against interruptions by enabling state persistence between runs. Specifically, we need to:

Identify the state of each process: Understand where each step left off before a failure.
Persist state information: Store this state somewhere that can be accessed and reused in subsequent runs.
Resume processing from the stored state: Modify the ETL workflow to skip already processed data.

#### Assumptions
We are dealing with a batch ETL process, not a real-time pipeline.
The data sources and targets are well-defined but might change over time.
Failures can occur at any stage of the ETL job: extraction, transformation, or loading.
The state information needs to be stored durably.

#### First-Principles Solution
Determine State Information: Each step in the ETL process should have a way to identify its progress. This could include timestamps, sequence numbers, or unique identifiers for rows processed.
Persistence Mechanism: Use a reliable storage solution (e.g., database, file system) to store state information between runs.
Resume Logic: Modify the job to check for existing state and only process new data.
Detailed Steps
Step 1: Define State Information
For extraction, use timestamps or sequence numbers to track rows processed.
For transformation, maintain a record of transformations applied.
For loading, keep track of successfully loaded records.
Step 2: Choose Storage for State
Database: Useful if you need complex state tracking and query capabilities.
File System: Simpler but less robust in terms of durability and accessibility.
Message Queue: Can be used to queue tasks and manage state through message acknowledgments.
Step 3: Implement Resume Logic
At the start of each run, read the last processed state from the chosen storage mechanism.
Skip records that have already been processed according to the stored state.
Update the state in real-time as new data is extracted, transformed, and loaded.

#### Example
Workflow

Extraction:

Use a primary key or timestamp to identify rows not yet processed.
Store these identifiers in a database table named extracted_rows.

Transformation:

Query the extracted_rows table for new records.
Process and transform data, updating a transformation state table (e.g., transformed_rows).

Loading:

Use the transformed row identifiers to load into the target database or file system.
Update the loaded_rows table with successful loads.
Implementing Resume Logic
PYTHON
# Pseudo-code for resume logic in a data pipeline script
def start_etl_job():
    # Load state from storage (e.g., database)
    last_processed_timestamp = get_last_processed_time_from_db()

    # Extraction step
    extracted_data = extract_data_since(last_processed_timestamp)

    # Transformation step
    transformed_data = transform(extracted_data)

    # Loading step
    load_transformed_data(transformed_data)

def load_transformed_data(data):
    loaded_rows = []
    for row in data:
        if not is_row_already_loaded(row.id):
            load_to_target_database(row)
            loaded_rows.append(row)
            update_loaded_state(row.id)
    return loaded_rows

# Helper functions
def get_last_processed_time_from_db():
    # Query database to get last processed timestamp
    pass

def extract_data_since(timestamp):
    # Extract data from source since the given timestamp
    pass

def transform(data):
    # Apply transformation logic
    pass

def load_to_target_database(row):
    # Load row into target database
    pass

def is_row_already_loaded(row_id):
    # Check if row has already been loaded
    pass

def update_loaded_state(row_id):
    # Update the state in storage to mark this row as loaded
    pass

#### Closing Summary
and Common Mistakes/Follow-Ups

#### Common Mistakes

Failing to store state information correctly or durably.
Not updating state during each step, leading to incomplete processing.
Overlooking edge cases like partial failures or concurrent runs.

#### Follow-Ups

Discuss how to handle concurrent runs and ensure atomicity.
Explore strategies for handling large datasets efficiently.

#### Cons
ider failover scenarios and recovery mechanisms.

**Comparison:**

| ID | Timestamp           |
|----|---------------------|
| 101| 2023-09-01T18:00:00Z|
| 102| 2023-09-01T19:00:00Z|

**Code:**

```python
# Pseudo-code snippet for resume logic in ETL job
def start_etl_job():
    last_processed_timestamp = get_last_processed_time_from_db()

    extracted_data = extract_data_since(last_processed_timestamp)
    transformed_data = transform(extracted_data)

    loaded_rows = load_transformed_data(transformed_data)
    update_state(loaded_rows)

# Helper functions
def get_last_processed_time_from_db():
    # Query database to get last processed timestamp
    pass

def extract_data_since(timestamp):
    # Extract data from source since the given timestamp
    pass

def transform(data):
    # Apply transformation logic
    pass

def load_to_target_database(row):
    # Load row into target database
    pass

def is_row_already_loaded(row_id):
    # Check if row has already been loaded
    pass

def update_loaded_state(row_id):
    # Update the state in storage to mark this row as loaded
    pass

# Resume logic for processing existing data
def load_transformed_data(data):
    loaded_rows = []
    for row in data:
        if not is_row_already_loaded(row.id):
            load_to_target_database(row)
            loaded_rows.append(row)
            update_loaded_state(row.id)
    return loaded_rows
```

**Example input / output:**

```text
Input:
- Extracted rows with IDs [101, 102, 103] and timestamps ["2023-09-01T14:00:00Z", "2023-09-01T15:00:00Z"]
- Transformed rows with IDs [101, 102] and timestamps ["2023-09-01T16:00:00Z", "2023-09-01T17:00:00Z"]
- Loaded rows with IDs [101, 102] and timestamps ["2023-09-01T18:00:00Z", "2023-09-01T19:00:00Z"]

Output:
- Load only row ID 103 into target database
```

**Data:**

```json
{
  "extracted_rows": {
    "id": [101, 102, 103],
    "timestamp": ["2023-09-01T14:00:00Z", "2023-09-01T15:00:00Z"]
  },
  "transformed_rows": {
    "id": [101, 102],
    "timestamp": ["2023-09-01T16:00:00Z", "2023-09-01T17:00:00Z"]
  },
  "loaded_rows": {
    "id": [101, 102],
    "timestamp": ["2023-09-01T18:00:00Z", "2023-09-01T19:00:00Z"]
  }
}
```

**Flow:**

1. No
2. Yes
3. No
4. Yes
5. Start ETL Job
6. Is State Information Available?
7. Initialize State
8. Extract Data Since LastProcessed Time
9. Transform Extracted Data
10. Load Transformed Data
11. Update Loaded State
12. End ETL Job
13. Load Existing Data
14. Is Row Already Loaded?
15. Load to Target Database &Update State

