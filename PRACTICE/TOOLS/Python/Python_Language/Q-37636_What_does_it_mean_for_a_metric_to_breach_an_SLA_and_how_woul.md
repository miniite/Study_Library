---
type: "practice"
difficulty: "Basic"
---

# What does it mean for a metric to breach an SLA, and how would you write Python code that flags any response time above a 500ms target in a list?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ OpenTelemetry Instrumentation](<../../../../TOOLS/Python/Python_Language/OpenTelemetry_Instrumentation.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To understand what it means for a metric to breach an SLA (Service Level Agreement) in the context of AIOps, let's first clarify that an SLA is a contract between a service provider and a customer, defining performance expectations such as response times. When a metric breaches this agreement, it means that the observed value has exceeded the agreed-upon threshold for acceptable performance. In our scenario, we are dealing with response time metrics.

#### Triage

The question asks us to write Python code that flags any response time above 500 milliseconds (ms) in a list of such times. The key elements here include:

Understanding SLA Breach: Identifying when the metric exceeds the predefined threshold.
Coding Logic: Implementing this logic in Python.

#### First-Principles Solution

We need to iterate through each response time in the list and check if it breaches the 500ms target. If a response time is greater than 500ms, we will flag it (for example, by adding a True value at that position).

Here's how you can achieve this:

Define the SLA threshold: Set a variable for the 500ms target.
Iterate through the list of response times and compare each one to the threshold.
Flagging: Use an output list or modify the input list (if allowed) to mark these instances.
Step-by-Step Solution

Let's write the Python code for this:

PYTHON
def flag_breach(response_times, target=500):
    """
    Flags response times above a specified threshold.

    :param response_times: List of response times in milliseconds.
    :param target: The SLA threshold in ms (default is 500ms).
    :return: A list where each element indicates if the corresponding response time breaches the SLA.
    """
    # Initialize an empty list to store flags
    flags = []

    for rt in response_times:
        if rt > target:
            flags.append(True)  # Flagging a breach
        else:
            flags.append(False)

    return flags

# Example input data
response_times = [300, 600, 450, 800, 200]

# Calling the function with default target (500ms)
flags = flag_breach(response_times)

print(flags)  # Output: [False, True, False, True, False]

#### Closing Summary
and Common Mistakes

The code successfully flags any response time above 500ms. A common mistake could be forgetting to initialize the flags list or incorrectly handling the comparison logic.

#### Common Mistakes/Follow-Ups
Incorrect Threshold Value: Ensure you set the correct threshold.
List Initialization: Make sure your flags list is properly initialized and has the same length as response_times.
Edge Cases: Consider empty lists or very short response time arrays in edge cases.

**Code:**

```python
def flag_breach(response_times, target=500):
    """
    Flags response times above a specified threshold.

    :param response_times: List of response times in milliseconds.
    :param target: The SLA threshold in ms (default is 500ms).
    :return: A list where each element indicates if the corresponding response time breaches the SLA.
    """
    # Initialize an empty list to store flags
    flags = []

    for rt in response_times:
        if rt > target:
            flags.append(True)  # Flagging a breach
        else:
            flags.append(False)

    return flags

# Example input data
response_times = [300, 600, 450, 800, 200]

# Calling the function with default target (500ms)
flags = flag_breach(response_times)

print(flags)  # Output: [False, True, False, True, False]
```

**Example input / output:**

```

Input:

PYTHON
Copy
response_times = [300, 600, 450, 800, 200]

Output:

PYTHON
Copy
[False, True, False, True, False]
```

**Data:**

```json
{
    "response_times": [300, 600, 450, 800, 200],
    "flags": [false, true, false, true, false]
}
```

