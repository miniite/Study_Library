---
type: "practice"
difficulty: "Expert"
---

# How do you set governance thresholds for how much autonomy you grant AIOps automations during a business-critical event window (e.g. peak-sale traffic) vs. normal operations, and who authorizes the change?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Governance Dashboards and Visibility](<../../../../11_AI_and_ML/Responsible_AI/AI_Governance/Governance_Dashboards_and_Visibility.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR

When setting governance thresholds for AI automation in AIOps systems during critical business events, it’s crucial to balance autonomy with oversight. You must define clear thresholds that align with risk tolerance and ensure there's a robust authorization process involving key stakeholders.

#### Triaging the Question

The core of this question is about defining and managing the levels of autonomy granted to AI-driven operations in AIOps systems during different operational phases (e.g., peak-sale traffic vs. normal operations). The interviewer is probing your understanding of risk management, decision-making processes, and stakeholder engagement.

#### First-Principles Solution
1. Understanding Critical Business Events
Identify Key Metrics: Determine what metrics (e.g., transaction volumes, system load) define critical business events.
Risk Tolerance Levels: Define risk tolerance levels that dictate the degree of autonomy allowed during these events.
2. Setting Governance Thresholds
Threshold Determination: Establish thresholds based on historical data and current operational norms. For instance, you might allow AI to operate autonomously up to a certain transaction volume but require manual intervention beyond it.
Sensitivity Analysis: Perform sensitivity analysis to understand the impact of varying autonomy levels.
3. Authorization Mechanisms
Approval Workflow: Implement an approval workflow where changes in autonomy thresholds require sign-off from key stakeholders (e.g., IT Ops, Risk Management).
Monitoring and Logging: Ensure continuous monitoring and logging of AI actions within these threshold limits to detect any anomalies.
Incident Response Plan: Have a clear incident response plan that activates when predefined thresholds are breached or when manual intervention is required.
4. Stakeholder Engagement
Communication Strategy: Develop a communication strategy to educate stakeholders on the benefits and risks of AI governance.
Regular Reviews: Conduct regular reviews and updates to governance policies based on feedback and changing business conditions.

#### Closing Summary

Setting effective governance thresholds for AIOps automations requires a nuanced approach that balances operational efficiency with risk management. By defining clear thresholds, implementing robust authorization mechanisms, and engaging key stakeholders, you can ensure responsible AI practices are followed during critical events while maintaining operational agility.

#### Common Mistakes/Follow-Ups
Over-Autonomization: Overly granting autonomy can lead to uncontrolled system behavior.
Under-Communication: Lack of clear communication channels with stakeholders can result in misunderstandings and missed opportunities for timely intervention.
Ignoring Feedback Loops: Failing to incorporate real-time feedback from operational data into governance policies can lead to suboptimal decision-making.

#### Follow-Up
Questions
How do you ensure that AI systems are auditable during critical events?
Can you walk through a scenario where a threshold breach led to an incident and how it was handled?

**Comparison:**

```
Thresholds	Autonomous Actions	Manual Checks Required
<= 10,000	Yes	No
> 10,000 & < 50,000 (Risk Tolerance = High)	Limited Autonomy with Manual Checks	Yes
>= 50,000	No	Yes
```

**Code:**

```python
# Example Python code for setting thresholds (hypothetical)
def set_autonomy_thresholds(transaction_volume, risk_tolerance):
    if transaction_volume < 10000:
        return "Autonomous"
    elif 10000 <= transaction_volume < 50000 and risk_tolerance == 'High':
        return "Limited Autonomy with Manual Checks"
    else:
        return "Manual Intervention Required"
```

**Example input / output:**

```text
Input:
- Transaction Volume: 12000
- Risk Tolerance: High

Output:
Autonomy Level: Limited Autonomy with Manual Checks
```

**Data:**

```json
{
  "thresholds": {
    "transaction_volume_threshold": 10000,
    "risk_tolerances": {
      "Low": {"max_transaction_volume": 25000, "autonomy_level": "Autonomous"},
      "High": {"min_transaction_volume": 10000, "max_transaction_volume": 50000, "autonomy_level": "Limited Autonomy with Manual Checks"}
    }
  },
  "stakeholders": ["IT Ops", "Risk Management"]
}
```

**Flow:**

1. Yes
2. High
3. No
4. Yes
5. No
6. Critical Event Detected
7. Is Transaction Volume >10,000?
8. Check Risk Tolerance
9. Transaction Volume < 50,000?
10. Manual Intervention Required
11. Limited Autonomy withManual Checks
12. Autonomous Actions

