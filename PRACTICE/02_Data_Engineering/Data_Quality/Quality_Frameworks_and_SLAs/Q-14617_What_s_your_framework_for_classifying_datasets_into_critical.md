---
type: "practice"
difficulty: "Deep"
---

# What's your framework for classifying datasets into criticality tiers, and how does the tier drive the strictness of SLAs, on-call, and check coverage?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Quality SLAs and Rule Testing](<../../../../02_Data_Engineering/Data_Quality/Quality_Frameworks_and_SLAs/Quality_SLAs_and_Rule_Testing.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When designing a data quality framework as an Architect/Principal Data Engineer, it's crucial to classify datasets into criticality tiers based on their impact on business operations. This classification then drives the strictness of Service Level Agreements (SLAs), on-call responsibilities, and check coverage. The key idea is that more critical datasets require more stringent monitoring and quicker resolution processes.

#### Triage

The primary goal here is to establish a tiered approach for data quality based on the business-criticality of the data. This involves understanding what data is critical (e.g., financials, customer PII), medium-risk (e.g., inventory), and low-risk (e.g., non-transactional logs). The tiers will then dictate how rigorously we monitor these datasets, the escalation paths for issues, and the resources allocated to ensure their quality.

#### Assumptions
Business Impact: We assume that certain data is more critical than others due to its direct or indirect impact on business operations.
Resources Allocation: There are limited resources (e.g., time, personnel) available for monitoring and resolving issues.
Risk Management: Critical datasets need immediate attention to mitigate potential business risks.

#### First-Principles Solution

Criticality Tiers:

Critical Tier: Data that is highly critical, such as financial records, customer PII, or real-time operational data (e.g., fraud detection systems).

SLA: 99.99% uptime with a maximum of 30 minutes downtime tolerance.
On-Call Responsiveness: Real-time alerting and 24/7 on-call support.
Check Coverage: High-frequency automated checks, manual verification, and regular audits.

High Tier: Data that is important but less critical, such as marketing analytics or historical financial reports.

SLA: 99.9% uptime with a maximum of 120 minutes downtime tolerance.
On-Call Responsiveness: Automated alerts with manual verification and regular scheduled checks.
Check Coverage: Medium-frequency automated checks and periodic audits.

Medium Tier: Data that is moderately critical, such as operational data from non-critical systems or legacy datasets.

SLA: 99.5% uptime with a maximum of 4 hours downtime tolerance.
On-Call Responsiveness: Automated alerts with scheduled reviews and basic manual checks.
Check Coverage: Basic automated checks and periodic reviews.

Low Tier: Data that is non-critical, such as historical logs or less frequently accessed data.

SLA: 98% uptime with a maximum of 24 hours downtime tolerance.
On-Call Responsiveness: Automated alerts with manual intervention only in emergencies.
Check Coverage: Minimal automated checks and occasional reviews.

SLA Implementation:

Alerting Systems: Implement alerting systems that can be configured based on the tier of data. Higher tiers should have more stringent thresholds for triggering alerts.
Incident Management: Establish a robust incident management process with clear escalation paths, prioritizing critical issues over non-critical ones.
Monitoring Tools: Use advanced monitoring tools to track and report on data quality metrics in real-time.

On-Call Responsibilities:

Tier-Based On-Call Schedules: Assign on-call responsibilities based on the criticality of the dataset, ensuring that higher-tier datasets have dedicated teams or individuals available.
Training and Support: Provide training and support for on-call personnel to handle high-priority issues effectively.

Check Coverage:

Automated vs Manual Checks: Balance between automated checks (faster response times) and manual checks (comprehensive coverage).
Regular Audits: Conduct regular audits of critical datasets to ensure that the checks are effective and up-to-date.

#### Closing Summary
+ Common Mistakes/Follow-Ups

By classifying data into criticality tiers, we can prioritize resources effectively, ensuring that high-priority issues are addressed promptly while maintaining cost efficiency. A common mistake is not aligning the SLAs with actual business needs; it’s crucial to regularly review and adjust these based on changing business requirements.

For follow-ups, consider discussing:

Integration with Business Processes: How does this framework integrate with existing business processes?
Performance Trade-offs: What are the trade-offs between different tiers in terms of cost and resource allocation?
Feedback Mechanisms: How will we measure the effectiveness of this framework over time?

**Comparison:**

```
Tier	SLA (Uptime)	Downtime Tolerance	On-Call Responsiveness	Check Coverage
Critical	99.99%	30 minutes	24/7 Support	High-Frequency Automated Checks, Manual Verification
High	99.9%	120 minutes	Scheduled Shifts	Medium-Frequency Automated Checks and Periodic Reviews
Medium	99.5%	4 hours	Automated Alerts with Scheduled Reviews	Basic Automated Checks and Periodic Reviews
Low	98%	24 hours	Automated Alerts with Manual Intervention Only in Emergencies	Minimal Automated Checks and Occasional Reviews
```

**Code:**

```python
# Example SLA Configuration
def configure_sla(tier):
    if tier == 'Critical':
        return {'uptime': 99.99, 'downtime_tolerance': 30}
    elif tier == 'High':
        return {'uptime': 99.9, 'downtime_tolerance': 120}
    # ... for Medium and Low tiers

# Example On-Call Schedule
def on_call_schedule(tier):
    if tier in ['Critical', 'High']:
        return "24/7 Support"
    elif tier == 'Medium':
        return "Scheduled Shifts"
    else:
        return "Manual Intervention Only"

# Example Check Coverage
def check_coverage(tier):
    if tier in ['Critical', 'High']:
        return "High-Frequency Automated Checks, Manual Verification"
    elif tier == 'Medium':
        return "Basic Automated Checks and Periodic Reviews"
    # ... for Low tier

# Usage
print(configure_sla('Critical'))
print(on_call_schedule('Medium'))
print(check_coverage('Low'))
```

**Example input / output:**

```text
Input:
- Dataset Tier: Critical

Output:
- SLA: Uptime = 99.99%, Downtime Tolerance = 30 minutes
- On-Call Responsiveness: 24/7 Support
- Check Coverage: High-Frequency Automated Checks, Manual Verification
```

**Data:**

```json
{
  "data_quality_tiers": {
    "critical": {
      "slas": {"uptime": 99.99, "downtime_tolerance": 30},
      "on_call": "24/7 Support",
      "check_coverage": "High-Frequency Automated Checks, Manual Verification"
    },
    "high": {
      "slas": {"uptime": 99.9, "downtime_tolerance": 120},
      "on_call": "Scheduled Shifts",
      "check_coverage": "Medium-Frequency Automated Checks and Periodic Reviews"
    },
    "medium": {
      "slas": {"uptime": 99.5, "downtime_tolerance": 4 * 60},
      "on_call": "Automated Alerts with Scheduled Reviews",
      "check_coverage": "Basic Automated Checks and Periodic Reviews"
    },
    "low": {
      "slas": {"uptime": 98, "downtime_tolerance": 24 * 60},
      "on_call": "Automated Alerts with Manual Intervention Only in Emergencies",
      "check_coverage": "Minimal Automated Checks and Occasional Reviews"
    }
  }
}
```

**Flow:**

1. Yes
2. No
3. Data Quality Tier Classification
4. Is Data Critical?
5. Define SLA, On-Call & CheckCoverage
6. Assign Lower Priority SLAs,On-Call & Checks
7. Implement Monitoring andAlerts
8. Maintain Regular Audits andReviews

