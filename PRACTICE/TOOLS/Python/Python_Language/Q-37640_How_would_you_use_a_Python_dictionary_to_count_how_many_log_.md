---
type: "practice"
difficulty: "Basic"
---

# How would you use a Python dictionary to count how many log lines fall under each severity level (INFO, WARN, ERROR) in a log file?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Log Filtering Scripts](<../../../../TOOLS/Python/Python_Language/Log_Filtering_Scripts.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To solve this problem, we'll leverage Python dictionaries to categorize and count log entries based on their severity levels—INFO, WARN, and ERROR. The key steps involve reading the log file line by line, checking each entry’s severity level, and updating a dictionary that keeps track of the counts for each severity.

First, let's break down what we're probing:

Reading from a File: We need to understand how to open and read a file in Python.
Dictionary Usage: Dictionaries are used to store key-value pairs, where keys represent severity levels and values are their respective counts.
Handling Edge Cases: This includes checking if the log entries are well-formed and dealing with potential errors.

Now, let's walk through the implementation step by step:

Open and Read the File:
We'll use Python's open() function to read from a file. The mode 'r' will be used for reading.
Initialize a Dictionary:
A dictionary named severity_counts will store the counts of each severity level.
Iterate Over Each Line and Count:
For each line, we'll check if it matches one of the severity levels (INFO, WARN, ERROR) and update the count accordingly.

Here’s the full implementation:

PYTHON
# Initialize a dictionary to hold counts for each severity level
severity_counts = {'INFO': 0, 'WARN': 0, 'ERROR': 0}

# Open and read the log file
with open('logs.txt', 'r') as file:
    # Iterate over each line in the file
    for line in file:
        # Check if the severity is one of the expected levels
        if 'INFO' in line:
            severity_counts['INFO'] += 1
        elif 'WARN' in line:
            severity_counts['WARN'] += 1
        elif 'ERROR' in line:
            severity_counts['ERROR'] += 1

# Print the final counts for each severity level
for severity, count in severity_counts.items():
    print(f'{severity}: {count}')

This code snippet assumes that log entries contain the words "INFO", "WARN", or "ERROR" as indicators of their respective severity levels. It also uses a simple with statement to ensure the file is properly closed after reading.

**Code:**

```python
# Initialize a dictionary to hold counts for each severity level
severity_counts = {'INFO': 0, 'WARN': 0, 'ERROR': 0}

# Open and read the log file
with open('logs.txt', 'r') as file:
    # Iterate over each line in the file
    for line in file:
        # Check if the severity is one of the expected levels
        if 'INFO' in line:
            severity_counts['INFO'] += 1
        elif 'WARN' in line:
            severity_counts['WARN'] += 1
        elif 'ERROR' in line:
            severity_counts['ERROR'] += 1

# Print the final counts for each severity level
for severity, count in severity_counts.items():
    print(f'{severity}: {count}')
```

**Example input / output:**

```plaintext
INFO: 547
WARN: 32
ERROR: 10

This output indicates that there were 547 INFO messages, 32 WARN messages, and 10 ERROR messages in the log file.
```

