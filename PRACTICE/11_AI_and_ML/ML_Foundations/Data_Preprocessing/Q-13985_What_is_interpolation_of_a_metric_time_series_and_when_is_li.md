---
type: "practice"
difficulty: "Intermediate"
---

# What is interpolation of a metric time series and when is linear interpolation an acceptable way to fill short gaps?

**Difficulty:** Intermediate · **Levels:** Fresher · [↑ Seasonal Baselines](<../../../../11_AI_and_ML/ML_Foundations/Data_Preprocessing/Seasonal_Baselines.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR

Linear interpolation is a method used to estimate missing values in a time series by fitting a straight line between two known points. It's commonly applied when data gaps are short and you need a quick, straightforward way to fill them.

#### Triaging the Question

The question is asking about interpolation, specifically focusing on linear interpolation for filling short gaps in a metric time series. The interviewer might be probing your understanding of:

What interpolation means - Estimating missing values based on known data points.
When to use linear interpolation - In scenarios where the gap between two data points is small and the relationship between them can reasonably be assumed to be linear.

#### Assumptions
and First-Principles

#### Assumptions

The time series has regular sampling intervals.
Missing values are short-term gaps (e.g., one or a few missing data points).
Step-by-Step Solution:

Understanding Interpolation:

Definition: Interpolation is the process of estimating unknown values between known data points.
Linear Interpolation: This method assumes that the relationship between two adjacent data points can be approximated by a straight line.

Conditions for Using Linear Interpolation:

Short Gaps: When there are only one or a few missing data points, linear interpolation is often acceptable because it requires minimal effort and provides a reasonable approximation.
Regular Sampling: The time series should have regular intervals; otherwise, the linearity assumption may not hold.

#### Example
Let's consider a metric time series with known values at times t1 and t2, and we want to fill the gap between them:

Time (t)	Metric Value
t0	V1
t1	V2
t1.5	?
t2	V3

Here, we have a gap between t1 and t2. Linear interpolation can be used to estimate the value at t1.5.

Formulating the Solution:

Given two known points (t1, V2) and (t2, V3), the interpolated value ( V_{interp} ) at time ( t ) is calculated as: [ V_{interp}(t) = V2 + (V3 - V2) \cdot \frac{(t - t1)}{(t2 - t1)} ]

Applying the Formula:

For t = 1.5: [ V_{interp}(1.5) = V2 + (V3 - V2) \cdot \frac{(1.5 - t1)}{(t2 - t1)} ]

#### Closing Summary

Linear interpolation is a simple and effective method for filling short gaps in time series data, especially when the underlying relationship between adjacent points can be reasonably approximated by a straight line. However, it's important to consider whether the gap length and regularity of sampling intervals justify its use.

#### Common Mistakes/Follow-Ups
Overusing Linear Interpolation: Using it for large gaps or irregularly sampled data might lead to inaccurate results.
Ignoring Edge Cases: Ensure that the time series has consistent sampling, otherwise, linear interpolation may not be appropriate.
Alternative Methods: Explore other methods like spline interpolation or using domain-specific models if the data requires a more sophisticated approach.

**Comparison:**

| Time (t) | Metric Value |
|----------|--------------|
| 14:00    | 50           |
| **16:30**| **?**        |
| 17:00    | 90           |

**Code:**

```python
def linear_interpolate(t1, V2, t2, V3, t):
    """
    Perform linear interpolation between two points (t1, V2) and (t2, V3).
    """
    return V2 + (V3 - V2) * ((t - t1) / (t2 - t1))
```

**Example input / output:**

```text
Given the time series data:
Time (t) | Metric Value
--------|--------------
2023-10-01T14:00 | 50
2023-10-01T16:00 | 70
2023-10-01T18:00 | 90

We want to fill the gap between t = 16:30 and t = 17:00.

Using linear interpolation:
V_interpolated = 70 + (90 - 70) * ((17:00 - 16:00) / (18:00 - 16:00)) = 70 + 20 * (3/2) = 85
```

**Data:**

```json
{
  "time_series": [
    {"timestamp": "2023-10-01T14:00", "value": 50},
    {"timestamp": "2023-10-01T16:00", "value": 70},
    {"timestamp": "2023-10-01T18:00", "value": 90}
  ]
}
```

**Flow:**

1. Time Series
2. Short Gaps
3. Linear Interpolation
4. Estimate Missing Values
5. Regular Sampling
6. Accurate Approximation
7. Acceptable Method
8. Overusing Linear Interpolation
9. Inaccurate Results

