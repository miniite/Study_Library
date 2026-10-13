---
type: "practice"
difficulty: "Basic"
---

# How would you write a Python script that reads a list of service response times and returns how many of them violated a 200ms SLO?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Comparing Time Windows](<../../../../TOOLS/Python/Python_Language/Comparing_Time_Windows.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address this problem, we need to create a simple Python script that takes a list of service response times as input and counts how many of these times violate a Service Level Objective (SLO) threshold of 200 milliseconds (ms). The core idea is straightforward: iterate through the list, compare each value against the SLO, and increment a counter if it exceeds the threshold.

First, let's triage the question:

The script needs to process a list of response times.
It must count how many response times exceed 200ms.
We need to implement this logic in Python.

Now, let's walk through the solution from first principles:

Define the Problem: We are given a list of service response times and need to identify how many responses took longer than 200ms.
Input Data: Assume we have a list of floats or integers representing milliseconds (ms).
Logic Implementation:
Initialize a counter variable violations to zero.
Iterate through the list of response times.
For each time, check if it exceeds 200ms. If so, increment the violations counter.

Here is the Python code that accomplishes this:

PYTHON
def count_violations(response_times):
    violations = 0
    slo_threshold = 200  # Service Level Objective threshold in milliseconds

    for time in response_times:
        if time > slo_threshold:
            violations += 1

    return violations

# Example usage
response_times = [350, 180, 750, 90, 450]
violations = count_violations(response_times)
print(f"Number of response times violating the SLO: {violations}")

This script defines a function count_violations that takes a list of response times and returns the number of violations. We use a simple loop to iterate over each time in the list, check if it exceeds 200ms, and increment our counter accordingly.

**Code:**

```python
def count_violations(response_times):
    violations = 0
    slo_threshold = 200  # Service Level Objective threshold in milliseconds

    for time in response_times:
        if time > slo_threshold:
            violations += 1

    return violations
```

**Example input / output:**

```text
Input: [350, 180, 750, 90, 450]
Output: Number of response times violating the SLO: 3
```

**Data:**

```python
response_times = [350, 180, 750, 90, 450]
```

