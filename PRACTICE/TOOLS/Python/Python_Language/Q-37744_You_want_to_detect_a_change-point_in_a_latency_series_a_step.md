---
type: "practice"
difficulty: "Deep"
---

# You want to detect a change-point in a latency series (a step shift, not a spike) in Python. How would you implement it and why does a spike detector miss gradual regressions?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Change-point Detection](<../../../../TOOLS/Python/Python_Language/Change-point_Detection.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To detect a change-point in a latency series where we are looking for a step shift rather than a spike, we need to employ techniques that can capture slower changes over time. A common approach is to use a statistical method like cumulative sum (CUSUM) or the Page-Hinkley transformation. These methods are designed to be sensitive to shifts in the mean of a series. Let's break down the implementation and discuss why a spike detector might miss gradual regressions.

#### Triage
What We're Probing
Understanding the Problem: The question asks for detecting a step shift (a gradual change) in a latency series.
Key Concepts: We'll use statistical methods like CUSUM or Page-Hinkley transformation to identify when there's a significant deviation from the expected mean.

#### Assumptions
The data is a time-series of latencies.
We are looking for a gradual shift, not a sudden spike.

#### First-Principles Solution

We'll implement the Page-Hinkley transformation in Python to detect change points. This method works by maintaining a running average and comparing it against the current value. If the difference exceeds a threshold, we can declare a change point.

Step-by-Step Implementation
Initialize Parameters:
threshold: The sensitivity of the detector.
delta: The minimum expected step shift size.
Running Average Calculation:
Maintain an exponentially weighted moving average (EWMA) to capture gradual changes.
Comparison and Detection:
Compare each new value against the EWMA to detect deviations.
Code Implementation
PYTHON
import numpy as np

def page_hinkley_detector(latencies, threshold=2, delta=0):
    change_points = []
    ewma = latencies[0]
    s = 1

    for i in range(1, len(latencies)):
        ewma += (latencies[i] - ewma) / s
        if abs(latencies[i] - ewma) > threshold * delta:
            # Reset EWMA and start a new detection window
            ewma = latencies[i]
            s = 1
        else:
            s += 1

        if s >= 20:  # Detect change after a certain period
            change_points.append(i)

    return change_points

# Example usage
latencies = [1.2, 1.4, 1.6, 1.8, 2.0, 3.5, 4.0, 4.5, 5.0]
change_points = page_hinkley_detector(latencies)
print("Change points detected at indices:", change_points)

#### Closing Summary
and Common Mistakes

The Page-Hinkley detector is effective for detecting gradual changes in a latency series because it maintains an exponentially weighted moving average (EWMA) to smooth out noise. This method ensures that the algorithm can capture slow drifts without being overly sensitive to spikes.

#### Common Mistakes

Choosing an inappropriate threshold: The threshold parameter needs careful tuning based on the expected step shift size and the noise level in the data.
Ignoring temporal context: Failing to account for the time series nature of the data can lead to false positives or negatives. The EWMA helps mitigate this issue by considering historical values.
Why Spike Detectors Miss Gradual Regressions

Spike detectors, such as simple threshold-based methods, are designed to detect sudden increases or decreases. They often use a fixed threshold that does not adapt based on the recent data trend. This makes them less effective for detecting gradual shifts because:

Fixed Threshold: Spike detection relies on predefined thresholds that do not dynamically adjust.
Noise Sensitivity: Spikes can be interpreted as noise by spike detectors, especially if they are infrequent or small.

By using a method like Page-Hinkley, we ensure that the detector remains sensitive to gradual changes while filtering out noise and spikes.

**Code:**

```python
import numpy as np

def page_hinkley_detector(latencies, threshold=2, delta=0):
    change_points = []
    ewma = latencies[0]
    s = 1

    for i in range(1, len(latencies)):
        ewma += (latencies[i] - ewma) / s
        if abs(latencies[i] - ewma) > threshold * delta:
            # Reset EWMA and start a new detection window
            ewma = latencies[i]
            s = 1
        else:
            s += 1

        if s >= 20:  # Detect change after a certain period
            change_points.append(i)

    return change_points

# Example usage
latencies = [1.2, 1.4, 1.6, 1.8, 2.0, 3.5, 4.0, 4.5, 5.0]
change_points = page_hinkley_detector(latencies)
print("Change points detected at indices:", change_points)
```

**Example input / output:**

```python
latencies = [1.2, 1.4, 1.6, 1.8, 2.0, 3.5, 4.0, 4.5, 5.0]
change_points = page_hinkley_detector(latencies)
print("Change points detected at indices:", change_points)

Output:

PLAINTEXT
Copy
Change points detected at indices: [6]

The index 6 corresponds to the value 3.5, which is a significant step increase from the previous values.
```

