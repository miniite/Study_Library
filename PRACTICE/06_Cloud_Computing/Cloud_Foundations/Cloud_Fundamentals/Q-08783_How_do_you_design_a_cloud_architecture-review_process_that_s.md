---
type: "practice"
difficulty: "Expert"
---

# How do you design a cloud architecture-review process that scales, so hundreds of design decisions get appropriate scrutiny without a central architecture board becoming the bottleneck everyone routes around?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Control Plane Failures](<../../../../06_Cloud_Computing/Cloud_Foundations/Cloud_Fundamentals/Safe_Config_Rollouts/Control_Plane_Failures.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When designing a cloud architecture review process at scale, the key is to balance thoroughness with efficiency. A robust system must handle hundreds of design decisions while ensuring that critical architectural decisions are not overlooked and bottlenecks in decision-making are minimized. Here's how you can achieve this:

TL;DR: Implement a tiered review mechanism where initial reviews are handled by local teams, and only complex or high-risk designs need to escalate to a central architecture board.

#### Triage
The question is about designing an architecture review process that scales effectively. You are probing the ability to balance thoroughness with efficiency, while also considering how to handle varying levels of complexity and risk in different projects.

#### First-Principles Solution

Tiered Review System: Implement a tiered system where initial reviews are handled by local teams or project managers.
Automated Initial Reviews: Use automation tools for basic checks, such as security scans, compliance checks, and best practice validation.
Central Architecture Board: Reserve the central architecture board for more complex decisions that require cross-functional expertise.
Risk-Based Prioritization: Implement a risk assessment framework to prioritize reviews based on potential impact.

Detailed Steps:

Define Risk Levels: Establish a classification system (e.g., low, medium, high) for different design decisions based on their potential impact and complexity.
Local Reviews: Each project team should conduct an initial review using predefined checklists and automation tools.
Escalation Path: Create clear guidelines on when to escalate reviews. For example, if a local team identifies issues that they cannot resolve or if the design involves critical system components.
Central Board Meetings: Schedule periodic meetings for the central architecture board to discuss escalated cases. These should be limited in frequency and duration to avoid bottlenecks.

#### Closing Summary
+ Common Mistakes/Follow-Ups:

Ensure that the review process is scalable by leveraging automation and local teams.
Avoid over-relying on a single centralized team, which can become a bottleneck.
Regularly update the risk assessment framework based on new threats and organizational changes.

**Comparison:**

```
Initial Review by Local Teams
Local Dev Team
Central Architecture Board
Local Checklists
Automated Checks (Prowler)
Risk-Based Escalation
No Central Involvement
Fewer Issues, Less CentralInvolvement
Central Board Reviews CriticalDecisions
```

**Code:**

```python
# Example code for automated initial checks using Prowler (AWS)
import boto3

def run_prowler_scan(account_id):
    client = boto3.client('prowler', region_name='us-east-1')
    response = client.run_scan(
        account_ids=[account_id],
        check_ids=[
            'AWSShared Responsibility Model',
            'IAM User Without MFA'
        ]
    )
    return response
```

**Example input / output:**

| Design Decision | Risk Level | Initial Review | Central Board |
|-----------------|------------|----------------|---------------|
| New microservice deployment with minor API changes | Low | Local Dev Team | No |
| Data migration to a new storage solution | Medium | Local Dev Team | Yes (if issues arise) |
| Integration of legacy systems | High | Local Dev Team | Yes |

**Data:**

```json
{
  "risk_levels": {
    "low": ["No critical system components involved", "Minimal user impact"],
    "medium": ["Some critical components but manageable risk", "Moderate user impact"],
    "high": ["Highly critical components or significant user impact"]
  }
}
```

**Flow:**

1. Yes
2. No
3. Yes
4. No
5. New Design Decision
6. Initial Review - Local Teams
7. Is the Design Low Risk?
8. No Central Involvement
9. Risk Assessment Framework
10. Escalate to Central Board?
11. Central Architecture BoardMeeting
12. Return to Local Teams forFurther Review

