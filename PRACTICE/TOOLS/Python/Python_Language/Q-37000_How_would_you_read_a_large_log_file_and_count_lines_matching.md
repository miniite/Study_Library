---
type: "practice"
difficulty: "Intermediate"
---

# How would you read a large log file and count lines matching multiple patterns in a single pass instead of reading it multiple times?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Log Filtering Scripts](<../../../../TOOLS/Python/Python_Language/Log_Filtering_Scripts.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To solve this problem, we need to write a Python script that reads a large log file only once and counts lines that match multiple specified patterns. The key idea is to use regular expressions (regex) in combination with a single pass through the file.

The question is probing your ability to handle large files efficiently by minimizing I/O operations and using regex for pattern matching. We'll assume you have access to a list of patterns, which we will define as patterns = ['ERROR', 'WARNING'].

Here's the step-by-step solution:

Import necessary libraries: We need the re library for regular expression operations.
Define your patterns: A list or set of regex patterns that you want to match against each line in the file.
Initialize counters for each pattern.
Read the log file once and process each line: Use a generator function if the file is extremely large to avoid loading the whole file into memory.

Let's break down the solution:

Import necessary libraries:

PYTHON
import re

Define your patterns:

PYTHON
patterns = ['ERROR', 'WARNING']
regex_patterns = [re.compile(pattern) for pattern in patterns]

Initialize counters:

PYTHON
counts = {pattern: 0 for pattern in patterns}

Read the log file and process each line:

PYTHON
with open('large_log_file.log', 'r') as file:
    for line in file:
        # Check if any of the patterns match the current line
        for regex_pattern, count in zip(regex_patterns, counts.values()):
            if regex_pattern.search(line):
                count += 1

print(counts)

This approach ensures that we read the log file only once while still being able to count lines matching multiple patterns.

