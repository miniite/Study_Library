---
type: "practice"
difficulty: "Intermediate"
---

# How would you apply different preprocessing steps to numerical and categorical columns within a single pipeline?

**Difficulty:** Intermediate · **Levels:** Fresher · [↑ Preprocessing Fundamentals](<../../../../11_AI_and_ML/ML_Foundations/Data_Preprocessing/Preprocessing_Fundamentals.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR: To preprocess both numerical and categorical data in a single pipeline, we need to apply distinct transformations to each type of column. We'll use techniques like scaling, normalization, one-hot encoding, and handling missing values. The key is understanding the nature of each type and selecting appropriate preprocessing steps.

The question asks about applying different preprocessing methods for numerical and categorical columns within a unified pipeline. This requires us to first identify the types of data (numerical vs. categorical) in our dataset, then apply specific transformations tailored to each. For numerical features, common techniques include scaling, normalization, and handling missing values. Categorical features often require one-hot encoding or similar methods.

#### Triage
The core of this question is about understanding how to handle different types of data within a single preprocessing pipeline. We need to consider the following:

Identifying the type of each column.
Selecting appropriate preprocessing steps for numerical and categorical columns.
Ensuring that these steps are applied in a way that maintains the integrity of the original data while preparing it for modeling.

#### First-Principles Solution

Identify Data Types: First, we need to categorize the columns into either numerical or categorical based on their characteristics (e.g., integer values, continuous values, categories).

Numerical Columns Preprocessing:

Scaling/Normalization: Apply techniques like Min-Max scaling or Standardization.
PYTHON
from sklearn.preprocessing import MinMaxScaler

scaler = MinMaxScaler()
scaled_data = scaler.fit_transform(numerical_columns)

Categorical Columns Preprocessing:

One-Hot Encoding: Convert categorical variables into dummy/indicator variables.
PYTHON
from sklearn.preprocessing import OneHotEncoder

encoder = OneHotEncoder(sparse=False)
encoded_data = encoder.fit_transform(categorical_columns)

Handling Missing Values:

Impute missing values using methods like mean, median, or mode for numerical columns and most frequent category for categorical columns.
PYTHON
from sklearn.impute import SimpleImputer

imputer = SimpleImputer(strategy='mean')
imputed_data = imputer.fit_transform(data)

Combining Preprocessing Steps: Use pipelines to combine these steps efficiently.

Data Transformation Pipeline:

Create a pipeline that applies the necessary transformations in sequence.
PYTHON
from sklearn.pipeline import Pipeline

preprocessing_pipeline = Pipeline([
    ('imputer', SimpleImputer(strategy='mean')),
    ('scaler', MinMaxScaler()),
    ('encoder', OneHotEncoder(sparse=False))
])

transformed_data = preprocessing_pipeline.fit_transform(data)

Validation and Testing: Ensure the pipeline works correctly by testing it with sample data.

Edge Cases:

Be mindful of edge cases, such as handling non-numeric entries in categorical columns or dealing with very large datasets that might require more efficient scaling methods.

Scaling Up:

For larger datasets, consider using more advanced techniques and libraries like Dask for distributed processing.

#### Closing Summary
By understanding the nature of numerical and categorical data, we can apply appropriate preprocessing steps within a unified pipeline to ensure that both types of columns are appropriately transformed before feeding them into machine learning models. Common pitfalls include not correctly identifying column types or overfitting by applying too many transformations.

#### Common Mistakes/Follow-Ups

Failing to properly identify and handle missing values.
Overly complex preprocessing steps leading to data leakage.
Not validating the pipeline with test data.

