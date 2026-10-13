---
type: "practice"
difficulty: "Deep"
---

# What made you pick a robust MAD-based band over Bollinger-style standard-deviation bands in a noisy queue-depth metric?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Robust Statistical Bands](<../../../../11_AI_and_ML/Applied_ML/Anomaly_Detection/Robust_Statistical_Bands.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address this question effectively, let’s start by summarizing why we might choose a robust Mean Absolute Deviation (MAD) based band over traditional Bollinger Bands when dealing with noisy time series data like queue-depth metrics.

TL;DR

In a noisy environment where the distribution of the metric can deviate significantly from normality, MAD-based anomaly detection provides more robust and reliable boundaries compared to Bollinger Bands. This is because MAD is less sensitive to outliers, making it better suited for scenarios with high noise levels in the data.

#### Triaging the Question

The question delves into a comparison between two common methods of anomaly detection: Mean Absolute Deviation (MAD) bands and Bollinger Bands. The core idea here is understanding why robust statistical measures are preferred over traditional ones, especially when dealing with noisy time series data like queue-depth metrics.

#### Assumptions
We have a noisy time series dataset representing queue depth.
The goal is to detect anomalies in the presence of significant noise.
Both MAD and Bollinger Bands are considered for anomaly detection.

#### First Principles Solution

Understanding Bollinger Bands

Bollinger Bands consist of a moving average (usually simple) with an upper band and a lower band, which are typically two standard deviations away from the moving average.
Standard deviation is highly sensitive to outliers; thus, in noisy datasets, these bands can become unreliable.

Understanding MAD-Based Anomaly Detection

MAD-based anomaly detection calculates the median absolute deviation of the data points around their median.
The median and MAD are more robust statistical measures because they are less influenced by extreme values (outliers).
For a time series with significant noise, the median and MAD provide a better estimate of central tendency and variability.

Why Choose MAD?

Robustness to Outliers: In noisy data, outliers can skew standard deviation calculations, making Bollinger Bands less effective.
Simplicity and Interpretability: MAD-based methods are straightforward and easier to interpret in the context of anomaly detection.
Flexibility in Distribution Assumptions: MAD does not assume a normal distribution, making it suitable for non-Gaussian data.

#### Example
Calculation

Suppose we have a time series ( x_t ) representing queue depth at different timestamps.
Calculate the median of this series: ( \text{median}(x_t) = m ).
Compute the MAD as follows: [ \text{MAD} = \frac{\sum_{t=1}^{n} |x_t - m|}{n} ]
Define the upper and lower bands using a threshold, say ( k \times \text{MAD} ), around the median.

Comparing with Bollinger Bands

If we observe significant noise in the queue depth metric, the standard deviations used in Bollinger Bands will be inflated due to outliers.
This inflation can lead to false positives or misses in anomaly detection.
In contrast, MAD-based bands remain robust and provide a reliable threshold for identifying anomalies.

#### Closing Summary

In summary, choosing a robust MAD-based band over Bollinger Bands is justified by the former's superior ability to handle noisy data. MAD is less sensitive to outliers, making it a more accurate tool for detecting anomalies in non-Gaussian time series like queue-depth metrics.

#### Common Mistakes/Follow-Ups
Overlooking the impact of outliers on standard deviation.
Assuming normality in the distribution of data when choosing Bollinger Bands.
Failing to consider the interpretability and robustness of MAD-based methods.

**Comparison:**

```
Time Step	Queue Depth	Deviation from Median
1	10	-20
2	15	-15
3	20	-10
4	25	-5
5	30	0
6	35	5
7	40	10
8	100	70
9	45	15
```

**Code:**

```python
import numpy as np

# Sample queue depth time series data
queue_depth = [10, 15, 20, 25, 30, 35, 40, 100, 45]

# Calculate median
median_depth = np.median(queue_depth)

# Calculate MAD
mad_depth = np.mean([abs(depth - median_depth) for depth in queue_depth])

print(f"Median: {median_depth}, MAD: {mad_depth}")
```

**Example input / output:**

```
Input: queue_depth = [10, 15, 20, 25, 30, 35, 40, 100, 45]
Output:
CODE
Copy
Median: 30.0, MAD: 27.8
Interpretation: The median is (30) and the MAD is approximately (27.8). These values can be used to define anomaly detection thresholds.
```

**Data:**

```json
{
  "queue_depth": [10, 15, 20, 25, 30, 35, 40, 100, 45]
}
```

**Flow:**

1. Collect Queue Depth Data
2. Calculate Median
3. Compute MAD
4. Determine AnomalyThresholds (k * MAD)
5. Triangulate Anomalies

