---
type: "practice"
difficulty: "Intermediate"
---

# How would you detect outliers in a numeric column such as an unusually large transaction amount?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Detecting Unit Changes](<../../../../02_Data_Engineering/Data_Quality/Validation/Detecting_Unit_Changes.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To detect outliers in a numeric column like an unusually large transaction amount, we need to identify values that are significantly different from other data points in the same column. This can be crucial for ensuring data quality and preventing anomalies from skewing our analysis or models. The general approach involves using statistical methods to define thresholds around typical behavior.

Firstly, let's assume you have a dataset with transaction amounts as one of its columns. Our goal is to identify any transactions that stand out because they are unusually large compared to the rest.

#### Triaging the Question

The question is asking us to develop a method for detecting outliers in a numeric column. The interviewer may be probing our understanding of statistical methods, data cleaning techniques, and how we handle anomalies in real-world data. We need to make some assumptions about the nature of the data and the acceptable level of deviation from the norm.

Step-by-Step Solution
Calculate Basic Statistics: Start by calculating basic statistics such as mean, median, standard deviation, or interquartile range (IQR) for the transaction amounts.
Define Outlier Thresholds: Use these statistics to define thresholds that will help us identify outliers. Common methods include using Z-scores, IQR-based thresholds, or even domain-specific knowledge.
Apply Thresholds and Identify Outliers: Apply the defined thresholds to the data to flag any values that are considered outliers.

#### Example
Solution

For simplicity, let's use the IQR method as it is robust to extreme values and works well with skewed distributions.

Step 1: Calculate Basic Statistics

Assume we have a dataset transactions.csv with columns transaction_id and amount.

SQL
SELECT AVG(amount) AS avg_amount,
       MEDIAN(amount) AS median_amount,
       STDDEV_SAMP(amount) AS std_dev,
       PERCENTILE_CONT(0.75) WITHIN GROUP (ORDER BY amount) OVER () - PERCENTILE_CONT(0.25) WITHIN GROUP (ORDER BY amount) OVER () AS iqr
FROM transactions.csv;
Step 2: Define Outlier Thresholds

Using the IQR method, we can define outliers as values below ( \text{Q1} - 1.5 \times \text{IQR} ) or above ( \text{Q3} + 1.5 \times \text{IQR} ).

SQL
WITH stats AS (
    SELECT AVG(amount) AS avg_amount,
           MEDIAN(amount) AS median_amount,
           STDDEV_SAMP(amount) AS std_dev,
           PERCENTILE_CONT(0.75) WITHIN GROUP (ORDER BY amount) OVER () - PERCENTILE_CONT(0.25) WITHIN GROUP (ORDER BY amount) OVER () AS iqr
    FROM transactions.csv
)
SELECT avg_amount, median_amount, std_dev, iqr,
       median_amount - 1.5 * iqr AS lower_threshold,
       median_amount + 1.5 * iqr AS upper_threshold
FROM stats;
Step 3: Apply Thresholds and Identify Outliers

Using the thresholds from the previous step, we can flag outliers.

SQL
WITH stats AS (
    SELECT AVG(amount) AS avg_amount,
           MEDIAN(amount) AS median_amount,
           STDDEV_SAMP(amount) AS std_dev,
           PERCENTILE_CONT(0.75) WITHIN GROUP (ORDER BY amount) OVER () - PERCENTILE_CONT(0.25) WITHIN GROUP (ORDER BY amount) OVER () AS iqr
    FROM transactions.csv
),
thresholds AS (
    SELECT median_amount - 1.5 * iqr AS lower_threshold,
           median_amount + 1.5 * iqr AS upper_threshold
    FROM stats
)
SELECT t.transaction_id, t.amount, s.lower_threshold, s.upper_threshold,
       CASE WHEN amount < lower_threshold OR amount > upper_threshold THEN 'OUTLIER' ELSE 'NORMAL' END AS outlier_flag
FROM transactions.csv t JOIN thresholds s ON 1=1;

#### Closing Summary
+ Common Mistakes/Follow-Ups

In this example, we used the IQR method to detect outliers. Other methods like Z-scores can also be effective depending on the distribution of data. It's important to consider the specific characteristics of your dataset and possibly combine multiple approaches.

#### Common mistakes
include not properly defining the thresholds or applying them without understanding their implications. Always validate the results with domain knowledge and consider the business context.

#### Follow-up
questions might explore edge cases, such as handling very small datasets, dealing with missing values, or integrating outlier detection into a larger data pipeline.

**Comparison:**

| transaction_id | amount   | lower_threshold | upper_threshold | outlier_flag  |
| 1           | 50       | 28.33          | 97.67           | NORMAL        |
| 2           | 60       | 28.33          | 97.67           | NORMAL        |
| 3           | 70       | 28.33          | 97.67           | NORMAL        |
| 4           | 80       | 28.33          | 97.67           | NORMAL        |
| 5           | 90       | 28.33          | 97.67           | NORMAL        |
| 6           | 100      | 28.33          | 97.67           | NORMAL        |
| 7           | 10000    | 28.33          | 97.67           | OUTLIER       |

**Code:**

```sql
WITH stats AS (
    SELECT AVG(amount) AS avg_amount,
           MEDIAN(amount) AS median_amount,
           STDDEV_SAMP(amount) AS std_dev,
           PERCENTILE_CONT(0.75) WITHIN GROUP (ORDER BY amount) OVER () - PERCENTILE_CONT(0.25) WITHIN GROUP (ORDER BY amount) OVER () AS iqr
    FROM transactions.csv
),
thresholds AS (
    SELECT median_amount - 1.5 * iqr AS lower_threshold,
           median_amount + 1.5 * iqr AS upper_threshold
    FROM stats
)
SELECT t.transaction_id, t.amount, s.lower_threshold, s.upper_threshold,
       CASE WHEN amount < lower_threshold OR amount > upper_threshold THEN 'OUTLIER' ELSE 'NORMAL' END AS outlier_flag
FROM transactions.csv t JOIN thresholds s ON 1=1;
```

**Example input / output:**

```text
Input:
transactions.csv

Output:
transaction_id | amount   | lower_threshold | upper_threshold | outlier_flag
-----------------------------------------------------------
1             | 50       | 28.33           | 97.67          | NORMAL
2             | 60       | 28.33           | 97.67          | NORMAL
3             | 70       | 28.33           | 97.67          | NORMAL
4             | 80       | 28.33           | 97.67          | NORMAL
5             | 90       | 28.33           | 97.67          | NORMAL
6             | 100      | 28.33           | 97.67          | NORMAL
7             | 10000    | 28.33           | 97.67          | OUTLIER
```

**Data:**

```plaintext
transactions.csv:
+------------+----------+
| transaction_id | amount   |
+------------+----------+
| 1           | 50       |
| 2           | 60       |
| 3           | 70       |
| 4           | 80       |
| 5           | 90       |
| 6           | 100      |
| 7           | 10000    |
+------------+----------+
```

**Flow:**

1. Load Data
2. Calculate Statistics
3. Define Thresholds
4. Apply Thresholds to IdentifyOutliers
5. Output Results
6. Validate Results with BusinessContext

