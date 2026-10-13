---
type: "practice"
difficulty: "Intermediate"
---

# How would you quarantine bad records in a Python pipeline (write them to a dead-letter file or table) instead of failing the whole batch on one bad row?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Data Reconciliation Scripts](<../../../../TOOLS/Python/Python_Language/Data_Reconciliation_Scripts.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To quarantine bad records in a Python pipeline, we'll ensure that the processing continues even if some rows are invalid. The core idea is to write these bad records into a separate "dead-letter" file or table and log appropriate warnings. This way, our data pipeline remains robust and doesn't fail due to a single bad row.

#### Triage
The question aims at understanding how to handle errors in a Python pipeline without halting the entire batch processing. We need to identify bad records, isolate them, and store them separately for later review or cleanup. Key assumptions include:

The data is ingested from an external source.
The processing involves multiple steps (e.g., transformation, validation).
Errors are logged and stored.

Step-by-step solution: Let's start by defining a function to process each record in the batch. For simplicity, assume we have a CSV file as input and need to validate each row before processing it further.

Define Validation Rules: Determine what constitutes a "bad" record (e.g., missing required fields, invalid data types).
Process Records: Iterate through each record, validate it against the rules.
Handle Bad Records: If a record fails validation, write it to a dead-letter file or table and log an error message.

#### First-Principles Solution
PYTHON
import pandas as pd
from datetime import datetime

def process_records(input_file: str, dead_letter_file: str):
    # Load input data
    df = pd.read_csv(input_file)

    # Define validation rules (example: check if 'age' is a number and within valid range)
    def validate_record(row):
        try:
            age = int(row['age'])
            if 0 <= age < 150:
                return True, row
            else:
                return False, None
        except ValueError:
            return False, None

    # Process each record
    results = []
    for index, row in df.iterrows():
        valid, clean_row = validate_record(row)
        if not valid:
            log_bad_record(index, row, dead_letter_file)
        else:
            results.append(clean_row)

    # Optionally write the processed data back to a file or further process it
    pd.DataFrame(results).to_csv('output.csv', index=False)

def log_bad_record(index: int, bad_row: dict, dead_letter_file: str):
    with open(dead_letter_file, 'a') as dl_file:
        timestamp = datetime.now().strftime('%Y-%m-%d %H:%M:%S')
        message = f"Bad record detected at index {index} | Timestamp: {timestamp}\n"
        for key, value in bad_row.items():
            message += f"{key}: {value}\n"
        dl_file.write(message)

#### Closing Summary
and Common Mistakes
Summary: The solution involves iterating through each record, validating it against predefined rules, and writing any bad records to a separate file. This ensures the pipeline continues processing even when encountering errors.

#### Common Mistakes
Failing the entire batch on one bad row.
Not properly logging or storing bad records for later analysis.
Overcomplicating the validation logic.

**Code:**

```python
import pandas as pd
from datetime import datetime

def process_records(input_file: str, dead_letter_file: str):
    # Load input data
    df = pd.read_csv(input_file)

    # Define validation rules (example: check if 'age' is a number and within valid range)
    def validate_record(row):
        try:
            age = int(row['age'])
            if 0 <= age < 150:
                return True, row
            else:
                return False, None
        except ValueError:
            return False, None

    # Process each record
    results = []
    for index, row in df.iterrows():
        valid, clean_row = validate_record(row)
        if not valid:
            log_bad_record(index, row, dead_letter_file)
        else:
            results.append(clean_row)

    # Optionally write the processed data back to a file or further process it
    pd.DataFrame(results).to_csv('output.csv', index=False)

def log_bad_record(index: int, bad_row: dict, dead_letter_file: str):
    with open(dead_letter_file, 'a') as dl_file:
        timestamp = datetime.now().strftime('%Y-%m-%d %H:%M:%S')
        message = f"Bad record detected at index {index} | Timestamp: {timestamp}\n"
        for key, value in bad_row.items():
            message += f"{key}: {value}\n"
        dl_file.write(message)
```

**Example input / output:**

```text
Input CSV (example.csv):
id,name,age
1,Alice,30
2,Bob,,45
3,Charlie,-5

Output:
Bad record detected at index 2 | Timestamp: 2023-10-17 12:00:00
name: Bob
age:

Processed output (output.csv):
id,name,age
1,Alice,30
```

**Flow:**

1. valid
2. invalid
3. Read CSV
4. Iterate over rows
5. Validate row
6. Process row
7. Log bad record
8. Write to dead-letter file
9. Continue processing

