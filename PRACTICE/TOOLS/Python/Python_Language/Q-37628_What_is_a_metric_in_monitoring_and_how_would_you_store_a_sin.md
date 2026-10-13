---
type: "practice"
difficulty: "Basic"
---

# What is a metric in monitoring, and how would you store a single time-series sample as a Python tuple of (timestamp value labels)?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Time-series Data Basics](<../../../../TOOLS/Python/Python_Language/Time-series_Data_Basics.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In AIOps, a metric represents quantitative information about system or application behavior over time, such as CPU usage or disk space. When monitoring systems, it's crucial to store these metrics efficiently and accurately. A single time-series sample can be stored using a Python tuple that encapsulates the essential components of the data point: timestamp, value, and labels.

To store a single time-series sample as a Python tuple, you need to break down the task into its fundamental parts:

Timestamp: This is the exact moment in time when the metric was recorded.
Value: The actual numerical or categorical measurement at that specific point in time.
Labels: Additional metadata that provides context about the sample, such as environment details, service names, etc.

Here’s how you can structure this tuple:

PYTHON
sample = (timestamp, value, labels)
Step-by-Step Solution

Identify Timestamp:

The timestamp should be precise and unambiguous.
You could use Python's datetime module to get the current time or a specific point in time.

Determine Value:

The value can be an integer, float, or string depending on the metric type (e.g., CPU usage as a percentage).
Ensure the value is appropriately formatted and relevant to the context.

Define Labels:

Labels are optional but highly recommended for additional context.

#### Example
s include environment (development, staging, production), service name, or any other relevant metadata.

#### Example

Let's say you want to record a CPU usage sample at 15:30 on February 12, 2024. The value is 75% and the labels are {"environment": "production", "service": "web_server"}. Here’s how you can construct the tuple:

PYTHON
from datetime import datetime

# Example timestamp
timestamp = datetime(2024, 2, 12, 15, 30)

# Value of CPU usage in percentage
value = 75.0

# Labels providing context
labels = {"environment": "production", "service": "web_server"}

# Construct the tuple
sample = (timestamp, value, labels)

#### Closing Summary
and Common Mistakes

#### Common Mistakes
Forgetting to include all necessary components in the tuple.
Not using a robust timestamp format.
Overlooking the importance of adding relevant labels for context.

By following this structure, you can effectively store and manage time-series data in Python, which is essential for monitoring systems in AIOps environments. Always ensure that your tuples are well-defined to maintain clarity and ease of use when processing or analyzing large datasets.

