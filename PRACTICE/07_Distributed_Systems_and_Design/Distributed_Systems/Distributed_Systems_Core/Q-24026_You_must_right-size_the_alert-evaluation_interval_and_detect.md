---
type: "practice"
difficulty: "Deep"
---

# You must right-size the alert-evaluation interval and detection window org-wide. Explain the tension between fast detection (short windows, more false positives from noise) and stable detection (long windows, slower to fire), and how you'd set adaptive, load-aware windows as a platform default.

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Gossip and Deduplication](<../../../../07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Clock_Skew_and_Time_Windows/Gossip_and_Deduplication.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When setting alert-evaluation intervals and detection windows in a distributed system, there's a fundamental tension between fast detection of issues (short windows) and stable detection that minimizes false positives from noise (long windows). To right-size these parameters as a platform default, we need to balance the speed of issue identification with the reliability of alerts. This involves understanding the nature of the data and traffic patterns, along with considering the impact on system performance and alert fatigue.

To address this challenge, I would adopt an adaptive approach that dynamically adjusts the windows based on the current load conditions and historical data trends. Here’s a step-by-step explanation:

Understand Data Characteristics: First, analyze the nature of the system's traffic to understand how often issues are expected to occur.
Define Baseline Load: Establish a baseline of normal operation by collecting metrics over time in low-traffic periods.
Set Default Windows: Start with shorter detection windows to quickly identify issues but monitor for false positives and adjust based on real-world performance.
Adaptive Mechanism: Implement an adaptive system that adjusts the window sizes dynamically:
Shorten the evaluation interval during peak load or after detecting a pattern of frequent alerts to reduce latency in issue discovery.
Lengthen the evaluation interval during off-peak times when the system is less noisy, but maintain a mechanism to detect long-term trends.

**Comparison:**

```
Window Size (Seconds)	Description
Short	For high load or frequent alerts
Long	For low noise and stable conditions
```

**Code:**

```python
import time

def adaptive_window_detector(current_load, baseline_load):
    if current_load > 1.5 * baseline_load:
        # Shorten window for faster detection during peak load
        return short_evaluation_window()
    else:
        # Adjust to longer window in off-peak times
        return long_evaluation_window()

def short_evaluation_window():
    return 30  # seconds

def long_evaluation_window():
    return 120  # seconds
```

**Example input / output:**

```text
# Example input and output
baseline_load = {"cpu_usage": [85, 90, 92], "memory_usage": [70, 75, 76]}
current_load = {"cpu_usage": [100, 103, 104], "memory_usage": [85, 87, 88]}

window_size = adaptive_window_detector(current_load, baseline_load)
print(f"Selected evaluation window: {window_size} seconds")
```

**Data:**

```json
{
  "baseline_load": {
    "cpu_usage": [85, 90, 92],
    "memory_usage": [70, 75, 76]
  },
  "current_load": {
    "cpu_usage": [100, 103, 104],
    "memory_usage": [85, 87, 88]
  }
}
```

**Flow:**

1. Yes
2. No
3. Detect Current Load
4. Is current_load > 1.5 *baseline_load?
5. Shorten Evaluation Window to30s
6. Lengthen Evaluation Windowto 120s
7. Monitor for False Positives
8. Re-evaluate after an hour

