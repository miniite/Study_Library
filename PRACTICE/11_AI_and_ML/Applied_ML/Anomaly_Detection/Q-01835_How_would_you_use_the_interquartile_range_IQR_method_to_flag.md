---
type: "practice"
difficulty: "Basic"
---

# How would you use the interquartile range (IQR) method to flag outliers in a metric?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Robust Statistical Bands](<../../../../11_AI_and_ML/Applied_ML/Anomaly_Detection/Robust_Statistical_Bands.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To detect anomalies using the Interquartile Range (IQR) method, we first need to understand that this technique is effective because it identifies data points that are significantly different from other observations in a dataset. The IQR measures statistical dispersion and helps us define a range of “normal” values based on quartiles.

#### Triaging the Question

The question asks how to use the IQR method for anomaly detection, which involves several steps: computing the first quartile (Q1), third quartile (Q3), calculating the IQR, setting up thresholds, and then flagging any points outside of these thresholds. The interviewer is probing your understanding of statistical methods in time series analysis.

Step-by-Step Solution

We'll start by defining what we need to do:

Compute Q1 and Q3: These represent the 25th and 75th percentiles, respectively.
Calculate IQR: This is defined as ( \text{IQR} = Q3 - Q1 ).
Set Thresholds for Anomalies: Typically, points outside the range [Q1 - 1.5 * IQR, Q3 + 1.5 * IQR] are considered outliers.
Apply the Method to Time Series Data: This involves iterating over each data point and checking if it falls within or outside these thresholds.

#### Example
Calculation

Suppose we have a time series dataset of server response times:

CODE
Time: [0, 1, 2, ..., n]
Response Times: [30, 35, 40, 45, 50, 55, 60, 65, 70, 75, 80, 90, 100, 110, 200]
Step 1: Compute Q1 and Q3
Sort the data: [30, 35, 40, 45, 50, 55, 60, 65, 70, 75, 80, 90, 100, 110, 200]
Q1 (25th percentile): Value at position ( \frac{15}{4} = 3.75 ), so it is the average of the 4th and 5th values: ( \frac{45 + 50}{2} = 47.5 )
Q3 (75th percentile): Value at position ( \frac{15 * 3}{4} = 11.25 ), so it is the average of the 11th and 12th values: ( \frac{80 + 90}{2} = 85 )
Step 2: Calculate IQR

[ \text{IQR} = Q3 - Q1 = 85 - 47.5 = 37.5 ]

Step 3: Set Thresholds
Lower Bound: ( Q1 - 1.5 * \text{IQR} = 47.5 - 1.5 * 37.5 = 47.5 - 56.25 = -8.75 )
Upper Bound: ( Q3 + 1.5 * \text{IQR} = 85 + 1.5 * 37.5 = 85 + 56.25 = 141.25 )
Step 4: Flag Outliers

Any value outside the range [-8.75, 141.25] is flagged as an outlier. In our dataset, 200 is clearly an outlier.

#### Closing Summary
and Common Mistakes

The IQR method effectively identifies outliers by focusing on the spread of data. It’s robust to extreme values and works well with non-normal distributions. However, it assumes that the data is roughly symmetric around its median, which might not always be the case in real-world scenarios.

#### Common Mistakes

Ignoring Data Distribution: The IQR method assumes a normal distribution, but this may not hold for all datasets.
Not Handling Outliers During Calculation: If outliers are present in the dataset used to calculate Q1 and Q3, they can skew these values.
Over-Simplification: Failing to account for time series-specific properties like seasonality or trends.

**Comparison:**

```
Time	Response Times
0	30
1	35
2	40
...	...
n-1	110
n	200
```

**Code:**

```python
import numpy as np

def detect_outliers_iqr(data):
    # Calculate first quartile (Q1)
    Q1 = np.percentile(data, 25)

    # Calculate third quartile (Q3)
    Q3 = np.percentile(data, 75)

    # Interquartile range
    IQR = Q3 - Q1

    # Define thresholds
    lower_bound = Q1 - 1.5 * IQR
    upper_bound = Q3 + 1.5 * IQR

    # Identify outliers
    outliers = data[(data < lower_bound) | (data > upper_bound)]

    return outliers

# Sample time series data
response_times = [30, 35, 40, 45, 50, 55, 60, 65, 70, 75, 80, 90, 100, 110, 200]
outliers = detect_outliers_iqr(response_times)
print("Detected outliers:", outliers)
```

**Example input / output:**

```

outliers = detect_outliers_iqr([30, 35, 40, 45, 50, 55, 60, 65, 70, 75, 80, 90, 100, 110, 200]) print("Detected outliers:", outliers)

CODE
Copy
Output:
```python
Detected outliers: [200]
```

**Data:**

```json
{
    "response_times": [30, 35, 40, 45, 50, 55, 60, 65, 70, 75, 80, 90, 100, 110, 200]
}
```

**Flow:**

1. Yes
2. Inside Range
3. Outside Range
4. No
5. Start
6. Is Data Sorted?
7. Compute Q1 and Q3
8. Calculate IQR (Q3 - Q1)
9. Set Thresholds: Lower = Q1 -1.5*IQR, Upper = Q3 + 1.5*IQR
10. Iterate Over Data Points
11. Continue
12. Flag as Outlier
13. End

