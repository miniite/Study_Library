---
type: "practice"
difficulty: "Deep"
---

# Your batch job writes millions of tiny files to the lake, wrecking downstream read performance. How do you fix output file sizing?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Bad Record Handling](<../../../../03_Big_Data_and_Streaming/Distributed_Processing/Batch_vs_Stream/Bad_Record_Handling.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When dealing with a scenario where a batch job is generating an excessive number of small files in a data lake, it significantly impacts downstream processing efficiency. The core issue here is optimizing the file size to balance storage and performance trade-offs without overwhelming downstream systems.

Firstly, let's clarify what we are trying to achieve: we need to reduce the number of small files while ensuring that the overall throughput and reliability of the batch job are not compromised. This involves understanding the nature of your data ingestion process and the requirements for downstream processing.

#### Triage

The question is essentially asking how to optimize file size in a batch processing system to improve read performance without sacrificing the integrity or usability of the output. The interviewer might be probing on understanding the principles of data partitioning, storage optimization, and potential trade-offs between file size and read performance.

#### First-Principles Solution

To address this issue, we need to balance two primary factors: the number of files and their size. Smaller files can lead to better parallel processing but more overhead in terms of metadata management and I/O operations. Conversely, larger files reduce overhead but might not be ideal for certain types of downstream processing.

Determine Optimal File Size:

Analyze the data characteristics: For instance, if the data is highly variable or has a wide range of values, smaller file sizes can help ensure that each file contains a manageable number of records.

File Naming and Grouping Strategy:

Implement consistent naming conventions to group files logically for easier downstream processing.
Use a rolling window strategy where you batch data into files over time, ensuring that the number of files generated within a given period is controlled.

Use Data Partitioning Techniques:

If possible, partition your data based on specific criteria (e.g., date ranges, geographic regions) to reduce file sizes and improve query performance.

Optimize Write Operations:

Ensure that the batch job writes files in bulk to minimize overhead. This can be achieved by optimizing the write logic to buffer data and flush it in larger chunks.

#### Closing Summary

By strategically managing file size, we can significantly enhance downstream read performance while maintaining the integrity of our data processing pipeline. Key strategies include determining an optimal file size based on data characteristics, implementing effective naming conventions, using partitioning techniques, and optimizing write operations.

#### Common Mistakes/Follow-Ups
Overpartitioning: Generating too many small files can negate the benefits of parallel processing.
Underestimating Metadata Overhead: Smaller files come with increased overhead in managing metadata.
Ignoring Data Characteristics: Failing to consider the specific nature and variability of data can lead to suboptimal file size decisions.
Practical Example

Suppose we have a batch job that processes customer transaction data. If transactions are generated every minute, we could decide to group them into files representing one hour of activity each. This would result in fewer files (e.g., 24 files per day) while still allowing for efficient parallel processing during the read phase.

**Comparison:**

```
2023-10-04
2023-10-04
2023-10-04
2023-10-04
2023-10-04
2023-10-04
2023-10-04
2023-10-04
2023-10-04
write_to_files
write_to_files
Transaction Processing
File Writing Strategy Example
```

**Code:**

```python
# Example Python code snippet to illustrate file writing strategy

import os
from datetime import datetime, timedelta

def write_to_files(transactions, file_prefix='transactions_'):
    # Define a time window (e.g., one hour)
    window_size = timedelta(hours=1)

    current_window_start = datetime.now() - timedelta(minutes=5)  # Start slightly earlier to catch the end of the window

    while transactions:
        current_window_end = min(current_window_start + window_size, datetime.now())

        file_name = f"{file_prefix}{current_window_start.strftime('%Y%m%d_%H')}.csv"
        with open(file_name, 'w') as file:
            for transaction in transactions:
                if current_window_start <= transaction['timestamp'] < current_window_end:
                    file.write(f"{transaction}\n")

        # Remove processed transactions
        transactions = [t for t in transactions if not (current_window_start <= t['timestamp'] < current_window_end)]

        # Move to the next window
        current_window_start += window_size

# Example usage
transactions = [
    {'timestamp': datetime.now() - timedelta(minutes=10), 'amount': 50},
    {'timestamp': datetime.now() - timedelta(minutes=20), 'amount': 75}
]
write_to_files(transactions)
```

**Example input / output:**

```text
Input:
[
    {"timestamp": "2023-10-04T10:00:00Z", "amount": 50},
    {"timestamp": "2023-10-04T10:10:00Z", "amount": 75}
]

Output:
transactions_20231004_10.csv
```

**Data:**

```json
[
    {
        "timestamp": "2023-10-04T10:00:00Z",
        "amount": 50
    },
    {
        "timestamp": "2023-10-04T10:10:00Z",
        "amount": 75
    }
]
```

**Flow:**

1. Yes
2. No
3. Start
4. Is Transaction Within CurrentWindow?
5. Write to File
6. Move to Next Time Window
7. Remove ProcessedTransactions
8. End

