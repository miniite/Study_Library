---
type: "practice"
difficulty: "Deep"
---

# How do you architect the gateway so it can be safely operated by teams in three time zones with follow-the-sun on-call; what config-change guardrails, staged-rollout defaults, and runbook automation reduce the reliance on any single expert being awake during a change?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Control Plane and Data Plane Design](<../../../../07_Distributed_Systems_and_Design/APIs/API_Gateway/Gateway_Routing_and_BFF_Patterns/Control_Plane_and_Data_Plane_Design.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When architecting an API Gateway that needs to be safely operated by teams across three time zones with follow-the-sun on-call rotations, we need to ensure robust guardrails and automated processes to reduce the risk of disruptions caused by operational windows or lack of expertise during specific times. This involves setting up comprehensive configuration change protections, staged rollouts, and automated runbook execution.

Firstly, the API Gateway should be designed with clear roles and responsibilities. Each team in each time zone will have defined permissions and access levels to ensure that changes can only be made by authorized personnel. We'll use a multi-tiered approach for config changes: initial proposal, review, approval, and finally deployment, all managed via a CI/CD pipeline.

To handle the follow-the-sun on-call model, we need to establish a clear schedule of responsibilities across the teams. For example, Team A in Zone 1 is responsible from 6 AM to 2 PM UTC; Team B in Zone 2 handles changes from 2 PM to 10 PM UTC; and Team C in Zone 3 manages operations from 10 PM to 6 AM UTC.

Next, we’ll implement a series of guardrails to safeguard against unauthorized or rushed changes. These include:

Change Proposals: Any config change must be proposed via an internal ticketing system.
Automated Reviews: Code reviews and automated testing should occur before any change is considered for deployment.
Approval Flows: Changes require approvals from at least two team members, one of whom should ideally come from a different shift to ensure cross-zone oversight.

For staged rollouts, we will adopt a canary release approach. This involves deploying the new configuration to a small subset of users or environments first. If there are no issues detected within an hour (or another defined timeframe), the change is gradually rolled out to all users.

To further minimize dependency on any single expert being awake during critical operations, we'll automate as much of the runbook execution as possible. For example, if a manual step in the runbook fails, it should trigger alerts and retries. Additionally, we can use automated monitoring tools to detect and alert on potential issues before they escalate.

**Comparison:**

```
Step	Description
1	Propose config changes via internal ticketing system.
2	Automated reviews and testing before deployment.
3	Multi-step approval process involving at least two team members.
4	Canary release to a small subset of users or environments.
5	Monitor for issues and retries on failures.
```

**Flow:**

1. Propose Config Change
2. Review & Test
3. Approve
4. Staged Rollout - CanaryRelease
5. Monitor for Issues
6. Retry on Failures
7. Apply Full Deployment

