---
type: "practice"
difficulty: "Deep"
---

# Explain how you'd architect a canary-of-canaries approach: rolling an automation out to a small slice of incidents before letting it act on all of them.

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Safe Closed-loop Remediation](<../../../../07_Distributed_Systems_and_Design/Software_Architecture/Closed_Loop_Automation/Safe_Closed-loop_Remediation.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

#### Overview

In a canary-of-canaries architecture, we divide the incident space into smaller, manageable slices and roll out our automated solution to these slices incrementally rather than all at once. This approach allows us to test the effectiveness of our automation in a controlled environment before scaling it up. By doing so, we mitigate risks associated with large-scale deployments and ensure that any issues are caught early on.

#### Triaging the Question

The core idea here is to implement a staged deployment strategy for automated responses or actions during incident management. The key areas the interviewer will be probing include:

How you would segment incidents into canaries.
Methods of rolling out the automation incrementally.
Criteria and mechanisms for determining when an automation can be scaled up.
Risk mitigation strategies in place.

#### First-Principles Solution

Incident Segmentation:

We need to classify our incidents based on severity, type, or other relevant criteria. For instance, we might segment them by criticality levels (e.g., high, medium, low).

Canary Deployment:

Start with the least critical or most recent incidents as canaries. This allows us to test the automation's effectiveness in a safe environment.
Monitor key performance indicators (KPIs) and business impact metrics for these initial segments.

Feedback Loop:

Implement real-time monitoring to gather insights on how the automation performs in the canary environment.
Analyze this data to determine if there are any issues or unexpected behaviors that need addressing before full-scale deployment.

Scaling Criteria:

Define clear criteria for when it is safe to scale up. This could be based on successful performance in the canary segments, meeting certain KPI thresholds, or receiving positive feedback from stakeholders.
Use A/B testing techniques if possible, comparing outcomes between canary and non-canary segments.

Risk Management:

Establish rollback mechanisms to quickly revert to previous states if issues arise during the canary phase.
Ensure that critical systems are not impacted by the initial rollouts.

#### Closing Summary
+ Common Mistakes/Follow-Ups

This approach ensures a robust and controlled rollout of automation, minimizing risks associated with large-scale deployments. By carefully managing which incidents get the initial exposure to the automation, we can iteratively improve our solutions based on real-world data without disrupting critical operations.

#### Common Mistakes

Failing to define clear criteria for when to scale up or roll back.
Overlooking the importance of continuous monitoring and feedback loops.
Not having rollback mechanisms in place.
Triaging Follow-Ups

How do you ensure that canaries are representative of the overall incident set?

Use stratified sampling techniques to ensure a balanced mix of criticalities, severities, etc., in the canary segments.

What metrics should be used for feedback loops?

Metrics could include resolution time, customer satisfaction scores, and system performance indicators like latency or throughput.

How do you handle incidents that are not part of the current canary set?

These incidents would continue to be managed using traditional methods until they fit into a future canary set based on updated segmentation criteria.

**Comparison:**

```
Yes
No
No
Incident Segmentation
Are Canaries Representative?
Feedback Loop
Stratified Sampling forCanaries
Scaling Up Based on Metrics
Rollback Mechanisms in Place?
Yes -> Continue Scaling
Risk Mitigation Plan
```

**Code:**

```python
def segment_incidents(incidents, severity_threshold):
    """
    Segments incidents based on their severity.
    :param incidents: List of incident objects with 'severity' attribute.
    :param severity_threshold: Threshold for considering an incident as high-severity.
    :return: Two lists - high-severity canaries and low-severity non-canaries.
    """
    canaries = [incident for incident in incidents if incident.severity > severity_threshold]
    non_canaries = [incident for incident in incidents if incident.severity <= severity_threshold]
    return canaries, non_canaries
```

**Example input / output:**

| Input Incidents | High Severity Canaries | Low Severity Non-Canaries |
|-----------------|-----------------------|--------------------------|
| [1, 2, 3, 4, 5]  | [3, 5]                 | [1, 2, 4]                |

**Data:**

```json
{
  "incidents": [
    {"id": 1, "severity": 3},
    {"id": 2, "severity": 4},
    {"id": 3, "severity": 5},
    {"id": 4, "severity": 2},
    {"id": 5, "severity": 6}
  ]
}
```

**Flow:**

1. +---------------------+
2. | Incident Segmentation |
3. +---------------------+
4. |
5. v
6. +-------------------+
7. | Canary Deployment |
8. +-------------------+
9. |
10. v
11. +-------------------+
12. | Feedback Loop     |
13. +-------------------+
14. |
15. v
16. +----------------------+
17. | Scaling Criteria Met?|
18. +----------------------+
19. | No
20. v
21. +-----------------------+
22. | Rollback Mechanisms in Place? |
23. +-----------------------+
24. | Yes
25. v
26. +---------------------+
27. | Continue Scaling    |
28. +---------------------+
29. | No
30. v
31. +-------------------+
32. | Risk Mitigation Plan|
33. +-------------------+

