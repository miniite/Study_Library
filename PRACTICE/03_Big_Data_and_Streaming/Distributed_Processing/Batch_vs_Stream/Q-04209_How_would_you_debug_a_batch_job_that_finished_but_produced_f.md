---
type: "practice"
difficulty: "Intermediate"
---

# How would you debug a batch job that finished but produced fewer rows than expected?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Bad Record Handling](<../../../../03_Big_Data_and_Streaming/Distributed_Processing/Batch_vs_Stream/Bad_Record_Handling.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To debug a batch job that completed but produced fewer rows than expected, we need to first understand what could have gone wrong and then systematically identify the issue. This involves checking input data integrity, processing logic correctness, and the job's execution environment.

#### Triage

The question is asking us to diagnose why our batch job, which should process a full dataset, ended up with fewer rows than expected in the output. The interviewer might be probing into your understanding of data flow, debugging techniques, and how to handle edge cases. We will assume that the input data size has not changed.

Step-by-Step Solution:
Check Input Data Integrity:
Verify if the input dataset is complete.
Confirm that there were no unexpected deletions or modifications during data transfer or storage.
Review Data Flow and Processing Logic:
Examine the ETL (Extract, Transform, Load) pipeline for any logical errors or edge cases.
Ensure that all necessary transformations are applied correctly.
Analyze Job Execution Environment:
Check if there were any resource constraints or failures during execution.
Validate that the job was executed as expected and no part of it was skipped.
Detailed Steps:
Input Data Verification:
Compare the actual input file size and content with the expected data.
Use tools like awk, sed, or a SQL query to inspect the first few rows for any anomalies.
ETL Pipeline Review:
Break down each step of the ETL process:
Extraction: Check if all required files were extracted correctly.
Transformation: Ensure that transformations like filtering, aggregation, or deduplication are applied as intended.
Loading: Verify if data was loaded into the correct target tables or files without issues.
Use logging to trace any errors or warnings during processing.
Execution Environment Check:
Review logs for any error messages or exceptions.
Ensure that the job had sufficient resources and did not encounter timeouts or failures.

#### Closing Summary

By methodically checking each stage of the data flow, we can pinpoint where the discrepancy occurs. This approach ensures a thorough understanding of both the data and the processing logic.

#### Common Mistakes/Follow-Ups
Missing Input Files: Ensure that all required files are present.
Incorrect Transformations: Double-check transformations for any off-by-one errors or logical mistakes.
Resource Limitations: Verify if there were resource constraints during execution.

Would you like to explore any specific tools or techniques further?

**Comparison:**

| ID | Value  |
|----|--------|
| 1  | data1  |
| 2  | data2  |
| 3  | NULL   |

**Code:**

```bash
# Example of comparing file sizes and inspecting data using Python
import os

def check_file_integrity(input_path, expected_size):
    actual_size = os.path.getsize(input_path)
    if actual_size != expected_size:
        print(f"File size mismatch: Expected {expected_size}, got {actual_size}")
```

**Example input / output:**

```text
# Example input and expected output comparison
input_data = [{"id": 1, "value": "data1"}, {"id": 2, "value": "data2"}, {"id": 3, "value": None}]
expected_output = [{"id": 1, "value": "data1"}, {"id": 2, "value": "data2"}]

# Transformations
def transform_data(data):
    # Example transformation: remove null values
    return [d for d in data if d['value'] is not None]

actual_output = transform_data(input_data)

print("Expected Output:", expected_output)
print("Actual Output:", actual_output)
```

**Data:**

```json
{
  "input_data": [
    {"id": 1, "value": "data1"},
    {"id": 2, "value": "data2"},
    {"id": 3, "value": null}
  ],
  "expected_output": [
    {"id": 1, "value": "data1"},
    {"id": 2, "value": "data2"}
  ]
}
```

**Flow:**

1. Input Data Check
2. ETL Process Review
3. Execution Environment Check
4. Output Verification

