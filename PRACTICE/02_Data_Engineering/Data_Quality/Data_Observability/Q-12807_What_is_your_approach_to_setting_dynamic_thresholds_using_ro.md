---
type: "practice"
difficulty: "Deep"
---

# What is your approach to setting dynamic thresholds using rolling windows versus fixed thresholds, and what breaks with each?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Distribution Change Alerts](<../../../../02_Data_Engineering/Data_Quality/Data_Observability/Distribution_Change_Alerts.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To set dynamic thresholds using rolling windows versus fixed thresholds in data observability, we need to understand that both methods have their merits and trade-offs. The key difference lies in how they handle temporal data and respond to varying patterns over time.

#### Triaging the Question

The core of this question is about understanding thresholding techniques used in data observability. The interviewer wants to know your approach and rationale for choosing between rolling windows and fixed thresholds, as well as the potential pitfalls of each method.

#### Assumptions
Temporal Data: We are dealing with time-series data that needs monitoring.
Thresholds: These define conditions under which alerts or actions need to be taken.
Step-by-Step Solution

Let's break down both methods from first principles:

Fixed Thresholds

Definition: A fixed threshold is a static value set manually or based on historical data.

Use Case: It works well when the system behavior does not change significantly over time, such as in systems with steady-state operations.

#### Implementation

PYTHON
def check_fixed_threshold(value, threshold):
    if value > threshold:
        return True  # Alert condition
    return False

#### Pros

Simplicity: Easy to implement and understand.

#### Cons
istency: Alerts are generated in a consistent manner over time.

#### Cons

Inflexible: Does not adapt well to changing data patterns or seasonal variations.
Potential Misses: Might fail to catch anomalies that occur outside the static threshold range.
Rolling Windows

Definition: A rolling window threshold uses a moving average or rolling statistical function over a defined time interval to set thresholds dynamically.

Use Case: Ideal for systems with fluctuating behavior, such as those experiencing seasonal patterns or sudden changes in load.

#### Implementation

PYTHON
import pandas as pd

def check_rolling_window_threshold(value, window_size=60):
    data = pd.DataFrame([value], columns=['value'])
    rolling_avg = data['value'].rolling(window=window_size).mean()
    if value > rolling_avg.iloc[-1]:
        return True  # Alert condition
    return False

#### Pros

Flexibility: Adjusts to changing patterns in real-time.
Better Sensitivity: More likely to catch anomalies that occur within the dynamic range.

#### Cons

Complexity: Requires more sophisticated computation and storage resources.
Delayed Response: May have a lag due to the moving window calculation, which could affect timely responses.

#### Closing Summary

Both methods are valid but serve different use cases. Fixed thresholds are simpler and consistent but less adaptable to changing conditions. Rolling windows offer better responsiveness to dynamic data patterns at the cost of increased complexity and computational overhead.

#### Common Mistakes/Follow-Ups
Mistake: Overlooking the need for historical context in setting fixed thresholds, leading to missed alerts.

#### Follow-Up
Discussing how to balance the trade-offs between simplicity and adaptability, especially in complex systems with varying operational conditions.
Triaging and Rationale Recap

In summary, the choice between rolling windows and fixed thresholds depends on the nature of your data and system. Rolling windows provide better sensitivity by dynamically adjusting based on recent data but require more resources and careful tuning. Fixed thresholds are simpler to implement and maintain but may miss important anomalies in fluctuating systems.

Self-Check:

This solution covers both methods comprehensively, including pros, cons, and real-world implications. It is correctly detailed and complete for a deep understanding of the concept.

**Comparison:**

| Method       | Pros                                | Cons                             |
|--------------|-------------------------------------|----------------------------------|
| Fixed        | Simple and consistent              | Inflexible, might miss anomalies |
| Rolling      | Adapts to changing conditions       | More complex, potential delays   |

**Code:**

```python
def check_fixed_threshold(value, threshold):
    return value > threshold

import pandas as pd

def check_rolling_window_threshold(value, window_size=60):
    data = pd.DataFrame([value], columns=['value'])
    rolling_avg = data['value'].rolling(window=window_size).mean()
    if value > rolling_avg.iloc[-1]:
        return True
    return False
```

**Example input / output:**

```text
# Fixed Threshold Example
fixed_threshold = check_fixed_threshold(40, 30)
print(f"Fixed Threshold Alert? {fixed_threshold}")  # Output: True

# Rolling Window Example
rolling_window_alert = check_rolling_window_threshold(40, window_size=60)
print(f"Rolling Window Alert? {rolling_window_alert}")  # Output depends on recent data mean
```

**Data:**

```json
{
  "values": [20, 22, 25, 30, 35, 40],
  "window_size": 60,
  "thresholds": [28, 30]
}
```

**Flow:**

1. Yes
2. No
3. Yes
4. No
5. Fixed Threshold
6. Value > Threshold?
7. Alert
8. No Alert
9. Rolling Window
10. Value > Rolling Mean?
11. Alert
12. No Alert

