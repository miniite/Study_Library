---
type: "practice"
difficulty: "Deep"
---

# How do you decide when data is 'too sensitive to put in the lake at all' versus 'protectable with governance', and what's your framework for that risk-appetite conversation with legal and the business?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Cross-border Access Governance](<../../../../05_Data_Governance_and_Security/Security_and_Privacy/Access_Governance/Cross-border_Access_Governance.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When deciding whether data should be placed directly into a data lake, or if it can be protected through governance mechanisms, we need to consider a comprehensive framework that balances business needs against risk appetites defined by legal and compliance requirements. This involves understanding the nature of the data, its value, potential risks, and the effectiveness of existing and proposed security measures.

#### Triage
What's Being Probed

The core of this question revolves around evaluating data sensitivity and determining appropriate storage and access governance strategies. The interviewer is probing your ability to articulate a risk-assessment framework, consider legal constraints, and effectively communicate with stakeholders about data protection trade-offs.

#### First-Principles Solution

Data Classification & Assessment:

Begin by classifying the data based on sensitivity levels (e.g., PII, financial information, intellectual property). Use standardized frameworks like GDPR or HIPAA for guidance.

Risk Analysis:

Evaluate the potential risks associated with each data type:
Disclosure Risks: How likely is it that sensitive data could be misused if exposed?
Impact of Breach: What would be the business and legal consequences of a breach involving this data?

Security & Governance Framework:

#### Cons
ider both technical and organizational measures to protect the data.
Technical Measures: Encryption, access controls, monitoring systems, etc.
Organizational Measures: Policies, training programs, incident response plans.

Stakeholder Engagement:

Engage with legal, compliance, and business stakeholders to align on risk appetites and acceptable levels of data exposure.

Decision Framework:

Use a decision matrix or similar tool to weigh the benefits and risks for each data type against the cost of securing it.

#### Example
Decision Matrix:
Data Type	Potential Risk (1-5)	Impact on Business (1-5)	Cost of Protection (1-5)	Total Score
Customer PII	5	4	2	9
Financial Data	3	3	4	7

#### Closing Summary

The decision to store or protect data in a data lake hinges on a thorough risk assessment and alignment with business objectives. A balanced approach considers technical, legal, and organizational factors to ensure that the most sensitive data is handled appropriately.

#### Common Mistakes/Follow-Ups
Overlooking Legal Requirements: Always ensure compliance with relevant regulations.
Ignoring Business Value: Failing to consider the strategic importance of certain datasets can lead to overly cautious decisions.
Lack of Stakeholder Engagement: Poor communication and alignment across teams can result in suboptimal security measures.

**Code:**

```python
# Example Python Code for Risk Assessment (simplified)
def assess_risk(data_type, potential_risk, impact_on_business, cost_of_protection):
    total_score = sum([potential_risk, impact_on_business, cost_of_protection])
    return total_score

# Sample Data
data_types = {
    "Customer PII": {"risk": 5, "impact": 4, "cost": 2},
    "Financial Data": {"risk": 3, "impact": 3, "cost": 4}
}

for data_type, details in data_types.items():
    total_score = assess_risk(data_type, **details)
    print(f"Total score for {data_type}: {total_score}")
```

**Example input / output:**

| Data Type    | Risk (1-5) | Impact on Business (1-5) | Cost of Protection (1-5) | Total Score |
|--------------|------------|--------------------------|-------------------------|-------------|
| Customer PII | 5          | 4                        | 2                       | 9           |
| Financial Data | 3       | 3                        | 4                       | 7           |

**Data:**

```json
{
  "Customer PII": {"risk": 5, "impact": 4, "cost": 2},
  "Financial Data": {"risk": 3, "impact": 3, "cost": 4}
}
```

