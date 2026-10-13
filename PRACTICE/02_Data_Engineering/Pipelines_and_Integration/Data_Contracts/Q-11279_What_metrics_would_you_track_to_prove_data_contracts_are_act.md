---
type: "practice"
difficulty: "Deep"
---

# What metrics would you track to prove data contracts are actually reducing incidents?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Data Contract Practice](<../../../../02_Data_Engineering/Pipelines_and_Integration/Data_Contracts/Data_Contract_Practice.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To ensure that data contracts are effectively reducing incidents, we need to track a variety of metrics that capture both the compliance and utility of the data. At a high level, these metrics should cover three main areas: contract adherence, data quality improvements, and incident reduction. By analyzing these metrics, we can understand whether our data contracts are making a tangible difference.

Firstly, contract adherence is crucial to ensure that all parties involved in the data pipeline are following the defined rules and standards. Metrics such as the percentage of data records that meet contract criteria, number of violations detected per day, and time taken to resolve violations can provide insights into how well contracts are being followed.

Secondly, data quality improvements should be monitored to assess whether enforcing data contracts leads to better data integrity. This includes tracking key performance indicators (KPIs) such as the completeness rate, accuracy rate, and consistency of data attributes across different datasets.

Lastly, incident reduction is the ultimate goal of implementing data contracts. By monitoring the number of incidents related to data quality issues before and after contract enforcement, we can evaluate the effectiveness of our data governance strategies. This includes tracking metrics such as the total number of incidents reported, average time taken to resolve an incident, and the percentage decrease in critical data-related errors.

In summary, a robust set of metrics will help us understand whether data contracts are leading to improved data quality and reduced incidents. By continuously monitoring these metrics, we can make informed decisions about our data governance policies and identify areas for further improvement.

#### Common mistakes
to avoid include tracking only superficial metrics without delving into the root causes of issues or neglecting long-term trends that might not be immediately apparent.

**Comparison:**

```
Metric	Description
Contract Adherence (%)	Percentage of data records that meet contract criteria
Data Quality KPIs	Metrics such as completeness rate, accuracy rate, and consistency across datasets
Incident Reduction Rate (%)	Percentage decrease in critical data-related errors over time
```

**Code:**

```python
# Example code snippet to track contract adherence metrics
def calculate_contract_adherence(records, criteria):
    violations = [record for record in records if not all(record[k] == v for k, v in criteria.items())]
    return (1 - len(violations) / len(records)) * 100

# Example code snippet to track data quality improvements
def calculate_data_quality_metrics(data_set):
    completeness_rate = sum(len(value) > 0 for value in data_set.values()) / len(data_set)
    accuracy_rate = # complex logic to calculate based on predefined rules
    return {
        "completeness_rate": completeness_rate,
        "accuracy_rate": accuracy_rate
    }
```

**Example input / output:**

```text
Input:
- Data Records: [
    {"field1": "value1", "field2": "value2"},
    {"field1": "", "field2": "value2"}, # incomplete record
    {"field1": "value1", "field2": ""}  # inconsistent record
]
- Criteria: {
    "field1": "value1",
    "field2": "value2"
  }

Output:
- Adherence Rate: 33.3% (2 out of 6 records meet criteria)
```

**Data:**

```json
{
  "data_records": [
    {"field1": "value1", "field2": "value2"},
    {"field1": "", "field2": "value2"}, # incomplete record
    {"field1": "value1", "field2": ""}  # inconsistent record
  ],
  "criteria": {
    "field1": "value1",
    "field2": "value2"
  }
}
```

**Flow:**

1. Yes
2. No
3. Data Contract Implementation
4. Are Contracts Adhered to?
5. Monitor Data Quality
6. Calculate Metrics
7. Analyze Incident Reduction
8. Review and Improve Policies
9. Identify Compliance Issues
10. Resolve Violations

