---
type: "practice"
difficulty: "Deep"
---

# A principal probe: you claim AIOps reduces MTTR, but faster detection of a cost anomaly doesn't help if remediation still needs a human with authority to change a budget. How do you close the detect-to-remediate loop for cost incidents specifically, and where does human-in-the-loop remain non-negotiable?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Consolidating Cost Tooling](<../../../../06_Cloud_Computing/Cloud_Cost/FinOps/Serverless_Cost_Visibility/Consolidating_Cost_Tooling.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the principal probe about closing the detect-to-remediate loop in FinOps with AIOps, we need to understand that while faster detection of cost anomalies is crucial, human intervention often remains essential due to the complexities and policy constraints involved. The key challenge lies in streamlining the communication between automated systems and human decision-makers.

#### Triaging the Question

The question probes how to integrate effective automation with necessary manual actions in a way that optimizes both efficiency and compliance. Specifically, it asks us to design a robust system that can quickly identify cost issues but also facilitate timely remediation by involving humans when needed. We must consider the current state of FinOps processes, potential bottlenecks, and the role of automation versus human oversight.

#### Assumptions
Cost Anomalies: These are defined as deviations from expected costs, which could be due to misconfigurations, inefficient resource usage, or other factors.
Human Decision-Makers: These individuals have the authority to make budgetary changes and enforce policy compliance.
Automated Detection: The AIOps system can identify cost anomalies with high accuracy but may lack the context for complex decision-making.

#### Solution
from First Principles
Step 1: Define the Workflow

The workflow consists of several stages:

Anomaly Detection: Continuous monitoring and real-time analysis by AIOps to flag potential cost issues.
Impact Assessment: Evaluation of the impact of identified anomalies on business operations and financial health.
Remediation Suggestion: Automated recommendation systems that propose potential corrective actions based on historical data and best practices.
Human Approval/Action: Decision-makers review suggestions, make necessary adjustments, and enforce changes.
Step 2: Designing the Workflow

To ensure effective communication between automated systems and humans, we can leverage a combination of automation and user interfaces (UIs) for decision-making:

Automated Alerts: Send notifications to relevant teams or individuals via email, Slack, or other platforms when anomalies are detected.
Remediation Suggestion Engine: Use machine learning models trained on historical data to predict the most likely corrective actions. Provide these suggestions to human decision-makers through a user-friendly interface.
Human Decision Interface: A web portal where authorized personnel can view detailed information about cost anomalies, review suggested remediations, and make final decisions.
Step 3: Integrating Automation with Human Oversight

To bridge the detect-to-remediate gap:

Decision Support System (DSS): Develop a DSS that integrates real-time data from cloud providers and internal metrics to provide context-aware recommendations.
Policy Compliance Checker: Ensure that suggested actions comply with predefined financial policies and governance rules.
Risk Assessment Tools: Provide tools that help decision-makers assess the risk associated with different remediation options.
Step 4: Ensuring Human-in-the-Loop

Human oversight is non-negotiable in scenarios involving:

High-Risk Decisions: Large-scale budget reallocations or changes that could impact multiple services.
Complex Policy Enforcement: Situations where policy enforcement requires human judgment, such as prioritizing service level agreements (SLAs) over cost-saving measures.
Exception Handling: Unusual circumstances that require manual review and approval.

#### Closing Summary

By integrating automated anomaly detection with structured decision support systems and user-friendly interfaces, we can significantly reduce the time between detecting a cost anomaly and taking corrective action. However, human oversight remains essential for handling high-risk decisions, complex policy enforcement, and exceptional cases.

#### Common Mistakes/Follow-Ups
Overreliance on Automation: Failing to involve humans in critical decision-making processes.
Lack of Contextual Information: Not providing sufficient context to human reviewers, leading to suboptimal decisions.
Insufficient Policy Integration: Ignoring existing policies and governance rules during automated remediation suggestions.

**Comparison:**

```
Anomaly	Cost	Expected Cost	Remediation Suggestions
S3	$1200.00	$800.00	Review usage patterns
ECS	$6000.00	$5000.00	Scale down instances
```

**Code:**

```python
# Example Code for Anomaly Detection and Remediation Suggestion

def detect_anomalies(data):
    # Implement anomaly detection logic using statistical models or machine learning
    anomalies = find_anomalies(data)
    return anomalies

def suggest_remediations(anomalies, policy):
    # Use historical data to generate remediation suggestions that comply with policies
    remediations = recommend_actions(anomalies, policy)
    return remediations
```

**Example input / output:**

```text
Input:
- Data on cloud services costs vs. expected costs

Output:
- List of anomalies identified
- Remediation suggestions with reasons and actions

Example Input/Output:
Input: {"service": "S3", "cost": 1200.0, "expected_cost": 800.0}
Output: {"anomalies": [{"service": "S3", "cost": 1200.0, "expected_cost": 800.0}], "remediations": [{"action": "Review S3 usage patterns", "reason": "High cost discrepancy"}]}
```

**Data:**

```json
{
  "anomalies": [
    {"service": "S3", "cost": 1200.0, "expected_cost": 800.0},
    {"service": "ECS", "cost": 6000.0, "expected_cost": 5000.0}
  ],
  "remediations": [
    {"action": "Scale down ECS instances", "reason": "Over-provisioning detected"},
    {"action": "Review S3 usage patterns", "reason": "High cost discrepancy"}
  ]
}
```

**Flow:**

1. Detect Anomalies
2. Suggest Remediations
3. Apply Remediation Suggestion
4. Review and Approve
5. Apply Changes
6. Close Ticket
7. Human Review Required
8. Re-Suggest/Approve

