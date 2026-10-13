---
type: "practice"
difficulty: "Expert"
---

# Describe how you would fold ATAM into a continuous architecture governance model — running mini-evaluations at RFC time — so major risks are caught before the full formal review. What triggers a full ATAM versus a lightweight scenario check?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Turning ATAM Findings into Principles](<../../../../07_Distributed_Systems_and_Design/Software_Architecture/Trade_off_Analysis/Turning_ATAM_Findings_into_Principles.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In a continuous architecture governance model, integrating Architectural Trade-off Analysis Method (ATAM) at the Request for Comments (RFC) stage ensures that critical architectural decisions are evaluated early and often. By performing lightweight ATAMs during RFC time, we can catch major risks before they become unmanageable in more formal reviews. This approach not only streamlines the review process but also ensures alignment with organizational goals and technical standards.

#### Triaging the Question

The question is essentially about how to integrate a robust risk assessment mechanism into an existing architecture governance framework. It requires us to define when a full ATAM should be conducted versus a lightweight scenario check, thereby optimizing resource utilization and ensuring that only significant architectural trade-offs are deeply analyzed.

Key Assumptions
RFCs Exist: We assume that the organization has established a system for proposing new features or changes.
Architecture Governance Framework: There is an existing framework in place to manage architecture decisions.
Risk Thresholds Defined: Clear criteria exist for what constitutes a "major risk" versus a "minor risk."
Step-by-Step Solution
Define the RFC Process:
An RFC includes a detailed proposal outlining the new feature or change, its impact on existing systems, and proposed solutions.
Lightweight Scenario Check:
For minor risks, a quick scenario check can be performed to ensure that basic architectural principles are followed without delving into deep trade-offs.
Triggering Full ATAM:
A full ATAM is triggered when the RFC presents significant architectural challenges or introduces complex technical changes that could impact multiple systems.

#### Example
Workflow
Initial Screening:
An initial review of the RFC by a team member (e.g., a Technical Lead) to ensure it meets basic quality standards.
Lightweight Analysis:
If the RFC involves standard features or minor technical changes, perform a lightweight analysis to verify compliance with established guidelines and best practices.
Full ATAM for Major Risks:
For more complex RFCs, schedule a full ATAM session involving cross-functional teams (e.g., architects, developers, product managers) to thoroughly evaluate the proposed architecture.

#### Common Mistakes
Over-engineering Lightweight Checks: Failing to distinguish between minor and major risks can lead to unnecessary overburden on the lightweight checks.
Ignoring Early Indicators: Not recognizing early signs of potential issues in an RFC can result in significant problems being introduced into the system later.

#### Closing Summary
+ Follow-Ups

By integrating ATAM into the RFC process, we ensure that critical architectural decisions are made with minimal risk. This approach balances thorough evaluation with efficient use of resources, making our architecture governance model both effective and practical. Future discussions could explore automating parts of the lightweight scenario check to further streamline the process.

**Comparison:**

| RFC ID | Title                    | Complexity | Impact   | New Tech Used? | Lightweight Check Result | Full ATAM Required |
|--------|--------------------------|------------|----------|----------------|-------------------------|--------------------|
| 123    | Implement New Logging Feature | 2          | medium   | false           | true                     | false              |

**Code:**

```python
# Example Lightweight Check Function
def perform_lightweight_check(rfc):
    # Simple checks based on predefined criteria
    if rfc.complexity < 3 and rfc.impact == 'low':
        return True
    else:
        return False

# Example Full ATAM Trigger Logic
def trigger_full_atam(rfc):
    if rfc.impact >= 'high' or rfc.new_technology_used:
        return True
    else:
        return perform_lightweight_check(rfc)
```

**Example input / output:**

```text
Input:
{
  "rfc": {
    "id": 123,
    "title": "Implement New Logging Feature",
    "complexity": 2,  # Scale: 1-5 (1=Simple, 5=Very Complex)
    "impact": "medium",  # 'low', 'medium', 'high'
    "new_technology_used": false
  }
}

Output:
{
  "lightweight_check_result": true,
  "full_atam_required": false
}
```

**Data:**

```json
{
  "rfc": {
    "id": 123,
    "title": "Implement New Logging Feature",
    "complexity": 2,  # Scale: 1-5 (1=Simple, 5=Very Complex)
    "impact": "medium",  # 'low', 'medium', 'high'
    "new_technology_used": false
  }
}
```

**Flow:**

1. yes
2. no
3. Receive RFC
4. Is Complexity < 3 and Impact== 'low'?
5. Perform Lightweight Check
6. Trigger Full ATAM
7. Check Passed, No ATAMRequired
8. Evaluate with ATAM Team

