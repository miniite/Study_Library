---
type: "practice"
difficulty: "Intermediate"
---

# How would you write a Python script that reads a metrics stream line by line and keeps a running maximum without storing all values?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Time-series Data Basics](<../../../../TOOLS/Python/Python_Language/Time-series_Data_Basics.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To solve this problem, we need to design a Python script that can process a continuous stream of metrics (e.g., from a log file or real-time data feed) and maintain an up-to-date maximum value while only storing a limited amount of recent values in memory. This ensures efficient usage of resources without needing to keep all historical data.

The core idea is to use a sliding window approach where we track the maximum value seen so far, but also store a fixed number of recent values to allow for potential backtracking if necessary. Here’s how you can achieve this:

Understand the Question and Assumptions:

The script needs to read metrics line by line.
We need to keep track of the maximum value seen so far.
Memory usage should be optimized, meaning we don't want to store all past values.

#### Solution
Approach:

Use a variable to hold the current maximum value.
Maintain a list or deque (double-ended queue) to store recent values for potential backtracking.
As each new line is read, update the maximum and manage the deque to keep it at a fixed size.

#### First-Principles Solution

Initialize an empty deque and set max_val to negative infinity or any minimum possible value.
For each line in the stream:
Parse the metric value.
Update max_val if the current value is greater than max_val.
Append the current value to the deque.
If the size of the deque exceeds the fixed limit, remove the oldest value from it.

Full Solution:

PYTHON
from collections import deque

def process_metrics(max_window_size):
    max_val = float('-inf')
    recent_values = deque([], maxlen=max_window_size)

    for line in lines:
        metric_value = parse_metric(line)  # Assume this function parses the value

        if metric_value > max_val:
            max_val = metric_value

        recent_values.append(metric_value)

        # Optionally, print or log `max_val` after processing each line

def parse_metric(line):
    # Example parsing logic (assuming values are integers in a string format)
    return int(line.strip())

process_metrics(10)  # Set the max window size as needed

#### Closing Summary
and Common Mistakes:

This solution ensures that we only store recent values, making it memory-efficient.
The use of a deque allows efficient addition and removal of elements from both ends.
A common mistake is not properly managing the deque to maintain its size limit.

