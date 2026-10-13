---
type: "practice"
difficulty: "Expert"
---

# You're accountable for a delivery platform serving 10000 engineers across 15 regions; define its SLOs, its error budget, and what you'd do when the pipeline platform itself blows its budget during a critical release window.

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Standard Deployment Strategies](<../../../../10_DevOps_and_Platform/Delivery/CI_CD/Rollback_Strategy/Standard_Deployment_Strategies.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In this scenario, we're tasked with defining Service Level Objectives (SLOs) and an error budget for a CI/CD platform serving 10,000 engineers across 15 regions. The goal is to ensure the system operates reliably while managing the risk of outages during critical release windows.

TL;DR
Define SLOs based on service availability and performance.
Establish an error budget to manage the allowable downtime within a release cycle.
Mitigate risks by prioritizing critical regions first, conducting thorough testing, and implementing robust monitoring and alerting systems.

#### Triaging the Question

The question is probing our ability to define and maintain high service quality in a large-scale, distributed system. It requires us to balance reliability with flexibility during critical release periods, ensuring that any planned outages are well-managed within predefined limits.

#### Assumptions
The platform must be available 99.5% of the time across all regions.
There is a fixed error budget of 0.5% for each release cycle.
Regions have varying criticality, which will influence prioritization during outages.
We need to consider both technical and human factors in managing the error budget.

#### First Principles Solution
Defining SLOs

Service Availability:

Target: 99.5% availability across all regions.
Breakdown:
Region A: 99.7%
Region B: 99.6%
Other Regions (C-Z): 99.4%

Response Time:

Target: Average response time under 100ms.
Breakdown:
All regions: 95th percentile under 300ms.
Establishing an Error Budget
The error budget is the total allowable downtime for a given release cycle, set at 0.5%.
This budget should be allocated across critical and non-critical regions based on their impact to users.
Mitigating Risks During Critical Release Windows

Prioritization:

Identify critical regions where any downtime could significantly affect user experience or business operations.

#### Example
If Region A is hosting a critical development team, it might get priority in error budget allocation.

Testing and Validation:

Conduct thorough pre-release testing, including load tests and smoke checks.
Utilize canary deployments to gradually roll out changes while monitoring system performance closely.

Monitoring and Alerting:

Implement real-time monitoring for critical metrics such as response time, error rates, and request volume.
Set up alerting systems to notify the team of any anomalies, ensuring they can respond quickly if an issue arises.

Rollback Plan:

Have a well-defined rollback strategy in place should the release introduce issues beyond the allowed error budget.
Ensure that the rollback process is automated and can be executed within minutes.

#### Closing Summary
The key to managing SLOs and error budgets lies in precise definition, diligent monitoring, and proactive risk mitigation.
By prioritizing critical regions, conducting thorough testing, and maintaining robust alerting systems, we can ensure that any planned outages are well-managed within our predefined limits.

#### Common Mistakes/Follow-Ups
Overlooking Critical Regions: Failing to identify and prioritize regions where downtime could have a significant impact.
Neglecting Post-Release Monitoring: Not setting up adequate monitoring systems to track the performance post-release, leading to delayed detection of issues.
Insufficient Testing: Relying solely on pre-release testing without using canary deployments or other advanced techniques.

**Comparison:**

```
Monitoring Metrics	Target	Current
Service Availability (%)	99.5	N/A
Response Time (ms) 95th percentile	<300	N/A
Error Rate (%)	≤1%	N/A
```

**Code:**

```
MERMAID
Copy
graph TD;
    A[Define SLOs] --> B1[Service Availability];
    A --> B2[Response Time];
    B1 --> C1[Region A 99.7%];
    B1 --> C2[Region B 99.6%];
    B1 --> C3[Other Regions (C-Z) 99.4%];
    B2 --> D1[Average Response Time < 100ms];
    D1 --> E1[95th Percentile < 300ms];

    F[Establish Error Budget] --> G[0.5% for each release cycle];
    G --> H1[Prioritize Critical Regions];
    G --> H2[Conduct Thorough Testing];
    G --> H3[Implement Monitoring and Alerting];
    G --> H4[Define Rollback Plan];

    I[Mitigating Risks] --> J[Identify Critical Regions];
    J --> K1[Canary Deployments for Critical Features];
    J --> L1[Real-Time Monitoring for Metrics];
    J --> M1[Automated Rollback Strategy];
```

**Example input / output:**

```text
Input:
- Release candidate deployed to critical regions.
- Monitoring systems detect high error rate in Region A.

Output:
- Alerting system triggers notifications for the team.
- Canary deployment initiated on a small subset of users in Region A.
- Monitoring continues; if errors persist, rollback strategy is executed.
```

**Data:**

```json
{
  "SLOs": {
    "ServiceAvailability": {
      "RegionA": 99.7,
      "RegionB": 99.6,
      "OtherRegions": 99.4
    },
    "ResponseTime": {
      "Percentile95": <300ms
    }
  },
  "ErrorBudget": {
    "TotalAllowedDowntime": 0.5,
    "CriticalRegionAllocations": [
      {"RegionA": 0.2, "Reason": "Development team critical"},
      {"OtherRegions": 0.15, "Reason": "Secondary critical regions"}
    ]
  },
  "MitigationPlan": {
    "MonitoringSystems": ["Real-time response time", "Error rates", "Request volume"],
    "AlertingSystems": [
      "Email notifications",
      "Slack integrations for team communication"
    ],
    "RollbackStrategy": "Automated, executes within minutes of detection"
  }
}
```

**Flow:**

1. Pre-Release
2. Canary Deploy
3. Monitor Performance
4. If OK, Proceed to Rollout
5. Else, Identify Issues
6. Rollout to Full User Base
7. Debug and Fix
8. Re-deploy with Fixes
9. Conduct Post-ReleaseMonitoring

