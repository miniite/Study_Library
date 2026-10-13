---
type: "practice"
difficulty: "Deep"
---

# How do you decide, at intake, when a use case should NOT use AI at all, and how do you make that 'should-we-even-do-this' gate a respected part of the process rather than bypassed?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Platform vs Team Governance Boundaries](<../../../../11_AI_and_ML/Responsible_AI/AI_Governance/Platform_vs_Team_Governance_Boundaries.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

At the intake stage of any AI project, it's crucial to determine whether AI is the right approach. This decision involves evaluating ethical considerations, legal implications, and the overall feasibility and value proposition. My approach begins with a thorough assessment to ensure that we respect this 'should-we-even-do-this' gate as an integral part of our process, rather than bypassing it for expediency.

#### Triage
The Question at Hand

The core question here is how to identify use cases where AI should not be applied. This involves evaluating whether the problem can be solved more effectively with traditional methods or if there are significant risks that outweigh the benefits. The interviewer likely wants you to outline a systematic process for making these decisions, ensuring that ethical and compliance considerations take precedence.

#### First-Principles Solution
Define Ethical and Legal Criteria: Establish clear criteria based on industry standards, legal requirements, and organizational values.
Risk Assessment: Evaluate potential risks associated with AI, such as bias, privacy concerns, and security vulnerabilities.
Feasibility Analysis: Assess whether the problem is inherently suited for AI or can be solved more efficiently using traditional methods.
Stakeholder Input: Engage key stakeholders to gather diverse perspectives and ensure alignment on project goals.
Step-by-Step Solution

Define Ethical Criteria:

Ensure that any use of AI aligns with ethical principles such as fairness, accountability, transparency, privacy, and security.
Refer to frameworks like the IEEE Global Initiative for Ethical Considerations in Artificial Intelligence and Autonomous Systems or ISO 29110.

Risk Assessment Framework:

Identify potential risks associated with AI implementations, including bias, data quality issues, and model drift.
Use techniques such as the NIST Risk Management Framework (RMF) to systematically identify and mitigate these risks.

Feasibility Analysis:

Evaluate whether traditional methods can provide a satisfactory solution that meets business objectives more cost-effectively or with fewer ethical concerns.

#### Cons
ider the complexity of data preprocessing, model training, deployment, and maintenance.

Stakeholder Engagement:

Engage key stakeholders to ensure buy-in and address any concerns related to transparency, accountability, and compliance.
Incorporate feedback from legal departments, compliance teams, and affected communities to make informed decisions.

#### Closing Summary

By systematically evaluating ethical criteria, assessing risks, analyzing feasibility, and engaging stakeholders, we can ensure that the 'should-we-even-do-this' gate is respected as an integral part of our process. This approach not only mitigates potential risks but also enhances the trustworthiness and value proposition of AI solutions.

#### Common Mistakes/Follow-Ups
Ignoring Ethical Considerations: Failing to consider ethical implications can lead to biased or discriminatory outcomes.
Bypassing Stakeholder Engagement: Rushing through the process without involving key stakeholders can result in projects that lack support and buy-in.
Overlooking Feasibility Analysis: Overestimating the benefits of AI without considering traditional methods may lead to unnecessary complexity and costs.

**Comparison:**

```
Criteria	Description
Data Quality	Measures the accuracy and reliability of input data. Low quality can lead to inaccurate models.
Model Complexity	Indicates whether a complex model is necessary or if simpler methods suffice.
```

**Code:**

```python
# Example Python function for risk assessment (simplified)
def assess_risk(use_case_details):
    risks = []
    if use_case_details.get('data_quality') < 0.8:
        risks.append("Data Quality Issues")
    if use_case_details.get('model_complexity', False):
        risks.append("Model Complexity")
    return risks
```

**Example input / output:**

```text
Input: use_case_details = {
    "data_quality": 0.75,
    "model_complexity": True,
    "legal_requirements": False,
    "ethical_criteria": True
}
Output: Risks: ['Data Quality Issues', 'Model Complexity']
```

**Data:**

```yaml
use_case_details:
  data_quality: 0.75
  model_complexity: true
  legal_requirements: false
  ethical_criteria: true
```

**Flow:**

1. Define Ethical Criteria
2. Risk Assessment
3. Feasibility Analysis
4. Stakeholder Engagement
5. Risks Identified
6. Gate Decision Made
7. Solution Feasible?
8. Final Approval

