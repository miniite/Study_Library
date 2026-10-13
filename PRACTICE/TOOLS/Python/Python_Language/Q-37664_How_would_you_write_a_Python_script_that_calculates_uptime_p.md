---
type: "practice"
difficulty: "Intermediate"
---

# How would you write a Python script that calculates uptime percentage from a list of up/down status samples over a day?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Percentage Change Calculations](<../../../../TOOLS/Python/Python_Language/Percentage_Change_Calculations.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To calculate the uptime percentage from a list of status samples, we need to determine how much time the system was operational (up) and then compute the ratio of that time to the total duration covered by our data. This problem involves parsing the input data, counting the number of "up" states, and calculating the resulting percentage.

#### Triaging the Question

The key elements are:

Input Data: A list of status samples.
Uptime Calculation: Counting the number of times the system was up.
Percentage Calculation: Determining the uptime as a percentage of total time covered by the data.

We can make some assumptions:

Each status sample is in the format [timestamp, 'up'/'down'].
The timestamps are uniformly spaced and represent one-minute intervals (common for such problems).
We'll consider the total duration to be 24 hours, which gives us 1440 samples if each timestamp represents a minute.

#### First Principles Solution

Let's break down the steps:

Parse the input data.
Count the number of "up" states in the list.
Calculate the uptime percentage based on the count and total duration.

Here is how you can implement this:

PYTHON
def calculate_uptime_percentage(status_samples):
    # Initialize counters
    up_count = 0
    total_samples = len(status_samples)

    # Iterate through the samples to count 'up' states
    for sample in status_samples:
        if sample[1] == 'up':
            up_count += 1

    # Calculate uptime percentage
    uptime_percentage = (up_count / total_samples) * 100

    return uptime_percentage

# Example input data
status_samples = [
    [0, 'down'], [1, 'up'], [2, 'up'], [3, 'down'], [4, 'up'],
    # ... more samples ...
]

uptime = calculate_uptime_percentage(status_samples)
print(f"Uptime percentage: {uptime:.2f}%")

#### Closing Summary
and Common Mistakes

This script correctly calculates the uptime percentage from a list of status samples. Key takeaways include:

Ensure the data format is consistent.
Handle edge cases where no 'up' states are present.

#### Common mistakes
to avoid:

Not converting timestamps to a common unit before comparison.
Miscounting or misinterpreting the input data format.
Forgetting to convert the count of "up" samples into a percentage.

