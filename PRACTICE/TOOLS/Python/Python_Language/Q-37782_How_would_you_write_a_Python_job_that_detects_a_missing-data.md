---
type: "practice"
difficulty: "Intermediate"
---

# How would you write a Python job that detects a missing-data / no-data condition on a metric (the exporter died) and distinguishes it from a legitimate zero value?

**Difficulty:** Intermediate · **Levels:** SDE-1/2, SDE-2/3 · [↑ Simple Anomaly Detection](<../../../../TOOLS/Python/Python_Language/Simple_Anomaly_Detection.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address this challenge, we need to create a Python script or job that can differentiate between a genuine zero value in a metric and a missing or no-data condition. This involves understanding how data is typically handled and identifying patterns that indicate the absence of data. The core idea is to implement logic that checks for these conditions based on historical data behavior.

First, let's clarify what we're probing: the Python script should be able to determine if there's a gap in the expected metric values due to an issue with the exporter (e.g., the data source or the collector) rather than just receiving zero as part of the normal operational state. This requires us to consider how metrics behave normally and establish thresholds or patterns that indicate a potential problem.

We'll assume:

The metric typically fluctuates within certain bounds.
There are known periods where the value should be non-zero (e.g., during business hours).
We have access to historical data for baseline behavior analysis.

Step 1: Define the normal range and patterns of the metric.

PYTHON
# Example thresholds based on observed historical data
lower_bound = 0  # Lower bound for zero values
upper_bound = 100  # Upper bound for non-zero values during expected periods

expected_periods = {  # Times when the metric is expected to be non-zero
    '9:00': 18,
    '17:00': 24
}

Step 2: Check current data against historical patterns.

PYTHON
import datetime

def check_metric(data_point):
    timestamp = datetime.datetime.now()

    # Check if the current time is within an expected non-zero period
    for start_time, duration in expected_periods.items():
        then = datetime.datetime.strptime(start_time, '%H:%M')
        now = (timestamp - then).total_seconds() / 3600
        if now < duration:
            break

    # If we're outside of these periods and the value is zero, it might be missing data
    if timestamp.hour not in expected_periods.keys():
        return "missing_data"

    # Check if the value falls within normal bounds
    if lower_bound <= data_point <= upper_bound:
        return "normal"
    else:
        return "anomaly"

# Example usage
print(check_metric(0))  # Should return 'missing_data' or 'normal'

Step 3: Handle edge cases and refine logic.

Ensure the script can handle variations in time zones, daylight saving changes, etc.
Implement logging to capture when the job identifies missing data.

#### Closing summary
This solution involves defining normal ranges and expected patterns for metric behavior. By checking current data against these thresholds and historical behavior, we can effectively distinguish between a legitimate zero value and a missing-data condition caused by an issue with the exporter.

#### Common mistakes/follow-ups

Not considering historical data or baseline metrics.
Ignoring edge cases like time zone changes.
Failing to log findings for further analysis.

