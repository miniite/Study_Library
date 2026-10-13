---
type: "practice"
difficulty: "Deep"
---

# How would you detect that a numeric column has suddenly changed units for example rupees becoming paise?

**Difficulty:** Deep · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Detecting Unit Changes](<../../../../02_Data_Engineering/Data_Quality/Validation/Detecting_Unit_Changes.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To detect that a numeric column has suddenly changed its units, such as from rupees to paise, we need to ensure data quality and consistency in our dataset. This problem can be approached by first understanding the nature of the unit change (multiplicative factor) and then validating this change through statistical methods or predefined thresholds.

Triaging the Problem

The core issue here is to identify a sudden change in the units within a numeric column, which could indicate data corruption or an error in the data pipeline. The key steps are:

Identify the Multiplicative Factor: Rupees to paise involves multiplying by 100.
Statistical Analysis: Use statistical methods to detect anomalies that might suggest a unit change.
Thresholds and Rules: Define thresholds or rules to flag potential issues.

#### First Principles Solution

We start with a simple example where we have a column amount in the dataset, which should be in rupees but is suddenly found to be in paise.

Data Preparation:

Load the historical data (before any changes).
Load the recent data (where the unit change might occur).

Identify Multiplicative Factor:

Calculate the multiplicative factor by dividing current values by historic values.
Check if all values are close to 100, indicating a potential unit change.

Statistical Analysis:

Compute statistical measures like mean and standard deviation of the calculated factors.
Use these metrics to set thresholds for anomaly detection.

Rule-Based Validation:

Define rules based on domain knowledge or historical data patterns.
Check if any values deviate significantly from expected norms.
Detailed Steps

Load Data:

SQL
SELECT * FROM sales_data WHERE date BETWEEN '2023-01-01' AND '2023-06-30';

Calculate Multiplicative Factors:

PYTHON
import pandas as pd

# Load historical and recent data
df_hist = pd.read_sql_query("SELECT * FROM sales_data WHERE date BETWEEN '2023-01-01' AND '2023-06-30'", conn)
df_recent = pd.read_sql_query("SELECT * FROM sales_data WHERE date BETWEEN '2023-07-01' AND '2023-12-31'", conn)

# Calculate multiplicative factors
factor = df_recent['amount'] / df_hist['amount']

Statistical Analysis:

PYTHON
mean_factor = factor.mean()
std_dev_factor = factor.std()

# Define thresholds for anomaly detection
lower_bound = mean_factor - 2 * std_dev_factor
upper_bound = mean_factor + 2 * std_dev_factor

# Flag potential issues
anomalies = (factor < lower_bound) | (factor > upper_bound)

Rule-Based Validation:

PYTHON
# Define domain-specific rules
rule1 = df_recent['amount'] < 0.01  # Paise should be positive and less than a certain threshold

# Combine statistical and rule-based checks
potential_issues = anomalies | rule1

#### Closing Summary
+ Common Mistakes/Follow-Ups

#### Closing Summary
By comparing historical and recent data, using statistical methods to identify outliers, and applying domain-specific rules, we can detect sudden changes in units within a numeric column.

#### Common Mistakes
Failing to consider the multiplicative factor for unit conversion.
Not defining appropriate thresholds or rules based on the specific context.
Ignoring edge cases where no change might be detected due to data noise.
Common Follow-Ups
How would you handle situations where there are multiple potential changes?
What other statistical methods could be used besides mean and standard deviation?
How would you integrate this detection into a real-time monitoring system?

**Code:**

```python
import pandas as pd

# Load historical and recent data
df_hist = pd.read_sql_query("SELECT * FROM sales_data WHERE date BETWEEN '2023-01-01' AND '2023-06-30'", conn)
df_recent = pd.read_sql_query("SELECT * FROM sales_data WHERE date BETWEEN '2023-07-01' AND '2023-12-31'", conn)

# Calculate multiplicative factors
factor = df_recent['amount'] / df_hist['amount']

# Statistical analysis
mean_factor = factor.mean()
std_dev_factor = factor.std()

# Define thresholds for anomaly detection
lower_bound = mean_factor - 2 * std_dev_factor
upper_bound = mean_factor + 2 * std_dev_factor

# Flag potential issues
anomalies = (factor < lower_bound) | (factor > upper_bound)

# Rule-based validation
rule1 = df_recent['amount'] < 0.01

# Combine statistical and rule-based checks
potential_issues = anomalies | rule1
```

**Flow:**

1. No
2. Yes
3. No
4. Yes
5. Load Historical Data
6. Are there recent changes?
7. No Changes Detected
8. Calculate Multiplicative Factors
9. Are statistical thresholds met?
10. Rule-Based Validation
11. Potential Issues Identified
12. No Potential Issues
13. Log and Alert for Review

