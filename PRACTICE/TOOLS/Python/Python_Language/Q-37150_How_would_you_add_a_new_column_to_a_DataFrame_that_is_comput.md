---
type: "practice"
difficulty: "Basic"
---

# How would you add a new column to a DataFrame that is computed from existing columns, for example a ratio of two features?

**Difficulty:** Basic · **Levels:** Architect/Principal, Fresher, SDE-1/2, SDE-2/3 · [↑ Pandas DataFrames](<../../../../TOOLS/Python/Python_Language/Pandas_DataFrames.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To add a new column to a pandas DataFrame that is computed based on existing columns, we'll follow these steps:

Import the necessary libraries.
Load or define the DataFrame with the required columns.
Compute the desired value using a lambda function or vectorized operations.
Assign this computed value as a new column in the DataFrame.

Let's go through each step in detail:

#### Triaging the Question

The question is asking how to add a new column to a pandas DataFrame where the values of this new column are derived from existing columns. This could involve simple arithmetic operations, such as calculating a ratio between two features.

#### Assumptions
and Key Idea
We assume you have a basic understanding of Python and pandas.
The calculation can be any valid operation on the existing data (e.g., sum, average, ratio).

#### Solution
from First Principles
Step 1: Import Libraries

First, we need to import pandas:

PYTHON
import pandas as pd
Step 2: Create or Load DataFrame

Let's assume we have a simple DataFrame with two columns feature1 and feature2:

PYTHON
data = {
    'feature1': [5, 7, 9],
    'feature2': [3, 4, 6]
}

df = pd.DataFrame(data)
Step 3: Compute the New Column

We want to create a new column ratio which is the ratio of feature1 to feature2. We can use vectorized operations provided by pandas for this:

PYTHON
df['ratio'] = df['feature1'].div(df['feature2'])
Step 4: Verify Output

To verify, we can print out the DataFrame:

PYTHON
print(df)

#### Closing Summary
and Common Mistakes/Follow-Ups

This approach is straightforward and efficient using pandas' built-in functions. However, it's important to note a few common mistakes:

NaN Handling: If feature2 contains zero values, this will result in a ZeroDivisionError. You can handle such cases by adding a check or using the where method:

PYTHON
df['ratio'] = df.apply(lambda row: row['feature1'] / row['feature2'] if row['feature2'] != 0 else None, axis=1)

#### Performance Considerations
While vectorized operations are generally efficient, for very large datasets, you might want to consider other optimizations or parallel processing.

#### Example
Output

Given the input DataFrame:

PYTHON
   feature1  feature2
0         5         3
1         7         4
2         9         6

The computed DataFrame would be:

PLAINTEXT
   feature1  feature2       ratio
0         5         3    1.666667
1         7         4    1.750000
2         9         6    1.500000

This example demonstrates a simple and effective way to add computed columns in pandas.

