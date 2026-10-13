---
type: "practice"
difficulty: "Intermediate"
---

# How would you preprocess CPU, memory, disk, and network metrics into a single multivariate feature vector per timestamp?

**Difficulty:** Intermediate · **Levels:** Fresher · [↑ Preprocessing Fundamentals](<../../../../11_AI_and_ML/ML_Foundations/Data_Preprocessing/Preprocessing_Fundamentals.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To preprocess and clean CPU, memory, disk, and network metrics into a single multivariate feature vector per timestamp, we need to ensure that the data is consistent, relevant, and ready for analysis. This process involves several steps, including handling missing values, normalizing or scaling features, and possibly applying domain-specific transformations.

#### Triaging the Question

The interviewer is probing your understanding of data preprocessing techniques, particularly in handling multivariate time-series data. They are likely interested in how you would address common issues like missing data, outliers, and ensuring that different metrics are comparable across timestamps. Key assumptions include:

The input data has a regular timestamp frequency.
The metrics can be transformed or normalized appropriately.

#### First-Principles Solution
Load and Inspect Data: Read the raw metrics from CSVs or a database, inspect for missing values, and understand the distribution of each metric.
Handle Missing Values: Impute or remove missing values based on domain knowledge or statistical methods.
Normalization/Scaling: Scale the features to ensure they are comparable across different metrics (e.g., range between 0 and 1).
Feature Engineering: Create a multivariate feature vector combining the normalized metrics for each timestamp.
Step-by-Step Solution
PYTHON
import pandas as pd
from sklearn.preprocessing import MinMaxScaler

# Load data
data = pd.read_csv('metrics.csv')

# Inspect and handle missing values
print(data.isnull().sum())
if 'CPU' in data.columns:
    data['CPU'].fillna(method='ffill', inplace=True)
if 'Memory' in data.columns:
    data['Memory'].fillna(method='bfill', inplace=True)

# Normalize features
scaler = MinMaxScaler()
features_to_scale = ['CPU', 'Memory', 'Disk', 'Network']
data[features_to_scale] = scaler.fit_transform(data[features_to_scale])

# Create a feature vector
feature_vectors = data[[col for col in data.columns if col not in ['timestamp']]]
print(feature_vectors.head())

#### Closing Summary
and Common Mistakes/Follow-Ups

In this solution, we handled missing values by forward-filling or backward-filling them based on the context. We then normalized the features using MinMaxScaler to ensure they are scaled appropriately for analysis. Finally, we created a multivariate feature vector combining all relevant metrics.

#### Common Mistakes

Forgetting to inspect and handle missing data.
Not normalizing or scaling features properly, leading to biased models.
Overfitting by applying complex transformations without justification.

#### Follow-Ups

How would you handle anomalies in the data?
What if the metrics have different units or scales?
Can you optimize this process for large datasets?

**Comparison:**

| CPU  | Memory | Disk | Network |
|------|--------|------|---------|
| 1.25 | 3072   | 8096 | 4000    |
| 1.25 | 3072   | 8096 | 4000    |
| CPU  | Memory | Disk | Network |
|------|--------|------|---------|
| 0.3333 | 0.5 | 0.5  | 0.5     |
| 0.3333 | 0.5 | 0.5  | 0.5     |

**Code:**

```python
import pandas as pd
from sklearn.preprocessing import MinMaxScaler

# Load data
data = pd.read_csv('metrics.csv')

# Inspect and handle missing values
print(data.isnull().sum())
if 'CPU' in data.columns:
    data['CPU'].fillna(method='ffill', inplace=True)
if 'Memory' in data.columns:
    data['Memory'].fillna(method='bfill', inplace=True)

# Normalize features
scaler = MinMaxScaler()
features_to_scale = ['CPU', 'Memory', 'Disk', 'Network']
data[features_to_scale] = scaler.fit_transform(data[features_to_scale])

# Create a feature vector
feature_vectors = data[[col for col in data.columns if col not in ['timestamp']]]
print(feature_vectors.head())
```

**Example input / output:**

```text
Before:
CPU,Memory,Disk,Network,timestamp
1.25,3072,8096,4000,2023-10-01 00:00:00
1.25,3072,8096,4000,2023-10-01 01:00:00

After:
CPU,Memory,Disk,Network
0.3333,0.5,0.5,0.5
0.3333,0.5,0.5,0.5
```

**Data:**

```plaintext
CPU,Memory,Disk,Network,timestamp
1.25,3072,8096,4000,2023-10-01 00:00:00
1.25,3072,8096,4000,2023-10-01 01:00:00
```

**Flow:**

1. Yes
2. No
3. Load Data
4. Inspect for Missing Values
5. Handle Missing Values?
6. Forward/Backward Fill
7. Continue
8. Normalize Features
9. Create Multivariate FeatureVector
10. Print or Use Feature Vectors

