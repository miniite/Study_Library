---
type: "practice"
difficulty: "Intermediate"
---

# Your daily pipeline sometimes runs on a partition that only half-landed and produces a low count. How do you make the job detect completeness and refuse to run on incomplete data?

**Difficulty:** Intermediate · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ End to End No Data Loss](<../../../../04_Data_Architecture_and_Strategy/Architecture_Patterns/Data_Architecture/Tiered_Lake_Storage/End_to_End_No_Data_Loss.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To ensure our pipeline detects incomplete data partitions and refuses to process them, we need to implement a robust mechanism that verifies the integrity of the data before kicking off any further processing tasks. This involves checking if all expected data files have arrived in the designated landing zone or raw storage area. If not, the job should be flagged as incomplete and skipped until the full set is available.

#### Triaging the Question

The core issue here is ensuring that our ETL (Extract, Transform, Load) pipeline only processes fully landed data partitions to avoid partial or inconsistent processing. The interviewer is probing your understanding of data quality checks, control flow in pipelines, and how to handle incomplete data gracefully. We'll assume that the data lands in a known directory structure and use common file-naming conventions.

#### First-Principles Solution
Identify Data Files: Define a naming convention for files based on partitions (e.g., year=2023/month=04/day=15).
Count Expected Files: Calculate the expected number of files per partition.
Verify Completeness: Compare the actual count of files to the expected count.
Conditional Execution: Refuse to run any downstream processing if the data is incomplete.
Detailed Steps

Define File Naming Convention

Use a consistent naming format like partition-year-month-day.csv.

Calculate Expected File Count

For daily partitions, this could be based on the number of files expected for a given day.
For example, if we expect 30 log files per day, then any partition with less than 30 files is incomplete.

Check File Existence and Count

Use a script to list all files in the landing zone directory.
Filter by the partition name to get the relevant files.
Compare this count against the expected number of files.

Conditional Execution Based on Completeness

If the file count matches or exceeds the expected value, proceed with processing.
Otherwise, log a warning and skip further steps until all files are present.

#### Example
Implementation

Here's an example Python script to achieve this:

PYTHON
import os
from datetime import datetime

# Define partition directory path
PARTITION_DIR = "/path/to/landing_zone"

def count_files(partition):
    # Full path of the partition folder
    partition_path = f"{PARTITION_DIR}/{partition}"

    # List files in the partition
    files = [f for f in os.listdir(partition_path) if os.path.isfile(os.path.join(partition_path, f))]

    return len(files)

def is_partition_complete(partition):
    expected_count = 30  # Example: expect 30 log files per day

    actual_count = count_files(partition)

    return actual_count == expected_count

# Example usage
partition_name = "2023-10-05"
if is_partition_complete(partition_name):
    print(f"Partition {partition_name} is complete, proceeding with processing.")
else:
    print(f"Warning: Partition {partition_name} is incomplete. Skipping processing.")

#### Closing Summary
and Common Mistakes

To ensure the pipeline runs smoothly and avoids partial or inconsistent data processing:

Ensure Consistent File Naming: Use a reliable naming convention to identify partitions.
Use Conditional Logic: Implement checks to verify file counts before starting any downstream processes.
Automate Logging: Log warnings when incomplete data is detected, which helps in diagnosing issues.

#### Common mistakes
include not having robust checks for data completeness or skipping necessary validation steps. Always ensure that the pipeline can handle and recover from incomplete data gracefully.

Self-Check

This solution covers all key aspects of detecting and handling incomplete partitions effectively.

