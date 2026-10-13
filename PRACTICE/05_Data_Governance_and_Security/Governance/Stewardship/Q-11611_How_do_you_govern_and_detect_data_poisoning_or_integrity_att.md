---
type: "practice"
difficulty: "Expert"
---

# How do you govern and detect data poisoning or integrity attacks on datasets that feed critical automated decisions such as credit or fraud scoring?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Governing Real-time Features](<../../../../05_Data_Governance_and_Security/Governance/Stewardship/Governing_Real-time_Features.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To ensure data governance and detection of data poisoning or integrity attacks in critical datasets, particularly those feeding automated decision systems like credit or fraud scoring, we need to establish a robust framework that includes data validation, anomaly detection, and continuous monitoring. This involves defining clear roles and responsibilities for data stewards, implementing rigorous data quality checks, setting up alert mechanisms, and leveraging machine learning techniques for real-time anomaly identification.

The question is essentially asking how to protect the integrity of our datasets by ensuring they are accurate, consistent, and reliable, especially when those datasets impact high-stakes automated systems. This means we need to cover several aspects: understanding what constitutes an attack, defining data quality metrics, implementing detection mechanisms, and having a response plan in place.

Firstly, let's clarify the assumptions:

We have access to metadata about the dataset (columns, types, expected ranges).
The system has a baseline of "normal" behavior established through historical data.
There are defined thresholds for acceptable data quality metrics.
We can implement a combination of rule-based and machine learning-based detection methods.

Now, let's break down the solution step-by-step:

Define Data Quality Metrics:

Accuracy (e.g., missing values, incorrect values).

#### Cons
istency (e.g., duplicate records, data type mismatches).
Completeness (e.g., null value counts).

Establish Baseline Behavior:

Use historical data to establish a baseline of normal behavior.
This can be done using descriptive statistics and visualizations like histograms, box plots.

Implement Data Validation Rules:

Create a set of validation rules based on the metadata (e.g., check for out-of-range values).

#### Example
rule: IF credit_score > 100 THEN flag_row AS 'Outlier'.

Continuous Monitoring and Alerting:

Set up real-time monitoring to detect deviations from the baseline.
Use a dashboard or alert system to notify relevant personnel when issues are detected.

Anomaly Detection with Machine Learning:

Train machine learning models on historical data to recognize patterns of normal behavior.
Implement these models in production to continuously check for anomalies.

#### Example
model: Isolation Forest for identifying outliers in real-time data streams.

Response Plan:

Develop a response plan that includes steps like data rollback, manual review, and communication protocols with stakeholders.
Ensure that the response is scalable and can handle various attack scenarios (e.g., targeted attacks vs. random noise).

Data Stewardship Roles:

Assign roles to data stewards responsible for maintaining and updating the validation rules and models.
Ensure regular audits of data quality metrics and model performance.

In summary, a comprehensive approach to data governance and integrity protection involves defining clear metrics, establishing baselines, implementing automated checks, continuously monitoring anomalies, and having robust response plans. This ensures that datasets used in critical decision systems remain reliable and secure from potential attacks.

#### Common mistakes
include over-reliance on manual processes, ignoring the importance of continuous learning models, and not involving data stewards in the process. Effective communication and collaboration are key to success.

**Comparison:**

```
Metric	Description
Accuracy	Percentage of incorrect or null values
Consistency	Presence of duplicates, type mismatches
Completeness	Missing value counts
```

**Code:**

```python
# Example Python code for data validation rule implementation

def validate_credit_score(score):
    if score < 0 or score > 100:
        return 'Outlier'
    return None

data = [
    {'credit_score': 85, 'risk_level': 'Low'},
    {'credit_score': -5, 'risk_level': 'High'},  # This will trigger an alert
]

for record in data:
    result = validate_credit_score(record['credit_score'])
    if result == 'Outlier':
        print(f"Alert: Outlier detected in credit score for {record}")
```

**Example input / output:**

```text
Input:
- Historical data on credit scores and risk levels.
Output:
- Data quality report with metrics like null counts, out-of-range values, duplicates, etc.
- Alerts when anomalies are detected (e.g., negative or very high credit scores).

Response plan:
1. Manual review of flagged records.
2. Communication to stakeholders about the incident and proposed corrective actions.
3. Update validation rules if necessary based on feedback from reviews.
```

**Data:**

```json
{
  "data_quality_metrics": {
    "accuracy": {
      "null_count": 0,
      "incorrect_values": ["NaN"]
    },
    "consistency": {
      "duplicate_records": false,
      "data_type_mismatches": []
    },
    "completeness": {
      "missing_percentage": 0
    }
  }
}
```

**Flow:**

1. Yes
2. No
3. Data Validation
4. Is Data Outlier?
5. Alert and Manual Review
6. Continue Monitoring
7. Update Rules and Models
8. Notify Stakeholders

