---
type: "practice"
difficulty: "Intermediate"
---

# Why must you call fit only on the training data for a scaler like StandardScaler, then transform the test data with the same fitted scaler?

**Difficulty:** Intermediate · **Levels:** Architect/Principal, Fresher, SDE-1/2, SDE-2/3 · [↑ Vector Normalization with NumPy](<../../../../TOOLS/Python/Python_Language/Vector_Normalization_with_NumPy.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

#### Overview
When using StandardScaler or similar scalers in machine learning pipelines, it is crucial to fit the scaler on only the training data and then use this fitted scaler to transform both the training and test datasets. This practice ensures that your model's performance is evaluated on a fair ground, without leaking any information from the test set into the training process.

#### Triage
The core of the question revolves around understanding how scalers work in machine learning pipelines and why it’s essential not to fit a scaler on both training and testing data. We will assume that the reader has basic knowledge of scikit-learn's StandardScaler and its role in feature scaling. The key aspects to address are the concept of model fairness, avoiding information leakage, and the correct usage of scalers.

#### First-Principles Solution
To explain this effectively, let’s break it down step by step:

Understanding Data Leakage: Data leakage occurs when there is an unintentional use of information not available at the time of training a model during the training process itself. This can lead to overly optimistic performance estimates on the test data.

The Role of Scalers: A scaler, like StandardScaler, transforms the features in such a way that each feature has zero mean and unit variance. The transformation is based on the statistical properties (mean and standard deviation) of the data it processes.

Fitting vs. Transforming:

Fit: This step calculates the necessary statistics (e.g., mean, standard deviation) for scaling.
Transform: This step uses these calculated statistics to scale the features in the dataset accordingly.

Why Fit on Training Data Only:

If you fit the scaler on both training and test data, you risk using information from the test set during the fitting process. For example, if your test data has outliers that are not present in the training data, this could skew the calculated mean and standard deviation.
By fitting only on the training data, you ensure that the model’s evaluation is based on a fair comparison, as both the training and test datasets are processed under the same conditions.

Practical Example:

PYTHON
from sklearn.preprocessing import StandardScaler
from sklearn.model_selection import train_test_split
import numpy as np

# Sample data
X = np.random.rand(100, 2) * 100

# Splitting the dataset into training and test sets
X_train, X_test = train_test_split(X, test_size=0.2, random_state=42)

# Fitting the scaler on the training data only
scaler = StandardScaler().fit(X_train)

# Transforming both training and test datasets with the fitted scaler
X_train_scaled = scaler.transform(X_train)
X_test_scaled = scaler.transform(X_test)

Ensuring Fair Evaluation: By fitting the scaler on the training data only, we ensure that the scaling process is based solely on the characteristics of the training dataset. This way, both training and test datasets are treated fairly, reflecting a realistic performance evaluation.

#### Closing Summary
and Common Mistakes/Follow-Ups: Always remember to fit scalers or other preprocessing steps on the training set only. Failing to do so can lead to overfitting or incorrect model performance evaluations. In more complex scenarios, such as nested cross-validation, you might need to apply the same fitting process within each fold of the outer validation loop.

A common mistake is to fit scalers directly on the entire dataset (training + test) without splitting them first. This can introduce unwanted information from the test set into the training process, leading to overly optimistic performance metrics and incorrect model evaluations.

