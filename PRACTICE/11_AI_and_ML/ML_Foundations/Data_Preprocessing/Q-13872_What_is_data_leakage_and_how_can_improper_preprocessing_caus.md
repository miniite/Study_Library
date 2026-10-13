---
type: "practice"
difficulty: "Deep"
---

# What is data leakage and how can improper preprocessing cause it?

**Difficulty:** Deep · **Levels:** Fresher · [↑ Imbalanced Data](<../../../../11_AI_and_ML/ML_Foundations/Data_Preprocessing/Imbalanced_Data.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Summary: Data leakage occurs when information from outside the training dataset is used to create a model, leading to overly optimistic performance estimates on unseen data. Improper preprocessing steps like data imputation and feature engineering can inadvertently introduce such leaks if not handled carefully.

#### Triaging the Question
The question delves into understanding the concept of data leakage—a common pitfall in machine learning where information from outside the training dataset is used during model development, causing performance measures to be overly optimistic. The core idea here involves identifying and preventing such leaks through thoughtful data preprocessing techniques.

#### First-Principles Solution

Understanding Data Leakage:
Definition: Data leakage happens when the test or future datasets have some information that is not present in the training dataset, leading to models that perform well on historical data but poorly on new, unseen data.
Common Scenarios:
Using features from a future date to predict past events (e.g., predicting today's stock prices using tomorrow’s news).
Preprocessing Steps and Their Potential Risks:
Feature Engineering: Features should only use information known at the time of training.

#### Example
If you're building a model for predicting customer churn, avoid including future knowledge like "customer will cancel next month" as a feature.
Data Imputation: Handling missing values can lead to leakage if not done carefully. For instance, imputing last month's sales to predict this month’s sales could introduce forward-looking information into the training data.
Correctness Checks:
Perform sanity checks on preprocessed datasets by ensuring no future information is included in past data.
Validate models using cross-validation techniques that mimic real-world usage, such as k-fold time series validation.

#### Closing Summary
+ Common Mistakes/Follow-Ups: Improper preprocessing can lead to significant issues like overfitting and overly optimistic performance metrics. To avoid these problems, always ensure that the training data is a fair representation of what will be unseen in production. Regularly validate models using appropriate techniques such as time series cross-validation or k-fold validation to detect potential leaks.

#### Common mistakes
include:

Using future events to predict past outcomes.
Improper imputation strategies leading to information leakage.
Failing to perform thorough checks on preprocessed data for consistency and correctness.

#### Follow-ups
could involve more detailed discussions about specific preprocessing techniques, their trade-offs, and best practices.

**Comparison:**

```
Original Data	Imputed (Improper)	Correct Imputation
100	150	200
200	None	200
300	400	300
```

**Code:**

```python
# Example of improper imputation causing leakage
import pandas as pd

# Simulate a dataset with some missing values in the 'next_month_sales' column
data = {'previous_month_sales': [100, 200, 300],
        'next_month_sales': [150, None, 400]}
df = pd.DataFrame(data)

def impute_missing_values(df):
    # This function incorrectly uses future data to fill missing values
    df['previous_month_sales'].fillna(df['next_month_sales'], inplace=True)
    return df

# Correct approach would be:
def correct_imputation(df):
    df.fillna(method='ffill', inplace=True)  # Forward fill for simplicity, adjust as needed
    return df
```

**Example input / output:**

```text
# Example input and output of the imputation functions
imputed_df_improper = impute_missing_values(df)
print(imputed_df_improper)

correct_imputed_df = correct_imputation(df)
print(correct_imputed_df)
```

**Data:**

```json
{
    "previous_month_sales": [100, 200, 300],
    "next_month_sales": [150, null, 400]
}
```

**Flow:**

1. Yes
2. No
3. Original Data
4. Is feature future-looking?
5. Correct Preprocessing
6. Check for Leakage
7. Impute Correctly
8. Model Training

