---
type: "practice"
difficulty: "Expert"
---

# How do you build organizational buy-in for data quality investment when leadership sees it as a cost center and wants headcount on features instead?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Quality SLAs and Rule Testing](<../../../../02_Data_Engineering/Data_Quality/Quality_Frameworks_and_SLAs/Quality_SLAs_and_Rule_Testing.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To build organizational buy-in for data quality investment, especially in an environment where leadership views it as a cost rather than a strategic asset, you need to demonstrate its long-term value and align it with broader business objectives. Here’s how I would approach this:

Firstly, the problem is that leadership perceives data quality initiatives as non-value-added activities, focusing instead on features that directly generate revenue or user satisfaction. However, the reality is that poor data quality can lead to significant operational inefficiencies, regulatory issues, and reduced decision-making effectiveness. To change this mindset, you must:

Demonstrate Business Impact: Show how improving data quality directly contributes to business goals such as better customer insights, improved product development, enhanced compliance, and cost savings.
Build a Data Quality Framework: Establish a robust framework that includes clear SLAs (Service Level Agreements), KPIs (Key Performance Indicators), and metrics for tracking progress.
Communicate Value: Present data quality as an investment in the digital backbone of the organization—like building a strong foundation for a skyscraper.

**Comparison:**

```
SLA Metrics	Target (%)
Data Availability	99.5
Data Accuracy	98
KPI Metrics (Example)	Value (%)
Data Availability	97.5
Data Accuracy	96
```

**Code:**

```text
# Example SLA and KPI framework

class DataQualityFramework:
    def __init__(self, availability=99.5, accuracy=98):
        self.availability = availability
        self.accuracy = accuracy

    def set_slas(self, availability=None, accuracy=None):
        if availability is not None:
            self.availability = availability
        if accuracy is not None:
            self.accuracy = accuracy

# Example KPIs
class DataQualityKPIs:
    def __init__(self, data_availability, data_accuracy):
        self.data_availability = data_availability
        self.data_accuracy = data_accuracy

    @staticmethod
    def calculate_kpi(data_frame):
        availability = (data_frame.shape[0] / len(set(data_frame['timestamp']))) * 100
        accuracy = sum(data_frame['expected_value'] == data_frame['actual_value']) / len(data_frame) * 100
        return DataQualityKPIs(availability, accuracy)
```

**Example input / output:**

```text
# Example of setting SLAs and calculating KPIs

framework = DataQualityFramework()
framework.set_slas(availability=99.7, accuracy=99)
kpi = DataQualityKPIs.calculate_kpi(data_frame)

print(f"SLA Availability: {framework.availability}%")
print(f"SLA Accuracy: {framework.accuracy}%")
print(f"Current KPI Availability: {kpi.data_availability}%")
print(f"Current KPI Accuracy: {kpi.data_accuracy}%")
```

**Data:**

```json
{
    "data_quality_slas": {
        "availability": 99.5,
        "accuracy": 98
    },
    "kpi_metrics": [
        {"timestamp": "2023-10-01", "availability": 97.5, "accuracy": 96},
        {"timestamp": "2023-10-02", "availability": 99, "accuracy": 98.5}
    ]
}
```

**Flow:**

1. Yes
2. Data Quality ImprovementPlan
3. Is data quality critical forbusiness goals?
4. Quantify impact and createSLAs
5. Develop KPIs to track progress
6. Present to leadership with acost-benefit analysis
7. Secure investment bydemonstrating ROI throughdata quality improvements

