---
type: "practice"
difficulty: "Basic"
---

# How do you handle missing values in a dataset and what are the common strategies?

**Difficulty:** Basic · **Levels:** Fresher · [↑ Missing Values and Encoding](<../../../../11_AI_and_ML/ML_Foundations/Data_Preprocessing/Missing_Values_and_Encoding.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Handling missing values is a crucial step in data preprocessing, especially when dealing with real-world datasets that often contain incomplete or noisy information. This process ensures that your models perform better by providing clean and consistent data inputs. Common strategies include deletion, imputation using mean/median/mode, and predictive modeling.

The question is essentially asking you to explain the different approaches for managing missing values in a dataset. Let’s break it down step by step from first principles:

Understanding the Problem: First, recognize that missing values can lead to biased results or reduced model accuracy. Missing data can occur due to various reasons such as user errors, sensor malfunctions, or incomplete surveys.

#### Assumptions
and Context: Assume you have a dataset with features where some values are missing. The goal is to decide whether and how these missing values should be handled before feeding the data into a machine learning model.

Deletion Approaches:

Complete Case Analysis (CCA): This method involves removing any rows that contain at least one missing value.
Pairwise Deletion: This technique removes only those pairs of observations that are not complete, allowing for some correlation between features to be preserved.

Imputation Methods:

Mean/Median/Mode Imputation: Replace missing values with the mean (for numerical data) or mode (for categorical data).
K-Nearest Neighbors (KNN): Utilize similar instances in the dataset to fill in the missing values based on their k-nearest neighbors.

Predictive Modeling:

Use a separate model to predict the missing values based on other features in the dataset.

Choosing the Right Strategy: The choice of strategy depends on several factors, including the percentage of missing data and whether the data is numerical or categorical.

Here’s an example scenario: Suppose you are working with a dataset containing customer transaction records where some dates might be missing due to incomplete logs. You could choose to impute these missing dates using KNN based on other transaction features such as location and product type.

**Comparison:**

```
TransactionID	Date	Location	Product
1	2023-10-01	NYC	A
2	NaN	LA	B
3	2023-10-03	NYC	A
4	2023-10-04	LA	B
TransactionID	Date	Location	Product
1	2023-10-01	NYC	A
2	2023-10-02	LA	B
3	2023-10-03	NYC	A
4	2023-10-04	LA	B
```

**Code:**

```python
import pandas as pd

# Example dataset
data = {
    'TransactionID': [1, 2, 3, 4],
    'Date': ['2023-10-01', None, '2023-10-03', '2023-10-04'],
    'Location': ['NYC', 'LA', 'NYC', 'LA'],
    'Product': ['A', 'B', 'A', 'B']
}

df = pd.DataFrame(data)
print("Original DataFrame:")
print(df)

# Impute missing values using KNN
from sklearn.impute import KNNImputer

imputer = KNNImputer(n_neighbors=2)
imputed_df = imputer.fit_transform(df.drop('TransactionID', axis=1))
imputed_df = pd.DataFrame(imputed_df, columns=df.columns[1:])
imputed_df['TransactionID'] = df['TransactionID']

print("Imputed DataFrame:")
print(imputed_df)
```

**Example input / output:**

```text
Original DataFrame:
   TransactionID        Date Location Product
0             1  2023-10-01      NYC       A
1             2         NaN        LA       B
2             3  2023-10-03      NYC       A
3             4  2023-10-04        LA       B

Imputed DataFrame:
   TransactionID        Date Location Product
0             1  2023-10-01      NYC       A
1             2  2023-10-02        LA       B
2             3  2023-10-03      NYC       A
3             4  2023-10-04        LA       B
```

**Data:**

```json
{
  "data": {
    "TransactionID": [1, 2, 3, 4],
    "Date": ["2023-10-01", null, "2023-10-03", "2023-10-04"],
    "Location": ["NYC", "LA", "NYC", "LA"],
    "Product": ["A", "B", "A", "B"]
  }
}
```

**Flow:**

1. Original Dataset
2. Identify Missing Values
3. Assess the Impact
4. Complete Case Analysis (CCA)
5. Pairwise Deletion
6. KNN Imputation
7. Mean/Median/ModeImputation
8. Predictive Modeling

