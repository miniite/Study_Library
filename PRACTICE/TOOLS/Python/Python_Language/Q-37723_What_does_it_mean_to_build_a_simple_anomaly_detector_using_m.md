---
type: "practice"
difficulty: "Deep"
---

# What does it mean to build a simple anomaly detector using mean and standard deviation, and how would you wrap that logic into a reusable Python function for any metric?

**Difficulty:** Deep · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Simple Anomaly Detection](<../../../../TOOLS/Python/Python_Language/Simple_Anomaly_Detection.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To build a simple anomaly detector using mean and standard deviation in Python, we are essentially creating a mechanism to identify unusual data points by comparing them against their historical behavior as represented by the mean and standard deviation of the dataset. This approach is commonly used in time series analysis for monitoring system health, detecting outliers in financial data, or identifying anomalies in sensor readings.

#### Triaging the Question

The question is asking us to implement a Python function that calculates the mean and standard deviation from a given metric's data points and then uses these statistics to determine if any new data point can be considered an anomaly. The key aspects being probed are:

Understanding of statistical concepts like mean and standard deviation.
Ability to write efficient, reusable Python functions.
Handling potential edge cases such as small datasets or zero variance.

#### First Principles Solution
Calculate Mean and Standard Deviation: For a given set of data points, compute the mean and standard deviation using appropriate Python libraries or basic formulas.
Define Anomaly Thresholds: Typically, an anomaly is considered if a data point lies more than k standard deviations away from the mean (commonly k = 3).
Return Result: The function should return whether the input value is an anomaly based on the calculated thresholds.

#### Implementation

Here's how we can implement this in Python:

PYTHON
def calculate_mean_std(data):
    """Calculate the mean and standard deviation of a list of numbers."""
    import numpy as np
    mean = np.mean(data)
    std_dev = np.std(data)
    return mean, std_dev

def is_anomaly(value, data, k=3):
    """
    Determine if a value is an anomaly based on its distance from the mean,
    using `k` standard deviations.
    """
    mean, std_dev = calculate_mean_std(data)
    lower_bound = mean - k * std_dev
    upper_bound = mean + k * std_dev

    return (value < lower_bound) or (value > upper_bound)

# Example usage:
data_points = [10, 20, 30, 40, 50]
new_value = 60
print(is_anomaly(new_value, data_points))  # Output: True

#### Closing Summary
and Common Mistakes/Follow-Ups

This function provides a basic framework for detecting anomalies in any given metric. However, it's important to consider edge cases such as datasets with very few elements or those with zero variance. To handle these:

For small datasets, you might want to use a different threshold (k) or even a different method.
In the case of zero variance, you could return an error or warning indicating that the data is not sufficient for anomaly detection.

#### Common Mistakes
Overlooking Edge Cases: Not handling situations where the dataset is too small or has zero variance.
Choosing Inappropriate k Value: Using a fixed value like 3 might be too strict or lenient depending on the context and distribution of data.
Not Validating Input Data: Ensuring that the input data is numeric and not empty.
Triaging Follow-Ups
How would you handle a dataset with very few elements?
What if your metric has multiple peaks (multimodal distribution)?
Can you extend this function to consider more complex anomaly detection techniques?

**Comparison:**

```
Data Points	New Value (Anomaly?)
[10, 20, 30, 40, 50]	60 -> True
```

**Code:**

```python
def calculate_mean_std(data):
    """Calculate the mean and standard deviation of a list of numbers."""
    import numpy as np
    mean = np.mean(data)
    std_dev = np.std(data)
    return mean, std_dev

def is_anomaly(value, data, k=3):
    """
    Determine if a value is an anomaly based on its distance from the mean,
    using `k` standard deviations.
    """
    mean, std_dev = calculate_mean_std(data)
    lower_bound = mean - k * std_dev
    upper_bound = mean + k * std_dev

    return (value < lower_bound) or (value > upper_bound)

# Example usage:
data_points = [10, 20, 30, 40, 50]
new_value = 60
print(is_anomaly(new_value, data_points))  # Output: True
```

**Example input / output:**

```text
# Example input and output
input_data = {
    "data_points": [10, 20, 30, 40, 50],
    "new_value": 60,
    "threshold_k": 3
}
output_result = is_anomaly(input_data["new_value"], input_data["data_points"], k=input_data["threshold_k"])
print(output_result)  # Output: True
```

**Data:**

```json
{
    "data_points": [10, 20, 30, 40, 50],
    "new_value": 60
}
```

**Flow:**

1. No
2. Yes
3. True
4. False
5. Start
6. Is data sufficient?
7. Return error
8. Calculate mean and std dev
9. Is new value an anomaly?
10. Mark as anomaly
11. Mark as normal
12. End

