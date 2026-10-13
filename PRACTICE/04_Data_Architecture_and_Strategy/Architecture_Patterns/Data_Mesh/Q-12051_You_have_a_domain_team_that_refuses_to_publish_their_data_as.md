---
type: "practice"
difficulty: "Deep"
---

# You have a domain team that refuses to publish their data as a product because it slows their feature velocity; how do you resolve this without top-down mandate backfiring?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Data Mesh Principles](<../../../../04_Data_Architecture_and_Strategy/Architecture_Patterns/Data_Mesh/Data_Mesh_Principles.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In Data Engineering, leveraging Data Mesh principles can help foster collaboration and productivity while respecting team autonomy. The challenge here is to align domain teams with data sharing objectives in a way that doesn’t hamper their core development goals. Let’s break down the problem, explore possible solutions from first principles, and then conclude with practical insights.

TL;DR

The key idea is to design an ecosystem where data is shared responsibly but owned independently by the domain teams. By implementing Data Mesh practices, we can empower these teams to share their data in a way that aligns with their workflows and enhances broader organizational benefits without imposing mandates that might hinder their velocity.

#### Triaging the Question

The question revolves around balancing autonomy and collaboration in a data-driven organization. The goal is to resolve a conflict where domain teams are prioritizing feature development over data sharing, which poses challenges for cross-team projects and data-driven strategies.

#### Assumptions
Domain teams have valuable data that could benefit other parts of the organization.
The domain team’s primary focus is on delivering features quickly.
There's a need to share this data while respecting each team’s autonomy.

#### First Principles Solution

Understand Team Incentives: Start by understanding why the domain team prioritizes feature velocity. Often, it’s due to tight deadlines or resource constraints. We need to ensure that any solution we propose does not disrupt their core work flow.

Decentralize Data Ownership: Embrace the concept of data mesh where each domain owns its data but shares it under certain conditions. This decentralization helps in aligning incentives and reducing friction.

Incentivize Value Creation: Encourage teams to see the value in sharing their data by providing tangible benefits, such as improved analytics, better cross-team collaboration, or enhanced feature development through insights.

Build a Data Fabric: Develop a robust infrastructure that supports seamless data sharing and governance without impeding team velocity. This involves creating a platform where domain teams can publish their data on-demand.

Promote Cross-Team Collaboration: Facilitate communication between the domain teams and other parts of the organization to show how shared data can drive business outcomes. Use real-world examples or case studies that highlight successful data sharing projects.

Implement Gradual Rollout: Start with a pilot project where selected teams can share their data under strict governance rules. This allows for learning and feedback before scaling up.

Practical Steps
Step 1: Conduct a Workshop

Organize a workshop to engage the domain team in discussions about how they see value in sharing their data. Use this opportunity to explain the broader benefits of data mesh principles.

Domain Team
Value in Data Sharing
Cross-Team Collaboration
Improved Analytics
Feature Development Insights
Enhanced Velocity
Step 2: Design a Data Fabric Platform

Leverage cloud-based data fabric solutions that allow domain teams to publish their data on-demand. Ensure the platform supports versioning, access controls, and audit logs.

Data Publisher
Versioned Data
Access Controls
Auditable Logs
Step 3: Implement Governance Rules

Define clear governance rules that ensure data quality and compliance without overburdening the domain teams. Use automated tools to enforce these rules.

JSON
{
  "rules": [
    {
      "ruleType": "dataQuality",
      "description": "Ensure data meets minimum quality standards."
    },
    {
      "ruleType": "accessControl",
      "description": "Implement role-based access control."
    }
  ]
}

#### Closing Summary
and Common Mistakes

The approach should focus on creating a win-win scenario where domain teams see the value in sharing their data while maintaining their autonomy. Common mistakes include:

Imposing top-down mandates without understanding team dynamics.
Failing to provide clear incentives for data sharing.
Overcomplicating governance rules, which can deter participation.

By adopting Data Mesh principles and carefully designing solutions that respect team autonomy, you can foster a culture of collaboration and innovation while ensuring data is shared responsibly.

Common Follow-Ups
How would you measure the success of your approach?
What are some potential challenges in implementing this solution?
Can you provide an example of how Data Mesh has been successfully implemented in another organization?

**Comparison:**

```
Rule Type	Description
Data Quality	Ensures data meets minimum standards.
Access Control	Implements role-based access control.
Audit Logs	Tracks data usage and changes.
```

**Code:**

```python
# Example Python code for defining data quality rules
def validate_data_quality(data):
    if not check_min_quality(data):
        raise ValueError("Data does not meet minimum quality standards.")
```

**Example input / output:**

```text
Input: Data record with values outside the allowed range.
Output: Validation error message indicating data does not meet minimum quality standards.
Request: Team A publishes a new dataset.
Response: Dataset is validated against predefined rules and published if valid, or marked for review otherwise.
```

**Data:**

```json
{
  "dataQualityRules": [
    {
      "check": "minValueCheck",
      "params": {"minValue": 10}
    },
    {
      "check": "maxValueCheck",
      "params": {"maxValue": 100}
    }
  ]
}
```

**Flow:**

1. Domain Team
2. Data Publisher
3. Data Fabric Platform
4. Cross-Team Collaboration
5. Improved Analytics
6. Enhanced FeatureDevelopment

