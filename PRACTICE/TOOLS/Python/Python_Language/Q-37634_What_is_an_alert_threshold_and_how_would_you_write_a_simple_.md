---
type: "practice"
difficulty: "Basic"
---

# What is an alert threshold, and how would you write a simple Python if-statement that prints an alert when CPU usage crosses 90 percent?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Flapping Detection](<../../../../TOOLS/Python/Python_Language/Flapping_Detection.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

An alert threshold in AIOps refers to a specific value at which a system triggers an alarm or notification, indicating that the system is experiencing performance issues beyond normal operation. For this scenario, we are focusing on CPU usage crossing 90 percent as our alert threshold.

To write a simple Python if-statement that prints an alert when CPU usage crosses 90 percent, follow these steps:

Understand the Problem: We need to check the CPU usage and trigger a message if it exceeds 90 percent.

#### Assumptions
The value of CPU usage is provided as input.
The threshold is fixed at 90 percent.

Here's how we can implement this in Python:

PYTHON
# Example CPU usage percentage (you would get this from monitoring tools)
cpu_usage = 85

# Alert threshold
threshold = 90

# Check if the CPU usage crosses the alert threshold
if cpu_usage > threshold:
    print("Alert: CPU usage has crossed 90 percent!")

This code snippet checks whether the cpu_usage variable is greater than the threshold value. If it is, it prints a message indicating that an alert should be triggered.

#### Closing Summary
and Common Mistakes

In this simple example, we've covered how to use Python's basic conditional logic to trigger alerts based on CPU usage. However, in real-world scenarios, you might need to consider more complex conditions, such as averaging CPU usage over time, setting different thresholds for different types of services, or integrating with monitoring tools that provide actual CPU usage data.

#### Common Mistakes
Hardcoding the Threshold: Always ensure that any hardcoded values are configurable through environment variables or configuration files.
Lack of Context: Not considering real-time data sources and using static values can lead to false alerts or missed issues.
Ignoring Edge Cases: For example, if cpu_usage is less than 0 or greater than 100, the code should handle such cases gracefully.

**Comparison:**

```
Input Variable	Value
cpu_usage	85
threshold	90
Output Action	Description
No Alert	When cpu_usage <= 90
Alert	When cpu_usage > 90
```

**Code:**

```python
# Example CPU usage percentage (you would get this from monitoring tools)
cpu_usage = 85

# Alert threshold
threshold = 90

# Check if the CPU usage crosses the alert threshold
if cpu_usage > threshold:
    print("Alert: CPU usage has crossed 90 percent!")
```

**Example input / output:**

```text
- **Input**: `cpu_usage = 85`, `threshold = 90`
- **Output**: No alert is printed.
- **Input**: `cpu_usage = 91`, `threshold = 90`
- **Output**: `Alert: CPU usage has crossed 90 percent!` is printed.
```

**Data:**

```json
{
  "cpu_usage": 85,
  "threshold": 90
}
```

**Flow:**

1. No
2. Get CPU Usage
3. CPU Usage >= Threshold?
4. Do Nothing

