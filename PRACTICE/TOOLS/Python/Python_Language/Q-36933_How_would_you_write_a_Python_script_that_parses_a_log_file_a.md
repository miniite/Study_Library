---
type: "practice"
difficulty: "Basic"
---

# How would you write a Python script that parses a log file and counts how many lines contain the word 'ERROR'?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Log Filtering Scripts](<../../../../TOOLS/Python/Python_Language/Log_Filtering_Scripts.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To tackle this problem, we need to create a Python script that reads a log file line by line, checks if each line contains the word 'ERROR', and keeps a count of such lines. The core idea is to leverage basic string operations and file handling in Python.

The question is probing your understanding of file I/O operations in Python, and how to use conditional statements for filtering content. We will assume that the log file is well-formatted and contains lines with words separated by spaces.

Let's break down the steps:

Open the file for reading.
Read each line from the file.
Check if 'ERROR' is present in the line.
Increment a counter if 'ERROR' is found.
Close the file after processing all lines.

Here’s how we can implement this:

**Code:**

```python
# Open the log file for reading
with open('log_file.txt', 'r') as file:
    error_count = 0  # Initialize counter

    # Read each line from the file
    for line in file:
        # Check if 'ERROR' is present in the line
        if 'ERROR' in line.split():
            error_count += 1

# Output the total count of lines containing 'ERROR'
print(f"Number of lines with 'ERROR': {error_count}")
```

**Example input / output:**

```

Sample Input (log_file.txt):

CODE
Copy
2023-10-01 14:30:27 INFO - Starting the process
2023-10-01 15:05:12 ERROR - Issue with database connection
2023-10-01 16:10:45 INFO - Process completed successfully
2023-10-01 17:22:58 ERROR - Network timeout detected

Sample Output:

CODE
Copy
Number of lines with 'ERROR': 2
```

