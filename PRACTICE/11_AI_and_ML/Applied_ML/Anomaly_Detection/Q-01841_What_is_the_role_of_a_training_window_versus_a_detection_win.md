---
type: "practice"
difficulty: "Intermediate"
---

# What is the role of a training window versus a detection window in an anomaly detector?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Multivariate Anomaly Correlation](<../../../../11_AI_and_ML/Applied_ML/Anomaly_Detection/Multivariate_Anomaly_Correlation.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In an anomaly detection system, particularly when dealing with time series data, the training window and detection window play crucial roles. The training window is where we gather historical data to build our model, while the detection window is used during runtime to evaluate new data points against the trained model to identify anomalies. Understanding these concepts helps in setting up an effective anomaly detection system.

The interviewer is likely probing your understanding of how the system learns from past data and applies that learning to real-time or near-real-time data. They might also be interested in how you handle initial training, online updates, and potential delays or lags between the two windows.

First-principles reasoning starts with defining what these terms mean:

Training Window: This is the period during which we collect historical data that our model will learn from. The goal here is to capture the normal behavior of the system so that when new data arrives, it can be compared against this "normal" baseline.

Detection Window: Once the model is trained on the training window data, it moves into detection mode where it processes real-time or near-real-time data points. Here, the model uses its learned knowledge to predict what the next value should be based on past patterns and flag any deviations as anomalies.

Assuming a scenario where we have time series data representing user activity on a website, let's break down the process:

Training Phase: Suppose we collect 30 days of user interaction data. During this phase, our algorithm analyzes the data to understand typical user behavior such as login patterns, session durations, and click-through rates.

Detection Phase: Once trained, the model starts processing new incoming data points in real-time or near-real-time. For instance, it might process a day's worth of data every hour. If there’s a sudden spike in logins that doesn’t match historical norms, the system flags this as an anomaly.

Now, let's discuss common considerations and edge cases:

Size of Training Window: A larger training window can capture more variability but may lag behind current trends. Conversely, a smaller window is faster to update but might miss subtle shifts.
Detection Delay: There’s always a delay between when data enters the detection window and when an anomaly is flagged. Managing this trade-off involves deciding how frequently new data should be processed after training.
Online Learning: Models can continue learning even as they are in use, updating their understanding of what constitutes "normal" behavior over time.

In summary, the training window ensures the model learns from a broad range of historical data, while the detection window allows it to adapt and apply this learning to real-time or near-real-time data. Balancing these aspects is key to building an effective anomaly detection system.

**Comparison:**

```
Timestamp	Value
2023-10-01	10
2023-10-02	15
...	...
2023-11-01	9
Timestamp	Predicted Value
2023-11-01T09:00Z	15
2023-11-01T09:05Z	14
Timestamp	Anomaly Flag
2023-11-01T09:05Z	1
```

**Code:**

```python
# Example code snippet for training and detecting anomalies

from datetime import timedelta
import pandas as pd
from statsmodels.tsa.stattools import adfuller

def train_model(training_data):
    # Perform initial analysis (e.g., stationarity check)
    result = adfuller(training_data['value'])
    if result[1] > 0.05:
        print("Data is non-stationary, needs differencing.")

    # Fit your model here
    pass

def detect_anomalies(detection_data):
    predictions = predict_next_values(detection_data)
    residuals = detection_data['value'] - predictions

    # Flag anomalies based on some threshold or statistical method
    anomalies = (residuals.abs() > 3 * residuals.std()).astype(int)

    return anomalies

def main():
    training_window_size = timedelta(days=30)  # Define your training window size
    detection_window_size = timedelta(hours=1)  # Define your detection window size

    current_time = pd.Timestamp.now()
    start_training = current_time - training_window_size
    end_detection = current_time + detection_window_size

    training_data = get_data_between(start_training, current_time)
    train_model(training_data)

    detection_data = get_data_between(current_time, end_detection)
    anomalies = detect_anomalies(detection_data)

    print("Detected anomalies:", anomalies)

main()
```

**Example input / output:**

```text
Input:
Training Data (30 days of user activity):
Timestamp       Value
2023-10-01      10
2023-10-02      15
...
2023-11-01      9

Detection Data (1 hour of new data):
Timestamp       Value
2023-11-01T09:00Z   14
2023-11-01T09:05Z   16

Output:
Predicted Values based on Training:
Timestamp       Predicted Value
2023-11-01T09:00Z   15
2023-11-01T09:05Z   14

Anomalies Detected (Based on Residuals):
Timestamp       Anomaly Flag
2023-11-01T09:05Z    1
```

**Data:**

```json
{
  "training_window": {
    "start_date": "2023-10-01",
    "end_date": "2023-11-01"
  },
  "detection_window": {
    "start_time": "2023-11-01T09:00:00Z",
    "end_time": "2023-11-01T10:00:00Z"
  }
}
```

**Flow:**

1. Yes
2. No
3. Yes
4. Yes
5. No
6. Start
7. Collect Historical Data(Training Window)
8. Is Training Complete?
9. Perform Analysis and ModelTraining
10. Process New Real-time Data(Detection Window)
11. Are Predictions Available?
12. Compare New Data withPredictions
13. Is Anomaly Detected?
14. Flag as Anomaly
15. Continue Monitoring
16. Notify Anomaly DetectionSystem

