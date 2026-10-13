---
type: "practice"
difficulty: "Deep"
---

# Your batch job succeeds but downstream consumers occasionally read partial output. How do you make the batch write atomic to consumers?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Bad Record Handling](<../../../../03_Big_Data_and_Streaming/Distributed_Processing/Batch_vs_Stream/Bad_Record_Handling.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To ensure that a batch job writes atomic outputs to downstream consumers, we need to understand the core issue: partial data consumption by consumers due to potential delays or race conditions in the processing pipeline. This problem arises when the batch job completes and writes its output, but some downstream components are still consuming from earlier states before fully processing the new output.

To address this, let's break down the key aspects:

Understand the Problem: Batch jobs typically write data to a storage system (e.g., HDFS, S3), which is then consumed by other systems in smaller batches or streams.
Identify Atomicity Requirements: For the batch job to be considered atomic to consumers, it should ensure that once an output file is written, it is fully processed and not partially read by downstream systems until it can guarantee full consumption.

#### Cons
ider Data Consistency Models: We need to choose a data consistency model that suits our use case, such as Strong Consistency or Eventual Consistency.

Let's reason through the solution from first principles:

1. Define the Problem
Batch Job Flow:
Batch job writes output files.
Downstream consumers read these files in smaller batches or streams.
Issue: Partial reads by downstream consumers due to delays in processing.
2. Ensure Atomic Writes

To ensure atomicity, we need a mechanism that ensures the entire output file is written and fully processed before any consumer starts reading from it. This can be achieved using a combination of techniques:

Step-by-Step Solution

Use Temporary Files:

Write output to temporary files.
Once all data is successfully written, rename or move these temporary files to the final destination.

File System Notifications:

Use file system notifications (e.g., inotify on Linux) to detect when a file has been fully written and notify downstream systems.

Metadata Management:

Maintain metadata about completed batches in a database or distributed storage.

#### Cons
umers can check this metadata before reading new output files, ensuring they only process fully written data.

Locking Mechanisms:

Implement locking mechanisms to prevent concurrent writes and ensure atomicity.
Use semaphores or distributed locks (e.g., ZooKeeper) to manage file access.

Changelog Approach:

For stream-based consumers, use a changelog approach where the batch job appends new data to an existing file instead of overwriting it.

#### Cons
umers can then process only the changes since the last known checkpoint.
3. Implementing Atomic Writes

Here’s how we could implement atomic writes using Python and HDFS as an example:

PYTHON
import os
from hdfs import InsecureClient

def write_batch_output(data, output_path):
    # Step 1: Define temporary file path
    temp_file_path = f"{output_path}.tmp"

    # Step 2: Write data to temporary file
    with open(temp_file_path, 'w') as temp_file:
        temp_file.write(data)

    # Step 3: Rename/Move temporary file to final output path
    client = InsecureClient('http://localhost:50070', user='hadoop')
    with client.write(output_path, overwrite=True) as writer:
        os.replace(temp_file_path, writer)

# Example usage
output_data = "This is the batch output data."
write_batch_output(output_data, "/user/hadoop/batch_output.txt")
4. Close Summary and Common Mistakes

By implementing these steps, we ensure that the batch job writes atomic outputs, reducing the likelihood of partial reads by downstream consumers. Common mistakes include:

Forgetting to handle temporary files: Not cleaning up or managing temporary files properly can lead to conflicts.
Improper locking mechanisms: Inadequate locking can cause race conditions and data corruption.
Lack of metadata management: Without proper metadata, it’s hard for consumers to know when new data is fully available.

