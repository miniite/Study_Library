---
type: "practice"
difficulty: "Basic"
---

# What is a counter metric versus a gauge metric, and how would you increment a counter stored in a Python dictionary each time an error is seen?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Time-series Data Basics](<../../../../TOOLS/Python/Python_Language/Time-series_Data_Basics.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In AIOps, metrics are crucial for monitoring system health and performance. Counters and gauges are two fundamental types of metrics used to track different kinds of data over time. A counter metric is a non-negative integer that monotonically increases; it records the number of events or occurrences. In contrast, a gauge metric can represent any numerical value at a specific point in time, such as a current temperature or memory usage.

To increment a counter stored in a Python dictionary each time an error is seen, we need to understand how dictionaries work and ensure that the operation is efficient and correct. Let's break this down step by step.

First, let's define what our data structure looks like. We'll use a dictionary where the key represents the event type or error name, and the value is the counter for that event.

PYTHON
error_counts = {}

Now, every time an error occurs, we want to increment the corresponding counter in this dictionary. Here’s how you can do it:

Check if the error already exists in the dictionary: If it does, increment its count.
If not, initialize the counter with a value of 1.

Here is a simple implementation:

PYTHON
def log_error(error_type):
    # Check if the error type is already in the dictionary
    if error_type in error_counts:
        # Increment the existing count
        error_counts[error_type] += 1
    else:
        # Initialize the counter for this error type to 1
        error_counts[error_type] = 1

# Example usage
log_error("NetworkTimeout")
log_error("DatabaseError")
log_error("NetworkTimeout")

print(error_counts)  # Output: {'NetworkTimeout': 2, 'DatabaseError': 1}

This approach ensures that the counter is always incremented correctly. It uses a dictionary to map error types to their respective counts.

