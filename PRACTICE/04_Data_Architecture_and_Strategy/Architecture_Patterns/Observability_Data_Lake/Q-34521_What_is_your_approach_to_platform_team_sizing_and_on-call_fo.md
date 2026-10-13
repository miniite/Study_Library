---
type: "practice"
difficulty: "Deep"
---

# What is your approach to platform team sizing and on-call for a lake that has become tier-0 infrastructure that all of AIOps depends on?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Detecting Ingestion Changes Behind Drift](<../../../../04_Data_Architecture_and_Strategy/Architecture_Patterns/Observability_Data_Lake/Detecting_Ingestion_Changes_Behind_Drift.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the challenge of scaling the platform team, particularly in terms of on-call responsibilities, for an Observability Data Lake (ODL) that has become tier-0 infrastructure critical to all AIOps operations, we need a comprehensive approach grounded in first principles. The core idea is to balance resource allocation with risk management, ensuring both performance and reliability while minimizing downtime.

The key factors include:

Team Structure: Define the team roles and responsibilities.
Risk Assessment: Evaluate the criticality of ODL components.
On-Call Strategy: Design a robust on-call schedule that ensures coverage without overburdening any individual or small group.
Resource Allocation: Ensure adequate staffing to handle both day-to-day operations and unexpected incidents.

Let’s break this down step by step:

#### Triage
Understanding the Question

The question is asking for an approach to sizing a platform team, specifically focusing on on-call responsibilities, given that the Observability Data Lake (ODL) has become tier-0 infrastructure essential to all AIOps operations. The interviewer is probing your understanding of:

Criticality and Impact: How critical ODL is to overall system reliability.
Risk Management: Strategies for managing risks associated with ODL failures.
Team Management: Effective team structure, roles, and responsibilities.
On-Call Planning: Balancing coverage needs with resource constraints.

#### First-Principles Solution
1. Define the Team Roles

The platform team should be structured into specialized roles to ensure a balanced skill set:

Data Engineers: Responsible for ODL infrastructure management, data pipelines, and ETL processes.
DevOps Engineers: Focus on CI/CD pipelines, monitoring tools, and automation scripts.
SREs (Site Reliability Engineers): Specialized in proactive maintenance, incident response, and reliability engineering.
2. Risk Assessment

Criticality assessment is crucial to prioritize which components of the ODL need more robust handling:

Component Breakdown: Identify critical vs non-critical data streams.
Failure Modes Analysis: Understand potential failure scenarios for each component.
Impact on AIOps: Assess how failures in specific parts affect overall AIOps operations.
3. On-Call Strategy

Designing a robust on-call schedule involves:

On-Call Rotations: Implement a rotating on-call schedule to distribute responsibility and avoid burnout.
On-Call Escalation: Establish clear escalation paths for critical issues, ensuring timely resolution.
Training and Drills: Regular training sessions and incident response drills to keep the team prepared.
4. Resource Allocation

Ensure adequate staffing by:

Headcount Planning: Allocate additional resources based on risk assessment and operational needs.
Skill Mismatch Mitigation: Cross-train engineers to handle multiple roles, reducing single points of failure.
Use of Tools and Automation: Leverage automation tools for routine tasks, allowing human resources to focus on critical issues.

#### Closing Summary

The approach involves a structured team organization with clear roles and responsibilities, informed by a thorough risk assessment. On-call strategies should balance coverage needs with resource constraints, ensuring both proactive maintenance and rapid response capabilities. By continuously monitoring and adapting the team structure, we can maintain high reliability and minimize downtime for critical AIOps operations.

#### Common Mistakes/Follow-Ups
Overlooking Risk: Failing to properly assess the criticality of different ODL components.
Lack of Rotation: Relying on a few individuals for on-call duties without adequate support, leading to burnout and high-risk situations.
Insufficient Training: Not providing regular training and drills to keep engineers prepared for incidents.

**Code:**

```python
# Example code snippet for risk assessment (simplified)
def assess_risk(data_stream):
    criticality_score = data_stream['critical']
    impact_score = data_stream['impact']
    return criticality_score * impact_score

# Sample input data
data_streams = [
    {'name': 'stream1', 'critical': 5, 'impact': 3},
    {'name': 'stream2', 'critical': 4, 'impact': 7}
]
risk_scores = [assess_risk(ds) for ds in data_streams]
```

**Example input / output:**

| Data Stream | Criticality Score | Impact Score | Risk Score |
|-------------|-------------------|--------------|------------|
| stream1     | 5                 | 3            | 15         |
| stream2     | 4                 | 7            | 28         |

**Data:**

```yaml
# Sample configuration for on-call rotations
on_call_rotation:
  - DataEngineer1
  - DevOpsEngineer2
  - SRE3
  - DataEngineer4
  - DevOpsEngineer5
  - SRE6
```

