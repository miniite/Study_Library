---
type: "practice"
difficulty: "Intermediate"
---

# Your object-store lake has millions of tiny files from a streaming job and queries have gotten slow. Explain the small-files problem in plain terms and describe the compaction job you'd schedule.

**Difficulty:** Intermediate · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Small Files and Compaction](<../../../../04_Data_Architecture_and_Strategy/Architecture_Patterns/Data_Architecture/Tiered_Lake_Storage/Small_Files_and_Compaction.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Summary

When dealing with an object store that contains millions of tiny files, the "small-files" problem can significantly degrade query performance due to increased metadata overhead and I/O operations. Compaction is a strategy used to reduce the number of small files by merging them into larger ones, thereby improving query efficiency.

#### Triaging the Question

The question asks us to explain the small-files problem and describe a compaction job. The key aspects are:

Problem: Understanding why millions of tiny files slow down queries.

#### Solution
Describing how a compaction job can address this issue by aggregating smaller files into larger ones.

#### First Principles Solution
Problem: Small Files in a Data Lake

In large-scale data processing, especially with streaming jobs, it's common to end up with millions of small files. This happens because each file might contain only a few records or even just one, leading to excessive metadata overhead and increased I/O operations.

Why Does It Matter?

Metadata Overhead: Each file has its own metadata (e.g., file name, size, timestamp), which consumes storage space in the object store.
I/O Operations: Reading millions of small files requires more disk reads, leading to slower query performance and increased latency.
Compaction Job: Merging Small Files

A compaction job is designed to mitigate these issues by consolidating smaller files into larger ones. This process can be done in several ways:

Aggregation Based on Time or Key: Group small files based on time intervals (e.g., hourly, daily) or keys (e.g., user IDs). For example, aggregating data from multiple hourly files into a single file for each day.
Incremental Compaction: This involves merging files in batches as new data comes in, rather than doing a one-time full compaction.

Key Steps:

Identify Small Files: Use metadata to identify files below a certain size threshold.
Determine Aggregation Criteria: Decide how to group these small files (e.g., by time, key).
Merge Files: Write the aggregated data into new, larger files.
Update Metadata: Update any necessary metadata to reflect the changes.

#### Example Scenario

#### Cons
ider a scenario where you have hourly streaming data for millions of users. Initially, each user's data is stored in its own small file. As these files accumulate over days or weeks, they lead to inefficiencies. A compaction job can merge these hourly files into daily files, reducing the number of files from thousands per day to just a few.

#### Implementation
Details
Scheduling: Schedule the compaction job to run periodically (e.g., nightly) to avoid overwhelming processing resources.
Concurrency Control: Ensure that multiple compaction jobs do not conflict with each other by using locks or semaphores.
Data Integrity: Validate the correctness of merged files and handle any potential data loss or corruption.

#### Closing Summary

Compaction is a critical strategy for managing small files in an object store, addressing metadata overhead and I/O inefficiencies. By merging smaller files into larger ones, compaction improves query performance and overall system efficiency.

#### Common Mistakes/Follow-Ups
Failing to Monitor: Not continuously monitoring the state of small files can lead to missed opportunities for compaction.
Overcomplicating Merging Logic: Over-engineering the compaction logic without considering real-world constraints (e.g., concurrency, data integrity) can introduce unnecessary complexity.
Triaging Common Concerns
How Often Should Compaction Be Scheduled?
Typically, compaction is scheduled nightly or during off-peak hours to minimize impact on other processes.
What Metrics Should I Monitor?
Key metrics include the number of small files, query performance, and storage usage before and after compaction.

**Comparison:**

```
File Name	Size
user_1_hourly_data.json	512B
user_2_daily_data.json	1MB
```

**Code:**

```python
# Example Python script for identifying small files
import os

def identify_small_files(directory, size_threshold):
    small_files = []
    for filename in os.listdir(directory):
        file_path = os.path.join(directory, filename)
        if os.path.isfile(file_path) and os.path.getsize(file_path) < size_threshold:
            small_files.append(filename)
    return small_files
```

**Example input / output:**

```text
Input:
- Directory: `/path/to/lake`
- Size Threshold: `1MB`

Output:
- Small Files Identified: `["user_1_hourly_data.json"]`
- Merged File Created: `daily_user_data.json` (containing aggregated data from user_1)
```

**Data:**

```json
{
  "directory": "/path/to/lake",
  "files": [
    {"name": "user_1_hourly_data.json", "size": 512},
    {"name": "user_2_daily_data.json", "size": 1048576}
  ]
}
```

**Flow:**

1. Yes
2. No
3. Identify Small Files
4. Are files < threshold?
5. Merge Files
6. Keep File as Is
7. Create Merged File
8. Continue Monitoring

