---
type: "practice"
difficulty: "Intermediate"
---

# What is the cost of NOT having lineage during a production incident? Give a concrete before/after from your experience.

**Difficulty:** Intermediate · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Lineage for Dynamic SQL](<../../../../05_Data_Governance_and_Security/Governance/Lineage/Lineage_for_Dynamic_SQL.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When a production incident occurs, not having data lineage can lead to significant delays in identifying and resolving the root cause. This is especially true when dealing with complex data pipelines that involve multiple transformations and sources. In my experience, the cost of lacking lineage during such an incident can be measured both in terms of time wasted and potential business impact.

To illustrate this, let's consider a scenario where we have a marketing analytics system for a retail company. This system relies on customer data from various sources, including CRM systems, transactional databases, and external market reports. The transformations are intricate, involving ETL jobs that join multiple datasets, apply complex filters, and generate aggregated metrics.

Before Lineage

In the absence of lineage, when an incident occurs in the marketing analytics system—let's say the customer churn rate suddenly spikes—we have no clear visibility into which part of our data pipeline might be causing this anomaly. We would need to manually trace back through each ETL job and transformation step by step. This process can take hours or even days, during which the business is left in a state of uncertainty, potentially making uninformed decisions based on incorrect data.

After Lineage

With proper lineage established, our incident response is significantly more efficient. When the churn rate spike occurs, we can quickly pinpoint the exact ETL job responsible for the transformation that led to this anomaly. This allows us to focus our investigation efforts precisely where they are needed, potentially reducing resolution time from days to just a few hours.

Key Idea: The Cost of Not Having Lineage
Time-to-Resolution: Without lineage, identifying the root cause can take significantly longer due to manual tracing.
Business Impact: Delays in resolving issues can result in incorrect business decisions based on faulty data.
Operational Efficiency: Regular maintenance and documentation of lineage can improve overall operational efficiency.

#### Common Mistakes
and Follow-Ups
Misunderstanding the Scope: Failing to fully understand the scope of your data transformations can lead to incomplete or incorrect lineage tracking.
Over-Engineering: While it's important to have robust lineage, over-complicating the process with unnecessary tools or processes can introduce its own overhead.
Lack of Automation: Manual documentation of lineage is error-prone and time-consuming. Implementing automated lineage tracking can save significant effort.

**Comparison:**

| Transformation ID | Name                | Inputs                     | Outputs                    |
|-------------------|---------------------|----------------------------|----------------------------|
| 1                 | Customer Data Join  | CRM System, Transaction DB | Joined Customer Data      |
| 2                 | Aggregation         | Joined Customer Data       | Churn Rate Metrics        |
| Incident ID   | Timestamp             | Impact                    | Resolved     |
|--------------|-----------------------|---------------------------|--------------|
| 1            | 2023-06-01T14:30:00Z | Increased Churn Rate      | False        |

**Code:**

```python
# Example of a simplified data lineage metadata schema
lineage_metadata = {
    "transformations": [
        {"id": 1, "name": "Customer Data Join", "inputs": ["CRM System", "Transaction DB"], "outputs": ["Joined Customer Data"]},
        {"id": 2, "name": "Aggregation", "inputs": ["Joined Customer Data"], "outputs": ["Churn Rate Metrics"]}
    ],
    "incidents": [
        {"id": 1, "timestamp": "2023-06-01T14:30:00Z", "impact": "Increased Churn Rate", "resolved": False}
    ]
}
```

**Example input / output:**

```text
Before Incident:
- Manual tracing of data lineage took 8 hours to identify the root cause.
- Business decisions were made based on incorrect churn rate metrics.

After Incident:
- Automated lineage tracking reduced time-to-resolution to 2 hours.
- Correct business decisions could be made in a timely manner.
```

**Data:**

```json
{
    "lineage": [
        {
            "id": 1,
            "transformationName": "Customer Data Join",
            "inputs": ["CRM System", "Transaction DB"],
            "outputs": ["Joined Customer Data"]
        },
        {
            "id": 2,
            "transformationName": "Aggregation",
            "inputs": ["Joined Customer Data"],
            "outputs": ["Churn Rate Metrics"]
        }
    ],
    "incidents": [
        {
            "id": 1,
            "timestamp": "2023-06-01T14:30:00Z",
            "impact": "Increased Churn Rate",
            "resolved": false
        }
    ]
}
```

**Flow:**

1. No
2. Yes
3. Production Incident
4. Has Lineage?
5. Manual Tracing
6. Root Cause Identified (8 hours)
7. Automated Lineage Tracking
8. Root Cause Identified (2 hours)
9. Business Impact

