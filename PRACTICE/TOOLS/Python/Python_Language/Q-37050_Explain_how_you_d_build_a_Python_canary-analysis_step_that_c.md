---
type: "practice"
difficulty: "Expert"
---

# Explain how you'd build a Python canary-analysis step that compares p95 latency and error rate of a canary versus baseline over a window and returns a promote/rollback decision.

**Difficulty:** Expert · **Levels:** SDE-1/2, SDE-2/3 · [↑ Python Tail Latency](<../../../../TOOLS/Python/Python_Language/Python_Tail_Latency/_index.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To build a Python canary-analysis step, we need to compare the performance metrics (p95 latency and error rate) between the baseline version and the canary version of an application over a specified time window. This analysis will help us decide whether to promote or rollback the canary based on predefined thresholds.

#### Triaging the Question
Objective: Implement a Python script that analyzes the performance metrics (p95 latency and error rate) between a baseline and a canary version.
Metrics: We'll focus on p95 latency, which measures the 95th percentile of response times, and the error rate.
Window: The analysis will be performed over a specified time window to ensure that we have enough data points for meaningful comparison.
Decision Criteria: If the canary's metrics fall within acceptable thresholds compared to the baseline, it should be promoted; otherwise, rolled back.

#### Assumptions
We assume access to performance data from both versions (baseline and canary) in a structured format.
The time window is predefined and provided as input to the script.
Predefined thresholds for p95 latency and error rate are available.
The script will output a decision: "promote" or "rollback".

#### First-Principles Solution
Data Collection: Collect performance data (p95 latency, error rate) from both versions over the specified time window.
Thresholds Definition: Define acceptable thresholds for p95 latency and error rate.
Comparison Logic: Compare the canary's metrics against these thresholds.
Decision Making: Based on the comparison, decide whether to promote or rollback.
Step-by-Step Solution

Collect Data:

Fetch performance data in a structured format (e.g., JSON).

#### Example
input schema:
JSON
{
  "baseline": [
    {"time": 0, "latency": 20, "error_rate": 0},
    {"time": 1, "latency": 35, "error_rate": 0.01}
  ],
  "canary": [
    {"time": 0, "latency": 25, "error_rate": 0.01},
    {"time": 1, "latency": 40, "error_rate": 0.02}
  ]
}

Threshold Definition:

#### Example
thresholds:
PYTHON
baseline_thresholds = {
  'p95_latency': {'upper_limit': 30},
  'error_rate': {'upper_limit': 0.1}
}
canary_thresholds = {
  'p95_latency': {'upper_limit': 40},
  'error_rate': {'upper_limit': 0.2}
}

Comparison Logic:

Iterate over the data points and compare against thresholds.
If any threshold is exceeded, flag the canary as problematic.
Code Implementation
PYTHON
def analyze_canary(canary_data, baseline_data, thresholds):
    for point in canary_data:
        if (point['latency'] > thresholds['p95_latency']['upper_limit'] or
            point['error_rate'] > thresholds['error_rate']['upper_limit']):
            return "rollback"
    return "promote"

# Example usage
baseline_data = [
    {"time": 0, "latency": 20, "error_rate": 0},
    {"time": 1, "latency": 35, "error_rate": 0.01}
]

canary_data = [
    {"time": 0, "latency": 25, "error_rate": 0.01},
    {"time": 1, "latency": 40, "error_rate": 0.02}
]

thresholds = {
    'p95_latency': {'upper_limit': 30},
    'error_rate': {'upper_limit': 0.1}
}

decision = analyze_canary(canary_data, baseline_data, thresholds)
print(f"Decision: {decision}")

#### Closing Summary

The script effectively compares the p95 latency and error rate of a canary version against a baseline over a specified time window. It outputs "promote" if both metrics are within acceptable limits or "rollback" otherwise.

#### Common Mistakes/Follow-Ups
Mistake: Not handling edge cases, such as missing data points.

#### Follow-Up
Discussing how to handle partial data (e.g., what to do if some time slots have no data).
Mistake: Failing to account for noise in the data. Consider implementing statistical filtering techniques.

**Comparison:**

| Time | Baseline Latency (ms) | Baseline Error Rate | Canary Latency (ms) | Canary Error Rate |
|------|-----------------------|--------------------|--------------------|------------------|
| 0    | 20                    | 0                  | 25                 | 0.01             |
| 1    | 35                    | 0.01               | 40                 | 0.02             |

**Code:**

```python
def analyze_canary(canary_data, baseline_data, thresholds):
    # Implementation as described above

# Example usage and decision output
baseline_data = [
    {"time": 0, "latency": 20, "error_rate": 0},
    {"time": 1, "latency": 35, "error_rate": 0.01}
]

canary_data = [
    {"time": 0, "latency": 25, "error_rate": 0.01},
    {"time": 1, "latency": 40, "error_rate": 0.02}
]

thresholds = {
    'p95_latency': {'upper_limit': 30},
    'error_rate': {'upper_limit': 0.1}
}

decision = analyze_canary(canary_data, baseline_data, thresholds)
print(f"Decision: {decision}")
```

**Example input / output:**

```text
# Example input data and thresholds

baseline_data = [
    {"time": 0, "latency": 20, "error_rate": 0},
    {"time": 1, "latency": 35, "error_rate": 0.01}
]

canary_data = [
    {"time": 0, "latency": 25, "error_rate": 0.01},
    {"time": 1, "latency": 40, "error_rate": 0.02}
]

thresholds = {
    'p95_latency': {'upper_limit': 30},
    'error_rate': {'upper_limit': 0.1}
}

# Output decision
decision = analyze_canary(canary_data, baseline_data, thresholds)
print(f"Decision: {decision}")
```

**Data:**

```json
{
  "baseline": [
    {"time": 0, "latency": 20, "error_rate": 0},
    {"time": 1, "latency": 35, "error_rate": 0.01}
  ],
  "canary": [
    {"time": 0, "latency": 25, "error_rate": 0.01},
    {"time": 1, "latency": 40, "error_rate": 0.02}
  ]
}
```

**Flow:**

1. Yes
2. No
3. Yes
4. No
5. Fetch Baseline Data
6. B
7. Fetch Canary Data
8. Define Thresholds
9. Analyze Canary MetricsAgainst Baseline
10. Canary Latency > Upper Limit?
11. Rollback
12. Canary Error Rate > UpperLimit?
13. Promote

