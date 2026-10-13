---
type: "practice"
difficulty: "Deep"
---

# How do you decide which of the six data quality dimensions — accuracy, completeness, consistency, timeliness, validity, uniqueness — actually deserve SLAs for a given dataset versus being nice-to-have checks?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Quality SLAs and Rule Testing](<../../../../02_Data_Engineering/Data_Quality/Quality_Frameworks_and_SLAs/Quality_SLAs_and_Rule_Testing.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When designing a data quality framework and service level agreements (SLAs) for a dataset, it's crucial to understand which of the six key dimensions — accuracy, completeness, consistency, timeliness, validity, and uniqueness — are critical enough to warrant formal SLAs. Here’s a step-by-step approach to decide on these priorities:

#### Overview
The goal is to identify which data quality metrics must be monitored closely and enforced strictly through SLAs because they directly impact business operations or decision-making processes.

#### Triage

Understanding the Question: The question revolves around determining the criticality of each data quality dimension for a specific dataset, deciding whether formal SLAs are necessary.
Key Assumptions: We assume that not all dimensions will have equal importance and that some may be more relevant or impactful depending on the use case.

#### First-Principles Solution

Business Impact Analysis (BIA): Begin by conducting a BIA to understand which metrics, if compromised, would significantly impact business operations, revenue, compliance, or customer satisfaction.
For instance, accuracy of financial data impacts accounting and billing processes directly.
Stakeholder Interviews: Engage with key stakeholders across the organization (e.g., finance, sales, HR) to gather insights on which dimensions are most critical for their roles.
Data-Driven Evaluation: Analyze historical data quality issues that have led to adverse business outcomes. For example, a lack of completeness in customer data can lead to missed sales opportunities.
Regulatory and Compliance Considerations: Ensure SLAs cover any regulatory or legal requirements specific to the industry (e.g., GDPR, HIPAA).
Operational Impact: Assess how each dimension affects day-to-day operations. Timeliness issues can disrupt workflows if critical data is not available when needed.

#### Closing Summary
+ Common Mistakes/Follow-Ups:

By focusing on the business impact and stakeholder needs, you can prioritize which dimensions should have formal SLAs.
A common mistake is treating all six dimensions equally without understanding their unique importance to different parts of the organization.

#### Follow-ups
might include: How do we monitor these quality metrics? What are the thresholds for acceptable performance under each SLA?

**Comparison:**

```
Metric	Business Impact Score (Avg)
Accuracy	7.0
Completeness	6.5
Consistency	4.5
Timeliness	8.0
Validity	4.0
Uniqueness	1.0
```

**Code:**

```python
# Example Python code to simulate BIA and data analysis steps
def conduct_bia(data_quality_metrics):
    critical_metrics = []
    for metric in data_quality_metrics:
        if is_critical(metric, business_impact_analysis()):
            critical_metrics.append(metric)
    return critical_metrics

def is_critical(metric, impact_scores):
    # This function would use a scoring system based on stakeholder feedback and historical issues
    return any(impact > 7 for impact in impact_scores.get(metric, [0]))

data_quality_metrics = ['accuracy', 'completeness', 'consistency', 'timeliness', 'validity', 'uniqueness']
critical_metrics = conduct_bia(data_quality_metrics)
print(critical_metrics)
```

**Example input / output:**

```text
Input:
- Data quality metrics with business impact scores from stakeholders

Output:
- Critical data quality dimensions that should have formal SLAs
```

**Data:**

```json
{
  "data_quality_metrics": [
    { "name": "accuracy", "business_impact_score": [8, 7, 6] },
    { "name": "completeness", "business_impact_score": [9, 5, 4] },
    { "name": "consistency", "business_impact_score": [7, 3, 2] },
    { "name": "timeliness", "business_impact_score": [6, 8, 9] },
    { "name": "validity", "business_impact_score": [5, 4, 3] },
    { "name": "uniqueness", "business_impact_score": [2, 1, 0] }
  ]
}
```

**Flow:**

1. Start
2. Conduct Business ImpactAnalysis
3. Identify Critical Metrics
4. Engage Stakeholders forFeedback
5. Evaluate Historical Issues
6. Assess Regulatory andCompliance Needs
7. Operational ImpactAssessment
8. Finalize SLA Prioritization
9. End

