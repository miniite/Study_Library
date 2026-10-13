---
type: "practice"
difficulty: "Intermediate"
---

# How would you use pandas to find columns with missing values and the percentage missing?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Detecting Unit Changes](<../../../../02_Data_Engineering/Data_Quality/Validation/Detecting_Unit_Changes.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address how we can use pandas to identify columns with missing values and calculate the percentage of missing data, let's break down the problem step by step.

First, it’s essential to understand what the question is asking: we need a method to detect which columns in our dataset have missing (NaN) values and compute the proportion of these missing values relative to each column. This task involves reading the data into a pandas DataFrame, using built-in functions, and calculating the required statistics.

#### Triaging the Question

The core of the question is about leveraging pandas’ capabilities for data cleaning and analysis. Specifically:

Identifying Columns with Missing Values: We need to check each column in our dataset.
Calculating Percentage of Missing Values: For each identified column, we will compute how much of its content is missing.

#### Assumptions
The input data is a CSV file or similar format that can be read by pandas.
The dataset is not excessively large to cause memory issues during processing.

#### First Principles Solution
Step 1: Import the Required Libraries
PYTHON
import pandas as pd
Step 2: Load the Data

Assume we have a CSV file named "data.csv" with some columns that might contain missing values.

PYTHON
df = pd.read_csv('data.csv')
Step 3: Find Columns with Missing Values

We can use pandas.isnull() to find missing (NaN) values and then sum these up for each column.

PYTHON
missing_values = df.isnull().sum()
print(missing_values)

This will give us a Series where the index is the column name, and the value is the count of missing values in that column.

Step 4: Calculate Percentage of Missing Values

To get the percentage, we divide the number of missing values by the total number of rows (length of the DataFrame) for each column.

PYTHON
percentage_missing = (df.isnull().sum() / df.shape[0]) * 100
print(percentage_missing)

This will provide us with a Series where each element represents the percentage of missing data in that particular column.

Step 5: Filter and Display Columns with High Missing Values

We may want to focus on columns with significant missing values, say more than a certain threshold. For example, let’s filter out columns with more than 20% missing values.

PYTHON
threshold = 20
high_missing_cols = percentage_missing[percentage_missing > threshold]
print(high_missing_cols)

#### Closing Summary
and Common Mistakes/Follow-Ups

This approach effectively finds columns with a significant amount of missing data, which is crucial for understanding the quality of your dataset. However, there are common pitfalls:

Memory Usage: If the DataFrame is very large, summing across all columns might be memory-intensive.
Data Imputation Techniques: Simply identifying columns with high missing values isn’t enough; we often need to decide how to handle these (e.g., impute or drop).

For follow-ups, you could discuss:

Data cleaning strategies based on the identified columns
Alternative methods for handling missing data
Scaling this process for larger datasets

