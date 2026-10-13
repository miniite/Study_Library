---
type: "practice"
difficulty: "Basic"
---

# How would you extract a numeric severity level from a log's textual level field like INFO, WARN, or ERROR?

**Difficulty:** Basic · **Levels:** Fresher · [↑ Parsing Semi-structured Logs](<../../../../11_AI_and_ML/ML_Foundations/Data_Preprocessing/Parsing_Semi-structured_Logs.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To extract a numeric severity level from a log's textual level field such as "INFO", "WARN", or "ERROR", we need to map these textual labels to corresponding numerical values. For instance, we might assign 1 to INFO, 2 to WARN, and 3 to ERROR. This process involves several steps: identifying the textual fields, mapping them to numeric levels, and ensuring any potential errors are handled gracefully.

The question is essentially asking us to create a function or method that takes a string representing a log level as input and returns an integer value corresponding to its severity. We need to consider edge cases like unknown log levels or unexpected inputs. Here’s how we can approach this from first principles:

Identify the Textual Fields: Recognize common textual values for log levels such as "INFO", "WARN", and "ERROR".
Mapping to Numerical Values: Define a mapping dictionary where each string corresponds to an integer.
Handling Unknown Inputs: Ensure that any input not matching our predefined levels is handled appropriately, either by raising an error or returning a default value.

Let’s go through the solution step-by-step:

Define the Mapping Dictionary:

PYTHON
log_level_mapping = {
    "INFO": 1,
    "WARN": 2,
    "ERROR": 3
}

Create a Function to Extract Severity Level:

PYTHON
def extract_severity(log_level):
    if not isinstance(log_level, str):
        raise ValueError("Log level must be a string")

    # Check if the log level is in our mapping dictionary
    severity = log_level_mapping.get(log_level.upper(), None)
    if severity is None:
        return -1  # Return -1 for unknown levels or handle as needed
    return severity

Test the Function:

PYTHON
test_log_levels = ["INFO", "WARN", "ERROR", "UNKNOWN"]
for log_level in test_log_levels:
    print(f"{log_level} -> {extract_severity(log_level)}")

This approach ensures that we handle known and unknown log levels gracefully. By using a dictionary, the mapping process is straightforward and efficient.

**Code:**

```python
def extract_severity(log_level):
    if not isinstance(log_level, str):
        raise ValueError("Log level must be a string")

    log_level_mapping = {
        "INFO": 1,
        "WARN": 2,
        "ERROR": 3
    }

    severity = log_level_mapping.get(log_level.upper(), -1)
    return severity

test_log_levels = ["INFO", "WARN", "ERROR", "UNKNOWN"]
for log_level in test_log_levels:
    print(f"{log_level} -> {extract_severity(log_level)}")
```

**Example input / output:**

```text
INFO -> 1
WARN -> 2
ERROR -> 3
UNKNOWN -> -1
```

