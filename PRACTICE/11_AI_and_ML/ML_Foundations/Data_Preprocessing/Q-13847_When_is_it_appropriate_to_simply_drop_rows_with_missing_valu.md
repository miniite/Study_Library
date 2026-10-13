---
type: "practice"
difficulty: "Basic"
---

# When is it appropriate to simply drop rows with missing values instead of imputing them?

**Difficulty:** Basic · **Levels:** Fresher · [↑ Reject vs Quarantine Bad Rows](<../../../../11_AI_and_ML/ML_Foundations/Data_Preprocessing/Reject_vs_Quarantine_Bad_Rows.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When dealing with missing data in a dataset, one common decision point is whether to simply drop rows containing missing values or to impute those values using some method. Dropping rows can be an appropriate choice under certain circumstances, but it's important to understand the implications and potential biases introduced by this approach.

#### Triaging the Question

The question essentially asks when dropping rows with missing values (commonly known as "listwise deletion") is a suitable strategy. The interviewer might be probing your understanding of data integrity, bias in data representation, and the impact on subsequent model training or analysis.

#### Assumptions
We are dealing with numerical or categorical features.
The dataset is relatively clean but contains some missing values.
The decision to drop rows versus impute them hinges on the nature and extent of the missingness.
Step-by-Step Solution

Understanding Missing Data Mechanisms:

MCAR (Missing Completely at Random): The probability of a value being missing is independent of both observed and unobserved values.
MAR (Missing at Random): The probability of a value being missing depends on the observed data but not the unobserved data.
MNAR (Not Missing at Random): The probability of a value being missing depends on the unobserved values themselves.

Impact on Analysis:

Dropping rows can lead to significant loss of information, especially if many rows are missing.
If you drop too many rows due to missingness, your sample size might become too small for meaningful analysis or training models with sufficient data.

Conditions Favoring Dropping Rows:

Low Missingness: When the percentage of missing values is very low (e.g., less than 5% per feature), dropping them often results in minimal loss.
Irrelevant Features: If certain features contain a high proportion of missing values and are not critical for your analysis or model, you can drop those rows without significant impact.

Practical Example: Suppose we have a dataset with 1000 entries where two features, "Age" and "Income," each have 5% missingness. If these features are less critical to our analysis (e.g., for a churn prediction model), dropping the rows might be acceptable.

Bias Considerations:

Dropping rows can introduce bias if the data is not MCAR or MAR. For example, if older users tend to have more missing age information due to their reluctance to share, dropping these rows could skew your analysis towards younger demographics.
Ensure that you do not unintentionally drop a significant number of observations from certain subgroups.

#### Closing Summary

Dropping rows with missing values is appropriate when the proportion of missing data is minimal and the features in question are not critical for the analysis or model training. However, it's crucial to carefully evaluate whether dropping these rows will introduce bias and significantly affect your dataset size. Always consider alternative imputation methods if possible.

#### Common Mistakes/Follow-Ups
Mistake: Overlooking the impact on sample size when missingness is high.

#### Follow-Up
How do you handle features with a higher percentage of missing values? Would it be better to impute them, or drop them entirely?

**Comparison:**

| Original DataFrame | Cleaned DataFrame after dropping rows with missing values |
|--------------------|----------------------------------------------------------|
| Age                | [25, 30, NaN, 45]                                         | [25, 30, 45]                               |
| Income             | [50000.0, 60000.0, NaN, 70000.0]                          | [50000.0, 60000.0, 70000.0]                |
| Churn              | [1, 0, 1, 0]                                              | [1, 0, 0]                                  |

**Code:**

```python
# Example code snippet for handling missing data in Python using pandas

import pandas as pd

# Sample dataset
data = {'Age': [25, 30, None, 45],
        'Income': [50000, 60000, None, 70000],
        'Churn': [1, 0, 1, 0]}
df = pd.DataFrame(data)

# Dropping rows with missing values
cleaned_df = df.dropna()

print("Original DataFrame:")
print(df)
print("\nCleaned DataFrame after dropping rows with missing values:")
print(cleaned_df)
```

**Example input / output:**

```text
Original DataFrame:
   Age   Income  Churn
0   25  50000.0      1
1   30  60000.0      0
2   NaN     NaN      1
3   45  70000.0      0

Cleaned DataFrame after dropping rows with missing values:
   Age   Income  Churn
0   25  50000.0      1
1   30  60000.0      0
3   45  70000.0      0
```

**Data:**

```json
{
  "data": {
    "Age": [25, 30, null, 45],
    "Income": [50000, 60000, null, 70000],
    "Churn": [1, 0, 1, 0]
  }
}
```

**Flow:**

1. Yes
2. No
3. If minimal missingness
4. Analyze MissingnessMechanism
5. Is the missing data MCAR?
6. Dropping Rows is Acceptable
7. Impute Values Instead
8. Check Sample Size Impact
9. Drop Rows if Minimal
10. Consider Bias and SubgroupImpact
11. Final Decision: Drop or Impute

