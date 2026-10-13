---
type: "practice"
difficulty: "Deep"
---

# How do you evaluate whether to buy a vendor's premium support tier versus building an internal database SRE capability?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Database Vendor and Platform Choices](<../../../../01_Databases_and_SQL/Database_Strategy/Vendor_and_Tooling/Database_Vendor_and_Platform_Choices.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To decide between buying a vendor’s premium support tier and building an internal Database SRE (Site Reliability Engineering) capability, you need to evaluate several factors that span cost, operational efficiency, risk management, and strategic alignment with your organization's goals. This evaluation involves understanding the current and future state of your database infrastructure, assessing the skills and resources required for both options, and considering the potential impact on your business continuity.

The core question is: How do we balance the trade-offs between external support that provides immediate resolution and internal expertise that can be tailored to our specific needs?

#### Triage
Vendor Support Tier Evaluation: Assess what premium support tiers offer, including response times, root cause analysis, proactive monitoring, and any additional services.
Internal SRE Capability Assessment: Evaluate the skills, resources, and tools needed for an internal SRE team to manage your database infrastructure effectively.
Cost Analysis: Compare the total cost of ownership (TCO) between both options, including direct costs like licensing fees, indirect costs such as training time, and any potential savings from reduced downtime.

#### First-Principles Solution

Current State Assessment:

Database Workloads: Analyze the current database workloads to understand peak loads, data volume, and transactional complexity.
Operational Requirements: Determine the required SLAs (Service Level Agreements) for your databases in terms of uptime, performance, and disaster recovery.
Incident History: Review historical incident logs to identify common issues and their frequency.

Vendor Support Tier Analysis:

Response Time: Evaluate the vendor’s promised response times under different tiers and compare them against your SLAs.
Proactive Monitoring: Assess the level of proactive monitoring offered by the support tier, including automated alerts and root cause analysis tools.
Additional Services: Consider any additional services such as regular health checks, performance tuning, and security assessments.

Internal SRE Capability Analysis:

Skill Set: Determine the skills required to manage your databases effectively, covering areas like database administration, monitoring, incident response, and automation.
Tooling & Infrastructure: Evaluate the tooling and infrastructure needed for an internal team, including monitoring tools (Prometheus, Grafana), alerting systems, and CI/CD pipelines.
Training & Development: Consider the ongoing training needs to keep the SRE team updated on the latest technologies and best practices.

Cost Analysis:

Direct Costs: Compare the licensing fees for different vendor tiers against internal development costs (salaries, training).
Indirect Costs: Factor in the time spent by your existing staff on support, troubleshooting, and maintenance.
Savings from Reduced Downtime: Estimate potential savings from reduced downtime provided by premium support versus self-managed solutions.

Risk Management:

Vendor Reliability: Assess the vendor’s reliability based on their track record and financial stability.
Disaster Recovery Plans: Ensure that your internal SRE team has robust disaster recovery plans in place, especially if you choose to go internal.
Compliance & Security: Consider compliance requirements (e.g., GDPR) and ensure both options meet necessary security standards.

#### Closing Summary

Deciding between a vendor’s premium support tier and building an internal Database SRE capability involves a detailed evaluation of the current state of your database infrastructure, operational requirements, cost analysis, skill set needs, and risk management. The key is to align these factors with your organization's strategic goals and ensure that you have a reliable and efficient solution in place.

#### Common Mistakes/Follow-Ups
Ignoring Internal Expertise: Overlooking the value of developing internal expertise can lead to missed opportunities for cost savings and tailored solutions.
Underestimating Vendor Costs: Failing to account for the true cost of support tiers, especially when factoring in time spent by your staff on coordination and follow-up.
Overlooking Risk Management: Neglecting risk management strategies can expose your organization to significant downtime risks if a premium tier vendor fails or an internal team is not well-prepared.

**Comparison:**

```
Factor	Vendor Support Tier	Internal Database SRE Capability
Response Time	Fast (1-2 hours)	Variable
Proactive Monitoring	Yes, automated alerts	Customizable
Skill Set	Varies by vendor	Tailored to specific needs
Tooling & Infrastructure	Vendor-provided tools	Custom development
Training & Development	Ongoing support	Self-directed learning
Cost of Ownership (TCO)	Lower, but with ongoing fees	Higher initial investment, lower maintenance
```

**Code:**

```python
# Example cost comparison code snippet
def compare_support_tier_vs_internal_sre(current_cost, support_tier_cost, internal_dev_cost):
    total_cost_support = current_cost + support_tier_cost
    total_cost_internal = current_cost + internal_dev_cost

    if total_cost_support < total_cost_internal:
        return "Vendor Support Tier is more cost-effective"
    else:
        return "Internal Database SRE Capability is more cost-effective"

# Sample data for comparison
current_cost = 50000
support_tier_cost = 30000
internal_dev_cost = 70000

print(compare_support_tier_vs_internal_sre(current_cost, support_tier_cost, internal_dev_cost))
```

**Example input / output:**

```text
Input:
- current_cost = 50000
- support_tier_cost = 30000
- internal_dev_cost = 70000

Output:
"Vendor Support Tier is more cost-effective"
```

**Data:**

```json
{
  "currentCost": 50000,
  "supportTierCost": 30000,
  "internalDevCost": 70000
}
```

**Flow:**

1. Current State Assessment
2. Vendor Support Tier Analysis
3. Internal SRE Capability Analysis
4. Response Time & ProactiveMonitoring
5. Skill Set & Tooling
6. Cost of Ownership
7. Skill Set & Tooling
8. Training & Development
9. Cost of Ownership
10. Compare with Internal SRE
11. Vendor Reliability
12. Risk Management Plans
13. Disaster Recovery
14. Compliance & Security
15. Total Cost Analysis
16. Internal vs Vendor Support
17. Vendor Financial Stability
18. Incident Frequency Analysis
19. Comprehensive DR Plan
20. Compliance Requirements
21. Final Decision

