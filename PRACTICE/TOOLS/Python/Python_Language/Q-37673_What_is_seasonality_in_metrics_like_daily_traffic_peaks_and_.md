---
type: "practice"
difficulty: "Deep"
---

# What is seasonality in metrics (like daily traffic peaks), and why does a naive fixed threshold in Python cause false alerts during expected busy hours?

**Difficulty:** Deep · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Flapping Detection](<../../../../TOOLS/Python/Python_Language/Flapping_Detection.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Seasonality refers to patterns that repeat over specific time intervals, such as daily traffic peaks or weekly usage trends. In AIOps, understanding and modeling seasonality is crucial because it helps in distinguishing between normal fluctuations and anomalies. A naive fixed threshold set without considering these patterns can lead to frequent false alerts during expected busy hours.

To deeply understand this issue, let's start by defining the concept of a threshold and how it might be implemented in Python. We'll then explore why a fixed threshold fails when seasonality is present and discuss better approaches using seasonality-aware anomaly detection techniques.

**Comparison:**

| Traffic Data | Check Result |
|--------------|--------------|
| 100          | False        |
| 200          | False        |
| 300          | False        |
| 400          | False        |
| 500          | False        |
| 600          | False        |
| 700          | False        |
| 800          | False        |
| 900          | False        |
| 1000         | False        |
| 1100         | True (Alert) |
| 1200         | True (Alert) |

**Code:**

```python
# Example of a naive threshold-based alerting system without considering seasonality

def check_threshold(value, threshold):
    if value > threshold:
        return True  # Alert triggered
    return False

# Simulated daily traffic data (in thousands)
traffic_data = [100, 200, 300, 400, 500, 600, 700, 800, 900, 1000, 1100, 1200] * 4
threshold = 1250

# Simulating a naive threshold check for the first few days
for day in range(len(traffic_data)):
    if check_threshold(traffic_data[day], threshold):
        print(f"Alert on Day {day+1}")
```

**Example input / output:**

```text
Simulating daily traffic data:
Day 1: No Alert (100)
Day 2: No Alert (200)
Day 3: No Alert (300)
Day 4: No Alert (400)
Day 5: No Alert (500)
Day 6: No Alert (600)
Day 7: No Alert (700)
Day 8: No Alert (800)
Day 9: No Alert (900)
Day 10: No Alert (1000)
Day 11: No Alert (1100)  # Alert triggered
Day 12: Alert on Day 12   # Alert triggered
```

**Data:**

```json
{
  "traffic_data": [100, 200, 300, 400, 500, 600, 700, 800, 900, 1000, 1100, 1200]
}
```

**Flow:**

1. Yes
2. No
3. Yes
4. Alert
5. No
6. Daily Traffic Data
7. Is value > threshold?
8. Trigger Alert
9. Normal Day, No Action
10. Seasonal Patterns Found
11. Adjust Threshold Based onSeasonality?
12. Apply Seasonal Adjustment
13. Re-run Check
14. Normal or Alert?

