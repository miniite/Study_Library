---
type: "practice"
difficulty: "Intermediate"
---

# Explain how you'd detect and handle a source that starts sending truncated or partial files intermittently.

**Difficulty:** Intermediate · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Handling Source Changes and Spikes](<../../../../02_Data_Engineering/Pipelines_and_Integration/ETL_ELT/Handling_Source_Changes_and_Spikes.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the issue of detecting and handling sources that send truncated or partial files, we need to implement a robust data ingestion strategy that includes both detection mechanisms and fallback plans. This approach will ensure that our ETL pipeline can handle such issues gracefully without compromising on data integrity.

#### Triage

The question is probing how you would design a system to monitor and manage the quality of incoming data streams. Specifically, we need to focus on:

Detecting when files are partially received.
Handling partial files in a way that minimizes data loss or corruption.
Ensuring minimal disruption to our overall ETL processes.

#### First-Principles Solution

First, let's define the problem: We receive log files or other types of flat-file data from various sources. These files can be truncated due to network issues, server crashes, or some other unforeseen circumstances. To handle this, we need a two-pronged approach:

File Verification: Implement checks to verify if a file has been fully received.
Data Handling: Decide how to process the data in case of partial files.
Step 1: File Verification

To detect truncated files, we can use checksums or metadata validation techniques. For instance, you could:

Calculate a checksum (e.g., MD5) of the file before and after transmission.
Validate the size of the file against expected sizes for different types of data.
Check for the presence of specific footer/end-of-file markers.

Here’s an example of how to implement this in Python:

PYTHON
import hashlib

def verify_file_integrity(file_path, expected_checksum):
    with open(file_path, 'rb') as f:
        file_content = f.read()
    actual_checksum = hashlib.md5(file_content).hexdigest()

    if actual_checksum == expected_checksum:
        print("File integrity verified.")
        return True
    else:
        print(f"Checksum mismatch detected: {actual_checksum} != {expected_checksum}")
        return False

# Example usage
file_path = '/path/to/your/file.log'
expected_checksum = 'expected_checksum_value'  # This should be set based on the expected checksum value
verify_file_integrity(file_path, expected_checksum)
Step 2: Handling Partial Files

If a file is detected as truncated or partial:

Log and Notify: Record the error in a log system and send an alert to relevant stakeholders.
Retry Mechanism: If possible, retry fetching the file from the source. This can be done using exponential backoff strategies to avoid overwhelming the server.
Partial Data Processing: Implement logic to process only the available data, ensuring that partial records are still utilized.

#### Closing Summary

In summary, we need a system that not only verifies the integrity of incoming files but also handles cases where data is partially received. By implementing checksums and logging mechanisms, we can ensure that our ETL processes remain robust and resilient to unexpected disruptions in data streams.

#### Common Mistakes
and Follow-Ups
Mistake: Overlooking partial file handling leads to data loss or corruption.

#### Follow-Up
Discuss the implications of different strategies (e.g., retrying vs. ignoring partial files) and how they affect overall system reliability and performance.
Mistake: Not having a proper alert mechanism can lead to unnoticed issues.

#### Follow-Up
Explain how setting up alerts for critical errors can help in maintaining continuous monitoring.

**Code:**

```python
import hashlib

def verify_file_integrity(file_path, expected_checksum):
    with open(file_path, 'rb') as f:
        file_content = f.read()
    actual_checksum = hashlib.md5(file_content).hexdigest()

    if actual_checksum == expected_checksum:
        print("File integrity verified.")
        return True
    else:
        print(f"Checksum mismatch detected: {actual_checksum} != {expected_checksum}")
        return False

# Example usage
file_path = '/path/to/your/file.log'
expected_checksum = 'expected_checksum_value'  # This should be set based on the expected checksum value
verify_file_integrity(file_path, expected_checksum)
```

**Flow:**

1. Yes
2. No
3. Start
4. Is file fully received?
5. Verify checksum and processdata
6. Log error, notify stakeholders,retry or handle partial data
7. Process data

