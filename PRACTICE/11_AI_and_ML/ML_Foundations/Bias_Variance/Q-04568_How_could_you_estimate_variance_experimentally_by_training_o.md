---
type: "practice"
difficulty: "Intermediate"
---

# How could you estimate variance experimentally by training on multiple bootstrap samples?

**Difficulty:** Intermediate · **Levels:** Fresher · [↑ Ensembles and Variance Reduction](<../../../../11_AI_and_ML/ML_Foundations/Bias_Variance/Ensembles_and_Variance_Reduction.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In this intermediate-level ML Engineer role, we'll explore how to estimate variance experimentally using multiple bootstrap samples. Bootstrap sampling is a resampling technique that allows us to approximate the distribution of our model's performance metrics by repeatedly training models on different subsets of the data. The key idea here is that each subset provides a slightly different view of the underlying data distribution, which helps in understanding how much the model’s predictions vary.

To estimate variance using bootstrap samples, we need to:

Understand what variance means in this context.
Generate multiple bootstrap samples from our training dataset.
Train models on these bootstrap samples.
Evaluate each model's performance.
Analyze the variability of these performances to gauge variance.

Let’s break it down step by step:

#### Triage

The question is probing how we can quantify the variance of a machine learning model’s predictions using resampling techniques. This involves:

Understanding the concept of variance in the context of model performance.
Learning about bootstrap sampling and its utility.
Implementing an experimental setup to demonstrate this technique.

#### Assumptions
We have a labeled dataset for training and testing our models.
The data is representative of the problem domain, but may contain some noise or variability that we want to account for in our model’s performance.
Our goal is to estimate how much the model's predictions vary due to changes in its training data.

#### First Principles Solution

Generate Bootstrap Samples:

Resample the training dataset with replacement to create multiple (say, 100) bootstrap samples of the same size as the original dataset.

Train Models on Each Sample:

For each bootstrap sample, train a model using the same algorithm and hyperparameters.

Evaluate Model Performance:

Evaluate the performance of each trained model on a separate validation or test set to get a measure of its predictive power.

Analyze Variability:

Compute the standard deviation of these performance metrics across different models. This will give us an estimate of the variance in our model’s predictions.

#### Example
Code

Here's a simple Python code snippet that demonstrates this process:

PYTHON
import numpy as np
from sklearn.utils import resample
from sklearn.datasets import make_classification
from sklearn.model_selection import train_test_split
from sklearn.ensemble import RandomForestClassifier
from sklearn.metrics import accuracy_score

# Generate synthetic data for demonstration purposes
X, y = make_classification(n_samples=1000, n_features=20, random_state=42)
X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.3, random_state=42)

# Number of bootstrap samples and iterations
n_bootstrap_samples = 100
bootstrap_sample_size = len(y_train) // 5

# Train models on each bootstrap sample and evaluate their performance
performance_scores = []
for _ in range(n_bootstrap_samples):
    # Generate a bootstrap sample
    X_resampled, y_resampled = resample(X_train, y_train, replace=True, n_samples=bootstrap_sample_size)

    # Train a model on the current bootstrap sample
    model = RandomForestClassifier()
    model.fit(X_resampled, y_resampled)

    # Predict on test set and evaluate performance
    predictions = model.predict(X_test)
    score = accuracy_score(y_test, predictions)
    performance_scores.append(score)

# Calculate variance of the performance scores
variance = np.var(performance_scores)
print(f"Estimated Variance: {variance}")

#### Closing Summary
+ Common Mistakes/Follow-Ups

By following these steps, we can estimate the variance of a model's predictions using multiple bootstrap samples. This approach helps us understand how much our model’s performance might vary due to changes in its training data.

#### Common Mistakes

Not accounting for enough resamples, leading to biased estimates.
Using too small or unrepresentative subsets for resampling, which could skew the results.
Failing to properly evaluate and analyze the performance metrics from multiple runs.

#### Follow-Ups

Discussing how this method relates to other variance estimation techniques like cross-validation.
Exploring more complex models and their variance characteristics.

#### Cons
idering the computational cost of running many bootstraps, especially for large datasets or complex models.

**Code:**

```python
import numpy as np
from sklearn.utils import resample
from sklearn.datasets import make_classification
from sklearn.model_selection import train_test_split
from sklearn.ensemble import RandomForestClassifier
from sklearn.metrics import accuracy_score

# Generate synthetic data for demonstration purposes
X, y = make_classification(n_samples=1000, n_features=20, random_state=42)
X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.3, random_state=42)

# Number of bootstrap samples and iterations
n_bootstrap_samples = 100
bootstrap_sample_size = len(y_train) // 5

# Train models on each bootstrap sample and evaluate their performance
performance_scores = []
for _ in range(n_bootstrap_samples):
    # Generate a bootstrap sample
    X_resampled, y_resampled = resample(X_train, y_train, replace=True, n_samples=bootstrap_sample_size)

    # Train a model on the current bootstrap sample
    model = RandomForestClassifier()
    model.fit(X_resampled, y_resampled)

    # Predict on test set and evaluate performance
    predictions = model.predict(X_test)
    score = accuracy_score(y_test, predictions)
    performance_scores.append(score)

# Calculate variance of the performance scores
variance = np.var(performance_scores)
print(f"Estimated Variance: {variance}")
```

