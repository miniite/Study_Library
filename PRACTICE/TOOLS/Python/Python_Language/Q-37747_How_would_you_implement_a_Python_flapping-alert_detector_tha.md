---
type: "practice"
difficulty: "Intermediate"
---

# How would you implement a Python flapping-alert detector that identifies a metric oscillating across a threshold and suppresses it, while still surfacing a persistent breach?

**Difficulty:** Intermediate · **Levels:** SDE-1/2, SDE-2/3 · [↑ Flapping Detection](<../../../../TOOLS/Python/Python_Language/Flapping_Detection.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To implement an effective Python flapping-alert detector, we need to identify when a monitored metric is oscillating around a threshold and then suppress alerts that are due to this oscillation. At the same time, if there’s a persistent breach (i.e., the metric consistently stays outside of the threshold), we must ensure that such breaches are surfaced. The key ideas here involve implementing sliding windows for detection, state tracking, and alert suppression logic.

#### Triage
The question asks us to design an algorithm in Python that can differentiate between oscillating metrics and persistent breaches. We need to implement a system that tracks recent metric values using a sliding window approach. If the metric oscillates within a certain range around the threshold, we suppress alerts. However, if it consistently stays outside this range, we trigger an alert.

#### Assumptions

The metric data is provided as a stream of values.
We have a threshold value that defines what constitutes a breach.
We need to set parameters for the sliding window and the oscillation tolerance.

Step-by-step solution from first principles:

Data Stream Processing: Assume we receive a continuous stream of metric values, e.g., metric_values = [value_1, value_2, ..., value_n].

Sliding Window Implementation:

Define the size of the sliding window (e.g., 30 samples) to consider recent data points.
Use this window to track the metric values over time.

Oscillation Detection:

For each new incoming value, calculate the standard deviation or range around a threshold.
If the metric oscillates within a certain tolerance (e.g., ±5% of the threshold), suppress the alert for that period.

Persistent Breach Identification:

Track whether the metric has stayed outside the threshold longer than a specified period (e.g., 10 out of the last 30 samples).
If it does, trigger an alert.

Here is how we might implement this in Python:

PYTHON
import numpy as np

class FlappingAlertDetector:
    def __init__(self, threshold, tolerance=5, window_size=30):
        self.threshold = threshold
        self.tolerance = tolerance / 100.0 * self.threshold
        self.window_size = window_size
        self.window = []
        self.last_alerted_at = None

    def update(self, value):
        if len(self.window) >= self.window_size:
            self.window.pop(0)

        self.window.append(value)

        # Calculate the range around the threshold
        lower_bound = self.threshold - self.tolerance
        upper_bound = self.threshold + self.tolerance

        current_range = np.max(self.window) - np.min(self.window)

        if (value < lower_bound or value > upper_bound) and not self.is_flapping():
            print(f"Potential breach detected: {value}")
            # Send alert here

    def is_flapping(self):
        count_outside_bounds = sum(1 for v in self.window if v < self.threshold - self.tolerance or v > self.threshold + self.tolerance)

        return count_outside_bounds <= 5

# Example usage
detector = FlappingAlertDetector(threshold=80)
metric_values = [75, 78, 82, 81, 79, 83, 77, 76, 84]
for value in metric_values:
    detector.update(value)

This implementation tracks recent values using a sliding window and checks if the new value is outside the tolerance range. If it is within this range for most of the last few samples, we suppress further alerts.

#### Closing Summary
+ Common Mistakes/Follow-Ups:

Ensure that the threshold and tolerance parameters are appropriately set to balance between sensitivity and noise.
Use a sliding window approach to manage state over time.

#### Cons
ider edge cases such as abrupt changes or long-term trends that might be missed by simple oscillation checks.

#### Follow-ups
could involve optimizing performance, adding more sophisticated statistical methods for anomaly detection, and integrating with alerting systems.

