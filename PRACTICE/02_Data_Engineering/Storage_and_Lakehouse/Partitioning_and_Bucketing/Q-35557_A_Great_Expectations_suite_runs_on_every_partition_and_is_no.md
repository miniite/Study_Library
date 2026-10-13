---
type: "practice"
difficulty: "Intermediate"
---

# A Great Expectations suite runs on every partition and is now the slowest step in the DAG — how do you scope validation to changed partitions?

**Difficulty:** Intermediate · **Levels:** SDE-1/2, SDE-2/3 · [↑ Concurrent Partition Overwrites](<../../../../02_Data_Engineering/Storage_and_Lakehouse/Partitioning_and_Bucketing/Concurrent_Partition_Overwrites.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR

When a Great Expectations suite runs on every partition, it can become a bottleneck if one or more partitions have not been updated since the last run. To optimize this process, we need to scope validation only to those partitions that have changed. This involves identifying which partitions have been modified and then running validations specifically on these changes.

#### Triaging the Question

The core of the question is about reducing unnecessary computation by validating only the relevant partitions in a Great Expectations suite. The interviewer might be probing your understanding of:

Partitioning and Bucketing: How data is organized and stored.
Change Detection: Techniques to identify which partitions have changed since the last run.
Great Expectations Workflow: How it operates on different partitions.
Performance Optimization: Strategies to reduce the time taken by the validation process.

#### Assumptions
We are dealing with a large-scale data pipeline where multiple partitions exist.
Each partition is stored as a distinct directory or database table.
Great Expectations suite runs periodically and needs to be optimized for performance.

#### First Principles Solution
1. Identify Modified Partitions

To determine which partitions have changed, we can use metadata tracking:

Metadata Tracking: Maintain a history of when each partition was last validated.
Change Detection Logic: Compare the timestamps or checksums of recent changes with the stored metadata to identify modified partitions.
PYTHON
import os
from datetime import datetime

# Example metadata file structure
METADATA_DIR = "/path/to/metadata"
PARTITION_DIRS = [f for f in os.listdir(METADATA_DIR) if os.path.isdir(os.path.join(METADATA_DIR, f))]

last_validation_timestamps = {partition: None for partition in PARTITION_DIRS}

def get_last_validation_time(partition):
    metadata_file_path = os.path.join(METADATA_DIR, partition, "last_validation.txt")
    if not os.path.exists(metadata_file_path):
        return None
    with open(metadata_file_path) as f:
        last_validated_at_str = f.read().strip()
        try:
            last_validated_at = datetime.strptime(last_validated_at_str, "%Y-%m-%d %H:%M:%S")
        except ValueError:
            return None  # In case of an invalid format
    return last_validated_at

def identify_modified_partitions(current_time):
    modified_partitions = []
    for partition in PARTITION_DIRS:
        last_validation_time = get_last_validation_time(partition)
        if not last_validation_time or current_time > last_validation_time + timedelta(days=1):  # Example threshold
            modified_partitions.append(partition)
    return modified_partitions

current_time = datetime.now()
modified_partitions = identify_modified_partitions(current_time)

print("Modified partitions:", modified_partitions)
2. Scope Validation to Modified Partitions

Once we have identified the modified partitions, we can update our Great Expectations suite configuration to run validations only on these partitions.

YAML
# Example Great Expectations YAML config for partitioned data
expectations:
  - name: "partition_1_expectation"
    batch_kwargs:
      path: "/path/to/data/partition_1"  # Update this with the modified partition paths
    expectations:
      - expectation_type: expect_column_values_to_be_between
        column: value_col
        min_value: 0
        max_value: 100

# Modify the batch_kwargs for each of the identified partitions
for partition in modified_partitions:
    # Update the YAML or directly set the batch kwargs dynamically
3. Implement Automation and Monitoring
Automate Metadata Updates: Ensure that metadata files are updated whenever a validation run is performed.
Monitoring and Alerts: Set up monitoring to alert on failures or excessive delays.

#### Closing Summary
+ Common Mistakes/Follow-Ups

By focusing the Great Expectations suite on only modified partitions, we can significantly reduce validation time. This approach balances thoroughness with efficiency, ensuring that our pipeline remains performant even as data volumes grow.

#### Common Mistakes

Overlooking Metadata: Failing to maintain proper metadata tracking can lead to unnecessary revalidation of unchanged partitions.
Inefficient Change Detection Logic: Using overly complex or inefficient change detection methods (e.g., full revalidation instead of incremental checks).
Ignoring Performance Impact: Not considering the performance impact of running validations on large datasets, leading to unnecessary computational overhead.

#### Follow-Ups

Discussing how to handle edge cases like data ingestion during validation runs.
Exploring further optimizations such as parallel processing or distributed validation across multiple nodes.

**Code:**

```python
import os
from datetime import datetime, timedelta

def get_last_validation_time(partition):
    metadata_file_path = os.path.join(METADATA_DIR, partition, "last_validation.txt")
    if not os.path.exists(metadata_file_path):
        return None
    with open(metadata_file_path) as f:
        last_validated_at_str = f.read().strip()
        try:
            last_validated_at = datetime.strptime(last_validated_at_str, "%Y-%m-%d %H:%M:%S")
        except ValueError:
            return None  # In case of an invalid format
    return last_validated_at

def identify_modified_partitions(current_time):
    modified_partitions = []
    for partition in PARTITION_DIRS:
        last_validation_time = get_last_validation_time(partition)
        if not last_validation_time or current_time > last_validation_time + timedelta(days=1):  # Example threshold
            modified_partitions.append(partition)
    return modified_partitions

current_time = datetime.now()
modified_partitions = identify_modified_partitions(current_time)

print("Modified partitions:", modified_partitions)
```

**Example input / output:**

```text
# Example input and output for the code snippets provided
current_time = datetime(2023, 10, 5) + timedelta(hours=8)
modified_partitions = identify_modified_partitions(current_time)

print("Modified partitions:", modified_partitions)
# Output:
# Modified partitions: ['partition_1', 'partition_2']
```

**Data:**

```yaml
expectations:
  - name: "partition_1_expectation"
    batch_kwargs:
      path: "/path/to/data/partition_1"  # Update this with the modified partition paths
    expectations:
      - expectation_type: expect_column_values_to_be_between
        column: value_col
        min_value: 0
        max_value: 100

for partition in modified_partitions:
    batch_kwargs = {
        "path": os.path.join("/path/to/data", partition)
    }
```

