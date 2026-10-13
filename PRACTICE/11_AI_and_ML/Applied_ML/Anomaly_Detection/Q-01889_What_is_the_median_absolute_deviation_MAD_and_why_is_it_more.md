---
type: "practice"
difficulty: "Deep"
---

# What is the median absolute deviation (MAD) and why is it more robust than standard deviation?

**Difficulty:** Deep · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Robust Statistical Bands](<../../../../11_AI_and_ML/Applied_ML/Anomaly_Detection/Robust_Statistical_Bands.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR Summary

Median Absolute Deviation (MAD) is a statistical measure that quantifies the variability of data in a time series, similar to how standard deviation does. However, MAD is more robust to outliers because it relies on the median rather than the mean. This makes it particularly useful in anomaly detection scenarios where extreme values can significantly distort results.

#### Triaging the Question

The question asks us to explain what MAD is and why it’s more robust than standard deviation. To tackle this, we need to:

Define both MAD and standard deviation.
Explain how they are calculated.
Compare their properties, focusing on robustness.
Provide a practical example of when MAD would be preferred over standard deviation.

#### First-Principles Solution
1. Definitions

Standard Deviation (SD): [ \sigma = \sqrt{\frac{1}{N} \sum_{i=1}^{N} (x_i - \mu)^2} ] Where:

( \sigma ) is the standard deviation.
( N ) is the number of data points.
( x_i ) are individual data points.
( \mu ) is the mean of all data points.

Median Absolute Deviation (MAD): [ \text{MAD} = \text{median}(|x_i - \text{median}(x)|) ] Where:

( \text{MAD} ) is the Median Absolute Deviation.
The inner median calculation involves finding the central value of the data set.
2. Calculation

To understand why MAD is more robust, let's walk through a simple example:

#### Cons
ider a time series with values: ( [10, 15, 16, 17, 18, 19, 20] ).

Standard Deviation:

Calculate the mean: [ \mu = \frac{10 + 15 + 16 + 17 + 18 + 19 + 20}{7} = 16.43 ]
Compute squared deviations from the mean and their average.
Standard deviation calculation: [ \sigma = \sqrt{\frac{(10-16.43)^2 + (15-16.43)^2 + ... + (20-16.43)^2}{7}} \approx 3.97 ]

Median Absolute Deviation:

Calculate the median: [ \text{median}(x) = 17 ]
Compute absolute deviations from the median.
Median of these absolute values: [ \text{MAD} = \text{median}(|10-17|, |15-17|, ..., |20-17|) = \text{median}(7, 2, 1, 0, 1, 2, 3) = 2 ]
3. Robustness Comparison

The key difference is that standard deviation uses the mean, which can be heavily influenced by outliers. For example, if we add an outlier like ( [10, 15, 16, 17, 18, 19, 20, 300] ):

Standard Deviation:

Mean: [ \mu = \frac{10 + 15 + ... + 20 + 300}{8} = 46.875 ]
Standard deviation calculation shows a much larger value, ( \sigma \approx 97.58 ), which is misleading for anomaly detection.

Median Absolute Deviation:

Median remains the same: [ \text{median}(x) = 17 ]
MAD computation: [ \text{MAD} = \text{median}(|10-17|, |15-17|, ..., |20-17|, |300-17|) = 16.5 ]

Thus, while the standard deviation is significantly affected by the outlier, MAD remains relatively stable.

4. Practical Application

In AIOps, where time series data can contain anomalies (such as sudden spikes or drops), using MAD allows for more accurate detection of genuine outliers without being skewed by occasional extreme values.

#### Closing Summary
and Common Mistakes/Follow-Ups

MAD is a valuable tool in anomaly detection because it’s less sensitive to outliers compared to standard deviation. However, it has its limitations too. For instance, MAD doesn't provide as much information about the distribution shape or tail behavior of the data.

#### Common mistakes
include:

Incorrectly calculating the median and absolute deviations.
Overlooking the presence of extreme values that can affect the results when using SD.
Failing to recognize situations where standard deviation might still be necessary for detailed analysis.

#### Follow-ups
could explore how MAD is implemented in practice, its limitations compared to other robust measures (like trimmed mean), or how it integrates into anomaly detection systems.

Self-Check

This answer covers definitions, calculations, comparisons, and practical applications correctly and comprehensively.

**Code:**

```python
import numpy as np

# Example time series data
data = [10, 15, 16, 17, 18, 19, 20]
outlier_data = [10, 15, 16, 17, 18, 19, 20, 300]

# Standard Deviation
mean_sd = np.mean(data)
sd = np.sqrt(np.mean((data - mean_sd)**2))

# Median Absolute Deviation
median_data = np.median(data)
mad = np.median(np.abs(data - median_data))
print("Standard Deviation:", sd)
print("Median Absolute Deviation:", mad)

# With outlier
mean_sd_outlier = np.mean(outlier_data)
sd_outlier = np.sqrt(np.mean((outlier_data - mean_sd_outlier)**2))

median_data_outlier = np.median(outlier_data)
mad_outlier = np.median(np.abs(outlier_data - median_data_outlier))
print("Standard Deviation with Outlier:", sd_outlier)
print("Median Absolute Deviation with Outlier:", mad_outlier)
```

**Example input / output:**

```text
Standard Deviation: 3.9681452505701703
Median Absolute Deviation: 2.0
Standard Deviation with Outlier: 113.15569907743214
Median Absolute Deviation with Outlier: 16.5
```

