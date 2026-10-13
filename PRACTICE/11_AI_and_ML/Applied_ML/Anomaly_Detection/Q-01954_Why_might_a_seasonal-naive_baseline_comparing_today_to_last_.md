---
type: "practice"
difficulty: "Intermediate"
---

# Why might a seasonal-naive baseline comparing today to last week outperform a fancy model, and when does it break?

**Difficulty:** Intermediate · **Levels:** SDE-1/2, SDE-2/3 · [↑ Seasonality and Trend Handling](<../../../../11_AI_and_ML/Applied_ML/Anomaly_Detection/Seasonality_and_Trend_Handling.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When considering anomaly detection in time series data, especially in AIOps environments where anomalies can significantly impact system performance or reliability, using a simple seasonal-naive baseline (comparing today's metric to last week’s same day) might outperform more complex models for several reasons. However, this approach has limitations and breaks down under certain conditions.

First, let’s outline why a naive seasonal comparison could work well:

Simplicity and Speed: It requires minimal computational resources and is fast to implement.
Baseline Performance: In many scenarios, anomalies are indeed periodic or related to the same day of the week from previous weeks.
Low Noise Sensitivity: Simple models can be less prone to false positives that might arise from model complexity.

Now, let’s break down when this approach breaks and why:

Irregular Anomalies: If anomalies occur due to unexpected events or new trends unrelated to historical patterns, the naive method will struggle.
High Variance in Historical Data: In datasets with high variance, last week's values might not be a reliable proxy for today’s expected values.
Seasonality Changes Over Time: Seasonal patterns can change over time due to various factors (e.g., holidays, events), and the naive method won’t adapt unless retrained.

In summary, while a naive seasonal approach is simple and effective in many common cases, it has significant limitations that could lead to poor performance under certain conditions. A more robust solution would involve combining this baseline with adaptive learning techniques or incorporating domain knowledge about potential anomalies.

**Comparison:**

```
Day	Value
Mon	20
Tue	25
Wed	30
Thu	35
Fri	40
Sat	45
Sun	50
Mon	60
Tue	70
Wed	80
```

**Code:**

```python
# Example of a simple anomaly detection function using naive seasonal comparison

def detect_anomaly_naive(series, window):
    today_value = series[-1]
    last_week_same_day_value = series[-7 - (window % 7)]

    # Calculate the difference as an anomaly score
    anomaly_score = abs(today_value - last_week_same_day_value)
    return anomaly_score > threshold

# Example usage
series = [20, 25, 30, 35, 40, 45, 50, 60, 70, 80]
threshold = 15
window = 1 # Daily window adjustment for simplicity
anomaly_detected_today = detect_anomaly_naive(series, window)
```

**Example input / output:**

```text
Input:
- series: [20, 25, 30, 35, 40, 45, 50, 60, 70, 80]
- threshold: 15
- window: 1

Output:
- Anomaly detected today? False (since the difference between today's value and last week’s same day is within the threshold)
```

**Data:**

```json
{
    "series": [20, 25, 30, 35, 40, 45, 50, 60, 70, 80],
    "threshold": 15,
    "window": 1
}
```

**Flow:**

1. Yes
2. No
3. Start
4. Is today's value significantlydifferent from last week's sameday?
5. Return True (AnomalyDetected)
6. Return False (No AnomalyDetected)
7. End
8. End

