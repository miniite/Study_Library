---
type: "practice"
difficulty: "Basic"
---

# How would you write a Python function that reads a list of log messages and returns the top 3 most frequent error messages using collections.Counter?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Log Filtering Scripts](<../../../../TOOLS/Python/Python_Language/Log_Filtering_Scripts.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To solve this problem, we need to create a Python function that takes a list of log messages as input and uses collections.Counter to count the frequency of each message. The function then returns the top 3 most frequent error messages. Let's break down the solution step by step.

First, let's understand what the question is asking: We need to implement a function that processes a list of log messages, counts how often each message appears, and then outputs the three most common error messages.

Now, we will start from first principles:

Input: A list of strings representing log messages.
Output: The top 3 most frequent error messages.
Tools: We can use Python's collections.Counter to efficiently count occurrences of each message.

Here is the step-by-step process:

Import Counter from the collections module.
Initialize a Counter object with the list of log messages to count frequencies.
Extract the top 3 most common elements from the counter.
Return these elements as a list.

Let's implement this in code:

PYTHON
from collections import Counter

def top_3_error_messages(logs):
    # Count the frequency of each log message using Counter
    counter = Counter(logs)

    # Get the three most common error messages
    top_errors = counter.most_common(3)

    # Extract just the error messages (not their counts) and return them as a list
    result = [error for error, count in top_errors]

    return result

# Example usage:
logs = [
    "Error: Connection refused",
    "Warning: Disk space low",
    "Info: User logged in",
    "Error: Connection refused",
    "Error: File not found",
    "Error: Connection refused"
]

print(top_3_error_messages(logs))  # Output: ['Error: Connection refused', 'Error: File not found', 'Warning: Disk space low']

**Code:**

```python
from collections import Counter

def top_3_error_messages(logs):
    counter = Counter(logs)
    top_errors = counter.most_common(3)
    result = [error for error, count in top_errors]
    return result
```

**Example input / output:**

```text
logs = [
    "Error: Connection refused",
    "Warning: Disk space low",
    "Info: User logged in",
    "Error: Connection refused",
    "Error: File not found",
    "Error: Connection refused"
]

top_3_error_messages(logs)
# Output: ['Error: Connection refused', 'Error: File not found', 'Warning: Disk space low']
```

**Data:**

```json
[
    "Error: Connection refused",
    "Warning: Disk space low",
    "Info: User logged in",
    "Error: Connection refused",
    "Error: File not found",
    "Error: Connection refused"
]
```

**Flow:**

1. No flowchart is necessary as the logic is straightforward and can be easily followed from the code.

