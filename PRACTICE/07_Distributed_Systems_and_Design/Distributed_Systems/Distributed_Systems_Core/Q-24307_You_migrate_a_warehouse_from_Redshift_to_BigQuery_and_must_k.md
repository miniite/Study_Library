---
type: "practice"
difficulty: "Expert"
---

# You migrate a warehouse from Redshift to BigQuery and must keep both in sync during the transition. Explain the dual-run reconciliation how you compare aggregates at scale and how you decide the acceptable discrepancy before cutover.

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Zero Downtime Sharding Migrations](<../../../../07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Distributed_Joins_in_MPP_Systems/Zero_Downtime_Sharding_Migrations.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

#### Overview

When migrating a warehouse from Redshift to BigQuery, ensuring data consistency is critical. This involves setting up a dual-run reconciliation phase where both systems operate in tandem, allowing real-time comparison of aggregates and identifying discrepancies before the cutover. The goal is to identify and resolve any inconsistencies so that the transition can be seamless.

#### Triaging the Question

The key aspects being probed are:

Dual-Run Reconciliation: Understanding how to set up a system where both Redshift and BigQuery operate simultaneously.
Aggregates Comparison at Scale: Techniques for efficiently comparing aggregates across large datasets.
Acceptable Discrepancy Thresholds: Determining when discrepancies are significant enough to affect the cutover decision.

#### First-Principles Solution
Step 1: Setting Up the Dual-Run Environment
Data Streaming: Implement a streaming pipeline that captures all data changes in Redshift and replicates them to BigQuery.
Synchronization: Ensure both systems are synchronized by using a watermark or timestamp for incremental updates.
Real-Time Monitoring: Use tools like Data Studio, Cloud Monitoring, or custom dashboards to monitor real-time discrepancies.
Step 2: Aggregates Comparison at Scale
Aggregation Strategy: Define the key metrics and aggregates that need to be compared (e.g., sum, average, count).
Sampling Techniques: Implement sampling strategies to reduce computational load while ensuring accuracy.
Data Partitioning: Use partitioning in BigQuery to optimize query performance and reduce latency.
Step 3: Discrepancy Analysis
Thresholds Determination: Define thresholds based on business requirements, acceptable error rates, and historical data variance.
Root Cause Analysis: Implement a system for identifying and investigating discrepancies.
Automated Resolution: Develop automated scripts to resolve minor discrepancies.
Step 4: Cutover Decision
Manual Review: Conduct manual reviews of high-discrepancy cases.
Impact Assessment: Assess the impact of discrepancies on business operations.
Final Cutover: Once all discrepancies are resolved or accepted, perform the cutover.

#### Closing Summary
and Common Mistakes/Follow-Ups

#### Closing Summary

The dual-run reconciliation process involves setting up a robust streaming pipeline, comparing aggregates at scale using efficient techniques, defining acceptable discrepancy thresholds, and ensuring thorough resolution before performing the final cutover. This approach ensures minimal disruption during the migration.

#### Common Mistakes
Ignoring Real-Time Discrepancies: Failing to monitor real-time discrepancies can lead to significant issues that may not be detected until after the cutover.
Overlooked Sampling Techniques: Inefficient sampling strategies can result in inaccurate comparisons and increased computational overhead.
Inadequate Thresholds: Setting too loose or strict thresholds can either delay the cutover unnecessarily or risk overlooking critical discrepancies.

#### Follow-Ups
How do you handle real-time data volume spikes during the dual-run phase?
What are some best practices for automating the resolution of minor discrepancies?
Can you discuss how you would scale this solution if the dataset size increases significantly?

**Comparison:**

| Key | Value (Redshift) | Value (BigQuery) | Discrepancy |
| --- | --------------- | ---------------- | ----------- |
| 1   | 10              | 9.5              | 4%          |
| 2   | 20              | 19.8             | 3%          |
| 3   | 30              | 29.7             | 3.0%        |

**Code:**

```python
# Example Python code for real-time monitoring and threshold setting
import pandas as pd

def monitor_discrepancies(redshift_data, bigquery_data):
    # Calculate discrepancies
    discrepancies = abs((redshift_data - bigquery_data) / redshift_data)

    # Define thresholds
    high_threshold = 0.1
    low_threshold = 0.05

    # Identify high-discrepancy cases
    high_discrepancies = discrepancies[discrepancies > high_threshold]

    return high_discrepancies

# Example usage
redshift_data = pd.DataFrame({'key': [1, 2, 3], 'value': [10, 20, 30]})
bigquery_data = pd.DataFrame({'key': [1, 2, 3], 'value': [9.5, 19.8, 29.7]})

high_discrepancies = monitor_discrepancies(redshift_data['value'], bigquery_data['value'])
print(high_discrepancies)
```

**Example input / output:**

```text
Input:
Redshift Data: [{'key': 1, 'value': 10}, {'key': 2, 'value': 20}, {'key': 3, 'value': 30}]
BigQuery Data: [{'key': 1, 'value': 9.5}, {'key': 2, 'value': 19.8}, {'key': 3, 'value': 29.7}]

Output:
High Discrepancies: [0.04999999999999964, 0.03]
```

**Data:**

```json
{
  "redshift_data": [
    {"key": 1, "value": 10},
    {"key": 2, "value": 20},
    {"key": 3, "value": 30}
  ],
  "bigquery_data": [
    {"key": 1, "value": 9.5},
    {"key": 2, "value": 19.8},
    {"key": 3, "value": 29.7}
  ]
}
```

**Flow:**

1. No
2. Yes
3. Real-time Data Streaming
4. Is Discrepancy > Threshold?
5. Monitor and Log
6. Investigate Root Cause
7. Automate Resolution ifPossible
8. Cutover Decision Review
9. Cutover Execution

