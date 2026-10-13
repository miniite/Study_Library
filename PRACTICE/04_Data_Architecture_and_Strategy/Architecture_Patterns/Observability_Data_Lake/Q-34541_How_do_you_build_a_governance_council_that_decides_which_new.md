---
type: "practice"
difficulty: "Expert"
---

# How do you build a governance council that decides which new AIOps use cases are allowed to write derived data back into the shared lake?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Mesh vs Central Observability Lake](<../../../../04_Data_Architecture_and_Strategy/Architecture_Patterns/Observability_Data_Lake/Mesh_vs_Central_Observability_Lake.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To build a governance council for deciding which new AIOps use cases can write derived data back into the shared lake, we need to establish a robust framework that ensures compliance, security, and alignment with organizational goals. The key is to design a system of checks and balances through a structured process involving various stakeholders.

Firstly, let's break down the core components:

Stakeholders: This includes AIOps engineers, data scientists, business analysts, legal advisors, IT ops, compliance officers, and security experts.
Use Case Evaluation: Each proposed use case must be evaluated based on predefined criteria such as relevance, impact, risk, and alignment with broader organizational strategies.

#### Triaging the Question

The question is essentially about establishing a governance framework to ensure that only vetted and compliant AIOps use cases are allowed to write data back into the shared lake. The interviewer likely wants to see how you would structure this process from first principles, including key stakeholders, decision-making criteria, and potential technical and non-technical challenges.

#### First Principles Solution
Define Criteria: Establish clear, objective criteria for evaluating new use cases. These should cover aspects like:
Relevance to business objectives.
Data sensitivity and impact on data integrity.
Compliance with internal policies and external regulations.
Form a Governance Council: Assemble a council of key stakeholders who will make final decisions. This group should have diverse expertise to ensure a holistic view.
Process Workflow:
Use Case Submission: Engineers submit new use cases via a standardized form.
Initial Review: Initial review by technical and security experts to assess feasibility and risk.
Stakeholder Feedback: Collect feedback from the governance council members based on their respective expertise (e.g., legal, compliance).
Decision Making: The council convenes to deliberate and make a decision based on the reviewed information.
Communication Plan: Ensure transparent communication of decisions and reasons for approval or denial.

#### Common Mistakes
Overlooking the role of technical risk in decision-making.
Failing to include diverse stakeholder perspectives.
Not defining clear, measurable criteria that can be objectively evaluated.
Lack of a structured process that ensures consistency across use cases.

#### Closing Summary
+ Follow-Ups

Building such a governance council requires careful planning and execution. The key is to ensure that the process is robust yet flexible enough to handle new and evolving AIOps use cases while maintaining high standards of data integrity, compliance, and security.

Would you like me to elaborate on any specific aspects? For example, how would we structure the initial review phase or what might be some common challenges in ensuring adherence to regulatory requirements?

**Comparison:**

```
Use Case Name	Description	Relevance Score	Impact Level
Anomaly Detection	Monitor system performance for anomalies.	8	5
Predictive Maintenance	Forecast equipment failures based on historical data.	9	4
```

**Code:**

```python
# Example code for use case submission form (simplified)
class UseCase:
    def __init__(self, name, description, relevance_score, impact_level):
        self.name = name
        self.description = description
        self.relevance_score = relevance_score  # Score based on business objectives
        self.impact_level = impact_level  # Risk assessment

# Example use case submission process
use_cases = [
    UseCase(name="Anomaly Detection",
            description="Monitor system performance for anomalies.",
            relevance_score=8,
            impact_level=5),
    UseCase(name="Predictive Maintenance",
            description="Forecast equipment failures based on historical data.",
            relevance_score=9,
            impact_level=4)
]

for use_case in use_cases:
    print(f"Use Case: {use_case.name}, Relevance Score: {use_case.relevance_score}")
```

**Example input / output:**

| Use Case Name      | Description                                          | Relevance Score | Impact Level |
|--------------------|------------------------------------------------------|-----------------|--------------|
| Anomaly Detection  | Monitor system performance for anomalies.           | 8               | 5            |
| Predictive Maintenance | Forecast equipment failures based on historical data. | 9              | 4            |

**Data:**

```yaml
# Example YAML schema for use case submission form
useCaseSubmissionForm:
  - name: Anomaly Detection
    description: Monitor system performance for anomalies.
    relevanceScore: 8
    impactLevel: 5
  - name: Predictive Maintenance
    description: Forecast equipment failures based on historical data.
    relevanceScore: 9
    impactLevel: 4
```

**Flow:**

1. Use Case Submission
2. Initial Review
3. Stakeholder Feedback
4. Governance Council Decision
5. Communication of Decision

