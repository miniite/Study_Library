---
type: "practice"
difficulty: "Deep"
---

# How does Datadog apply machine learning to operational telemetry for anomaly and outlier detection?

**Difficulty:** Deep · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Anomaly and Forecast Monitors](<../../../../TOOLS/Datadog/Datadog_Platform/Anomaly_and_Forecast_Monitors.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

#### Overview

Datadog uses advanced machine learning techniques to analyze and process large volumes of operational telemetry data, identifying anomalies and outliers in real-time or near-real-time. This involves a combination of unsupervised learning methods like clustering and probabilistic models, as well as supervised learning for specific use cases.

#### Triaging the Question

The core of this question is understanding how Datadog leverages machine learning to detect unusual behavior in operational data. The interviewer is likely probing your knowledge of:

Types of machine learning algorithms used.
Data processing and preprocessing steps involved.

#### Implementation
details such as feature extraction and model selection.

#### First-Principles Solution
Data Collection and Preprocessing

Datadog collects a vast amount of operational telemetry data from various sources, including logs, metrics, traces, and synthetic monitoring. This raw data is first cleaned and preprocessed to remove noise and irrelevant information.

Raw Telemetry Data
Cleaning & Preprocessing
Feature Extraction
Feature Engineering

Feature extraction is crucial as it involves transforming raw data into meaningful features that can be used by machine learning models. For example, time series data might be transformed into features like mean, standard deviation, or rate of change.

C
Time Series Features
Model Selection and Training

Datadog uses a combination of unsupervised and supervised learning techniques:

Unsupervised Learning: Clustering algorithms (e.g., DBSCAN, K-means) to group similar data points.
Supervised Learning: For specific use cases where labeled data is available.
C
Unsupervised Learning
D
Supervised Learning
Anomaly Detection

Once the models are trained, they are used to detect anomalies in real-time or near-real-time. This involves monitoring new data points and comparing them against learned patterns.

E
Anomaly Detection
F
Outlier Detection
Post-Processing and Visualization

Detected anomalies and outliers are then post-processed to filter false positives and provide actionable insights. Datadog visualizes this data through intuitive dashboards and alerts, enabling quick response times.

G
Post-processing
H
Visualization
Deployment and Monitoring

Finally, these models are deployed in production environments with continuous monitoring to ensure they adapt to changing operational conditions. Regular retraining might be required based on evolving data characteristics.

#### Closing Summary
and Common Mistakes/Follow-Ups

In summary, Datadog uses a combination of unsupervised and supervised machine learning techniques to detect anomalies and outliers in operational telemetry data. The key steps include data preprocessing, feature engineering, model training, real-time anomaly detection, post-processing, and visualization.

#### Common Mistakes

Overfitting models on historical data.
Failing to address false positives during the post-processing step.
Not adapting models for changing operational conditions.

#### Follow-Ups

How does Datadog handle cold start problems?
What are some challenges in scaling this system across different types of telemetry data?

**Comparison:**

| Time | Metric1 | Metric2 |
|------|---------|---------|
| 1    | 1       | 0.9     |
| 2    | 2       | 0.8     |
| 3    | 3       | 1.1     |
| 4    | 4       | 1.2     |
| 5    | 5       | 0.7     |

**Code:**

```python
# Example feature extraction code
import pandas as pd

def extract_features(df):
    # Simple example: calculate mean and standard deviation for each metric
    features = df.describe().transpose()
    return features
```

**Example input / output:**

| Input Data | Feature Extraction Output |
|------------|--------------------------|
| metric1: [1, 2, 3, 4, 5] | mean=3, std=1.41 |
| metric2: [0.9, 0.8, 1.1, 1.2, 0.7] | mean=1.0, std=0.16 |

**Data:**

```json
{
  "data": {
    "metric1": [1, 2, 3, 4, 5],
    "metric2": [0.9, 0.8, 1.1, 1.2, 0.7]
  }
}
```

**Flow:**

1. Data Collection
2. Cleaning & Preprocessing
3. Feature Extraction
4. Unsupervised Learning
5. Anomaly Detection
6. Supervised Learning
7. Outlier Detection
8. Post-processing
9. Visualization

