---
type: "practice"
difficulty: "Intermediate"
---

# How would you write a Python function that classifies a latency value into buckets (fast, normal, slow) for a simple histogram of response times?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Percentage Change Calculations](<../../../../TOOLS/Python/Python_Language/Percentage_Change_Calculations.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To classify a given latency value into one of three categories—fast, normal, and slow—we need to define thresholds that separate these categories. The function should take in the latency (in milliseconds) as an input and return the appropriate category based on predefined ranges.

#### Triage

The question is asking for a Python function to categorize response times into fast, normal, or slow buckets. We need to assume certain threshold values for what constitutes "fast," "normal," and "slow." Typically, these thresholds can be derived from domain knowledge or empirical data. For this example, let's define:

Fast: 0-25ms
Normal: 26-100ms
Slow: 101ms and above

#### First-Principles Solution

First, we'll write a simple Python function that takes the latency value as input. We will then use conditional statements to determine which category it falls into based on our defined thresholds.

PYTHON
def classify_latency(latency):
    if latency <= 25:
        return "fast"
    elif 26 <= latency <= 100:
        return "normal"
    else:  # latency >= 101
        return "slow"

#### Closing Summary
and Common Mistakes/Follow-Ups

The function is straightforward, but there are a few common mistakes to be aware of:

Inconsistent Thresholds: Ensure that the thresholds you choose make sense for your specific use case.
Edge Cases: Handle scenarios where the latency could be exactly at a threshold value. For instance, if 25ms is considered "fast," should 25ms also return "normal"?
Logging and Testing: Always include appropriate logging or unit tests to validate that the function behaves as expected.

#### Example
PYTHON
# Example usage of the function
latencies = [10, 30, 75, 105]
for latency in latencies:
    print(f"Latency: {latency}ms -> Category: {classify_latency(latency)}")

Output:

CODE
Latency: 10ms -> Category: fast
Latency: 30ms -> Category: normal
Latency: 75ms -> Category: normal
Latency: 105ms -> Category: slow
Self-Check

This solution is correct and complete.

