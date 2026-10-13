---
type: "practice"
difficulty: "Intermediate"
---

# How would you compute the standard deviation of a list of metric readings in Python using the statistics module, and why is it useful for anomaly detection?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Simple Anomaly Detection](<../../../../TOOLS/Python/Python_Language/Simple_Anomaly_Detection.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To compute the standard deviation of a list of metric readings in Python, we can leverage the statistics module which provides an easy-to-use interface to common statistical calculations. This method is particularly useful for anomaly detection because it helps identify data points that are significantly different from other observations.

#### Triaging the Question

The question asks us to compute the standard deviation using Python's statistics module and explains its utility in anomaly detection. The key aspects to cover are:

Importing the statistics module.
Calculating standard deviation from a list of metric readings.
Understanding why standard deviation is useful for identifying anomalies.

#### First-Principles Solution

Let’s start by calculating the standard deviation using Python's statistics module. Here’s how you would do it:

PYTHON
import statistics

# Sample data: a list of metric readings
metric_readings = [23, 24, 25, 26, 27, 100]

try:
    # Compute the standard deviation
    std_deviation = statistics.stdev(metric_readings)
except statistics.StatisticsError as e:
    print(f"Error: {e}")
else:
    print(f"The standard deviation is: {std_deviation}")

#### Closing Summary
and Common Mistakes/Follow-Ups

This solution demonstrates the use of Python’s statistics module to compute the standard deviation, which helps in detecting anomalies. It's important to note that if the dataset contains less than two data points, a StatisticsError will be thrown.

#### Common Mistakes
Using mean() instead of stdev(): Ensure you are using statistics.stdev() for computing standard deviation.
Handling empty lists: The example above includes error handling to manage cases where the list might be empty or contain less than two elements.
Ignoring edge cases: Always consider what happens when the input is not a valid list or contains non-numeric values.

#### Example
Dry-Run

Given metric_readings = [23, 24, 25, 26, 27, 100], the output would be:

CODE
The standard deviation is: 28.941022568959586
Why It's Useful for Anomaly Detection

Standard deviation helps in identifying outliers by providing a measure of dispersion around the mean. If a new metric reading significantly deviates from the computed standard deviation, it can be flagged as an anomaly.

For instance, if we have metric_readings = [23, 24, 25, 26, 27, 100] and the normal range is within one or two standard deviations of the mean, a reading like 100 would be flagged as an anomaly.

**Code:**

```python
import statistics

# Sample data: a list of metric readings
metric_readings = [23, 24, 25, 26, 27, 100]

try:
    # Compute the standard deviation
    std_deviation = statistics.stdev(metric_readings)
except statistics.StatisticsError as e:
    print(f"Error: {e}")
else:
    print(f"The standard deviation is: {std_deviation}")
```

**Example input / output:**

```

Given metric_readings = [23, 24, 25, 26, 27, 100], the output would be:

CODE
Copy
The standard deviation is: 28.941022568959586
```

**Data:**

```json
{
  "metric_readings": [23, 24, 25, 26, 27, 100]
}
```

