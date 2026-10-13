---
type: "practice"
difficulty: "Expert"
---

# What metrics would you track to measure CI/CD health (DORA metrics)?

**Difficulty:** Expert · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ DORA Metrics](<../../../../10_DevOps_and_Platform/Delivery/CI_CD/Deployment_Observability/DORA_Metrics.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To measure Continuous Integration and Continuous Deployment (CI/CD) health, it's crucial to adopt a set of Key Performance Indicators (KPIs) that align with the DevOps Research and Assessment (DORA) framework. These metrics provide a comprehensive view of how effectively your development team is integrating code changes and deploying them in a safe and stable manner.

The DORA metrics are:

Deployment Frequency
Lead Time for Changes
Change Failure Rate
Mean Time to Restore

#### Triage
Understanding the Question and Assumptions

The question asks about metrics to measure CI/CD health, which means we need to focus on these specific KPIs as they are widely recognized in the DevOps community. The assumptions here include:

You have a mature CI/CD pipeline.
Your team is actively deploying code changes multiple times a day.

#### First-Principles Solution
1. Deployment Frequency (DF)

This metric measures how often your team deploys new features or fixes to production. A higher deployment frequency indicates better communication and collaboration between development, testing, and operations teams.

2. Lead Time for Changes (LTC)

Lead time is the average amount of time taken from the creation of a change request to its final release into production. Lower lead times indicate that your team can quickly respond to customer feedback and market changes.

3. Change Failure Rate (CFR)

This metric tracks the rate at which deployments fail due to issues or bugs introduced by new code. A lower failure rate is desirable as it indicates a more stable deployment process and higher confidence in the quality of deployed code.

4. Mean Time to Restore (MTR)

When a deployment fails, MTR measures how quickly the system can be restored to a working state. Lower values of MTR indicate faster recovery times, which can reduce customer dissatisfaction and operational costs.

#### Closing Summary
+ Common Mistakes/Follow-Ups

To ensure robust CI/CD practices, tracking these DORA metrics is essential. However, it's important to remember that high deployment frequency alone does not guarantee success; the quality of deployments is equally crucial. Additionally, while these metrics provide valuable insights, they should be complemented by other metrics such as code coverage, test execution rates, and incident management.

#### Common mistakes
include focusing solely on deployment frequency without ensuring that all deployed changes are of high quality or failing to monitor the impact of failed deployments on user experience and business operations. Follow-ups might involve integrating additional metrics related to security and compliance, customer feedback loops, and continuous improvement practices.

**Comparison:**

```
Metric	Description	Target Value
Deployment Frequency (DF)	Number of deployments to production per week. Higher is better.	> 5
Lead Time for Changes (LTC)	Average time from code commit to deployment. Lower is better.	< 24 hours
Change Failure Rate (CFR)	Percentage of deployments that fail due to bugs or issues. Lower is better.	< 1%
Mean Time to Restore (MTR)	Average time to recover from a failed deployment. Faster is better.	< 30 minutes
```

**Code:**

```
MERMAID
Copy
graph TD
    A[Deployment Frequency] --> B(DF)
    C[LTC Lead Time for Changes] --> D(LTC)
    E[Change Failure Rate] --> F(CFR)
    G[MTR Mean Time to Restore] --> H(MTR)
```

**Example input / output:**

| Input Metric | Value   |
|--------------|---------|
| Deployment Frequency (DF) | 10       |
| Lead Time for Changes (LTC) | 8.5      |
| Change Failure Rate (CFR) | 3%       |
| Mean Time to Restore (MTR) | 20 minutes |

**Data:**

```json
{
  "metrics": {
    "deploymentFrequency": 10,
    "leadTimeForChanges": 8.5,
    "changeFailureRate": 3,
    "meanTimeToRestore": 20
  }
}
```

**Flow:**

1. Code Change
2. Integration/Testing
3. Deploy
4. Monitor & Feedback
5. Release to Production
6. Measure Metrics

