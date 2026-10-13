---
type: "practice"
difficulty: "Deep"
---

# How do you observe and alert on a sudden change in the distribution of a categorical column (e.g. country codes)?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Distribution Change Alerts](<../../../../02_Data_Engineering/Data_Quality/Data_Observability/Distribution_Change_Alerts.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To effectively observe and alert on a sudden change in the distribution of a categorical column, such as country codes, we need to set up robust data observability mechanisms that can detect anomalies in real-time. This involves tracking changes in the frequency or proportion of different categories over time, and triggering alerts when these changes exceed predefined thresholds.

The key idea is to continuously monitor the distribution of the categorical column and compare it with historical baselines to identify any significant deviations. By setting up a monitoring pipeline that can handle large volumes of data efficiently, we ensure that we can quickly respond to issues before they escalate into larger problems.

**Comparison:**

```
Category	Current Distribution (%)	Baseline Distribution (%)	Change Threshold Exceeded?
US	50	25	No
CA	25	25	Yes
GB	15	25	No
CN	10	25	No
```

**Code:**

```python
import pandas as pd

def detect_distribution_change(current_data, baseline_data):
    # Calculate current distribution
    current_distribution = current_data['country_code'].value_counts(normalize=True)

    # Calculate baseline distribution
    baseline_distribution = baseline_data['country_code'].value_counts(normalize=True)

    # Identify categories with significant changes
    changed_categories = []
    for category in current_distribution.index:
        if not pd.isna(category) and (abs(current_distribution[category] - baseline_distribution.get(category, 0)) > threshold):
            changed_categories.append(category)

    return changed_categories
```

**Example input / output:**

```text
# Example input data
current_data = pd.DataFrame({
    'country_code': ['US', 'CA', 'GB', 'CN']
})

baseline_data = pd.DataFrame({
    'country_code': ['US', 'GB', 'CA', 'CN']
})

# Detect distribution change
changed_categories = detect_distribution_change(current_data, baseline_data)
print(changed_categories)  # Output: ['CA']
```

**Data:**

```json
{
  "current_data": [
    {"country_code": "US"},
    {"country_code": "CA"},
    {"country_code": "GB"},
    {"country_code": "CN"}
  ],
  "baseline_data": [
    {"country_code": "US"},
    {"country_code": "GB"},
    {"country_code": "CA"},
    {"country_code": "CN"}
  ]
}
```

**Flow:**

1. Yes
2. No
3. Data Ingestion
4. Preprocessing
5. Is Distribution Stable?
6. Alerts Suppressed
7. Distribution Changed Detected
8. Generate Alert
9. Notify Stakeholders
10. Monitor and Adjust Thresholdsas Needed

