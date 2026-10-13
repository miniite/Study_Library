---
type: "practice"
difficulty: "Basic"
---

# How would you write a Python pipeline step that appends a load_timestamp and source_file column to every record for traceability?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Data Reconciliation Scripts](<../../../../TOOLS/Python/Python_Language/Data_Reconciliation_Scripts.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To write a Python pipeline step that appends load_timestamp and source_file columns to every record, we need to understand the basic steps involved in processing data with Python. The goal is to ensure each record has metadata added for traceability purposes.

The question really probes your understanding of how to manipulate records within a loop or using list comprehensions, as well as your ability to handle timestamps and file names effectively. We'll assume we are working with a list of dictionaries representing our data records, although in practice, this could be any iterable structure like a DataFrame from pandas.

Step-by-Step Solution
Understand the Input Data: Assume we have a list of dictionaries, each representing a record.
Add load_timestamp and source_file Columns: Loop through each dictionary and add these two new keys with appropriate values.
Return the Modified List: The function should return the modified list of records.
Full Solution

Here's how we can implement this in Python:

PYTHON
import datetime

def append_traceability_info(records, source_file_path):
    """
    Appends `load_timestamp` and `source_file` to each record for traceability.

    :param records: List of dictionaries representing data records
    :param source_file_path: Path to the source file as a string
    :return: Modified list of dictionaries with added metadata
    """
    # Get current timestamp
    load_timestamp = datetime.datetime.now().isoformat()

    # Loop through each record and append necessary information
    for record in records:
        record['load_timestamp'] = load_timestamp
        record['source_file'] = source_file_path

    return records

# Example usage
data_records = [
    {"id": 1, "name": "Alice"},
    {"id": 2, "name": "Bob"}
]

source_file_path = "/path/to/source/file.csv"

modified_records = append_traceability_info(data_records, source_file_path)
print(modified_records)

#### Closing Summary
and Common Mistakes

This solution ensures that each record in the list now has two additional metadata fields: load_timestamp with the current timestamp when the function was executed, and source_file with the path of the input file. This is crucial for maintaining traceability and auditability.

#### Common mistakes
to avoid include not handling different data structures correctly (e.g., if records are in a DataFrame instead of a list of dictionaries) or forgetting to include the necessary metadata fields.

**Code:**

```python
import datetime

def append_traceability_info(records, source_file_path):
    load_timestamp = datetime.datetime.now().isoformat()

    for record in records:
        record['load_timestamp'] = load_timestamp
        record['source_file'] = source_file_path

    return records
```

**Example input / output:**

```text
data_records = [
    {"id": 1, "name": "Alice"},
    {"id": 2, "name": "Bob"}
]

source_file_path = "/path/to/source/file.csv"

modified_records = append_traceability_info(data_records, source_file_path)
print(modified_records)

# Output:
[
    {'id': 1, 'name': 'Alice', 'load_timestamp': '2023-10-05T14:30:00.123456', 'source_file': '/path/to/source/file.csv'},
    {'id': 2, 'name': 'Bob', 'load_timestamp': '2023-10-05T14:30:00.123456', 'source_file': '/path/to/source/file.csv'}
]
```

**Data:**

```json
[
    {"id": 1, "name": "Alice"},
    {"id": 2, "name": "Bob"}
]
```

