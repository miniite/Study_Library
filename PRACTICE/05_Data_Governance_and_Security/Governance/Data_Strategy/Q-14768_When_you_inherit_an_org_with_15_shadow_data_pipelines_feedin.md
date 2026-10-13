---
type: "practice"
difficulty: "Expert"
---

# When you inherit an org with 15 shadow data pipelines feeding ML models nobody governs, what's your first 90-day plan to bring them under governance without breaking prod?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Governing Synthetic Training Data](<../../../../05_Data_Governance_and_Security/Governance/Data_Strategy/Governing_Synthetic_Training_Data.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When inheriting a situation where 15 shadow data pipelines feed ML models and no formal governance exists, the first step is to understand the current state, assess risks, define goals, and develop a strategic plan. This involves several critical activities, including inventorying the pipelines, understanding their data sources, and mapping them against regulatory requirements.

#### Triage

The question is essentially asking how to systematically bring 15 shadow data pipelines under governance within a 90-day period without disrupting production. The interviewer is probing your ability to:

Understand the current state of the pipelines.
Identify key risks and dependencies.
Define a plan that balances short-term needs with long-term governance goals.
Ensure minimal disruption to existing operations.

#### Assumptions
You have limited access to documentation or previous knowledge about these pipelines.
There is no immediate regulatory requirement forcing action on this.
The ML models are in production and critical for business operations.
The organization has some form of governance framework, but it’s not currently applied.

#### First-Principles Solution
Day 1: Inventory and Assessment

Objective: Create a detailed inventory of the pipelines to understand their structure, data sources, and dependencies.

Gather Documentation: Review any available documentation, such as code repositories, Jira tickets, or Slack channels.
Survey Stakeholders: Conduct surveys or interviews with key stakeholders (e.g., developers, data scientists) to gather insights into the pipelines.
Automated Discovery: Use automated tools to discover and document existing pipelines.
Day 7: Risk Assessment

Objective: Identify potential risks associated with disrupting these pipelines without detailed understanding.

Impact Analysis: Perform a risk assessment by analyzing the dependencies of each pipeline on production models and other systems.
Criticality Evaluation: Determine which pipelines are critical for core business operations versus those that can be addressed later.
Day 14: Develop Governance Framework

Objective: Define a governance framework aligned with organizational needs.

Regulatory Compliance: Align the framework with relevant regulatory requirements (e.g., GDPR, HIPAA).
Data Quality Standards: Establish data quality standards and metrics.
Access Control Policies: Implement role-based access control policies.
Audit Trails: Define mechanisms for logging and auditing activities.
Day 21: Pilot Governance Implementation

Objective: Start implementing governance practices on a small subset of pipelines to identify any issues early.

Pilot Pipelines: Choose one or two representative pipelines to apply the new governance framework.
Monitor Impact: Continuously monitor these pipelines for any disruptions and gather feedback from stakeholders.
Day 30: Refine Governance Framework

Objective: Use insights gained from the pilot phase to refine the governance framework.

Adjust Policies: Make necessary adjustments based on feedback and observed impact.
Document Changes: Ensure all changes are well-documented for future reference.
Day 60: Full Rollout

Objective: Gradually roll out the new governance practices to all pipelines while maintaining production integrity.

Batched Implementation: Implement the governance framework in batches, starting with less critical pipelines.
Continuous Monitoring: Use continuous integration and monitoring tools to ensure smooth operation.
Day 90: Review and Feedback

Objective: Conduct a review of the initial rollout and gather feedback for future improvements.

Status Update Meetings: Hold regular meetings with stakeholders to discuss progress and address any concerns.
Feedback Loop: Create a feedback loop mechanism where stakeholders can provide ongoing input on governance practices.

#### Closing Summary
+ Common Mistakes/Follow-Ups

This approach ensures that you methodically bring the shadow pipelines under governance while minimizing disruption. Common mistakes include rushing into implementation without adequate planning, neglecting stakeholder engagement, or failing to account for data quality issues. Follow-ups might involve expanding the scope of your governance framework based on initial findings and continuing to monitor the health of these pipelines.

**Code:**

```python
# Example code snippet for automated pipeline discovery
import os
from subprocess import run

def discover_pipelines(directory):
    # Use a tool like `find` or equivalent to list files in the directory
    result = run(['find', directory, '-type', 'f', '-name', '*.py'], capture_output=True, text=True)
    return [file for file in result.stdout.split('\n') if file]  # Filter out empty strings

pipelines = discover_pipelines('/path/to/codebase')
print(pipeliens)
```

**Example input / output:**

| Pipeline | Action Taken |
|----------|--------------|
| Customer Segmentation | Documented data sources, identified dependencies. |
| Product Recommendations | Automated discovery script run, pipeline code reviewed. |

**Data:**

```json
{
  "inventory": {
    "pipeline_1": {
      "name": "Customer Segmentation",
      "data_sources": ["CRM", "Sales"],
      "dependencies": ["Marketing Automation"]
    },
    "pipeline_2": {
      "name": "Product Recommendations",
      "data_sources": ["E-commerce", "User Interactions"],
      "dependencies": []
    }
  }
}
```

