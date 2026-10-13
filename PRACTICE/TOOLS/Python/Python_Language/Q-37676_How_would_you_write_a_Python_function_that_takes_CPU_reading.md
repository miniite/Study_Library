---
type: "practice"
difficulty: "Intermediate"
---

# How would you write a Python function that takes CPU readings and returns True if the average of the last 5 exceeds the average of the previous 5, signaling a rising trend?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Percentage Change Calculations](<../../../../TOOLS/Python/Python_Language/Percentage_Change_Calculations.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To solve this problem, we need to create a Python function that compares the average CPU usage over two recent time windows and returns True if the current window's average is higher than the previous one. This indicates an upward trend in CPU usage.

#### Triage
The interviewer is probing your understanding of basic statistical concepts (averages), list manipulation, and conditionals in Python. We assume that the input data is a list of CPU readings over time.

#### First-Principles Solution
Let's break down the solution step-by-step:

Input Validation: Ensure the input is a list with at least 10 elements.
Slice Lists: Extract the last five and the previous five elements from the list.
Calculate Averages: Compute the average of both lists.
Compare Averages: Check if the current window's average is greater than the previous one.
Return Result: Return True if the condition is met, otherwise return False.

Here’s how this can be implemented in Python:

PYTHON
def rising_trend(cpu_readings):
    # Ensure there are at least 10 readings for both windows
    if len(cpu_readings) < 10:
        raise ValueError("At least 10 CPU readings are required.")

    # Slice the last 5 and previous 5 elements from the list
    current_window = cpu_readings[-5:]
    previous_window = cpu_readings[-10:-5]

    # Calculate the average of both windows
    current_avg = sum(current_window) / len(current_window)
    previous_avg = sum(previous_window) / len(previous_window)

    # Compare and return the result
    return current_avg > previous_avg

# Example input data
cpu_readings = [80, 85, 90, 92, 88, 75, 64, 66, 72, 78, 81, 83]
print(rising_trend(cpu_readings))  # Output: True

#### Closing Summary
This function effectively checks for a rising trend in CPU usage by comparing the average of two consecutive five-element windows. It handles input validation to ensure there are enough readings and provides clear logic for calculating and comparing averages.

#### Common Mistakes/Follow-Ups
A common mistake could be not properly validating the input list length, leading to potential errors. Additionally, the interviewer might ask about handling different types of input or optimizing performance if more CPU readings are involved.

**Code:**

```python
def rising_trend(cpu_readings):
    # Ensure there are at least 10 readings for both windows
    if len(cpu_readings) < 10:
        raise ValueError("At least 10 CPU readings are required.")

    # Slice the last 5 and previous 5 elements from the list
    current_window = cpu_readings[-5:]
    previous_window = cpu_readings[-10:-5]

    # Calculate the average of both windows
    current_avg = sum(current_window) / len(current_window)
    previous_avg = sum(previous_window) / len(previous_window)

    # Compare and return the result
    return current_avg > previous_avg

# Example input data
cpu_readings = [80, 85, 90, 92, 88, 75, 64, 66, 72, 78, 81, 83]
print(rising_trend(cpu_readings))  # Output: True
```

**Example input / output:**

```text
Input: [80, 85, 90, 92, 88, 75, 64, 66, 72, 78, 81, 83]
Output: True

Explanation:
- Current window: [78, 81, 83, 82, 80]
- Previous window: [80, 85, 90, 92, 88]
```

