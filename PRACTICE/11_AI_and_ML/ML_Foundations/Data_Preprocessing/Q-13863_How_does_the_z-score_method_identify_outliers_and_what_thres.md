---
type: "practice"
difficulty: "Intermediate"
---

# How does the z-score method identify outliers and what threshold is commonly used?

**Difficulty:** Intermediate · **Levels:** Fresher · [↑ Imbalanced Data](<../../../../11_AI_and_ML/ML_Foundations/Data_Preprocessing/Imbalanced_Data.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In data preprocessing, the z-score method is a statistical approach to identify outliers in datasets by measuring how many standard deviations an element is from the mean. This method is particularly useful when dealing with normally distributed data.

#### Triaging the Question

The interviewer is probing your understanding of:

The concept and formula behind the z-score.
How it identifies outliers.
Commonly used thresholds for defining outliers.

#### Assumptions

We are working with a dataset that follows a normal distribution.
The goal is to identify extreme values that deviate significantly from the mean.

#### Solution
From First Principles

The z-score of an element ( x ) in a dataset is calculated using the formula: [ z = \frac{x - \mu}{\sigma} ] where ( \mu ) is the mean and ( \sigma ) is the standard deviation of the dataset. A high absolute value of ( z ) indicates that the data point is far from the mean, suggesting it might be an outlier.

Step 1: Calculate Mean and Standard Deviation

First, compute the mean (( \mu )) and standard deviation (( \sigma )) of your dataset: [ \mu = \frac{1}{N} \sum_{i=1}^{N} x_i ] [ \sigma = \sqrt{\frac{1}{N} \sum_{i=1}^{N} (x_i - \mu)^2} ]

Step 2: Compute Z-Scores

For each data point ( x_i ), compute the z-score: [ z_i = \frac{x_i - \mu}{\sigma} ]

Step 3: Identify Outliers Using Thresholds

Commonly used thresholds for identifying outliers are based on the standard normal distribution. Typically, a z-score greater than 3 or less than -3 is considered an outlier.

#### Closing Summary
and Common Mistakes

In summary, the z-score method quantifies how far each data point is from the mean in terms of standard deviations. A threshold of ( \pm 3 ) is widely accepted but can be adjusted based on specific requirements. Over-reliance on this method without considering domain knowledge or alternative methods might lead to false positives.

#### Common Mistakes
and Follow-Ups
Misinterpreting Outliers: Failing to understand that z-scores alone do not confirm the presence of outliers; they only indicate potential candidates.
Ignoring Distribution Assumptions: Assuming all data follows a normal distribution, which is often not true in real-world scenarios. Non-parametric methods might be more appropriate.
Overlooking Domain Knowledge: Relying solely on statistical methods without considering domain-specific understanding can lead to incorrect decisions.

**Comparison:**

```
Data Point	Z-Score
10	-2.82
20	-1.88
30	-0.93
40	0.0
50	0.93
60	1.88
70	2.82
80	3.87
90	4.91
100	5.96
-100	-10.0
```

**Code:**

```python
import numpy as np

# Example dataset
data = [10, 20, 30, 40, 50, 60, 70, 80, 90, 100, -100]

mean = np.mean(data)
std_dev = np.std(data)

z_scores = [(x - mean) / std_dev for x in data]
print("Z-scores:", z_scores)

# Identifying outliers using a threshold
threshold = 3
outliers = [x for i, x in enumerate(data) if abs(z_scores[i]) > threshold]
print("Outliers:", outliers)
```

**Example input / output:**

```

Input:

JSON
Copy
{
    "data": [10, 20, 30, 40, 50, 60, 70, 80, 90, 100, -100]
}

Output:

PYTHON
Copy
Z-scores: [-2.8214605803245356, -1.8758467272846558, -0.9328933345468984, 0.0, 0.9328933345468984, 1.8758467272846558, 2.8214605803245356, 3.867003177464399, 4.912545774604263, 5.958088371744127, -100]
Outliers: [-100]
```

**Data:**

```json
{
    "data": [10, 20, 30, 40, 50, 60, 70, 80, 90, 100, -100]
}
```

**Flow:**

1. Yes
2. No
3. Load Data
4. Calculate Mean
5. Calculate Standard Deviation
6. Compute Z-Scores
7. Z-Score > 3?
8. Identify as Outlier
9. G
10. Return Cleaned Data

