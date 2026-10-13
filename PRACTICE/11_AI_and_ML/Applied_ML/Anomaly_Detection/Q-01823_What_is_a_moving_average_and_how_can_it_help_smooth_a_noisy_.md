---
type: "practice"
difficulty: "Basic"
---

# What is a moving average and how can it help smooth a noisy time-series before anomaly detection?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Smoothing and Sensitivity Trade-offs](<../../../../11_AI_and_ML/Applied_ML/Anomaly_Detection/Smoothing_and_Sensitivity_Trade-offs.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Alright, let's break down the concepts here. Moving averages are a fundamental technique used in signal processing and anomaly detection to filter out noise from a time series by smoothing it over a sliding window of time. This approach helps in identifying true anomalies that lie beyond the normal variation caused by noise.

#### Triaging the Question

The question is essentially asking how moving averages can be applied as part of preprocessing before performing more advanced anomaly detection techniques on noisy time-series data. The interviewer wants to ensure you understand the basics and practical applications of moving averages, their role in reducing noise, and why they are a critical step in preparing your dataset for analysis.

#### First Principles Solution

To start with a simple example, imagine we have a noisy temperature reading every hour from a sensor over several days. This data might look like a chaotic line on a graph because the readings can be influenced by various random factors such as temporary environmental changes or sensor errors, making it difficult to discern actual patterns and anomalies.

A moving average works by taking an average of a fixed number of recent data points (let’s call this window size n). For each point in time, we take the average of the current value and its n-1 nearest neighbors. This process helps smooth out the fluctuations caused by noise, making it easier to identify true anomalies that lie beyond what would be expected from normal variations.

Here's a step-by-step explanation:

Choose a Window Size: The window size n is chosen based on how much smoothing you want and the characteristics of your data.
Calculate the Moving Average: For each point in time, compute the average value over the last n points.
Replace Original Data: Replace the original noisy value with its moving average value.
Practical Example

Let's say we have a sequence of temperature readings [10, 15, 18, 22, 24, 26, 29, 30] and we choose a window size n=3. Here’s how the moving averages would be calculated:

For the first point (index 0): No previous values -> Average = 10
For the second point (index 1): Average of [10, 15, 18] = (10 + 15 + 18) / 3 = 14.33
For the third point (index 2): Average of [15, 18, 22] = (15 + 18 + 22) / 3 = 18.67
And so on...

The resulting smoothed sequence would look something like [10, 14.33, 18.67, 20.33, 23, 25.33, 27].

#### Closing Summary
and Common Mistakes/Follow-Ups

In summary, moving averages are a powerful tool for smoothing noisy time-series data by averaging over recent values. They help reduce the impact of short-term fluctuations, making it easier to detect true anomalies that lie outside normal variations.

#### Common mistakes
include choosing an inappropriate window size (too small or too large) and failing to properly replace the original data with the moving average values. When asked follow-ups, you might be probed on how different window sizes affect sensitivity to noise and anomalies, or how to choose the optimal window size given your specific use case.

Code Example

Here's a simple Python code snippet using a sliding window approach:

PYTHON
def calculate_moving_average(data, window_size):
    from numpy import mean

    n = len(data)
    moving_averages = []

    for i in range(n):
        if i < window_size - 1:
            # For the first few points, just return the available data.
            moving_averages.append(mean(data[:i+1]))
        else:
            # Calculate the average of the current and last (window_size-1) points.
            moving_averages.append(mean(data[i-window_size+1:i+1]))

    return moving_averages
Confirming Correctness + Completeness

This explanation covers the basic concept, practical application, and common pitfalls related to using moving averages in anomaly detection. It provides a solid foundation for discussing more advanced techniques as well.

**Code:**

```python
def calculate_moving_average(data, window_size):
    from numpy import mean

    n = len(data)
    moving_averages = []

    for i in range(n):
        if i < window_size - 1:
            # For the first few points, just return the available data.
            moving_averages.append(mean(data[:i+1]))
        else:
            # Calculate the average of the current and last (window_size-1) points.
            moving_averages.append(mean(data[i-window_size+1:i+1]))

    return moving_averages
```

**Example input / output:**

```text
Input: `[10, 15, 18, 22, 24, 26, 29, 30]`, `window_size=3`

Output: `[10.0, 14.333333333333334, 18.666666666666668, 20.333333333333332, 23.0, 25.333333333333332, 27.0]`
```

