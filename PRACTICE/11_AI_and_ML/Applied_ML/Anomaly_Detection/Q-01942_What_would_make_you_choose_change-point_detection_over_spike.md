---
type: "practice"
difficulty: "Deep"
---

# What would make you choose change-point detection over spike detection for a gradual throughput degradation?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Dynamic vs Static Alerting](<../../../../11_AI_and_ML/Applied_ML/Anomaly_Detection/Dynamic_vs_Static_Alerting.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Change-point detection and spike detection are both valuable techniques in anomaly detection, but they serve different purposes. For a scenario involving gradual throughput degradation, change-point detection is often preferred because it can effectively identify more subtle shifts over time. Let's explore the core idea, the assumptions involved, and then build a step-by-step solution from first principles.

Change-point detection focuses on identifying points in time where the statistical properties of a time series abruptly change. This contrasts with spike detection, which typically looks for sudden, isolated deviations. For gradual degradation, we're interested in detecting subtle trends or shifts that may not be as obvious as spikes.

#### Triaging the Question

The question is asking us to decide between two techniques based on their suitability for identifying a gradual change in throughput. We need to consider factors such as the nature of the data, the desired outcome, and potential trade-offs.

#### Assumptions
The system has a baseline performance that gradually degrades over time.
We want to identify points where this degradation starts or accelerates.
The changes are not abrupt but gradual and can be subtle.

#### First-Principles Solution

To choose change-point detection for detecting gradual throughput degradation, we need to understand its core principles and how it differs from spike detection. Change-point detection involves modeling the time series data and looking for statistically significant shifts in parameters such as mean or variance over time.

Step 1: Model Selection

For a gradually degrading system, a statistical model that can capture gradual changes is essential. A common approach is to use a piecewise constant model where the data is segmented into intervals with constant behavior between segments. We can then detect points where these segments change.

Step 2: Change-Point Detection Algorithms

Several algorithms are available for detecting change-points, such as the Binary Segmentation algorithm or more advanced methods like the PELT (Pruned Exact Linear Time) method. These algorithms help identify significant shifts by comparing different segmentations of the time series data.

Step 3: Thresholding and Significance Testing

To ensure that detected changes are not due to random noise, we apply significance tests. This involves setting a threshold for determining whether a change is statistically significant or just an artifact of the data's inherent variability.

Step 4: Evaluation Metrics

Evaluate the performance using metrics like false positive rate (FPR) and true positive rate (TPR). For gradual degradation, TPR should be high to ensure we catch all relevant changes, while controlling FPR to minimize unnecessary alerts.

#### Closing Summary
+ Common Mistakes/Follow-Ups

Change-point detection is a powerful tool for identifying gradual shifts in time series data. It excels when the goal is to detect subtle trends or changes that might not be immediately apparent through spike detection alone. Key considerations include model selection, algorithm choice, and appropriate thresholding.

#### Common mistakes
include using inappropriate models or algorithms that do not capture gradual changes well. Over-relying on statistical significance without considering domain knowledge can also lead to false positives. Follow-ups could involve exploring advanced techniques like Bayesian change-point detection for more nuanced analysis.

**Comparison:**

```
Time	Throughput
0	10
1	12
2	9
3	8
4	7
5	6
6	5
7	4
8	3
9	2

Change Points: [5, 9]
```

**Code:**

```python
# Example of a simple piecewise constant model using Binary Segmentation
def binary_segmentation(data, max_segments):
    # Implementing the Binary Segmentation algorithm here
    pass

# Detecting significant changes using PELT method
from ruptures import Pelt

model = 'rbf'
change_points = Pelt(model=model).fit_predict(data)
```

**Example input / output:**

```

Input:

JSON
Copy
{
  "throughput_data": [10, 12, 9, 8, 7, 6, 5, 4, 3, 2]
}

Output:

JSON
Copy
{
  "change_points": [5, 9],
  "degradation_detected": true
}
```

**Data:**

```json
{
  "data": [10, 12, 9, 8, 7, 6, 5, 4, 3, 2]
}
```

**Flow:**

1. Yes
2. Start
3. Is Gradual Degradation?
4. Apply Change-Point DetectionAlgorithm
5. Identify Significant Changes
6. Evaluate with Metrics (TPR,FPR)
7. Output Change Points &Evaluation Results

