---
type: "practice"
difficulty: "Deep"
---

# Your ML-on-telemetry job runs as a scheduled pipeline; how would you deploy a change to the feature pipeline without corrupting the feature store the online detector reads from?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Infrastructure Drift Detection](<../../../../10_DevOps_and_Platform/Delivery/CI_CD/Infrastructure_Drift_Detection.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When deploying changes to an ML-on-telemetry CI/CD pipeline, especially when this involves interacting with a feature store that's being used by an online detector, it’s crucial to ensure that no data corruption or service disruption occurs. The key is to introduce change management practices that allow for controlled updates while maintaining the integrity of the feature store.

#### Triage
and Assumptions

The question revolves around how to safely update a CI/CD pipeline that impacts the feature store, without affecting the online detector's real-time operations. We assume:

A robust feature store is in place.
The online detector relies on this feature store for real-time predictions.
There are multiple stages of the CI/CD process: build, test, and deploy.

#### First Principles Solution

To manage changes to a pipeline that interacts with a critical system like an online detector, we can adopt a staged deployment approach. This involves:

Feature Store Segregation: Ensure there's a dedicated feature store for development and another for production.
Version Control: Implement version control for the features in the feature store.
Canary Deployments: Gradually roll out changes to ensure they don't impact critical services.
Detailed Steps

Feature Store Segregation:

Create a separate environment (e.g., dev or staging) where developers can modify and test new features without affecting production data.

Version Control for Features:

Use version control systems like Git to manage changes in the feature store metadata and feature definitions.
Ensure that each change is documented, tested, and approved before being deployed.

Canary Deployments:

Implement a canary deployment strategy where a small percentage of traffic uses the new features while the rest continues to use the old ones.
Monitor real-time performance metrics to detect any anomalies or issues early.

Automated Testing:

Run automated tests on both the feature store and the online detector to ensure that changes do not introduce bugs or service disruptions.
Use unit, integration, and end-to-end testing frameworks like JUnit, PyTest, or custom scripts tailored for your use case.

Rollback Plan:

Have a rollback plan in place if any issues are detected during the canary deployment phase.
This could involve using versioning to revert changes quickly if needed.

Monitoring and Logging:

Set up comprehensive monitoring and logging mechanisms to track feature usage, performance, and anomalies.
Use tools like Prometheus, Grafana, or AWS CloudWatch for real-time visibility into the system's health.

#### Closing Summary

By following these best practices, we can ensure that changes to our CI/CD pipeline are deployed safely without disrupting the online detector. Key points include segregating environments, version control, canary deployments, automated testing, and robust monitoring. These steps help in maintaining feature store integrity and real-time system reliability.

#### Common Mistakes/Follow-Ups
Failing to Test Thoroughly: Skipping thorough testing before deployment can lead to production issues.
Not Implementing Canaries Properly: Failing to use a gradual rollout approach increases the risk of widespread outages.
Ignoring Monitoring and Logging: Without adequate visibility, it’s hard to detect and resolve issues quickly.

