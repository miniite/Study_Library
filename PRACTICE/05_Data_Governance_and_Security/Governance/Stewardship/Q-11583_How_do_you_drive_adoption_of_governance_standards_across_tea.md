---
type: "practice"
difficulty: "Expert"
---

# How do you drive adoption of governance standards across teams that see it as overhead, using incentives rather than mandates?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Canonical Model vs Domain Autonomy](<../../../../05_Data_Governance_and_Security/Governance/Stewardship/Canonical_Model_vs_Domain_Autonomy.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Driving adoption of governance standards without mandates requires a strategic approach that aligns with team priorities and leverages intrinsic motivators. The key is to understand why teams might resist such standards and then craft a solution that not only addresses their concerns but also rewards them for compliance. In this scenario, the core idea is to frame governance as an enabler of success rather than an overhead burden.

To begin triaging the question, we need to recognize that resistance often stems from perceived inefficiencies or lack of value added by these standards. The role of a database developer architect is to identify and mitigate these concerns while simultaneously highlighting the benefits that governance can bring to the table. This involves crafting a balanced approach that includes both carrots (incentives) and sticks (non-mandatories), but ultimately focuses on creating an environment where compliance becomes a natural part of the team's workflow.

First, let’s break down the key steps:

Identify Pain Points: Understand why teams are resistant to governance standards.
Leverage Intrinsic Motivators: Frame governance as a tool for better data quality and reduced risk.
Craft Customized Solutions: Develop tailored incentive programs that resonate with each team’s unique needs.

Now, let's delve into the detailed solution:

Step 1: Identify Pain Points

Teams might see governance standards as overhead because:

They perceive it as time-consuming and not directly contributing to their primary goals.
There is a lack of clear benefits or value proposition.
They fear that compliance will lead to reduced agility.
Step 2: Leverage Intrinsic Motivators

To drive adoption, we need to highlight the intrinsic benefits of governance:

Improved Data Quality: Better data quality leads to more accurate insights and better decision-making.
Reduced Risk: Compliance with standards can prevent costly mistakes and regulatory issues.
Operational Efficiency: Automated compliance checks reduce manual workloads and improve operational efficiency.
Step 3: Craft Customized Solutions

Develop tailored incentive programs that align with team goals:

Performance Metrics: Introduce metrics for data quality, such as completeness rates or accuracy scores. Teams can be rewarded based on these metrics.
Recognition Programs: Implement recognition programs where teams and individuals who meet governance standards are publicly acknowledged.
Training and Development: Offer training sessions that not only teach the standards but also demonstrate how compliance leads to better outcomes.

#### Implementation
Example

Let’s consider a specific example:

Metric-Based Incentives: For a team responsible for customer data, metrics such as the percentage of records with complete fields could be tracked. A bonus pool can be allocated based on achieving these targets.
Recognition Programs: A monthly newsletter highlighting teams that have met or exceeded governance standards and sharing their success stories.
Training Sessions: Conduct regular training sessions to ensure all team members are up-to-date on the latest governance practices and understand how they benefit from compliance.

#### Closing Summary

By focusing on intrinsic motivators, customizing incentives, and clearly communicating the benefits of governance standards, we can drive adoption across teams. This approach not only addresses their concerns but also aligns with their goals, making governance a natural part of the workflow rather than an overhead burden.

#### Common Mistakes

Failing to understand the root causes of resistance.
Over-relying on extrinsic motivators without addressing underlying issues.
Implementing one-size-fits-all solutions that don’t resonate with individual teams.

#### Follow-Up
Questions
How do you measure the success of your governance initiatives?
Can you give an example of a team that was resistant to governance standards and how you turned it around?

**Comparison:**

```
Team	Metric	Target	Current Status
Sales	Data Quality Improvement	25%	10%
Sales	Compliance Rate	90%	85%
Sales	Reduced Risk Events	3	2
```

**Code:**

```sql
-- Example SQL for tracking data completeness metrics
SELECT COUNT(*) - SUM(CASE WHEN field IS NULL THEN 1 ELSE 0 END) / COUNT(*)
FROM customer_data;
```

**Example input / output:**

| Input | Output |
|-------|--------|
| Customer Data with Missing Fields | Complete Records Counted, Incomplete Field Counted for Compliance Metrics |

**Data:**

```json
{
  "team": "Sales",
  "metrics": {
    "data_quality_improvement": 25,
    "compliance_rate": 90,
    "reduced_risk_events": 3
  }
}
```

**Flow:**

1. Yes
2. No
3. Identify Pain Points
4. Does Team See Value?
5. Leverage Intrinsic Motivators
6. Develop Customized Solutions
7. Implement & Monitor
8. Review Success & Iterate
9. Communicate Benefits Clearly
10. Customize Incentive Programs
11. Engage Teams Regularly
12. Monitor & Adjust as Needed

