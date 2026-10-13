---
type: "practice"
difficulty: "Basic"
---

# How does StandardScaler work and what does a standardized feature look like statistically?

**Difficulty:** Basic · **Levels:** Fresher · [↑ Seasonal Baselines](<../../../../11_AI_and_ML/ML_Foundations/Data_Preprocessing/Seasonal_Baselines.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

StandardScaler is a common data preprocessing technique used in machine learning to standardize features by removing the mean and scaling to unit variance. This transformation helps in improving the performance of many machine learning algorithms that assume or perform better with input features having zero mean and unit variance.

To understand how StandardScaler works, we need to break down the process step-by-step:

Centering: Subtract the mean from each feature.
Scaling: Divide by the standard deviation (square root of the variance).

Let's see this in action with a simple example. Suppose you have two features: Feature 1 and Feature 2, which are not standardized.

#### Triage

The question asks for an explanation of how StandardScaler works, focusing on the transformation process and what a standardized feature looks like statistically. The core idea is to understand the mathematical operations involved in standardizing data.

#### First-Principles Solution

Input Data: Consider two features X1 and X2.

Suppose we have a dataset with these two features:
PYTHON
import numpy as np

X = np.array([[10, 20], [30, 40], [50, 60]])

Compute Mean: [ \mu_X = \frac{1}{n} \sum_{i=1}^{n} x_i ] For X1 and X2, the mean is calculated as follows:

Centering (Subtract Mean):

Subtract the computed mean from each value in the feature.

Compute Standard Deviation: [ \sigma_X = \sqrt{\frac{1}{n} \sum_{i=1}^{n} (x_i - \mu)^2} ] The standard deviation is then used to scale the centered values.

Scaling (Divide by Standard Deviation):

Divide each value in the feature by its standard deviation.
Detailed Steps

Compute Mean:

PYTHON
mean_X1 = np.mean(X[:, 0])
mean_X2 = np.mean(X[:, 1])

Centering (Subtract Mean):

PYTHON
X_centered_1 = X[:, 0] - mean_X1
X_centered_2 = X[:, 1] - mean_X2

Compute Standard Deviation:

PYTHON
std_dev_X1 = np.std(X[:, 0])
std_dev_X2 = np.std(X[:, 1])

Scaling (Divide by Standard Deviation):

PYTHON
X_scaled_1 = X_centered_1 / std_dev_X1
X_scaled_2 = X_centered_2 / std_dev_X2

#### Example

Given the dataset X:

PLAINTEXT
| Feature 1 | Feature 2 |
|-----------|-----------|
|     10    |     20    |
|     30    |     40    |
|     50    |     60    |

The transformed dataset X_scaled would be:

PLAINTEXT
| Feature 1 (Scaled) | Feature 2 (Scaled) |
|-------------------:|-------------------:|
|          -1.22     |           -1.22    |
|           0        |            0       |
|          1.22      |           1.22     |

#### Closing Summary

StandardScaler works by centering and scaling the features to have zero mean and unit variance, respectively. The key steps involve computing the mean and standard deviation of each feature, then subtracting the mean from each value (centering) and dividing by the standard deviation (scaling). This ensures that all features contribute equally to the model's learning process.

#### Common Mistakes
Not Centering First: Some implementations might scale directly without centering, leading to incorrect results.
Ignoring Data Types: Ensure numerical data is handled correctly; categorical or binary data should not be scaled in the same manner as continuous data.
Standard Deviation of Zero: Features with zero standard deviation cannot be standardized and need special handling.

**Code:**

```python
import numpy as np

# Sample dataset
X = np.array([[10, 20], [30, 40], [50, 60]])

# Compute mean
mean_X1 = np.mean(X[:, 0])
mean_X2 = np.mean(X[:, 1])

# Centering (Subtract Mean)
X_centered_1 = X[:, 0] - mean_X1
X_centered_2 = X[:, 1] - mean_X2

# Compute Standard Deviation
std_dev_X1 = np.std(X[:, 0])
std_dev_X2 = np.std(X[:, 1])

# Scaling (Divide by Standard Deviation)
X_scaled_1 = X_centered_1 / std_dev_X1
X_scaled_2 = X_centered_2 / std_dev_X2

print("Centered Feature 1:", X_centered_1)
print("Centered Feature 2:", X_centered_2)
print("\nScaled Feature 1:", X_scaled_1)
print("Scaled Feature 2:", X_scaled_2)
```

