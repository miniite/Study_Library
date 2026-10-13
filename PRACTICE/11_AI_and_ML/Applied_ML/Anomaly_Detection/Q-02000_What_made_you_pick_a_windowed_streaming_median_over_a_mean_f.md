---
type: "practice"
difficulty: "Deep"
---

# What made you pick a windowed streaming median over a mean for a heavy-outlier real-time metric?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Robust Statistical Bands](<../../../../11_AI_and_ML/Applied_ML/Anomaly_Detection/Robust_Statistical_Bands.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When dealing with real-time metrics in an AIOps environment, robustness against outliers is crucial to ensure accurate anomaly detection. One common approach to handling this is by choosing the windowed streaming median over the mean for calculating anomalies. The reason I picked a windowed streaming median is due to its resilience to extreme values and its ability to provide more reliable estimates in noisy environments.

The key idea behind using the windowed streaming median lies in how it calculates the central value of a set of data points within a moving window, making it less susceptible to skewing by outliers. In contrast, the mean can be heavily influenced by large deviations, leading to misleading anomaly detection results. Let's break down this solution step-by-step.

Problem Statement: The question revolves around why I chose the windowed streaming median over the mean for a real-time metric with heavy-outliers.

Core Concept: We need an algorithm that can handle noisy data and provide stable, reliable anomaly detection.

#### Assumptions

The environment involves real-time metrics where anomalies must be detected quickly.
Outliers are common in the data stream.

#### Solution
from First Principles:

Mean Calculation: The mean is calculated by summing all values and dividing by the number of samples. This makes it sensitive to outliers, as a single extreme value can significantly alter the result.
Median Calculation: The median, on the other hand, is determined by sorting the data points and selecting the middle value (or average of two middle values if the dataset size is even). It is less affected by outliers because it only considers the central position in the sorted list.

Windowed Streaming Median:

In a windowed approach, we define a sliding window over the recent data points.
For each new incoming data point, update the window to include the newest and exclude the oldest data point.
Recalculate the median within this updated window to get the current value.

Advantages of Windowed Streaming Median:

Robustness: It reduces the impact of outliers by focusing on the central values rather than all points.
Real-Time Adaptation: By using a sliding window, it can adapt to changes in the data distribution over time.
Performance: Efficient algorithms exist for maintaining the median within a window, ensuring low computational overhead.

#### Implementation
Example: Suppose we have a window of size 10 and a new incoming value x. The steps are as follows:

PYTHON
# Assume data is a list storing the last 10 values
if len(data) == 10:
    old_value = data.pop(0)
    data.append(x)
else:
    data.append(x)

# Sort the window and find the median
sorted_data = sorted(data)
n = len(sorted_data)
median = (sorted_data[n//2 - 1] + sorted_data[n//2]) / 2 if n % 2 == 0 else sorted_data[n//2]

return median

#### Closing Summary
and Common Mistakes: The windowed streaming median provides a robust approach to anomaly detection in real-time metrics with heavy outliers. It is less sensitive to extreme values compared to the mean, making it more reliable. However, common mistakes include not properly implementing the sliding window mechanism or failing to account for edge cases like empty windows or very small windows.

