---
type: "practice"
difficulty: "Expert"
---

# How do you measure whether the medallion architecture is actually delivering ROI vs the old pipelines? What metrics do you put in front of the CFO?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Medallion Layer Quality](<../../../../02_Data_Engineering/Storage_and_Lakehouse/Medallion_Architecture/Medallion_Layer_Quality.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When evaluating whether a Medallion Architecture (Bronze/Silver/Gold) delivers Return on Investment (ROI) versus traditional pipelines, we need to focus not just on technical benefits but also business outcomes. The key is to align the architecture with strategic business goals and measure both direct and indirect impacts. This involves setting up a comprehensive framework that includes performance metrics, cost savings, data quality improvements, and enhanced decision-making capabilities.

First, let's break down what Medallion Architecture aims to achieve: it separates data into three tiers—Bronze (raw), Silver (aggregated), and Gold (analyzed). Each tier serves different purposes and enables efficient data processing and storage. The goal is to reduce redundant ETL processes, improve data quality, and enable faster analytics.

Now, let's dive into the metrics we would put in front of the CFO:

Data Quality Metrics:

Data Completeness: Ensure all required fields are populated.
Data Consistency: Measure discrepancies between sources.
Data Accuracy: Assess how well data aligns with business rules and expectations.

Performance Metrics:

Query Response Time: Monitor how fast queries run on each tier.
Throughput: Track the amount of data processed per unit time.
Resource Utilization: Analyze CPU, memory, and storage usage.

Cost Savings:

Storage Cost: Compare the cost of storing raw vs. aggregated data.
Compute Cost: Evaluate the cost savings from optimized processing workflows.
Maintenance Cost: Measure reduced operational overhead due to simplified pipelines.

Operational Efficiency Metrics:

ETL Job Failures: Monitor and reduce failures in data ingestion processes.
Data Latency: Ensure data is available for analysis as soon as possible.
Data Availability: Confirm that critical data points are always accessible.

Business Impact Metrics:

Decision Quality: Assess how better data informs strategic business decisions.
Revenue Growth: Tie analytics insights to increased sales or market share.
Customer Experience Improvement: Measure impacts on customer satisfaction and retention.

To implement these metrics, we would set up a robust monitoring system that includes:

Dashboards: Create visual representations of key performance indicators (KPIs) using tools like Tableau or Power BI.
Alerting Mechanisms: Configure automated alerts for critical issues such as data quality violations or performance bottlenecks.
Regular Reporting: Generate monthly/quarterly reports summarizing the state of the architecture and its impact.

For example, a simple dashboard might look like this:

MARKDOWN
| Metric              | Current Status    | Target     |
|---------------------|-------------------|------------|
| Data Completeness   | 98%               | >= 99%     |
| Query Response Time | <5 seconds        | <=3 seconds|
| Storage Cost Savings| $10,000/month      | $12,000/month |

**Comparison:**

```
Metric	Current Status	Target
Data Completeness	98%	>= 99%
Query Response Time	<5 seconds	<=3 seconds
```

**Code:**

```python
# Example Python script to fetch and process data quality metrics
import pandas as pd

def get_data_quality_metrics():
    # Simulated data
    data = {
        'metric': ['Data Completeness', 'Query Response Time'],
        'current_value': [98, 4.5],
        'target_value': [99, 3]
    }
    df = pd.DataFrame(data)

    return df

metrics_df = get_data_quality_metrics()
print(metrics_df)
```

**Example input / output:**

```text
- **Input**: Raw data from various sources.
- **Output**: Processed and aggregated Silver/Gold level data, metrics dashboard, and alerts for critical issues.
```

**Data:**

```json
{
  "data": [
    {
      "metric": "Data Completeness",
      "current_value": 98,
      "target_value": 99
    },
    {
      "metric": "Query Response Time (seconds)",
      "current_value": 4.5,
      "target_value": 3
    }
  ]
}
```

**Flow:**

1. Yes
2. No
3. Raw Data Ingestion
4. Is Data Quality Good?
5. Aggregate Silver Level
6. Fix Issues, Re-ingest
7. Generate Gold-Level Analytics
8. Maintain Dashboards andAlerts
9. Monitor for Degradation

