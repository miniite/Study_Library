---
type: "practice"
difficulty: "Deep"
---

# How do you construct a data platform maturity model to communicate progress to the board — what dimensions do you score and what's the anti-pattern of gaming the model?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Late and Out-of-order Data Strategy](<../../../../04_Data_Architecture_and_Strategy/Platform_Strategy/Data_Platform_Strategy/Late_and_Out-of-order_Data_Strategy.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When constructing a data platform maturity model, we aim to provide a clear, strategic framework that captures the evolution of our data capabilities from basic to advanced levels. This model serves as a communication tool between technical teams and non-technical stakeholders like the board of directors. The core dimensions typically include Data Governance, Data Management, Analytics & Business Intelligence, and Technology Infrastructure. However, it's crucial to avoid "gaming" the model by ensuring that scoring is objective and grounded in real-world performance.

#### Triage

The question asks us to outline a robust data platform maturity model and identify its anti-patterns. We need to focus on defining meaningful dimensions of maturity and highlight potential biases or manipulations (anti-patterns) within these metrics.

#### First-Principles Solution
Define Dimensions: Establish clear, measurable dimensions that align with business objectives.
Objective Scoring Criteria: Develop a transparent scoring system based on verifiable data points.
Regular Audits & Reviews: Implement regular assessments to ensure consistency and accuracy.
Stakeholder Engagement: Ensure all stakeholders understand the model’s purpose and value.
Dimensions of Maturity

Data Governance:

Definition: Policies, processes, and controls that ensure data quality and compliance.
Metrics: Data stewardship maturity, policy enforcement effectiveness, data lineage completeness.

Data Management:

Definition: Processes for acquiring, integrating, storing, securing, and managing data.
Metrics: Data catalog accuracy, data integration success rate, data security breaches.

Analytics & Business Intelligence (BI):

Definition: Utilization of data analytics to drive business decisions and insights.
Metrics: BI tool adoption, data-driven decision-making frequency, predictive modeling accuracy.

Technology Infrastructure:

Definition: Capabilities in tools, platforms, and services that support data operations.
Metrics: Cloud platform utilization, automation level of data processes, infrastructure reliability.
Anti-Patterns

Gaming the Model:

Symptom: Manipulating scores to make the current state look better than it is.

#### Example
Over-reporting the success rate of data governance without actual improvements in policies and enforcement.

Lack of Consistency:

Symptom: Inconsistent scoring across different teams or time periods, leading to unreliable comparisons.

#### Example
Using self-reported metrics for governance but relying on automated tools for technology infrastructure.

Ignoring Key Metrics:

Symptom: Focusing excessively on easily measurable aspects while neglecting crucial underlying issues.

#### Example
Emphasizing the number of data sources in a catalog without addressing data quality or lineage issues.

Misaligned Objectives:

Symptom: Scoring based on metrics that are not aligned with business goals, leading to misdirected efforts.

#### Example
Prioritizing technology infrastructure improvements over data governance and BI initiatives, which may not directly impact business outcomes.

#### Closing Summary

A well-constructed data platform maturity model is essential for aligning technical efforts with strategic business objectives. By defining clear dimensions and metrics, ensuring objective scoring, regular audits, and stakeholder engagement, we can build a robust framework that provides meaningful insights to the board of directors. Common pitfalls like gaming the model, lack of consistency, ignoring key metrics, and misaligned objectives need to be actively managed to maintain integrity and credibility.

#### Common Mistakes
Over-relying on self-reported data without rigorous validation.
Focusing too narrowly on technology aspects while neglecting governance and analytics.
Inconsistent scoring across time periods or teams.
Allowing biases or political motivations to skew the model's outcomes.

**Comparison:**

```
Dimension	Score
Data Governance	75%
Data Management	85%
Analytics & BI	90%
Technology Infra.	65%

Total Maturity Score: 80%
```

**Code:**

```python
# Example code for a basic maturity scoring system (simplified)
def score_maturity(dimensions):
    total_score = 0
    max_score = len(dimensions) * 100 # Assuming each dimension has a maximum of 100 points

    for dim, value in dimensions.items():
        total_score += value

    return round((total_score / max_score) * 100)
```

**Example input / output:**

```

Input:

JSON
Copy
{
  "dimensions": {
    "DataGovernance": 75,
    "DataManagement": 85,
    "AnalyticsBI": 90,
    "TechnologyInfrastructure": 65
  }
}

Output:

Total Score: 80%
```

**Data:**

```json
{
  "dimensions": {
    "DataGovernance": 75,
    "DataManagement": 85,
    "AnalyticsBI": 90,
    "TechnologyInfrastructure": 65
  }
}
```

**Flow:**

1. Yes
2. No
3. Define Dimensions
4. Are Metrics Objective?
5. Develop Scoring System
6. Implement Regular Audits
7. Refine Metrics & Scoring
8. Engage Stakeholders
9. Maintain Consistency

